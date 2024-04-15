# jakewilliami.github.io

**NOTE:** I think this blog is broken.  I seem to have updated `about.html` and I recall working on a new template that never properly pushed.  I should re-assess the template and workflows again at some point.  As this is static, I should consider using one of the good static site generators written in Julia.

## Blog

This blog uses the [chalk](https://github.com/nielsenramon/chalk) theme.  See also: [ptsurbeleu/jekyll-theme-chalk](https://github.com/ptsurbeleu/jekyll-theme-chalk).

## Updating the repository

You can add posts to [`_posts/`](./_posts/), and update the about section in [`about.html`](./about.html).  Once you are happy with your changes, commit them with a descriptive commit message.  Now you can publish your changes to [jakewilliami.github.io](https://jakewilliami.github.io) but running [the deploy script](./bin/deploy) (make sure you run this from the master branch, or any branch that isn't already `gh-pages`).  I believe to deploy you need to have Jekyll installed.
