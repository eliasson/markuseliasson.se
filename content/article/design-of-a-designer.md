---
title: "Design of a Designer"
date: 2024-08-16T23:17:21+02:00
description: "The old man and the sea"
draft: true
---

I recently had the great opportunity to be part of a team that developed a piece of software that enabled users to
design constructions of a certain type, while following product rules and recommendations from the manufacturer.

The client operates in a construction type of domain where they produce a whole range of products that need to be
assembled in order to provide a solution for their customer’s needs. They have websites, catalogs and clear assembly
instructions detailing which pieces that are compatible and how to fit things together.

Our task was to build a web application where their customers could design their solutions using said company’s products.

> We need you to build a new drawing program for us.

Why on earth would anyone build their own design tool? Is it not enough to rely on existing CAD software?

## Computer Aided Design

TODO Explain that CAD software, while great, is not a sales tool.

## Objectives

Given the conclusion that the design software is as much a sales tool as a technical tool we set out with the following
objectives in our project:

**Users should not have to be product experts** - Users are experts in their own domain, they use products to help them
solve a particular problem of theirs. They do not know all the details about them and they most likely do not have the
time or interest to learn it. And I believe it is safe to assume that most users are not frequent users of the system.

**If it can be designed it can be built** - the tool should guide the users to design solutions where parts are correctly
combined using best practices. Invalid constructions should not be possible to design.

**It must scale** - the tool should scale from beginners to power users, independently of the sales organization.
Onboarding of new users should not require dedicated training.

## Domain

As you might have noticed I have stayed away from naming our client and their specific domain. It is not relevant for
this article and instead the examples used to illustrate the design will be generic. Let’s pretend we’re about to
develop a software to design a system of old school overhead power lines.

- There are multiple types of posts; wood and steel.
- There are different heights for posts, 9 and 12 meters.
- There are two types of wiring between the posts; iron and steel.

## Design principles

The team building this software relied on the following principles during the development of the program.

- All code must be tested, everyone on the team has been using Test Driven Development for many years.
- Product data should be imported to the application.
- Product rules should not be hard-coded but modeled as part of the product data or expressed next to it.
- Domain logic should be kept out from any rendering.

While not clearly defined at the start of the project these principles were already agreed upon by the team from earlier
projects and a long history of using Domain Driven Design, Extreme Programming and Test-Driven Development.

For readers unfamiliar with these concepts, the reasons are:

- Domain experts should define the data and properties of products as they know it best.
- Product lifecycle and additions should not require software development.
- Rendering UI, drawings, etc should be separated from domain logic to allow for independent changes and life cycles.
- TDD gives us a sound design as well as confidence to change and improve the software for years to come.
