---
layout: post
published: true
title: 'Utils: recording screen gifs'
subtitle: Add some motion to your life!
---
Have you ever seen some cool videos on those fancy and well-known GitHub projects' READMEs Like [this one](https://github.com/wikiti/extension-networking/tree/master/examples/tic-tac-toe#description)? Oh, well, that repo it's mine, and it isn't a famouse one (sigh).

Anyways, those *videos* are actually *GIF*s files, loaded as images. If you want to do so, it's very easy to do so. You only have to record the video, convert it into a *GIF*, and reference it in your markdown file.

## Recording & converting

The desired behaviour for an application to record your screen should be as simple as selecting a region, and pressing a *record* button, and export the recording as a *GIF* file. Fortunately, there are many tools which work like explained. Here is a list of tools for each common operating systems:

- Linux: [Peek](https://github.com/phw/peek)
- Windows: [ScreenToGif](https://screentogif.codeplex.com/), [Recordit](http://recordit.co/)
- Mac: [Giphy](https://itunes.apple.com/es/app/giphy-capture.-the-gif-maker/id668208984?mt=12), [Recordit](http://recordit.co/)

## Reference

Just place your *GIF* file somewhere in your repository, and reference it within your markdown file.

````md
![Alternative text](relative/path/to/your/file.gif)
````

Remember that, if you're using GitHub's markdown editor (for example, on issues or pull requests), you may just drag and drop your *GIF*. You can also reference urls:

````md
![Alternative text](http://www.example.com/example.gif)
````

And this should be the result:

![Lawl gif]({{site.baseurl}}/img/mV9bFz7.gif)
