---
layout: post
published: true
title: 'Rails: How to reivent the wheel'
subtitle: That's how it shouldn't be made
date: 2016-11-24T11:00:00.000Z
---
Today, I saw the following code snippet in some source code:

```rb
def group_by_attribute1(items)
  array = items.map { |i| [i.attribute1, i] }
  array.inject(Hash.new{ |h,k| h[k]=[] }) do |h(k,v)|
    h[k] << v
    h
  end
end
```

Aaaaaah... **MASTE PEESE!**

Of course, there is no need to explain why that's so wrong. That's why the wheel shouldn't be reinvented. Best of all, this code passed all quality checks of its repository.

I think that's why `group_by` was invented:

```rb
def group_by_attribute1(items)
  items.group_by(&:attribute1)
end
```

See? Much easier and cleaner.
