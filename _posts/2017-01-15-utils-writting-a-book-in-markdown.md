---
layout: post
published: false
title: 'Utils: Writting a book in Markdown'
subtitle: Is that even possible?
---
Say you want to write your very own software or coding book. The first option to proceed is, of course, a text processor like [LibreOffice Writer](https://www.libreoffice.org/discover/writer/). Don't use that. You may use instead a more complex toolchain, like [LaTeX](https://www.latex-project.org/). If you are as lazy as me, you probably won't want to learn how to layout a book using LaTeX. So... what's left?

Here's the best and easiest way: use [Markdown](https://es.wikipedia.org/wiki/Markdown). Of course, it's not as powerful as LaTeX, but it's fast, easy and lightweight.

## Where do we start?

If you don't know Markdown, I recommend you to read [this tutorial](http://www.markdowntutorial.com/). Since we're not serving the book as is (some scattered *.md* files) we need something to transform those source *.md* files into a convinient format like *.pdf* or *.pub*. Please, greet your new friend [Pandoc](http://pandoc.org/).