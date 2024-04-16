# jakewilliami.github.io (v2)

## Blog

For some time, I struggled with Jekyll.  This was due to friction with the build system and dependencies.  Unfortunately, it was certainly the best on the market for a long time.  During framework research for [my other website](https://www.wikiwand.com/en/Gaston_Leroux), I considered using a language I am familiar with; e.g., using a SSG such as [Franklin.jl](https://github.com/tlienart/Franklin.jl/) or [StaticWebPages.jl](https://github.com/Humans-of-Julia/StaticWebPages.jl).  However, I ended up finding [Zola](https://github.com/getzola/zola) (at time of writing&mdash;April, 2024&mdash;I am using [v0.18.0](https://github.com/getzola/zola/tree/v0.18.0))..Zola appears to be simple with few dependencies, developed in Rust (a language that I know and hold in high regard), and growing in popularity.

The theme is an important talking point.  The blog [used to](https://github.com/jakewilliami/jakewilliami.github.io/tree/7b72aa9b21af5eb41af8219bafc1a1dc878040aa/archive/v1) use Pavel Tsurbeleu's theme, [Chalk](https://github.com/ptsurbeleu/jekyll-theme-chalk) Gem-based [Jekyll](https://github.com/jekyll/jekyll) (ported from its [original Jekyll theme by Nielsen Ramon](https://github.com/nielsenramon/chalk)).  (Well, it [appears](https://github.com/jakewilliami/jakewilliami.github.io/commit/6284fa74a5d8d2c42cbd3ea5e29b03d66fedea9b#diff-ecd98a7d9df5d7ee49f62a1bbc08186e0b6a4198ed186932f441d94faca751f4) that I used to use [the Time Machine theme by Jon Rohan](https://web.archive.org/web/20201204211522/https://jakewilliami.github.io/), albeit fleetingly.)  Unfortunately, these have since grown obsolete with a lack of maintenance.  Between this and friction with the Ruby pipeline/build system using bloated dependencies (Gems; NPM), I decided to switch away from it all.  The present theme is based on [Chester How's Jekyl theme, Tale](https://github.com/chesterhow/tale) ([`361d8e33`](https://github.com/chesterhow/tale/tree/361d8e337536e4bdd8b110edac0836a56d6f2541)).  The theme was them [ported to Zola by Aaran Xu](https://github.com/aaranxu/tale-zola) ([`5108a4ae`](https://github.com/aaranxu/tale-zola/tree/5108a4ae31352ecd3aa3d7ab8fc85038975f46a8)), then [picked up by Miguel Pimentel and renamed as Mabuya](https://github.com/semanticdata/mabuya) ([`9ff7ef60`](https://github.com/semanticdata/mabuya/tree/9ff7ef60c4f4a9632abe01a3b39672d027c24de7)), and finally adapted by _moi_.


## Updating the repository

You will primarily want to make changes in the [`content/`](./content/) directory.  This is where all of your public facing material should be.  You may occasionally need to make configuration changes in [`templates/`](./templates/) or [`static`](./static/).

To test changes locally, simply run
```bash
zola serve
```

**TODO: update deplpoyment instructions once that has been set up**


