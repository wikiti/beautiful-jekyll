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
class A < SimpleDelegator
end

class B < SimpleDelegator
end

class C < SimpleDelegator
end

class Original
end

obj = C.new(B.new(A.new(Original.new)))
```

Crazy, right?


TODO