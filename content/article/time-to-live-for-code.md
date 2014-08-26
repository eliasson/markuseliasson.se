+++
Categories = []
Description = ""
Tags = []
draft = false
title = "Time to live for code"
date = 2014-08-26T09:10:38Z
+++

> Perfection is achieved, not when there is nothing more to add, but when
>  there is nothing left to take away.
>
> -- *Antoine de Saint-Exupery*

Over the years I have become more and more obsessed about removing code. Some
of it surely relates to my OCD tendencies, although I would like to think
that the majority comes from being a good developer.

Deleting code is nothing new, dead code should simply not exist in any code base
and once a piece of code has served its purpose it should be deleted. Still, in
almost all of my assignments there are plenty of code that could (and should
have) be deleted - since it no longer used.

For me, the biggest problem with dead code is that it is still part of the
*cognitive load* for the programmer. It's there, I cannot ignore it, I see it
when I browse source code, it is (hopefully) executed as part of my test cases,
it is showing up when I search for code usage.

I guess there are mainly two reasons for why dead code is still in the code
base, **fear** and **sloppiness** (stress, laziness and time constraints falls
under sloppiness).


## Code structure

The way we structure our code is mainly based on function or technique - and
sometimes both.

Lets say you're implementing a *sign up* feature where your customers can be be
notified on the next iPhone launch. You might put that together with your
*product code*, perhaps the *user profile*, or next to some *email* code lying
around. Do you make a separate CSS file for it, is the markup well separated?

Have you considered the *Time To Live* for this feature? It might be a long
lived piece, or it might just be used a few weeks for the iPhone?

Did you consider making this tiny piece of functionality a separate module?
Sure the overhead can be quite big based on languages, build systems, etc.
(looking at you Maven).

I would assume that this feature has a quite short TTL (compared to the other
parts of the web shop), and I would definitely put it as a separate module -
*all of it*.


*Why?* - If I can remove this feature easy (without surgeon knife) - I argue
that it beats (any) overhead cost in making it a separate module.

Compare the "all over the place"-pattern in this django-ish setup:

    acme/products/templates/iphone-signup.html (new)
    acme/products/static/css/products.css (modified)
    acme/products/forms.py (modified)
    acme/products/views.py (modified)
    acme/products/tests.py (modified)
    acme/products/urls.py (modified)


With the isolated one:

    acme/iphonesignup/templates/iphone-signup.html (new)
    acme/iphonesignup/static/css/iphone-signup.css (new)
    acme/iphonesignup/forms.py (new)
    acme/iphonesignup/views.py (new)
    acme/iphonesignup/tests.py (new)
    acme/iphonesignup/urls.py (new)
    acme/settings.py (edited)
    acme/urls.py (edited)

When the signup period is over, which one do you think is easiest to remove?
And what if there have been substantial code churn in the product app, would not
that be just great?


## Planning

When you plan your work, make it a practice to ask yourself - when can I delete
this code?

Most of the times the answer will be, not anytime soon, then just carry on
as normal. But if the answer is days / weeks / months - consider putting the
feature in a separate module. Even if it will cost you a few hours more, you
will benefit once you easily can get rid of the feature.


## Follow up

I have been working in teams where both product owners and developers come and
goes. After a while features are forgotten, (heck sometimes they are even
implemented twice!), you need to stay alert and challenge product owners on when
you can delete your features.

Also, consider using some sort of metrics collected from production, if a feature
is not used, do what you can to delete any trace of it.


## Rust

As developers we are far to eager to write code, when we really should be
finding ways to delete code.

Code does not rust or get worse over time (as compared to many other types of
engineering). Code gets rusty because the developer makes it rusty, an act of
keyboard typing.

Leaving code in a code base that is not used anymore is an excellent way of
destroying your code.


Grouping source code on TTL when TTL is short is one way to make it easier to
get rid of dead code - it will help with both **fear** and **sloppiness** as it
is easier to empty the garbage if it is nicely put in a bin compared to picking
up the litter all over the place.


Markus
