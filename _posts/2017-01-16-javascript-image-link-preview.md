---
layout: post
published: false
title: 'Javascript: Image link preview'
subtitle: Like on Imgur's comments
---

Have you ever been on [Imgur.com](http://imgur.com)? When writing comments, if you insert an image url, it can be previewed by hovering the mouse over the link. Something like this:

![Imgur image preview]({{site.baseurl}}/img/gif-preview.gif)

In this post, I'm going to implement something similar with HTML, CSS and JavaScript. Let's go!

## Design

The desired behaviour is basically the following:

1. We seek for links on our post/comment/page/etc, and flag them with some kind of special identifier (a class, for example).
2. When the user hovers that link, a popover is created with the link's image, reducing its size.
  - Show a spinner or message while the image is loading.
  - Hide the spinner or message after the image has been loaded.

The popover will be a box with absolute position, placed via javascript under the image's link.

## Implement

We'll divide our problem into subproblems to fix it easily. To speed up the process, we're going to use [jQuery](https://jquery.com/) for nodes and events handling.

### Creating a simple modal

First, we need to create a modal box. That should be easy; it's just a `div`ontainer with some style. You can place it at the end of the `body` tag.

```html
<div id="image-preview" class="hidden">
  <span class="loading">Loading...</span>
  <div class="image-preview-wrapper">
    <!-- Our image will go inside this container. -->
  </div>
</div>
```

```css

/* Popover box. */
#image-preview {
  position: absolute;

  padding: 5px;

  min-width: 100px;
  max-width: 500px;
  min-height: 100px;
  max-height: 500px;

  background-color: black;
  color: white;

  z-index: 1;
}

.hidden {
  visibility: hidden;
}
```

Now, our popover style is done. Of course, you can tweak the css as you like. Your imagination is your limit!

### Auto converting links

...

### Showing the modal after hover

Here's the tricky part! We need to show the modal after hovering the mouse over an image link. For example, imagine we've a link like this:

```html
<div class="comment">
  Look at this image: <a class="image-preview-link" href="http://www.example.com/image.png">http://www.example.com/image.png</a>.
</div>
```

After hovering the mouse over it, we need to execute three actions:

1. Place the popover **under** the image link.
2. Create an image tag referencing the image, and clear previous ones.
2. Show the popover.

Which can be implemented with something like this:

```js
$(document).on("hover", ".image-preview-link", function() {
  // Step 1: Place the popover under the image link.
  // Note that `offset()` will return an object with the form `{ left: x, top: y }`
  var position = $(this).offset(); 
  $("#image-preview").css(position);

  // Step 2: Create the image tag.
  $("#image-preview image-preview-wrapper").empty().append(
    $("<img>", { src: $(this).attr("href") })
  );
  ...

  // Step 3: Show the popover.
  $("#image-preview").addClass("hidden");
});
```

After unhovering the mouse over it, we need to undo the previous actions, which can be reduced to:

1. Hide the popover.

### Hidden the spinner after image load

...

## Conclusion

...

{JSFiddle example here}

## References

...
