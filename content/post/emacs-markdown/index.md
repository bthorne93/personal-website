---
title: "Emacs Markdown Mode"
date: 2018-04-08T18:07:27-04:00
permalink: /posts/2018/04/08/emacs-monolith
tags:
  - foss
  - emacs
  - editor

image:
  caption: 'Image credit: [Photo by Jason Leung on Unsplash](https://unsplash.com/photos/0sBTrm726C8'
  focal_point: ""
  preview_only: false

---

# Markdown mode

`emacs` has available a markdown aware mode dedicated to the provision of markdown syntax, namely `markdown-mode`, and it is awesome. In the eternal quest for the perfect work flow I have always found that switching between the command-line and a standalone editor to be annoying, due to the fast debug cycle I defualt to when developing code. As a solution I am biting the bullet and committing to learning how to emacs properly.

## The package installer

This deserves a note in itself, but suffice for now to say that `emacs` has a rudimentary package archive and installer, similar to `pip`, which you can access via `M-x package-install`. To get started with markdown mode, you need to add the MELPA package repository to the list of archives your emacs installation is aware of. Add this to your `~/.emacs.d/init.el`:

```emacs
(require 'package)
  (add-to-list
   'package-archives
  ;; '("melpa" . "http://stable.melpa.org/packages/") ; many packages won't show if using stable
   '("melpa" . "http://melpa.milkbox.net/packages/")
   t))
```

Once this is installed, when you edit a file with the `.md` suffix, emacs should automatically load `markdown-mode`.  For an in-depth [tutorial](https://jblevins.org/projects/markdown-mode/ "Tutorial in getting markdown-mode to work").

-------------------------------------------------------------------------------

## A few important keyboard shortcuts

- '`C-c C-c C-p`' processes the markdown to html and previews it in the default browser, this does not create any intermediate `html` files.

- '`C-c C-s <x>`' where, x=(i, b, c). This formats the active-area text to italic, bold, and code formatting environments.

- '`C-c C-i <x>`' where x=(l, i). This enters a UI mode to create links or images.

## Customization mode

Enter the customization mode using `M-x customize-mode`, in which you can edit the command used to process markdown to html. For this option I use:

```
pandoc --from=markdown --to=html --standalone --mathjax --syntax=pygments
```

These options tell pandoc to create a single `html` document, using MathJax to process tagged math environments (rather than the default basic math awareness), and to use `pygments` syntax highlighting for code blocks. The resulting HTML looks quite basic, but nice enough.



