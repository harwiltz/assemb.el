# `assemb.el`

`assemb.el` is a little Emacs Lisp build system that simplifies the
process of building modular documents. It's particularly suited
towards programatically automating the process of assembling files to
be exported using `org-mode`'s suite of tools. It supports
hierarchical targets, "generic" targets, and incremental builds.

## Some Use Cases
* I first started working on `assemb.el` to eliminate redundancy when
  I was writing personal statements for PhD applications. `assemb.el`
  helped me reuse the parts that were common to most essays, and to
  easily compose documents from smaller parts.
* I frequently use `org-mode` when writing notes as I run experiments
  and such. Sometimes I like to demonstrate some algorithms I'm
  working on by embedding source code in my notes, which is then
  executed every time I export my document so the results are
  displayed on the exported product. Sometimes these little programs
  take a noticeable time to run though, and it's annoying to wait for
  it every time I want to export my work. `assemb.el`'s incremental
  building capability saves me lots of time here.
  
## Examples
To use `assemb.el`, build targets are specified in a file called
`assemb.el` in the root of your project.

### Hello, World

Below is a simple _Hello, World_ example.

```lisp
(require 'assemble)

(defun default () (hello-world))

(assemble-target "hello-world" depending on '()
  (message "Hello, Newman"))
```

To build your project, simply execute `M-x assemble`. You'll be
prompted for a target name, but if you press Enter without giving one
it'll execute the target specified in your `default` function.

### `org-mode` exports
Here we'll make a build target to export an org-mode file to LaTeX.

```lisp
(require 'assemble)

(defun default () (foo.pdf))

(assemble-target "foo.pdf" depending on '(foo.org)
  (defile "foo"
    (insert "\\usepackage{custom-latex-style}\n")
	(append-file-to-buffer "foo.org")
	(assemble-latex-bibtex "foo")))
```

The `defile` function is used to make a temporary buffer to export. We
use `insert` and `append-file-to-buffer` to edit the contents of the
buffer to be exported.

### Generic targets
In the previous example, we made a build target that might be useful
for all `.org` files, it is not exactly particular to `foo.org`. Here
we'll show how to improve this to generate a website from several
`.org` files.

```lisp
(require 'assemble)

(defun default () (website))

(assemble-target "website" depending on '(foo.html bar.html))

(assemble-wildcard "html" depending on '(_org style.css)
  (defile _filename
    (insert-css "style.css")
	(append-file-to-buffer _filename)
	(assemble-html (concat _filename ".html"))))
```

Here we define a build target called `"website"` that simply depends
on the files `foo.html` and `bar.html`. Since we wish to build all
HTML files the same way, we use `assemble-wildcard` to define a
build-target for all `.html` files (note that this can be overriden by
a build target for those `.html` files that you want to treat
differently). The `_org` dependency corresponds to the `.org` file
with the same name as the `.html` file being built. Likewise,
`_filename` refers to the base name of the file.
