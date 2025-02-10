---
title: "Polyglot Sphinx Documentation"
date: 2125-03-20 # TODO change to publish
permalink: /blog/2025/03/polyglot-sphinx
tags:
  - documentation
  - tips and tricks
---

# Polyglot Sphinx Documentation: User report from documenting too many languages

I like programming languages. The more of them the better. I have even had the
pleasure of working on a couple projects with code in 5 or more languages at once.

This is what I call fun.

The fun part gets a little less fun when you want to create a documentation site
for one of these projects. But, it can be done. I hesitate to say that it can even be
done in a way that isn't horrible.

## The set up

While >=5 languages is a lot, I expect a lot of people will have to deal with at least
2, at some point. Any sufficiently advanced Python project will eventually collect some
compiled code, and many scientific projects will start out in C++ or Fortran before
realizing the users are in Python or Julia, and they need some wrappers.

In my cases, I have had a C++ core and wrappers in (some subset of):
Python, R, Julia, TypeScript, and Rust.

Each of these languages has its own documentation style and native tools. But I
want one website to host the output of all of them.

## The goal

Eventually, we'd like a website that has a "languages" page.
This page will have one subpage for each language, and that
subpage will have roughly the same format:

- Installation
- An example program
- The API documentation

The goal is that this third section will always be generated from
the source code. It is **not** a goal that these pages look exactly
the same.

## My solution

I decided to start with [Sphinx](https://www.sphinx-doc.org/en/master/).
This tool is likely familiar to many Python users, but it is not as Python-specific
as it first appears. A crucial thing that sets Sphinx apart is that it is programmable,
both through an official extension API, and through the ability to inject whatever
code you want into the build process through the `conf.py` file.

Therefore, all you need to do to get a Polygot Sphinx site is the same as a polyglot human:
teach it a lot of languages.

The basics for a new language are:

- Investigate if the language has a Sphinx extension. For example, any Doxygen-using
  code can be documented with [breathe](https://breathe.readthedocs.io/en/latest/).
- If not, see if you can output the documentation in a format that Sphinx can understand.
  Natively this is reStructuredText, but the [myst-parser](https://myst-parser.readthedocs.io/en/latest/)
  extension adds support for a pretty wide range of Markdown-like formats, which ends up being more
  useful for this task.
- If the above don't already exist, you can either write your own, or bail out to generating web pages
  and either embedding them in the Sphinx site, or simply linking to them.

| Language   | Native Tool   | Sphinx Extension       | Markdown Outputting Tool                               |
| ---------- | ------------- | ---------------------- | ------------------------------------------------------ |
| C++        | Doxygen       | breathe                | several, none that are well-regarded enough to mention |
| Python     | Sphinx        | (builtin)              | pydoc-markdown                                         |
| Julia      | Documenter.jl | Sphinx-Julia (broken!) | DocumenterMarkdown.jl                                  |
| TypeScript | JSDoc         | sphinx-js              | jsdoc2md                                               |
| R          | roxygen2      | -                      | rd2markdown                                            |
| Rust       | rustdoc       | sphinxcontrib-rust     | ish: cargo-readme, cargo-doc2readme                    |

Note that the fact that the final column is mostly complete means that
people seeking to use non-Sphinx tools, that can read Markdown, still have
a good chance of success, but the exact tips I give here may not apply.

The rest of this work is a series of tips, tricks, and code snippets for each of the
languages I have worked with. I hope it helps!

The examples are all taken from either the
[BridgeStan](https://github.com/roualdes/bridgestan) or [TinyStan](https://github.com/WardBrian/tinystan) projects.
If you just want to see what the output looks like, you can browse the language sections of both sites:

- https://roualdes.github.io/bridgestan/latest/languages.html
- https://brianward.dev/tinystan/latest/languages.html

## General tip: Good Sphinx defaults

I'm not going to fully explain Sphinx and reStructuredText here. There are many
good guides out there, and plenty of examples in the wild. However, if you
are just hoping to get a site up and running, here are some good extensions:

- If you don't like any of the built-in themes, I tend to use
  [`pydata-sphinx-theme`](https://pydata-sphinx-theme.readthedocs.io/en/latest/).
  In particular, it has nice support for dark mode and multiple versions of the documentation if
  you want to support those.
- I basically always throw [`sphinx_copybutton`](https://pypi.org/project/sphinx-copybutton/)
  on my extensions list. It adds a "copy" button to all code blocks.
- A few of the [built-in, language-agnostic extensions](https://www.sphinx-doc.org/en/master/usage/extensions/index.html)
  are also good. A few ones worth considering are `sphinx.ext.mathjax` if you want to write LaTeX,
  `sphinx.ext.viewcode` and `sphinx.ext.linkcode` to provide links to the source code in documentation,
  and `sphinx.ext.autosectionlabel` to automatically generate anchors for each section.

## General tip: Making a language optional

Any polygot project is liable to end up with developers who are only interested in one
of the languages. It's nice if they don't have to be able to install all the others to
edit and build the documentation.

I use a common idiom for this in the couple projects I maintain. The `conf.py` fails
gracefully on missing dependencies, unless the build is happening in a CI environment.

For example:

```python
import os
import subprocess

# handles the common cases of GitHub Actions and ReadTheDocs
RUNNING_IN_CI = os.environ.get("CI") or os.environ.get("READTHEDOCS")

# LANGUAGE A
try:
    # code that imports the extension or tries to run the native tool, e.g.:
    print("Checking C++ doc availability")
    import breathe
    subprocess.run(["doxygen", "-v"], check=True, capture_output=True)
except Exception as e:
    # if we are in a CI environment, we want to know about this
    if RUNNING_IN_CI:
        raise e
    else:
        # otherwise, we can just not build for this language
        print("Breathe/doxygen not installed, skipping C++ Doc")
        exclude_patterns += ["languages/c-api.rst"]
else:
    # if the above check succeeded, we can now do whatever language-specific config
    extensions.append("breathe")

# LANGUAGE B ...
```

The user will still need Sphinx and its Python dependencies, but they won't need
the complete set of tools for every language you support just to build the docs.

This also works well if the markdown outputs of languages without Sphinx support
are checked into the repository -- the build will just use (possibly stale) versions
of those pages, which is fine for a user who isn't working on them.

## Doxygen family: C, C++, Fortran, etc.

Doxygen supports a surprising number^["C, Python, PHP, Java, C#, Objective-C, Fortran, VHDL, Splice, IDL, and Lex", according to their homepage]
of languages, but most often I see it used for C and C++.

The `breathe` extension is _the_ best option for any Doxygen-using code. It produces
the nicest looking output of any of the tools I will describe in this post, rivialing the
built-in Python support.

I also showed the majority of the configuration you will need in the tip above.
Besides checking that the library and `doxygen` executable are available, you
will need to tell Breathe a bit about your project. Here's the entire rest of the config for
one of my projects (still in `conf.py`, and note that because these are just variable
declarations they're harmless to put outside the `try` block):

```python
# output directory
breathe_projects = {"bridgestan": "./_build/cppxml/"}
breathe_default_project = "bridgestan"
# where to find your Doxygen-commented code
breathe_projects_source = {"bridgestan": ("../src/", ["bridgestan.h"])}
```

The actual documentation page can then use the `:autodoxygenfile:` directive
to generate the documentation for a file. For example:

```rst
.. autodoxygenfile:: bridgestan.h
    :project: bridgestan
    :sections: func typedef var
```

The Breathe documentation covers a lot more options, and is worth a read if you
are using Doxygen for your project, but the above should get you pretty far.

### Bonus tip: Macro preprocessing

If your C(++) code has macros, you might get better results by enabling some [preprocessing
in Doxygen](https://www.doxygen.nl/manual/preprocessing.html) through some extra configuration in `conf.py`.

For example, BridgeStan has a `BS_PUBLIC` macro which is used to mark functions
as exported from a shared library. Doxygen doesn't like the `__attribute` and `__declspec`
attributes that this expands to by default, so we use the following to define it away:

```python
breathe_doxygen_config_options = {
    "ENABLE_PREPROCESSING": "YES",
    "MACRO_EXPANSION": "YES",
    "EXPAND_ONLY_PREDEF": "YES",
    "PREDEFINED": "BS_PUBLIC=",
}
```

## Python

Python has built-in support in Sphinx. I hesitate to suggest that I have much
to add for people who have already used Sphinx for Python, but I will mention
a couple extra-useful extensions and options:

### Intersphinx

Sphinx has a built-in feature called [intersphinx](https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html)
which allows you to link to other Sphinx-built documentation frictionlessly.
Just add `"sphinx.ext.intersphinx"` to your `extensions` list in `conf.py`,
and then add a `intersphinx_mapping` dictionary to the same file. For example:

```python
intersphinx_mapping = {
    "python": ( "https://docs.python.org/3/", None),
    "numpy": ("https://numpy.org/doc/stable/", None),
}
```

Now, a directive like

```rst
:py:func:`os.path.join`
```

will link directly to the Python documentation.

### Autodoc

Another built-in extension, [autodoc](https://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html),
provides the Python equivalents of the `autodoxygenfile` directive I showed above.

For example, it lets you write blocks like

```rst
.. autoclass:: bridgestan.StanModel
   :members:
```

and automatically generate documentation for that class and all its members.

This can save a lot of boilerplate, and is extra helpful because it can automatically pick
up new methods as you add them. There are a lot more options than just `autoclass`, so
I recommend reading more in the documentation.

## Julia

Julia is the first language in the "Get it to generate Markdown" category. There
is a `Sphinx-Julia` package in existence, but it was last substantially updated in 2018,
and I have been unable to get it to work with modern versions of Sphinx.

Luckily, Julia makes this task relatively easy. Julia packages typically use the
`Documenter.jl` package to generate documentation, and there is a
[`DocumenterMarkdown.jl`](https://github.com/JuliaDocs/DocumenterMarkdown.jl)
package that can be asked to generate Markdown files which have been processed,
e.g. by expanding `@docs` sections and linking to the source code. This plays
nicely with unrecognized Markdown directives, so you can use myST directives
in the Julia documentation and they will be passed through to allow Sphinx to
render them.

There's a catch, of course, which is that DocumenterMarkdown is not very actively
maintained, so it does require an older version of Documenter.jl. This has been
fine in my experience, as many of the more recent versions of Documenter.jl seem
to be focused on improving HTML output, which we don't need. Still, it's worth being
aware of.

At any rate, the config is relatively simple. You write your Julia docs as you would
for any Julia package, except you set the `format` argument to `Markdown` in the
`makedocs` call. I also copy the generated files into the Sphinx source directory.
Here's my entire `make.jl` file:

```julia
using Documenter, BridgeStan
using DocumenterMarkdown

makedocs(
    format = Markdown(),
    repo = "https://github.com/roualdes/bridgestan/blob/main{path}#{line}",
)

cp(
    joinpath(@__DIR__, "build/julia.md"),
    joinpath(@__DIR__, "../../docs/languages/julia.md");
    force = true,
)
cp(
    joinpath(@__DIR__, "build/assets/Documenter.css"),
    joinpath(@__DIR__, "../../docs/_static/css/Documenter.css");
    force = true,
)
```

On the Sphinx side, you'll now need the `myst-parser` extension to read the Markdown,
and to make sure the Documenter css is included:

```python
extensions = [
    # ...
    "myst_parser",
]
suppress_warnings = ["myst.xref_missing"] # Julia doc generates raw html links
html_static_path = ["_static"]
html_css_files = [
    "css/Documenter.css",
]
```

Finally, if you want to have the Julia doc build run as part of the Sphinx build,
you can use another `try` block like the one I showed for C++ above:

```python

try:
    print("Building Julia doc")
    subprocess.run(
        ["julia", "--project=.", "./make.jl"],
        cwd=pathlib.Path(__file__).parent.parent / "julia" / "docs",
        check=True,
    )
except Exception as e:
    # fail loudly in Github Actions
    if RUNNING_IN_CI:
        raise e
    else:
        print("Failed to build julia docs!\n", e)
```

### Bonus tip: Helping people navigate duplicated files.

If you use this method (and follow my advice to check-in the copied result in the Sphinx folder),
there is one downside: There will be two `julia.md` files in your repository. One, the actual
source, will be in the directory of your Julia package, and the other, which is overwritten on
build, will be in the Sphinx directory.

To help people keep this straight, I add a note to the top of the source file in a `@raw html` block,
which gets reproduced in the Sphinx output. For example:

````markdown
```@raw html
% NB: If you are reading this file in docs/languages, you are reading a generated output!
% This should be apparent due to the html tags everywhere.
% If you are reading this in julia/docs/src, you are reading the true source!
% Please only make edits in the later file, since the first is DELETED each re-build.
```
````

## TypeScript (and JavaScript)

https://github.com/jsdoc2md/jsdoc-to-markdown
https://github.com/mozilla/sphinx-js

## R

https://github.com/Genentech/rd2markdown

## Rust

Rust, like Julia, uses Markdown extensively in its own documentation system. However,
Rust was one of the few items I more or less gave up on, and my documentation
page was just a link to the docs.rs site which is automatically generated by
uploading a crate to crates.io.

However, while writing this article, I found [`sphinxcontrib-rust`](https://gitlab.com/munir0b0t/sphinxcontrib-rust).
This ends up being pretty similar to the doxygen and R cases. It runs a Rust program
and then produces a Markdown file which Sphinx can read. You can then include this file
as a section in a larger, hand-written document on the language.

Finding this actually delayed the writing of this article, because I took
some time to try it out in BridgeStan. I found a few issues, but the maintainer was
incredibly responsive and they have all been resolved or sufficiently worked around.

So, you know the drill. Here's what I have in `conf.py`:

```python
try:
    print("Checking Rust doc availability")
    subprocess.run(["cargo", "--version"], check=True, capture_output=True)
except Exception as e:
    if RUNNING_IN_CI:
        raise e
    else:
        print("Rust not installed, skipping Rust Doc")
        exclude_patterns += ["languages/rust.md", "languages/_rust"]
else:
    extensions.append("sphinxcontrib_rust")
    # minimum needed for Rust, but you may need more myst
    # extensions depending on your specific docstrings!
    myst_enable_extensions += ["colon_fence", "attrs_block"]
    rust_crates = {
        "bridgestan": str(pathlib.Path(__file__).parent.parent / "rust"),
    }
    rust_doc_dir = "languages/_rust"
    rust_rustdoc_fmt = "md"
    rust_generate_mode = "changed" if not RUNNING_IN_CI else "always"
```

This will produce a `lib.md` file in the `languages/_rust/CRATE_NAME/` directory,
which contains only the API documentation for the crate.
You can then include this in another Markdown file wherever you want it to
appear, like with the R example above. Be aware of the
[limitations](https://sphinxcontrib-rust.readthedocs.io/en/latest/docs/limitations.html),
and it is worth checking that your docs still look reasonable under `cargo doc`.

Tip: You can use `:start-line: 2` to skip the `# Crate CRATE_NAME` line that `sphinxcontrib-rust` generates.

## General tip: Github Pages

- `sphinx.ext.githubpages` is a built-in extension that just generates the `.nojekyll`
  file that GitHub Pages needs to serve a folder as raw html.

- force pushing to gh-pages
