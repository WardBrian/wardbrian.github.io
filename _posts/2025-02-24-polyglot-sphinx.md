---
title: "Polyglot Sphinx Documentation"
date: 2025-02-24
permalink: /blog/2025/02/polyglot-sphinx
tags:
  - documentation
  - tips and tricks
  - programming languages
---

# Polyglot Sphinx Documentation: User report from documenting too many languages

I like programming languages. The more of them the better. I have even had the
pleasure of working on a couple projects with code in 5 or more languages at once.

This is what I call fun.

The fun part gets a little less fun when you want to create a documentation site
for one of these projects. But, it can be done. I might even dare to say that it can be
done in a way that isn't horrible. This is more of a family recipe than a cookbook --
I think everyone's set-up will be just different enough to make a true tutorial impossible.

## The project

While >=5 languages is a lot, I expect a lot of people will have to deal with at least
2, at some point. Any sufficiently advanced Python project will eventually collect some
compiled code, and many scientific projects will start out in C++ or Fortran before
realizing the users are in Python or Julia, and they need some wrappers.

In my cases, I have had a C++ core and wrappers in (some subset of)
Python, R, Julia, TypeScript, and Rust.

Each of these languages has its own documentation style and native tools for emitting it
in various formats. But I want one unified way to generate a website which documents all of them.

## The goal

Eventually, we'd like a website that has [a "languages" page](https://roualdes.github.io/bridgestan/latest/languages.html).
This page will have one subpage for each language, and that
subpage will have roughly the same format:

- Installation
- An example program
- The API documentation

The goal is that this third section will always be generated from
the source code. Additionally, we'd like this source code to be annotated
following the standard practices for the specific language it is written in.
Doing so makes it easier for people who are familiar with a given language to
contribute, and it keeps things like built-in help systems working.

It is **not** a goal that these pages look _exactly_ the same.

## My solution

I decided to start with [Sphinx](https://www.sphinx-doc.org/).
This tool is likely familiar to many Python users, but it is not as Python-specific
as it first appears. A crucial thing that sets Sphinx apart is that it is _highly_ programmable,
both through an official extension API, and through the ability to inject whatever
code you want into the documentation build process through
[Sphinx's `conf.py` file](https://www.sphinx-doc.org/en/master/usage/configuration.html).

Therefore, all you need to do to get a Polygot Sphinx site is the same as a polyglot human:
teach it a lot of languages.

The basics for a new language are:

- Investigate if the language has a Sphinx extension. For example, any Doxygen-using
  code can be documented with [breathe](https://breathe.readthedocs.io/).
- If not, see if you can output the documentation in a format that Sphinx can understand.
  Natively this is reStructuredText, but the [myst-parser](https://myst-parser.readthedocs.io/)
  extension adds support for a pretty wide range of Markdown-like formats, which ends up being more
  useful for this task.
- If the above don't already exist, you can either write your own, or bail out to generating web pages
  and either embedding them in the Sphinx site, or simply linking to them.

| Language   | Native Tool                                       | Sphinx Extension                                                      | Markdown Outputting Tool                                                                                                   |
| ---------- | ------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| C++        | [Doxygen](https://www.doxygen.nl/)                | [breathe](https://breathe.readthedocs.io)                             | several, none that are well-regarded enough to mention                                                                     |
| Python     | [Sphinx](https://www.sphinx-doc.org/)             | (builtin)                                                             | similarly, several options depending on docstring style                                                                    |
| Julia      | [Documenter.jl](https://documenter.juliadocs.org) | Sphinx-Julia (broken!)                                                | [DocumenterMarkdown.jl](https://documentermarkdown.juliadocs.org)                                                          |
| TypeScript | [JSDoc](https://jsdoc.app/)                       | [sphinx-js](https://github.com/mozilla/sphinx-js)                     | [jsdoc2md](https://github.com/jsdoc2md/jsdoc-to-markdown)                                                                  |
| R          | [roxygen2](https://roxygen2.r-lib.org/)           | -                                                                     | [rd2markdown](https://github.com/Genentech/rd2markdown)                                                                    |
| Rust       | [rustdoc](https://doc.rust-lang.org/rustdoc/)     | [sphinxcontrib-rust](https://gitlab.com/munir0b0t/sphinxcontrib-rust) | ish: [cargo-readme](https://github.com/webern/cargo-readme), [cargo-doc2readme](https://github.com/msrd0/cargo-doc2readme) |

Note that the final column is mostly complete -- meaning that
people seeking to use Markdown-friendly (non-Sphinx) tools still have
a good chance of success, even if the exact tips I give here may not apply.

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
of the languages. It's nice if they don't have to install all the others to
edit and build the documentation.

I use a common idiom for this in the couple projects I maintain. The `conf.py` fails
gracefully on missing dependencies, _unless_ the build is happening in a CI environment.

For example:

```python
import os
import subprocess

# handles the common cases of GitHub Actions and ReadTheDocs
RUNNING_IN_CI = os.environ.get("CI") or os.environ.get("READTHEDOCS")
BASE_DIR = pathlib.Path(__file__).parent.parent

# LANGUAGE A
try:
    # code that imports the extension or tries to run the native tool, e.g.:
    print("Checking C++ doc availability")
    import breathe
    subprocess.run(["doxygen", "-v"], check=True)
except Exception as e:
    # if we are in a CI environment, we want to know that we're missing a dependency
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

A would-be contributor will still need Sphinx and its Python dependencies,
but they won't need the complete set of tools for every language supported by
the project just to build the docs.

This also works well if the markdown outputs of languages without Sphinx support
are checked into the repository -- the build will just use (possibly stale)
checked-in versions of those pages, which is fine for a user who isn't working on them.

## Doxygen family: C, C++, Fortran, etc.

[Doxygen](https://www.doxygen.nl/) supports a surprising number of languages
("C, Python, PHP, Java, C#, Objective-C, Fortran, VHDL, Splice, IDL, and Lex", according to their homepage),
but it is the "default" tool for C and C++.

The [`breathe` extension](https://breathe.readthedocs.io/en/latest/) is _the_ best option for any
Doxygen-using code. It produces the nicest looking output of any of the tools I will describe in
this post, rivaling the built-in Python support.

I also showed the majority of the configuration you will need in the tip above.
Besides checking that the library and `doxygen` executable are available, you
will need to tell Breathe a bit about your project. Here's the entire rest of the config for
one of my projects (still in `conf.py`, and note that because these are just variable
declarations they're harmless to put outside the `try` block):

```python
# output directory for generated files
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

If your C(++) code has macros, especially ones that expand to `__attribute` or other items that change
the signature of the function you're documenting, you may find that these degrade the appearance of the
result from Doxygen. In these cases, you might get better results by enabling some [preprocessing
in Doxygen](https://www.doxygen.nl/manual/preprocessing.html) through some extra configuration in `conf.py`
and making them expand to something more friendly, such as the empty string.

For example, BridgeStan has a `BS_PUBLIC` macro which is used to mark functions
as exported from a shared library (using `__attribute` and `__declspec`). The
following configuration in `conf.py` tells Doxygen to preprocess the code before
parsing, and in particular to replace `BS_PUBLIC` with nothing:

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

Now, a [cross-reference directive](https://www.sphinx-doc.org/en/master/usage/domains/python.html#cross-referencing-python-objects) like

```rst
:py:func:`os.path.join`
```

will link directly to the Python documentation for `os.path.join()` on docs.python.org.

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

Luckily, Julia makes the task of generating markdown relatively easy. Julia packages
typically use the [`Documenter.jl`](https://documenter.juliadocs.org) package to
generate documentation, and there is a
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
    repo = "https://github.com/WardBrian/TinyStan/blob/main{path}#{line}",
)

cp(
    joinpath(@__DIR__, "build/julia.md"),
    joinpath(@__DIR__, "../../../docs/languages/julia.md");
    force = true,
)
cp(
    joinpath(@__DIR__, "build/assets/Documenter.css"),
    joinpath(@__DIR__, "../../../docs/_static/css/Documenter.css");
    force = true,
)
```

Back in Sphinx's `conf.py`, you'll now need the `myst-parser` extension to read the Markdown,
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
        cwd=BASE_DIR / "clients" / "julia" / "docs",
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

To help people keep this straight, I add a note to the top of the source Markdown
file in a `@raw html` block, which gets reproduced in the Sphinx output. For example:

````markdown
```@raw html
% NB: If you are reading this file in docs/languages, you are reading a generated output!
% This should be apparent due to the html tags everywhere.
% If you are reading this in julia/docs/src, you are reading the true source!
% Please only make edits in the julia/docs/src file, since the first is DELETED each re-build.
```
````

## TypeScript (and JavaScript)

My advice here assumes you've documented your TypeScript code with JSDoc-style comments.

There is a `sphinx-js` [extension from Mozilla](https://github.com/mozilla/sphinx-js) that can
read JSDoc comments, but it was marked as a public archive during the writing of this post.
There does appear to be [a fork by the Pyodide project](https://github.com/pyodide/sphinx-js-fork)
that is still active.

I didn't know about this package before now, so I actually built a different solution,
based around a tool called [`jsdoc2md`](https://github.com/jsdoc2md/jsdoc-to-markdown).
Which I guess I will be sticking to, at least until the sphinx-js maintenance status
is more clear.

My existing solution is pretty similar to the Julia case, where I build a Markdown file
and copy it into the Sphinx source directory, but it requires a bit more configuration.

First, there are some JavaScript developer dependencies for your `package.json`.
Note that the `babel` mentions are only necessary if you are using TypeScript,
and `@godaddy/dmd` is technically optional, but I have found it to greatly improve
the quality of the rendered output.

```json
  "devDependencies": {
    "@babel/cli": "^7.25.9",
    "@babel/core": "^7.26.0",
    "@babel/preset-env": "^7.26.0",
    "@babel/preset-typescript": "^7.26.0",
    "@godaddy/dmd": "^1.0.4",
    "jsdoc-babel": "^0.5.0",
    "jsdoc-to-markdown": "^9.0.5",
  }
```

(version numbers are just what I have installed, you can probably use newer ones)

And a configuration file for jsdoc2md. I have this live in a `doc` sub-folder of
the TypeScript client. Again, a lot of it is really only necessary for TS:

```json
{
  "source": {
    "includePattern": ".+\\.ts(doc|x)?$",
    "excludePattern": ".+\\.(test|spec).ts"
  },
  "template": "language-doc.hbs",
  "plugins": ["plugins/markdown", "node_modules/jsdoc-babel", "@godaddy/dmd"],
  "heading-depth": 3,
  "babel": {
    "extensions": ["ts", "tsx"],
    "ignore": ["**/*.(test|spec).ts"],
    "babelrc": false,
    "presets": [
      ["@babel/preset-env", { "targets": { "node": true } }],
      "@babel/preset-typescript"
    ],
    "plugins": []
  }
}
```

The template file `language-doc.hbs` (also in this `typescript/doc/` directory)
is a [Handlebars](https://handlebarsjs.com/)
file that gets filled with the output of the jsdoc2md command. A really barebones
example would just be `{{>main}}`, which includes the main template from the
`jsdoc-to-markdown` package. I use
[mine](https://github.com/WardBrian/tinystan/blob/85836d28060892e216b1e66cd100a9a3197f6fc7/clients/typescript/doc/language-doc.hbs)
to add some installation instructions and the like. Of note is the ability to pre-(re-?)declare
some links to fix broken ones jsdoc2md generates.
**My kingdom for a consistent rule of how markdown section headers get turned into html headers.**

Finally, in package.json, I provide a script which helps run it all:

```json
  "scripts": {
    "doc": "jsdoc2md --template ./doc/language-doc.hbs --plugin @godaddy/dmd --heading-depth=3 --configure ./doc/jsdoc2md.json --files src/*.ts "
  }
```

Astute readers will notice that there is some duplicated config between this command and
the json file. At least for the version of jsdoc2md I was using, I couldn't get it to
read all of the items out of the json file, so some ended up repeated. I left them in the
json file for clarity and aspirational reasons.

This command will write the Markdown to standard output. The rest of the config occurs in
`conf.py` to run this command and place the output:

```python
try:
    print("Building JS doc")
    # this allows you to set 'YARN' to 'npm run', for example
    yarn = os.getenv("YARN", "yarn").split()
    ret = subprocess.run(
        yarn + ["--silent", "doc"],
        cwd=BASE_DIR / "clients" / "typescript",
        check=True,
        capture_output=True,
        text=True,
    )
    with open("./languages/js.md", "w") as f:
        f.write(ret.stdout)

# you know the drill...
except Exception as e:
    if RUNNING_IN_CI:
        raise e
    else:
        print("Failed to build JS docs!\n", e)
```

Assuming you have `myst-parser` installed and listed
as an extension in `conf.py`, you should be good to go with your (Type|Java)Script docs.

## R

R is another language in this bucket of "get it to generate Markdown" solutions, but getting
it right also requires a bit more pre- and post-processing, which makes it more interesting
than me just linking to another "something2md" package.

It does, however, _involve_ another "something2md" package. In this case, it's
[`rd2markdown`](https://github.com/Genentech/rd2markdown), where `Rd` is the
R documentation format. Typically, these files themselves are generated by a package
called `roxygen2`, which processes comments in the source code. Of course, `roxygen2`'s format is
markdown-adjacent, so this ends up being a lossy process that ultimately converts
something like markdown into something that isn't, and then back into a more processed
form of markdown. Remember when I said this was fun?

Anyway, while `rd2markdown` is on CRAN, the development version on GitHub is more up-to-date,
so I assume that version in the following instructions.

In contrast to my general flow here, I'm actually going to start by showing the `conf.py`
code:

```python
try:
    print("Building R doc")
    subprocess.run(
        ["Rscript", "convert_docs.R"],
        cwd=BASE_DIR / "clients" / "R",
        check=True,
    )

except Exception as e:
    # fail loudly in Github Actions
    if RUNNING_IN_CI:
        raise e
    else:
        print("Failed to build R docs!\n", e)
```

So, at this level, it looks almost exactly like Julia. Assuming that the
`convert_docs.R` script it is calling in my R client directory isn't too bad ...

```R
# Converts R documentation (.Rd) files to markdown (.md) files for use in
# Sphinx.

library(rd2markdown)
library(roxygen2)

roxygen2::roxygenize()

files <- list.files("man", pattern = "*.Rd")

# we only want to doc exported functions, so we need
# to read the NAMESPACE file
namespace <- paste0(readLines("NAMESPACE"), collapse="\n")

for (f in files){
    # strip off .Rd
    name <- substr(f, 1, nchar(f)-3)

    if (!grepl(name, namespace, fixed=TRUE)){
        print(paste0("Skipping unexported ", name))
        next
    }

    # read .Rd file and convert to markdown
    rd <- rd2markdown::get_rd(file = file.path(".", "man", f))
    md <- rd2markdown::rd2markdown(rd, fragments = c())
    # replaces the headers with more appropriate levels for embedding
    # hopefully one day we can just pass level=3, see
    # https://github.com/Genentech/rd2markdown/issues/41
    md_indented <- gsub("(#+)", "\\1##", md)

    # write it to the docs folder
    writeLines(md_indented, file.path("..", "..", "docs", "languages", "_r",
        paste0(name, ".md")))

}
```

oh.

I mean, it certainly isn't horrible, but it's a good bit hackier than the others.
This is also the result of a lot of trial and error, so it's considerably _less_
hacky than it once was.

First: we run `roxygenize` to make sure our `.Rd` files are actually up to date
in our `man` directory. Then, we'd like to loop over all of them, but, in general,
you may have private/unexported functions that you don't want to document. So,
we check if each name is in the `NAMESPACE` file, and skip it if not.

Finally, we read each `.Rd` file, convert it to markdown, and write the result to the `docs`
directory. Because this will be embedded in a larger document, we increase the depth
of the markdown headers. As noted in the comment, doing this manually may become
unnecessary one day.

Unlike the others so far (though [spoilers!] _like_ Rust), I don't try to get the top-level
doc (installation instructions, example, etc) into this system. Instead, I write those
in a standard markdown file in my Sphinx source, and include the generated API docs, e.g.:

````markdown
... additional documentation above this point ...

## API Reference

```{include} ./_r/StanModel.md

```

### Compilation utilities

```{include} ./_r/compile_model.md

```

```{include} ./_r/set_bridgestan_path.md

```
````

### Bonus tip: R6 classes

If you happen to have an API that uses an [R6 class](https://r6.r-lib.org/index.html),
you will find that the generated markdown includes a table of methods
at the top with broken links. I've found the easiest thing to do is
just delete this in `conf.py` after calling the above R script, e.g.:

```python
    # delete some broken links in the generated R docs
    StanModel = pathlib.Path(__file__).parent / "languages" / "_r" / "StanModel.md"
    text = StanModel.read_text()
    start = text.find("### Public methods")
    end = text.find("### Method `")
    text = text[:start] + text[end:]
    StanModel.write_text(text)
```

## Rust

Rust, like Julia, uses Markdown extensively in its own documentation system. However,
Rust was one of the few items I more or less gave up on, and my documentation
page was just a link to the docs.rs site which is automatically generated by
uploading a crate to crates.io.

However, while writing this article, I found
[`sphinxcontrib-rust`](https://gitlab.com/munir0b0t/sphinxcontrib-rust).
This ends up being pretty similar to the doxygen and R cases. It runs a Rust program
and then produces a Markdown file which Sphinx can read. You can then include this file
as a section in a larger, hand-written document on the language if you want an
`autodoc`-like experience, or use the new Rust directives it adds to craft a more
hand-made page.

Finding this package actually delayed the writing of this article, because I took
some time to try it out in BridgeStan. I found a few issues, but the maintainer was
incredibly responsive and the issues have all been resolved or sufficiently worked around.

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
        "bridgestan": str(BASE_DIR / "rust"),
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

Tip: You can use `:start-line: 2` in the `{include}` directive to skip the
`# Crate CRATE_NAME` line that `sphinxcontrib-rust` generates.

## General tip: Github Pages

Assuming you want to host the output of this process as a [GitHub Pages](https://pages.github.com/) site,
here are some bonus tips:

- `sphinx.ext.githubpages` is a built-in extension that just generates the `.nojekyll`
  file that GitHub Pages needs to serve a folder as raw html.

- I highly recommend force-pushing to the gh-pages branch in your CI job that re-builds
  the documentation. This prevents your `.git` folder from growing in size due to storing
  old commits of the built documentation. As a general rule, you never need this output, since
  it should be reproducible from the commit that triggered the rebuild (your build is deterministic, _right_?).

- If you took my earlier advice and are using pydata-sphinx-theme, it's relatively
  simple to use their [version-picker](https://pydata-sphinx-theme.readthedocs.io/en/stable/user_guide/version-dropdown.html)
  by serving the site from a subdirectory. On merges to `main`/`trunk`, you can delete and re-create
  a folder called something like `latest`, or `development`, and during releases the release action
  can build a copy in the directory named after the new version and update the `versions.json` file.
  See BridgeStan's CI for this in action:
  - Release: [updating version metadata](https://github.com/roualdes/bridgestan/blob/main/docs/add_version.py)
  - Release: [running docs action](https://github.com/roualdes/bridgestan/blob/main/.github/workflows/release.yaml#L137-L143)
  - [Docs CI](https://github.com/roualdes/bridgestan/blob/main/.github/workflows/docs.yaml)

## Go out and cook

Like any recipe, you will learn much more the first time you try it yourself
than you ever could from reading it. While each language ends up being finicky
in its own way, the general structure of adding a new one is pretty
streamlined in this style, and the end result is quite satisfying.

If you find a problem with the above, or want to suggest a better way, please
[open an issue](https://github.com/WardBrian/wardbrian.github.io/issues/new).
I can't guarantee _support_, per se, but if I can I will try to keep this post as
a living document over time.
