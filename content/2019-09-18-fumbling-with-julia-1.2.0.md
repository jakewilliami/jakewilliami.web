+++
title = "Fumbling with Julia 1.2.0"
date = 2019-09-18
+++

I was first told about Julia very recently by a friend from mathematics class.  He told me that it is very good for maths, but not very well known.

*Disclaimer: this little post is transcribed verbatim from my notebook.  Current Jake cannot necessarily verify the correctness of some of this information.*

Well I have played around a little with it.  I found a few important directories for it.

Running Julia using terminal:
```bash
open /urs/local/bin/julia
```

Scripts you can make and use are stored in
```bash
/Applications/Julia-1.2.app/Contents/Resources/Scripts
```

The `startup.jl` file can be found at
```bash
/Applications/Julia-1.2.app/Contents/Resources/julia/etc/julia
```

In Julia, to get `<package>`, type
```julia
Pkg.add("<package>")
Pkg.build("<package>")
using <package>
```

Packages will be stored at
```bash
~/.julia/packages/
```
