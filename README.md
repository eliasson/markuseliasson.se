# Site of Markus Eliasson

This is the personal site and blog for http://www.markuseliasson.se, powered by
the static site generator [Hugo](http://hugo.spf13.com).


## Theme

Currently there is an embedded theme called _tidy_ used. If the theme plays out well
and I finish some minor extensions I will extract it to a separate theme.

Remaining features:

[ ] Improve grid for mobile layout
[ ] Add support Disqus comments

## Getting started

Probably not of great interest to anyone but me, but due to my frequency of
blogging I really need these commands written down.

    # Create a new article
    $ hugo new article/my-new-article.md

    # Run a local server
    $ hugo server --theme=x10 --buildDrafts --watch

    # Generate only published pages with this command
    hugo -t x10
    cd public
    git add .
    git commit -m "Build site"
    git push


Copyright (c) 2014-2019 - Markus Eliasson - All rights reserved.
