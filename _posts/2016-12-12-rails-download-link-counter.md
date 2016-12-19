---
layout: post
published: true
title: 'Rails: Download link counter'
subtitle: Keep track of it!
---
It's 4:00 AM in the morning. You're finishing your amazing Rails WebApp which will let final users download free content without paying anything. You're almost done, polishing some details. However, you lack of one feature, and you don't know how to implement it (*really? are you serious? after all this?*): track how many times has a file been downloaded.

Fear not, my friend! I've the solution for you, right on this post.

## Problem

As stated on that amazing introduction, we need to *track* how many times a file has been downloaded. There is no way for Rails routes to persist how many times a route has been accessed (actually, it can, but rack middleware is required). So, how do we solve this?

## Solution

Models. Yup. That's it. We'll store how many times a link has been accessed by using a database record. We need to store the path (e.g. the file route) and the download count. There are many ways to store that information in the database. For example:

- Store download `count` into an existent model.
- Create a new model, storing `controller`, `action`, `file_id` ... and download `count`.
- Create a new model, storing the `path` and `count`.
- Create a new model, storing the `count`, and relating it with the downloadable model.

I think that the third option is the most flexible of all of them, because we can track anything we want; from file download to per route access, building some cool stats for ~~software destroyers~~ marketing team.

The table scheme will be the following for this case:

| id: integer | path: text       | count: integer |
| ----------- | ------------------ | -------------- |
|           1 | /files/32/download |            243 |
|           2 | /files/24/download |          12560 |
|         ... |                ... |            ... |

Easy, right? Note that *path* should be indexed to speed up queries.

After setting the model, the controllers must increment the corresponding path record every time is accessed.

Now, it's time to work!

## Implementation

First, we need a model to store the table's information. We should call it... `RequestCounter`. Original, isn't it?

This is pretty straightforward:

```sh
$ rails generate model request_counter path:text counter:integer
```

We want `path` to be primary key and `count` to be `0` as default, so we need to update the migration file to look like this:

```rb
class CreateRequestCounters < ActiveRecord::Migration
  def change
    create_table :request_counters do |t|
      t.text :path, index: true # indexed attribute
      t.integer :counter, default: 0 # 0 as default

      t.timestamps
    end
  end
end
```

Now,let's add some methods to the model:

```rb
class RequestCounter < ActiveRecord::Base
  def inc
    self.class.increment_counter :counter, self.id
  end
end
```

So, why should we use [`ActiveRecord::Base::increment_counter`](http://apidock.com/rails/ActiveRecord/Base/increment_counter/class) instead of [`ActiveRecord::Base#increment`](http://apidock.com/rails/ActiveRecord/Base/increment) or `+= 1`? The answer  is simple: we want our increments to be atomic. Imagine a race condition where to users download the same file at the very same time; this may lead to conflicts and the counter may be incremented by `1` instead of `2`, and we don't want that. Instead, `increment_counter` will perform the increment on the database layer of our application (instead of doing something like `SET COLUMN TO VALUE`, it'll `INCREMENT COLUMN BY 1`).

And now, the controller. Each time a download link is accessed, the related counter should be increment. To handle any kind of link (even paths), we'll put the main code in the `ApplicationController` class:

```rb
class ApplicationController < ActionController::Base
  def register_path_request
    request_counter = RequestCounter.find_or_create_by path: request.path
    request_counter.inc
  end
  # ...
end
```

Isn't the [`find_or_create_by`](http://apidock.com/rails/v4.0.2/ActiveRecord/Relation/find_or_create_by) method cool? If the record is not found, then it's created! After that, the counter is incremented by one with `RequestCounter#inc`.

- *But... the `ApplicationController#register_path_request` is not called anywhere! How do we use it?*

That's a good question! Well use [`after_action`](http://apidock.com/rails/v4.0.2/AbstractController/Callbacks/ClassMethods/after_action) callback in one of our controllers to call this method. For example, imagine that we have the following controller:

```rb
class FilesController < ApplicationController
  # GET /files/:id
  def show
    send_file "path/to/file_#{params[:id]}.txt"
  end

  # ...
end
```

We'll add an `after_action` to track each time `/files/:id` is requested:

```rb
class FilesController < ApplicationController
  after_action :register_path_request, only: [:show]

  # ...
end
```

And you're done! Every time you access the very same path or route, the counter will be created (if not found) and incremented by one. Try different implementations and use the one that fits better for your needs.

## Conclusions

So, in summary:

- Use a model to store the information in the database.
- Track requests count by path, action-controller, model, etc.
- Use action callbacks to increment the counters automatically.

Happy coding!
