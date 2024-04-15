# jakewilliami.github.io (v2)

## Blog

This blog considered using [Franklin.jl](https://github.com/tlienart/Franklin.jl/)

This blog uses [Zola](https://github.com/getzola/zola) ([v0.18.0](https://github.com/getzola/zola/tree/v0.18.0))

Used to use the [Chalk](https://github.com/ptsurbeleu/jekyll-theme-chalk) Gem-based [Jekyll](https://github.com/jekyll/jekyll) theme (ported from its [original Jekyll theme by Nielsen Ramon](https://github.com/nielsenramon/chalk)).  unfortunately it stopped being updated in 2017 and 2019 respectively, and was having too much greif with ruby verisons on macos and bloated dependencies (gems, npm, etc.)

Deciding between the following themes:
  - [Emily](https://github.com/kyoheiu/emily_zola_theme) (at [`1c1d560c`](https://github.com/kyoheiu/emily_zola_theme/tree/1c1d560c9ea209a988b78ab2a3514bf5c6846f29))
  - [Kita](https://github.com/st1020/kita) (at [`04a31a78`](https://github.com/st1020/kita/tree/04a31a78f8b2a697c51b93e31aeead79d39d9936))
  - [Mabuya](https://github.com/semanticdata/mabuya) (at [`9ff7ef60`](https://github.com/semanticdata/mabuya/tree/9ff7ef60c4f4a9632abe01a3b39672d027c24de7))
  - [Papermod](https://github.com/cydave/zola-theme-papermod) (at [`0aea7bb0`](https://github.com/cydave/zola-theme-papermod/tree/0aea7bb064c508e0e67417a405b0304c40b588e6))
  - [Sam](https://github.com/janbaudisch/zola-sam) (at [`890b51b4`](https://github.com/janbaudisch/zola-sam/tree/890b51b4105fd2e63f5e417e5cb63b8e25d5721f))

## Updating the repository

You will primarily want to make changes in the [`content/`](./content/) directory.  This is where all of your public facing material should be.  You may occasionally need to make configuration changes in [`templates/`](./templates/) or [`static`](./static/).

To test changes locally, simply run
```bash
zola serve
```

**TODO: update deplpoyment instructions once that has been set up**

## Version 1

See previous version at [`v1/`](./archive/v1/).
