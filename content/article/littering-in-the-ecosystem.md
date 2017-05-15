+++
Categories = ["Code"]
Description = "Are we littering the JavaScript ecosystem, or are we setting up a smorgasboard of choices?"
Tags = ["JavaScript"]
date = "2017-05-15T22:53:30+02:00"
title = "Littering in the ecosystem"
draft = false
+++
The other day I was in a discussion on the general situation with JavaScript development. I argued that the JavaScript community many time feels like a youth centre, everyone trying to show off themselves and not everyone taking responsibility on what they do. I even expressed that publishing many small NPM packages should be seen as littering the JavaScript ecosystem. Heck, one [developer](https://www.npmjs.com/~mafintosh) published over 500 packages and this behaviour seems to be encouraged by the community.

Today, I revisited these thoughts and asked myself _why_ I feel this way. What is the problem with having many packages to choose from? Sure, the dependencies your application have each introduce a risk - but the choice of additional dependencies cannot be seen as a risk but a smorgasbord, right? Does my feeling have any reason, or is it just that I am unused to these small modules comparing to Java or Python ecosystems?

Some argue that people seems to have forgotten how to program, that we need help left padding a string. I don't worry too much about that, using tested and proven code make sense and allows you to focus on adding value to your product or to your client.

After some thinking I reduced my concern down to two issues:

* **Discoverability**, having many modules sharing the same namespace makes it harder to search for a package that fit you needs. Also, using nonsense or silly package names trying to be unique does not help.
* **Maintainability**, can one developer with 500 packages really maintain all of those? Just managing PR can be a tedious job. Would it not be better with fewer packages that each have several maintainers?

Think about the value your package brings, does it have to be a package? Does it really motivate a new package or can would it be better off as a module in an existing package? Can it just be a gist or a git repository allowing for people to discover and vendor if needed?

If you publish a package you have the responsibility throughout its life-cycle, allow your potential consumers to distinguish between an abandoned package and a fully functional package that is just feature complete.

Don't get me wrong, I think that small modules are a good thing, but some of the existing ones are [tiny](https://github.com/gummesson/is-empty-object), these should not be individual packages and there are far too many abandoned packages still around, littering the ecosystem.