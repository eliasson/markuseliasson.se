+++
date = "2017-02-02T23:40:47+01:00"
title = "Managing your JavaScript dependencies"
description = "JavaScript has a fast moving ecosystem and if you do not manage your dependencies you risk breaking your application."
tags = ["JavaScript"]
draft = false
+++
Recently a colleague of mine and I had a discussion on npm dependencies, in where he asked _“Why is it bad to have many dependencies, really, why do you care?”._ I think it is a fair question, npm gets bashed all the time for resulting in a ridiculous number of dependencies, is that really a problem?

I don’t think the number of dependencies is the major problem though. Sure too many dependencies can lead to [performance issues](https://nolanlawson.com/2016/08/15/the-cost-of-small-modules/) and Windows did have its [problem](https://github.com/npm/npm/issues/3697) coping with npm's file structure. The major problem, in my experience, is that developers still - almost a year after the [left pad incident](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/) - do a poor job managing their dependencies.

The way I see it is that each dependency is a risk. It can introduce a bug, a breaking change or just disappear. If you have many dependencies (including transient dependencies) the risk that one of them will break increase the more you have. Do you trust all of these maintainers? Do you trust npm, Inc. to keep the registry online, and free, 24/7?

Depending on your project it might be OK not being able to reproduce a previous build. But not all organisations move fast and break things, some still work using waterfall process. There it is absolutely crucial to be able to reproduce build both during QA and after release for potential emergency corrections. Not only do you have to be able to reproduce, you might need to step only a single dependency up or down - how can you guarantee that no other dependency was changed?

Most of these advice is based on first-hand experience, helping a Swedish governmental organization with their frontend development.


## Use NPM

Last few years more and more developers have gone the route of just using npm for dependency management. If you are still using Bower for managing your frontend dependencies you should really let it go, it does not add anything npm cannot do and it still [lack important features](https://github.com/bower/bower/issues/505) such as locking versions.

Also, with the increase support for module exports and ES2015, I would argue that it is easier to use npm + [Browserify](http://browserify.org), [rollup.js](http://rollupjs.org) or [webpack](https://webpack.github.io/) than to mix package managers.


## Use semantic versioning

Npm comes with semantic versioning for packages and you should use that. Besides from version, npm do support [other values](https://docs.npmjs.com/files/package.json#dependencies):

* HTTP URL:s
* Git URL:s
* File system paths
* Tags
* Etc.

Do **not** rely on anything but semantic versions. Using any other type of dependencies will make it close to impossible to reproduce a build unless you have full control over the remote media. Also, running `npm install` will not update git repositories, and `npm update` might [behave different](https://github.com/npm/npm/issues/1727) depending on host.

_Sure, it might be OK using tags during development phase, as long as you know what you are doing._


## Lock your dependency tree

The way npm is built, it will honour [SemVer](https://docs.npmjs.com/misc/semver), but as long as your dependencies resolve accordingly (i.e. respects `*`, `~`, `^` and any range definition) it will not be updated unless you explicitly run `npm update`. And running a fresh `npm install` might very well give two users different dependency versions based on npm cache and such.

So, you, your colleagues and your CI server might not be running the same version of your dependencies. Again, this might or might not be ok in your situation, but it will definitely make it harder to reproduce builds.

In order to solve this, npm has a built in feature called [shinkwrap](https://docs.npmjs.com/cli/shrinkwrap). Using this feature the npm client will generate a parallel file named `npm-shrinkwrap.json` that locks down the versions for all installed packages in `package.json`, and as long as that file exist any `npm install` will use the package versions stated therein instead of resolving according to SemVer.


## Roll your own registry

Most rely on http://www.npmjs.org when downloading packages, it's free, it has **tons** of packages and they do a great job keeping it online. Still, it is a commercial company running that service, [npm, Inc.](https://www.npmjs.com/about) Do you trust them to always keep your dependencies around? What if the service suddenly goes away, transfers to a paid service, or it gets bought and shut down?

During the left-pad incident, npm, Inc. decided to transfer a package name in use to another owner than the current owner due to brand infringement claims. What guarantee is there that something like this does not happen again?

What you should do is host your own repository, that proxies npmjs.org. That way you can keep a copy of all packages you rely on, including the version history (based on your installations). So even if you do not have the need to publish internal packages, keeping a proxy registry can be extremely valuable in the future, if a package is taken off the Internet.

There are several free, open source, solutions available - I have successfully used the npm support in [Sonatype Nexus](https://www.sonatype.com/nexus-repository-oss) in several projects.


## Yarn

In 2016 Facebook introduced [Yarn](https://yarnpkg.com/), a drop-in replacement for the npm client. It still use the same `node_modules` directory and the same package registry as npm and can be used side-by-side with npm.

The most important difference Yarn has over npm is that it is deterministic. It automatically locks down all versions of your dependency tree, so everyone installing the same project will guaranteed get the exact same versions of dependencies.

So basically, Yarn is like npm + shrinkwrap, only better.

Also, besides from being deterministic and npm-compatible it is _way faster_ than npm when installing packages - you should definitely [try it out](https://yarnpkg.com/docs/install).


## Still not safe

Even if you do all these things you are still not safe. The way npm packages are constructed it might not contain the actual code you want, rather a package can hook up to the [npm-lifecycle](https://docs.npmjs.com/misc/scripts) and execute scripts as part of the installation. These scripts can do *whatever* the executing user have access to, most commonly it download files from off the Internet and install on your system.

E.g. [`phantomjs`](https://github.com/Medium/phantomjs/blob/master/package.json) use an install hook to download the correct version of phantom based on the current platform.

While this might seem like a good thing, it makes it **very** hard to keep a backup of the version you rely on. An internal npm registry will only keep the package content, any `pre/post/install` action is entirely up to the package maintainer. While some offer configuration to specify alternative URL:s for where additional files are downloaded - all such config is package specific. Another example is [node-sass](https://github.com/sass/node-sass) which downloads source code in order to build node bindings for libsass.

You need to be aware of which packages pulls stunts like this if you need to have a fully reproducible environment.

_A good way to test your builds is to block traffic to Internet during a clean build (with clean cache), that way it is easier to detect any outside source being requested._


## To summarize

The size and available options in the JavaScript ecosystem is one of its great strengths, you should use dependencies - but at least be aware of the risk they introduce. Maybe you don't need, or cannot afford, to have 100% reproducible builds - just make sure you call that decision before the _SHTF_.

* Only use npm or yarn for package management
* Use semantic versions, not tags, not git repositories, etc.
* Use shrinkwrap or yarn for version locking
* Use your own registry as proxy
* Be aware of any package that relies on external sources
