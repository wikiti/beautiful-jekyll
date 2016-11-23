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
  validate :progress, presence: true, inclussion: { within: 0..100 }

  # You'll probably want to use a better naming system, or move this code
  # into a different context, like a helper.

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

  # POST /jobs/work
  def work
    # Just create a task status, and send it as a parameter to your worker.
    # For example, for active job:
    ExampleJob.perform_later task_status

    # Don't forget to return information about the task object:
    render status: :created, json: task_status
  end
  
  # GET /jobs/poll/:id
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
  end
  
  member do
    get :poll
  end
end
````

### Worker

TODO

### Views

TODO

## Conclusion

TODO

