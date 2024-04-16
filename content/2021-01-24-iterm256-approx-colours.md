+++
title = "Approximating GitHub Language Colours in iTerm-256 Colour Terminal"
date = 2021-01-24
+++

When I first made [`ls.py`](https://github.com/jakewilliami/scripts/blob/master/python/ls.py), as a good exercise to teach me programming, I went through GitHub and manually tried to match colours.

Today, I ported a [script](https://github.com/jakewilliami/scripts/blob/master/python/rgb2iterm256.py) to convert RGB tuples to iTerm-256 approximations [into Julia](https://github.com/jakewilliami/scripts/blob/master/python/rgb2iterm256.jl).

With a quick Julia command call, you can actually take the information you need from this:
```bash
LANG_WANTED="input lang here"; curl https://raw.githubusercontent.com/github/linguist/master/lib/linguist/languages.yml > languages.yml; julia -E 'import Pkg; Pkg.add.(["YAML", "OrderedCollections", "Colors"]); using YAML; include("$(homedir())/projects/scripts/python/rgb2iterm256.jl"); f = YAML.load_file("languages.yml"); for k in keys(f); col = get(f[k], "color", ""); if !isempty(col); print(k, ":\t\t"); main(col); end; end'; rm languages.yml | grep -i "$LANG_WANTED"
```

This simply parses a language script, and prints their iTerm-256 colour code.  This really helps when you want to do something like `ls.py` or `ls.c`, where it prints certain extensions in different colours.  Of course, this is all just for aesthetics, and for a bit of fun.  But it's always fun porting things to Julia!  Go well.
