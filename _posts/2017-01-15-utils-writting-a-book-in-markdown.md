---
layout: post
published: true
title: 'Utils: Writting a book in Markdown'
subtitle: Is that even possible?
---
Say you want to write your very own software or coding book. The first option to proceed is, of course, a text processor like [LibreOffice Writer](https://www.libreoffice.org/discover/writer/). Don't use that. You may use instead a more complex toolchain, like [LaTeX](https://www.latex-project.org/). If you are as lazy as me, you probably won't want to learn how to layout a book using LaTeX. So... what's left?

Here's the best and easiest way: use [Markdown](https://es.wikipedia.org/wiki/Markdown). Of course, it's not as powerful as LaTeX, but it's fast, easy and lightweight.

## Where do we start?

If you don't know Markdown, I recommend you to read [this tutorial](http://www.markdowntutorial.com/). Since we're not serving the book as is (some scattered *.md* files) we need something to transform those source *.md* files into a convinient format like *.pdf* or *.pub*. Please, greet your new friend [Pandoc](http://pandoc.org/).

## Installing

Please, check [this page](http://pandoc.org/installing.html) for more information. On ubuntu, it can be installed with this command:

```sh
sudo apt-get install pandoc
```

<a href="https://www.youtube.com/watch?v=n3TAEaTCJHg" target="_blank">Easy peasy lemon squeezy</a>.

Also, we'll be using [make](https://www.gnu.org/software/make/), so don't forget to install it:

```sh
sudo apt-get install make
```

## Folder structure

Before creating scattered files like a maniac, let's create a folder tree structure to keep files organized. For example:

```
my-book/         # Root directory.
|- build/        # Folder used to store builded (output) files.
|- chapters/     # Markdowns files; one for each chapter.
|- images/       # Images folder.
|  |- cover.png  # Cover page for epub.
|- metadata.yml  # Metadata content (title, author...).
|- Makefile      # Makefile used for building our books.
```

Simple, isn't it? Don't worry abuot the files' content; I'll explain each file on the following sections.

You can find the template used on this post in [this GitHub repository](TODO). 

## Setup generic data

First, we need a title and an author, don't we? That's the purpose of *metadata.yml*. Here's a example:

```yml
---
title: My book title
author: Daniel Herzog
rights:  Creative Commons Attribution 4.0 International
language: en-US
tags: [book, my-book, etc]
abstract: |
  Your summary text.
---
```

You get the idea, right? You can find the list of all available keys on [this page](http://pandoc.org/MANUAL.html#extension-yaml_metadata_block).

## Creating chapters

Creating a new chapter is as simple as creating a new markdown file in your *chapters/* folder. Something like this:

```
chapters/01-introduction.md
chapters/02-installation.md
chapters/03-usage.md
chapters/04-references.md
```

Pandoc will join them automatically. All you need to specify is at least a title:

```md
# Introduction

This is the first paragraph of the introduction chapter.

## First

This is the first subsection.

## Second

This is the second subsection.
```

Each title (*#*) will represent a chapter, while each subtitle (*##*) will represent a chapter's section. You can use as many levels as markdown supports (5 or 6).

### Links between chapters

Imagine you want to reference a chapter from another; when the reference is pressed, it will redirect you to that section. It can be achieved with anchor links:

```md
// chapters/01-introduction.md
# Introduction

For more information, check the [Usage] chapter.

// chapters/02-installation.md
# Usage

...
```

Just use the chapter's title name. If you want to rename the reference, use this syntax:

```md
For more information, check [this](#usage) chapter.
```

Anchor names should be downcases, and spaces, colons, semicolons... should be replaced with hyphens. Instead of `Chapter title: A new era`, you have: `chapter-title-a-new-era`.

### Links between sections

It's the same as chapters' links:

```md
# Introduction

## First

For more information, check the [Second] section.

## Second

...
```

Or, renamed:

```md
For more information, check [this](#second) section.
```

Same rules apply here as chapters' links.

## Captions

### Insert an image

To insert an image with a caption use Markdown syntax:

```md
![A cool seagull.](images/seagull.png)
```

Pandoc will automatically convert the image into a caption.

If you want to resize the image, you can use this syntax:

```md
![A cool seagull.](images/seagull.png){ width=50% height=50% }
```

Also, to reference an image, use LaTeX labels:

```md
Please, admire the gloriousnes of Figure \ref{seagull_image}.

![A cool seagull.\label{seagull_image}](images/seagull.png)
```

### Insert a table

Just insert a Markdown table, and use the `Table: Your table description` syntax to add a caption:

```md
| Index | Name |
| ----- | ---- |
| 0     | AAA  |
| 1     | BBB  |
| ...   | ...  |

Table: This is an example table.
```

If you want to reference a table, use LaTeX labels:

```md
Please, check Table /ref{example_table}.

| Index | Name |
| ----- | ---- |
| 0     | AAA  |
| 1     | BBB  |
| ...   | ...  |

Table: This is an example table.\label{example_table}
```

### Insert a math formula

Wrap a LaTeX math equation between `$` delimiters:

```md
This, $\mu = \sum_{i=0}^{N} \frac{x_i}{N}$, the mean equation, ...
```

Pandoc will transform them automatically into images using online services.

If you want to center the equation instead of inlining it, use double `$$` delimiters:

```md
$$\mu = \sum_{i=0}^{N} \frac{x_i}{N}$$
```

## Output

We'll be using a *Makefile* to automatize the building process. Instead of using the *pandoc cli util*, we're going to use some *make* commands. The *Makefile* looks like this:

```make
BUILD = build
OUTPUT_FILENAME = book
METADATA = metadata.yml
CHAPTERS = chapters/*.md
TOC = --toc --toc-depth=2
COVER_IMAGE = images/cover.png
LATEX_CLASS = report
MATH_FORMULAS = --webtex

all: book

book: epub html pdf

clean:
	rm -r $(BUILD)

epub: $(BUILD)/epub/$(OUTPUT_FILENAME).epub

html: $(BUILD)/html/$(OUTPUT_FILENAME).html

pdf: $(BUILD)/pdf/$(OUTPUT_FILENAME).pdf

$(BUILD)/epub/$(OUTPUT_FILENAME).epub: $(METADATA) $(CHAPTERS)
	mkdir -p $(BUILD)/epub
	pandoc $(TOC) -S $(MATH_FORMULAS) --epub-metadata=$(METADATA) --epub-cover-image=$(COVER_IMAGE) -o $@ $^

$(BUILD)/html/$(OUTPUT_FILENAME).html: $(CHAPTERS)
	mkdir -p $(BUILD)/html
	pandoc $(TOC) $(MATH_FORMULAS) --standalone --to=html5 -o $@ $^

$(BUILD)/pdf/$(OUTPUT_FILENAME).pdf: $(METADATA) $(CHAPTERS)
	mkdir -p $(BUILD)/pdf
	pandoc $(TOC) $(MATH_FORMULAS) -V documentclass=$(LATEX_CLASS) -o $@ $^
```

Don't worry! If you're not familiarized with *make*, you don't need to actually understand how that works.

### Export to PDF

Use this command:

```sh
make pdf
```

The generated file will be placed in *build/pdf*.

Please, note that PDF file generation requires some extra dependencies (~ 800 MB):

```sh
sudo apt-get install texlive-latex-base texlive-fonts-recommended texlive-latex-extra 
```

### Export to EPUB

Use this command:

```sh
make epub
```

The generated file will be placed in *build/epub*.

### Export to HTML

Use this command:

```sh
make html
```

The generated file(s) will be placed in *build/html*.

### Extra configuration

If you want to configure the output, you'll probably have to look the [Pandoc Manual]() for further information about pdf (LaTeX) generation, custom styles, etc.

## References

- [Pandoc](http://pandoc.org/)
- [Pandoc Manual](http://pandoc.org/MANUAL.html)
- [Wikipedia: Markdown](http://wikipedia.org/wiki/Markdown)

Happy coding!
