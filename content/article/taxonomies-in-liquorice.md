+++
Categories = ["Hugo"]
Description = "Taxonomies in liquorice"
Tags = ["liquorice"]
draft = false
date = "2014-11-11T22:21:10+01:00"
title = "Taxonomies in liquorice"

+++

My Hugo theme [liquorice](https://github.com/eliasson/liquorice) was updated
yesterday to add simple support for [Taxonomies](http://gohugo.io/taxonomies/overview/).
Both *categories* and *tags* is supported both as list view and as the ingress
part of a single article. 


In order to use it you need to configured in your site configuration (see
[documentation](http://gohugo.io/taxonomies/usage/)), in TOML
this is basically adding:

    [indexes]
        category = "categories"
        tag = "tags"


You then add the categories and / or tags per article in the front-matter like:

    +++
    Categories = ["Hugo"]
    Description = "Taxonomies in liquorice"
    Tags = ["liquorice"]
    ...
    +++


Liquorice supports either both categories and tags, only one or neither of them.
