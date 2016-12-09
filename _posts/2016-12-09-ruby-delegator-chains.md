---
layout: post
published: false
title: 'Ruby: Delegator chains'
---
Have you ever heard of [`SimpleDelegator`](http://ruby-doc.org/stdlib-2.2.1/libdoc/delegate/rdoc/SimpleDelegator.html)? It's a pretty useful tool; it's basically used to extend an instance's behaviour with more methods without modifying its base class (extension is good for code maintainability!).

For example, imagine that you have a class like this:

```rb
class User
  attr_accessor :first_name, :last_name
  
  def initialize(first_name, last_name)
    self.first_name = first_name
    self.last_name = last_name
  end
end
```

Now, if you want add a new method to a `User` that won't be used in all contexts (for example, only in views), you shouldn't do it. Why taint your wonderful class with ugly new code that will extend your to 1000 lines? Instead, just create a new *delegator* class.

```rb
class PrintableUser < SimpleDelegator
  def full_name
    "#{first_name} #{last_name}"
  end
end
```

And use it like this:

```rb
user = User.new 'Daniel', 'Herzog'
printable = PrintableUser.new(user)

puts user.full_name
```

`SimpleDelegator`'s internal implementation is actually very simple (and inneficient): it redefines the `method_missing` method to lookup on the *delegated* object. This means that multiple delegators may be chainned to implement multiple features:

```rb
require 'delegate'

class A < SimpleDelegator
end

class B < SimpleDelegator
end

class C < SimpleDelegator
end

class Original
end

obj = C.new(B.new(A.new(Original.new)))
puts obj.class # C
```

Crazy, right? But there is a problem: `obj` is no longer an `Original` instance, but a `C` instance which delegates to `B`, which delegates to `A`, which delegates to `Original`. This can be a problem when passing delegated objects to other methods without checking it's class (duck typing rules!). For example, on [active-record](https://github.com/rails/rails/tree/master/activerecord), you can save an object to the database if it refers a delegated object:

```rb
class User < ActiveRecord::Base
  belongs_to :group
end

class Group < ActiveRecord::Base
  has_many :users
end

class PrintableUser < SimpleDelegator
  # ...
end

# ...

user = User.create
printable_user = PrintableUser.new(user)
group = Group.last
group.users << printable_user # <- ¡ERROR! Unexpected PrintableUser type, expected User.
```

To fix this issue, you can implement an `original` (or whatever name) method to retrieve the original object. To access a delegated object within the delegator, you need to use [`__getobj__`](http://ruby-doc.org/stdlib-2.2.1/libdoc/delegate/rdoc/SimpleDelegator.html#method-i-__getobj__). Since `self` refers to the current instance, you must iterate over all delegated objects until someone does not respond to `__getobj__`:

```rb
class A < SimpleDelegator
end

class B < SimpleDelegator
end

class C < SimpleDelegator
end

class Original
  def original
    obj = self
    obj = self.__getobj__ while obj.respond_to? :__getobj__
    obj
  end
end

obj = C.new(B.new(A.new(Original.new)))
puts obj.class # C
puts obj.original.class # Original
```

Tada! Also, you can create circular dependencies with delegated objects:

```rb
class A < SimpleDelegator
end

class B < SimpleDelegator
end

a = A.new(nil)
b = B.new(a)

a.__setobj__(b)
```

Leading to a `SystemStackError: stack level too deep` everytime you use `a` or `b`. Be careful!!
