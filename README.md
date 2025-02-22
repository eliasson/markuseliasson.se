# Site of Markus Eliasson

This is the personal site and blog for http://www.markuseliasson.se, powered by
the static site generator [Hugo](http://hugo.spf13.com).

## Getting started

Probably not of great interest to anyone but me, but due to my frequency of
blogging I really need these commands written down.

    # Create a new article
    hugo new article/my-new-article.md

    # Run a local server
    hugo server --theme=eliasson --buildDrafts --watch

    # Generate only published pages with this command
    hugo -t eliasson -d ../eliasson.github.io
    cd ../eliasson.github.io
    git add .
    git commit -m "Build site"
    git push

Copyright (c) 2014-2025 - Markus Eliasson - All rights reserved.
