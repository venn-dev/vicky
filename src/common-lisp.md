# Common Lisp

The recommended way is to use sbcl, quicklisp and asdf (included with quicklisp)

## Linux installation

Install sbcl and quicklisp (`cl-quicklisp` on ubuntu) with your package manager.

Load `quicklisp.lisp` installed by the package manager with the sbcl interpreter:

  $ sbcl --load /usr/share/common-lisp/source/quicklisp/quicklisp.lisp

The installation path changes depending on the distro and even between ubuntu versions.
Check package files from package manager documentation to find it.

In the sbcl REPL run:

  * (quicklisp-quickstart:install)

This should download some dependencies and print " ==== quicklisp installed ====== " together with some final tips.

And then:

  * (ql:add-to-init-file)

And press enter. This loads quickstart every time you start an sbcl REPL, otherwise you have to start it with `sbcl --load ~/quickstart/quickstart.lisp`.

## Starting a new project

Create your projects in subdirectories of `~/common-lisp` (you mean need to create it).

If your project name is `hello-world` create `~/common-lisp/hello-world` folder, and add a `hellow-world.asd` file that describes your project and its dependencies:

~/common-lisp/hello-world/hello-world.asd
```
(require "asdf")
(defsystem "lisp-web"
  :author ""
  :license ""
  :description "test"
  :depends-on ()
  :components ((:file "main")))
```


## Hunchentoot

