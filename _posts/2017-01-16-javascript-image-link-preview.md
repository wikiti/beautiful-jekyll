---
layout: post
published: false
title: 'Javascript: Image link preview'
subtitle: Like on Imgur's comments
---

Have you ever been on [Imgur.com](http://imgur.com)? When posting comments, if you insert an image url, it can be previewed by hovering the mouse over the link. Something like this:

![Imgur image preview]({{site.baseurl}}/img/gif-preview.gif)

In this post, We're going to implement something similar with HTML, CSS and JavaScript. Let's go!

## Design

The desired behaviour is basically this:

1. Seek for links on our post/comment/page/etc, and flag them with some kind of special identifier (a class, for example).
2. When the user hovers that link, a popover is created with the link's image.
  - Show a spinner or message while the image is loading.
  - Hide the spinner or message after the image has been loaded.
3. When unhovering, the popover is closed.

The popover will be a box with absolute position, placed via javascript under the image's link.

## Implement

We'll divide our problem into subproblems ([Divide and conquer!](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithm)) to fix it easily. To speed up the process, we're going to use [jQuery](https://jquery.com/) for nodes and events handling.

### Creating a simple popover

First, we need to create a modal box. That should be easy; it's just a `div` container placed on an absolute position (over everything) with some CSS style. You can place it at the end of the `body` tag; the position doesn't really matter matter.

```html
<div id="image-preview" class="hidden">
  <span class="loading">Loading...</span>
  <div class="image-preview-wrapper">
    <!-- Our image will go inside this container. -->
  </div>
</div>
```

And the related CSS:

```css

/* Popover box. */
#image-preview {
  position: absolute;
  overflow: hidden;

  margin-top: 20px;
  padding: 5px;

  min-width: 100px;
  max-width: 400px;
  min-height: 100px;
  max-height: 400px;

  background-color: black;
  color: white;

  z-index: 1;
}

/* Image preview tag. */
#image-preview .image-preview-wrapper img {
  max-width: 400px;
  max-height: 400px;
}

/* Hidden helper. */
.hidden {
  visibility: hidden;
}
```

Done! Remember that you can tweak the CSS as you like. Your imagination is your limit!

### Auto converting links

We need a way to track our links. Imagine you have plain text comments. Something like this:

```html
<div class="comment">
  Look at this image: http://www.example.com/image.png.
</div>
```

Now, that link is not an anchor; is just text. If you are building an static website, you can modify all links and transform them manually into something like this:

```html
<div class="comment">
  Look at this image: <a class="image-preview-link" href="http://www.example.com/image.png">http://www.example.com/image.png</a>.
</div>
```

But if you're working on a dynamic website (with real comments) we need to transform those *text-links* into real anchors. If this is your case, you'll probably need to setup that conversion either on the client side or server side.

If you have some containers (like a `div` with a class `.comment`) with plain text, you can use this basic working example to transform links.

```js
// Wait for the DOM to load.
$(document).ready(function() {
  // This is a basic URL matches; it won't cover all cases.
  // Only basic urls that ends on .png, .gif, .jpg or .jpeg.
  var re = /(https?:\/\/(?:[a-z0-9\-]+\.)+[a-z0-9]{2,6}(?:\/[^/#?]+)+\.(?:jpg|jpeg|gif|png))/g;

  // Search on all divs with class "comment", and replace the html of the
  // matched links by ading anchor tags.
  // The '$1' represents the matched text, which are the links found on the text.
  $('div.comment').html(function () {
    return $(this).html()
      .replace(re, "<a class='image-preview-link' target='_blank' href='$1'>$1</a>"); 
  });
});
```

You can find more advices to achieve this task on [this StackOverflow question](http://stackoverflow.com/questions/37684/how-to-replace-plain-urls-with-links).

### Showing the modal

Here's the tricky part! We need to show the modal after hovering the mouse over an image link. For example, imagine we've a tagged link like this:

```html
<div class="comment">
  Look at this image: <a class="image-preview-link" href="http://www.example.com/image.png">http://www.example.com/image.png</a>.
</div>
```

After *mouse-overing* it, we need to execute three actions:

1. Place the popover **under** the image link.
2. Create an image tag referencing the image, and clear previous ones.
3. Show the popover.

Which can be implemented with something like this:

```js
$(document).on("mouseover", ".image-preview-link", function() {
  // Step 1: Place the popover under the image link.
  // Note that `offset()` will return an object with the form `{ left: x, top: y }`
  var position = $(this).offset();
  $("#image-preview").css(position);

  // Step 2: Clear previous images and create the new image tag.
  var img = $("<img>", { src: $(this).attr("href") });
  $("#image-preview .image-preview-wrapper").empty().append(img);

  // Step 3: Show the popover.
  $("#image-preview").removeClass("hidden");
});
```

After unhovering, we need to undo the previous actions, which can be reduced to:

1. Hide the popover.

Which translates into javascript into something like this:

```js
$(document).on("mouseout", ".image-preview-link", function() {
  // Step 1: Hide the popover.
  $("#image-preview").addClass("hidden");
});
```

### Hidde the "Loading" message after the image loads

After the image has been loaded (some images will load too quickly), we need to remove the *"Loading..."* message, and show the image after it has been fully loaded. This can be achieved by adding a callback. When appending the image, add a `load()` callback to it:

```js
var img = $("<img>", { src: $(this).attr("href") });

// NEW CODE!
$("#image-preview .loading").show();
img.hide();

img.load(function() {
  $("#image-preview .loading").hide();
  img.show();
});
// ------

$("#image-preview .image-preview-wrapper").empty().append(img);
```

## Conclusion

Now, you can tweak your popover as you like. Here's a basic [working example on JSFiddle](https://jsfiddle.net/wikiti/4gf00hjv/):

<iframe src="https://jsfiddle.net/wikiti/4gf00hjv/embedded/result"></iframe>

In summary:

- You need to transform plain text links into anchor tags.
- You need to validate that links are valid images (must end in *.png*, *.gif*...).
- You can edit and setup your modal as you like.

## References

- [jQuery](http://jquery.com)
- [Imgur](http://imgur.com)

Happy coding!
