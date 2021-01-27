---
layout: post
title: Setting up CI on GitHub Actions, and deploying docs for Julia
---

I didn't know this when I first started learning Julia, but if you write a function; e.g.

```julia
function my_sum(a::T...) where {T <: Number}
    return reduce(+, a)
end
```

If you add a string immediately before the function definition:

```julia
"This is my bad version of a sum function because it takes any number of values, but they all must be the same type of number."
function my_sum(a::T...) where {T <: Number}
    return reduce(+, a)
end

"""
This is my...

...multi-line comment.

Pretty cool, huh?  Even has *markdown* `support`.
"""
my_identity_function(a) = a
```

When you go into help mode (`?` in the Julia REPL), your string will display as documentation for that function:

```
help?> my_sum
search: my_sum

  This is my bad version of a sum function because it takes any number of values, but they all must be the same type of number.

help?> my_identity_function
search: my_identity_function

  This is my...

  ...multi-line comment.

  Pretty cool, huh? Even has markdown support.
```

Adding these, which are called "docstrings", are very useful for anyone using your package.  Furthermore, they are useful for you to come back to six months later, to remind yourself how exactly you use your functions and `struct`s, etc.

There is a very helpful tool called [`Documenter.jl`](https://github.com/JuliaDocs/Documenter.jl) which takes advantage of those docstrings, and builds a documentation static website.  All you need to do is make five or so folders (directories), about three files, and of course have `Documenter.jl` installed.

## 1. Installing `Documenter.jl`

To install `Documenter.jl`, you need to enter Julia's REPL and type

```julia
julia> import Pkg; Pkg.add("Documenter")
```

## 2. Creating the necessary _folders_ for `Documenter.jl`

From the command line, enter the directory in which your package lies.  For example, I might be in the folder `~/projects/MyCoolPackage.jl/`, which will probably look like this:

```
.
├── LICENSE
├── Manifest.toml
├── Project.toml
├── README.md
├── src
│   └── MyCoolPackage.jl
└── test
    └── runtests.jl
```

Now run the following from the command line:

```bash
$ mkdir -p docs/build/; mkdir -p docs/src/; mkdir -p .github/workflows/
```

Now your directory structure should look something like this:

```
.
├── .github
│   └── workflows
├── .gitignore
├── LICENSE
├── Manifest.toml
├── Project.toml
├── README.md
├── docs
│   ├── build
│   └── src
├── src
│   └──  MyCoolPackage.jl
└── test
    └── runtests.jl
```

## 3. Creating the necessary _files_ for `Documenter.jl`

From the command line, run the following:

```bash
$ touch docs/src/index.md; touch docs/make.jl; touch .github/workflows/CI.yml
```

Now (this is an _important step_) you need to run

```julia
$ cd docs

$ julia --project=

julia> import Pkg; Pkg.activate(".")

julia> Pkg.add("Documenter")

julia> Pkg.add("") # HERE YOU NEED TO INSTALL ANY DEPENDENCIES YOUR PACKAGE HAS
```

Now that you have **installed any dependencies your package has in the `docs` folder**, your directory structure will look something like this:

```
.
├── .github
│   └── workflows
│       ├── CI.yml
│       ├── CompatHelper.yml
│       └── TagBot.yml
├── .gitignore
├── LICENSE
├── Manifest.toml
├── Project.toml
├── README.md
├── docs
│   ├── Manifest.toml
│   ├── Project.toml
│   ├── build
│   ├── make.jl
│   └── src
│       └── index.md
├── examples
│   ├── Manifest.toml
│   ├── Project.toml
│   ├── basic.jl
│   └── tm.jl
├── src
│   └── MyCoolPackage.jl
└── test
    └── runtests.jl
```

You may notice some other files in `.github/workflows/`.  Scroll to the end to find out what they contain and how they work.

**This is good.**  We are making good progress.

## 4. Writing to the files

Now that all necessary files are made, it is time to fill them up.  I will paste here the bare-bones of working code:

#### `docs/make.jl`

This file is the file that is run when the documentation is being made.  This is arguably the most important file, and should contain at least one function call on _how_ to make the documentation.  Usually you will also want a function call to tell `Documenter.jl` how to _deploy_ the docs (which is where `.github/workflows/CI.yml` come into play, but I will get to that soon).  This is an example of the `make.jl` file:

```julia
include(joinpath(dirname(@__DIR__), "src", "MyCoolPackage.jl"))
using Documenter, .MyCoolPackage

Documenter.makedocs(
    clean = true,
    doctest = true,
    modules = Module[MyCoolPackage],
    repo = "",
    highlightsig = true,
    sitename = "MyCoolPackage Documentation",
    expandfirst = [],
    pages = [
        "Index" => "index.md",
    ]
)

deploydocs(;
    repo  =  "github.com/username/MyCoolPackage.jl.git",
)
```

#### `docs/src/index.md`

This file will tell `Documenter.jl` how to structure your documentation.  Once again, here is an extremely simple example:

~~~markdown
# MyCoolPackage.jl Documentation

```@contents
```

```@meta
CurrentModule = MyCoolPackage
DocTestSetup = quote
    using MyCoolPackage
end
```

## Adding MyCoolPackage.jl
```@repl
using Pkg
Pkg.add("MyCoolPackage")
```

## Documentation
```@autodocs
Modules = [MyCoolPackage]
```

## Index

```@index
```
~~~

#### `.github/workflows/CI.yml`

Finally, this is the CI (continuous integration) part, which _actually does the deploying_.  All you need to have is something like the following:

~~~yaml
name: CI
# Run on master, tags, or any pull request
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC (8 PM CST)
  push:
    branches: [master]
    tags: ["*"]
  pull_request:
jobs:
  test:
    name: Julia ${{ matrix.version }} - ${{ matrix.os }} - ${{ matrix.arch }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        version:
          - "1.0"   # old LTS
          - "1.5"   # current
          - "nightly"   # Latest Release
        os:
          - ubuntu-latest
          - macOS-latest
          - windows-latest
        arch:
          - x64
          - x86
        exclude:
          # Test 32-bit only on Linux
          - os: macOS-latest
            arch: x86
          - os: windows-latest
            arch: x86
    steps:
      - uses: actions/checkout@v2
      - uses: julia-actions/setup-julia@v1
        with:
          version: ${{ matrix.version }}
          arch: ${{ matrix.arch }}
      - uses: actions/cache@v1
        env:
          cache-name: cache-artifacts
        with:
          path: ~/.julia/artifacts
          key: ${{ runner.os }}-test-${{ env.cache-name }}-${{ hashFiles('**/Project.toml') }}
          restore-keys: |
            ${{ runner.os }}-test-${{ env.cache-name }}-
            ${{ runner.os }}-test-
            ${{ runner.os }}-
      - uses: julia-actions/julia-buildpkg@latest
      - run: |
          git config --global user.name Tester
          git config --global user.email te@st.er
      - uses: julia-actions/julia-runtest@latest

  docs:
    name: Documentation
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: julia-actions/setup-julia@v1
        with:
          version: '1'
      - run: |
          git config --global user.name name
          git config --global user.email email
          git config --global github.user username
      - run: |
          julia --project=docs -e '
            using Pkg;
            Pkg.develop(PackageSpec(path=pwd()));
            Pkg.instantiate();'
      - run: julia --project=docs docs/make.jl
        env:
          GITHUB_TOKEN: $\{\{ secrets.GITHUB_TOKEN \}\}

~~~

The first part of this file will run some tests, and the second part will deploy the documentation.

**Push all of this to the repo.**

## 6. Allowing branch `gh-pages` to be accessed by `username.github.io` to deploy docs — the _final_ step

The last thing we need to do is to go to your repository at [https://github.com/username/MyCoolPackage.jl/](https://github.com/username/MyCoolPackage.jl/), go into `Settings > Options > GitHub Pages`, and select the `gh-pages` branch, then press save.  This will allow GitHub Pages to access the `gh-pages` branch created by `Documenter.jl`.

**And you are done!**

## Post Script: other notes.

You might want to add some tags to your `README.md`:

~~~markdown
<h1 align="center">
    MyCoolPackage.jl
</h1>

<!-- [![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://username.github.io/MyCoolPackage.jl/stable) -->
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://username.github.io/MyCoolPackage.jl/dev)
[![CI](https://github.com/invenia/PkgTemplates.jl/workflows/CI/badge.svg)](https://github.com/username/MyCoolPackage.jl/actions?query=workflow%3ACI)
[![Code Style: Blue](https://img.shields.io/badge/code%20style-blue-4495d1.svg)](https://github.com/invenia/BlueStyle)
![Project Status](https://img.shields.io/badge/status-maturing-green)
~~~

There are two other workflows I really like to have: one is a `CompatHelper`, which ensures your package's dependencies stay up-to-date; and one is `TagBot`, which will automatically update the version number based on the version of your package.

Here are the files:

#### `CompatHelper.yml`

```yaml
name: CompatHelper
on:
  schedule:
    - cron: 0 0 * * *
  workflow_dispatch:
jobs:
  CompatHelper:
    runs-on: ubuntu-latest
    steps:
      - name: Pkg.add("CompatHelper")
        run: julia -e 'using Pkg; Pkg.add("CompatHelper")'
      - name: CompatHelper.main()
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          COMPATHELPER_PRIV: ${{ secrets.DOCUMENTER_KEY }}
        run: julia -e 'using CompatHelper; CompatHelper.main()'
```

#### `TagBot.yml`

```yaml
name: TagBot
on:
  schedule:
    - cron: 0 0 * * *
  workflow_dispatch:
jobs:
  TagBot:
    runs-on: ubuntu-latest
    steps:
      - uses: JuliaRegistries/TagBot@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          ssh: ${{ secrets.DOCUMENTER_KEY }}
```
