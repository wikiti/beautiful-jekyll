---
layout: post
published: true
title: 'Rails: Asynchronous status tracker'
subtitle: Track your background jobs from javascript!
---
## Introduction

Imagina that you need to do a heavy task on your Rails server. If you're brute enought, you'll throw your logic into your controller, and let that long request stay there for a couple of minutes. Instead, you create an asynchronous job (with [Resque](https://github.com/resque/resque), [Sidekiq](https://github.com/mperham/sidekiq/), [ActiveJob](https://github.com/rails/rails/tree/master/activejob)...).

Now everything is great; your tasks are executing in background, and letting those server production threads handle more requests. But... what if you just want to know how much time will it take to process that task? How does the client (*javascript*) know the progress of that well-encapsulated task worker?

If you're using [Rails 5](http://guides.rubyonrails.org/5_0_release_notes.html), you'll know that [actioncable](https://github.com/rails/rails/tree/master/actioncable) is the best way to go. What if you're not? Then this post is for you, my friend!

On this post, I'll try to explain a simple logic to achieve this feature.

## Implementation

This feature needs some code in the following parts of your application:

- **Model:** Create a new model to handle `TaskStatus`es.
- **Controller:** Add some endpoints to poll the `TaskStatus` information.
- **Worker:** Update the `TaskStatus` progress.
- **View:** Create some *javascript* logic to request a job to the server, and keep track of it.

### Model

First off, generate your model:

````sh
$ rails g model TaskStatus progress:float
````

And migrate your database:

````sh
$ rake db:migrate
````

You'll want to add some methods and logic to your model. Be creative!

````ruby
class TaskStatus < ActiveRecord::Base
  # 0 means not done, 50 means half done, 100 means done
  validate :progress, presence: true, inclussion: { within: 0..100 }

  # You'll probably want to use a better naming system, or move this code
  # into a different context, like a helper.

  def done?
    progress == 100
  end

  def set(n)
    self.update! progress: n
  end
  
  def step(n)
    self.increment progress: n
  end
end
````

### Controller

The controller should have two endpoints; one to request and perform a background job, and one to poll
the current `TaskStatus` progress.

For example, this should be a valid controller:

````rb
class JobsController < ApplicationController

  # GET /jobs/
  def index
  end

  # POST /jobs/work
  def work
    # Just create a task status, and send it as a parameter to your worker.
    # For example, for active job:
    ExampleJob.perform_later task_status

    # Don't forget to return information about the task object:
    render status: :created, json: task_status
  end
  
  # GET /jobs/:id/poll
  def poll
    # Just return the task object. You may want to place this endpoint into another controller,
    # called TaskStatusesController, or FooBarController, or whatever.
    # Note that `task_status` will retrieve the status object from the database.
    render status: :ok, json: task_status
  end
  
  private
  
  # If params[:id] is present, search the object on the database. Otherwise, create a new one!
  def task_status
    @task_status ||= params[:id].present? ? TaskStatus.find(params[:id]) : TaskStatus.create!(progress: 0)
  end
end
````

Also, don't forget to update your `config/routes.rb` file:

````rb
resources :jobs do
  collection do
    post :work
    get :index
  end
  
  member do
    get :poll
  end
end
````

### Worker

The worker or background job should update the `TaskStatus` object at will. For example:

````rb
class ExampleJob < ActiveJob::Base 
  queue_as :default

  # This job will take 10 seconds to be fully performed!
  def perform(task_status)
    100.times do
      sleep 0.1
      task_status.step 1
    end
  end 
end
````

If you are iterating items, and performing an slow action on each item, just divide 100 between the number of items, and increment the `TaskStatus` object with that value:

````rb
unit = 100.0 / items.count
items.each do |item|
  item.slow_method
  task_status.step unit
end
````

### Views

Finally, setup some javascript to use our controller's endpoints. For example, in your index view:

````erb
<%= button_tag 'Work!', id: 'request-button' %>
<span>Progress: </span>
<span id="progress"></span>
````

And the *javascript* code (using *jQuery*):

````js
$(function() {
  // Poll interval; how often should we ask the server about TaskStatus?
  var POLL_INTERVAL = 100; // ms
  // TaskStatus content (json).
  var task_status = {};

  // When pressing the "Work!" button ...
  $('#request-button').click(function() {
    $(this).prop('disabled', true);
  
    // ... request a job by using an ajax request.
    $.ajax({
      method: 'POST',
      url: '/jobs/work',
      contentType: 'json'
    }).done(function(data) {
      task_status = data;
      // When the request finishes, start polling the status 
      setTimeout(updateStatus, POLL_INTERVAL);
    });
  });

  // Check the current status of the TaskStatus object.
  var updateStatus = function() {
    $.ajax({
      method: 'POST',
      url: '/jobs/' + task_status.id + '/poll',
      contentType: 'json'
    }).done(function(data) {
      task_status = data;
   
      // Done!
      if(task_status.progress == 100) {
        $('#progress').text('Done!');
      }
      // Not done!
      else {
        $('#progress').text(task_status.progress + '%');
        // Ask again in POLL_INTERVAL ms.
        setTimeout(updateStatus, POLL_INTERVAL);
      }
    });
  };
});
````

Now, run that server, go to your view, press that button, and the background task progress!

## Conclusion

Now, there is no need to use websockets or complex solutions to create a basic status tracking system. Almost *everything* is possible with simple *HTTP* requests. However, polling on web applications aren't always a good idea; those constant requests generate overhead on the server side.

