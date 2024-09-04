---
title: "The Design of a Designer"
date: 2024-09-03T20:00:00+02:00
description:
  Here is a walkthrough of the software design of a construction design tool I have been working on lately. It is a
  long read, focusing on the user interaction of such a tool in a complex domain, together with the benefits and
  challenges of this design. I always enjoy reading about how a piece of software is designed, the rationale behind
  the decisions, etc. This article is my way of giving back.
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

Most CAD software are highly advanced 3D-modeling tools.
{{< sidenote >}}<img src="/designer/picoCAD.jpg" alt="picoCAD" /><a class="no-feedback" href="https://johanpeitz.itch.io/picocad" target="_new">The fabulous picoCAD</a>{{< /sidenote >}}
They provide incredible levels of details, simulations, stress analysis, integrations to product data and a long list
of other features.

CAD tools are also aimed at technical people that care about most of those details. The tool we were asked to develop
is both a technical tool and a sales tool. The target audience for said tool might very well be in charge of both the
purchase and assembly of the products sold.

One might argue that such a tool is still a Computer Aided Design tool, but it is not really what most people would
consider a _“CAD-tool”_.

## Objectives

Given the conclusion that the design software is as much a sales tool as a technical tool we set out with the following
objectives in our project:

**Users should not have to be product experts** - Users are experts in their own domain, they use products to help them
solve a particular problem of theirs. They do not know all the details about them and they most likely do not have the
time or interest to learn it. And I believe it is safe to assume that most users are not frequent users of the system.

**If it can be designed it can be built** - the tool should guide the users to design solutions where parts are
correctly combined using best practices. Invalid constructions should not be possible to design.

It must scale - the tool should scale from beginners to power users, independently of the sales organization.
Onboarding of new users should not require dedicated training.


## Domain

As you might have noticed I have stayed away from naming our client and their specific domain.
{{< sidenote >}}<img src="/designer/our-domain.png" alt="Our domain" />Our domain, old school power lines.{{< /sidenote >}}
It is not relevant for this article and instead the examples used to illustrate the design will be generic. Let’s pretend we’re about to
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
projects and a long history of using **Domain Driven Design**, **Extreme Programming** and **Test-Driven Development**.

For readers unfamiliar with these concepts, the reasons are:

- Domain experts should define the data and properties of products as they know it best.
- Product lifecycle and additions should not require software development.
- Rendering UI, drawings, etc. should be separated from domain logic to allow for independent changes and life cycles.
- TDD gives us a sound design as well as confidence to change and improve the software for years to come.

## Architecture and software design

The architecture is pretty standard, there is a front-end and a back-end service.

{{< fullwidth title="Architecture overview" source="/designer/overview.png" >}}

The responsibility between these parts is that the back-end is responsible for:

- Authenticate via OpenID Connect
- Store product data.
- Integrate with an external system to handle orders and quotation requests.
- Export the design of a project to
- 2D drawing.
- 3D model.
- Bill of material.

The front-end is where the user designs a solution in a 2D canvas with 3D rendering as a companion view. And where the
user initiates the actions to order and to export the drawing in various formats.

We chose to make the application front-end heavy{{< note 1 >}} in order to get as snappy interactions as possible.
The rest of the article will focus on the front-end parts and pieces.
{{< sidenote >}}1. In responsibility, not in bytes.{{< /sidenote >}}

## User interaction

Remember that the UI should not implement any domain logic? We decided that the UI should only do at most three things:

- Render drawing items (e.g. post and wires).
- Showing the user possible actions that can be taken (e.g. place post, delete post).
- Trigger actions initiated by the user.

However, the UI should not itself decide which actions are available, nor should it have to handle situations where an
initiated action could not be performed. Getting the last part right is important, as it is a very frustrating user
experience when you try to do something and first after you initiate an action the system lets you know that it is not
currently possible.

The solution we chose is that the UI asks the program what options should be made available, and these options contain
a command that can be executed if the user should choose to select it.


## Creating a tiny route

Imagine that you are about to design a simple electricity route.

- You start off with nothing and drag-and-drop a post onto a canvas from a toolbox.
- You click to focus on the newly added post and arrows in all four directions appear.
- When you click the right arrow a new post is added with wires connected to the first post. The new post has three
  arrows present, and you select the bottom one.
- Yet another post is added, also with wiring to its previous post. The previous post is automatically rotated 45
  degrees to form a corner.

{{< fullwidth title="Designing a tiny route" source="/designer/tiny-route.png" >}}

## Step-by-step walkthrough

Let us walk through this step by step.

Skipping the initial post, here is a simplified diagram on what happens when a user selects a post and acts on one of
the arrows presented (i.e. the transition from image 2 to 3).

{{< fullwidth title="Step by step" source="/designer/step-by-step.png" >}}

When the user selects the post, the 2D shape renderer will ask the `SolutionWorkingSet` which options are available (1).
Option are modeled as `CommandOption`, each represents a single option with meta-data such as:

- **Type**, if this represents an action, movement, menu item, keyboard, etc.
- **Label**, used if this option is displayed as a menu item.
- **Command**, if this option is selected by the user, there is a pre-constructed command that should be executed. This
  command is fully constructed and `CommandOption` will never be created with a command that cannot be executed.

In our example four arrows were presented for the user. These are represented by four different `CommandOption`, one for
each direction. The type of these options will be of the type action, it has a label that can be used as tooltip by the
2D renderer and carry a command of the type `ExtendFromPost` that will be executed upon the user clicking the arrow
(action).

These options are given to the 2D renderer that displays each option as a blue arrow, which is determined by the type
and command in this case.


### Identifying available options

Later in the illustration (screenshot 4), the user selects another post where only three arrows (options) are present.
While this is obvious for a highly trained electricity planner such as yourself, the program still needs to have some
way of determining the available options should only be for three directions, since one is already in use.

The chosen approach for this is to combine the following into something we refer to as the `RuleEngine`.

- **A rich domain model** - a post can tell how many available connections that are available (among other things).
- **Product data** - which posts and wires are compatible with each other.

The `RuleEngine` implements methods to get the available alternatives for specific situations, e.g. to get the available
extensions from a post. These alternatives are not just the direction, they contain all the changes that need to be
made in order to fulfill the alternative.

In the case of extending the electricity route into the available directions it includes:

- The new wires between the both posts.
- The new post.
- A post to replace the originating post with wires attached to it.

All items are fully constructed, have the correct position for each alternative, and all alternatives are possible to
execute.

Below is a pseudocode{{< note 2 >}} implementation of how such a method is implemented in the `RuleEngine`.
{{< sidenote >}}2. A love child between Scala and TypeScript.{{< /sidenote >}}

{{< code >}}
```
def availableExtensionsFromPost(post, world, preferences) =
    for connection in post.availableConnections():
        // As the originating post will be muted (with wiring)
        // for each alternative, a clone is needed 
        val originatingPost = post.clone()
        
        // Calculate where the new post will be positioned.
        // This is based on the originating post's position
        // the direction (e.g north, east, south, west) and the user
        // preferences on center-to-center distance
        val newPostPosition = calculateNewPostPosition(
            originatingPost.position,
            connection.direction,
            preferences)        
        
        // Create the new post together with the wires to add 
        val newPost = Post(newPostPosition)
        val wire = Wires(Position.Zero)

        // If the new route intersects / collides with any
        // existing wiring this alternative should be skipped
        if world.intersects([wire, newPost]):
            continue

        // The result contains a replacement post (with the wiring
        // attached) and the new items.
        // The point of interest will later be used to render the 
        // option arrows in the UI.
        yield ExtendResult(
            replacements = [originatingPost],
            added = [newPost, wires],
            pointOfInterest = post.position) 
```
{{< /code >}} 

As illustrated in the code above, the `RuleEngine` is responsible to ensure that each alternative is viable. The
alternative where the new wires would intersect with something existing is not a valid alternative and not included
in the result.

The `RuleEngine` handles what is possible to do and what items of the world that will be affected. It is the
`SolutionWorkingSet` that is the glue between the graphical representation and the domain model.

The above method would have been called when the user selects the post item in the 2D rendering. Where the UI will call
`SolutionWorkingSet.optionsForItem` to get the options to render.

{{< code >}}
```
def *optionsForItem(id) =
    val item = this.world.byId(id)
    
    // Use our method to get the possible alternatives for extensions
    val alternatives = this.ruleEngine.availableExtensionsFromPost(
      item, 
      this.world. 
      this.preferences)
      
    for alt in alternatives:
        // Create the command that will be executed if the user selects
        // this option.
        // Each command has different needs, this one will only need to
        // replace one post and add the new items to the world upon execution.
        val command = ExtendFromPostCommand(alt.replacement, alt.added)
        
        // Return an option that should be rendered as an Action at the given position
        yield Option(type = Action, position = alt.pointOfInterest, command))
```
{{< /code >}}

### Executing commands

Now that the UI has received the options available it needs to show them somehow. In our case each option represents a
possible direction, which is illustrated with a blue arrow.

![Executing a command](/designer/executing-command.png)

The renderer code that generates the UI above upon a post selection looks something like this:

{{< code >}}
``` 
// Method called when user selects a post
def showOptions(itemId) =
    val options = this.workingSet.optionsForItem(itemId)
    for option in options:
        if (option.type == Action)
            // Create the UI ornament that should execute the command
            // when acted upon
            val ornament = BlueArrow(
                position = option.position,
                onClick = this.workingSet.execute(option.command))
            this.canvas.add(ornament)
```
{{< /code >}}

As you can see the UI is quite “dumb”, it will only get available options, filter out the relevant ones and then create
the UI control that should be used to represent it.

Again, the available options only return viable options and the commands are already constructed; the code needed to
implement the “action handler” when a blue arrow is clicked is very simple, it just executes the command.

When the user clicks the arrow the command is executed by calling the `SolutionWorkingSet.execute`:

{{< code >}}
```
def executeCommand(command):
    // Handle the undo and redo stack 
    updateUndoStack()
    
    // To ensure that the command does not alter the world
    // directly make a copy that we later throw away
    val world = this.world.clone()
    
    // Execute the specific command
    val executionResult = command.execute(world)
    
    // Update the world with any new, updated or removed items 
    this.world.update(executionResult)

    // Generate the instructions on what is needed to render
    // (add, update, delete) and perform a partial re-render of
    // the view
    val renderInstructions = this.createRenderInstructions(executionResult)
    this.renderer.rerender(renderInstructions)
```
{{< /code >}}

The `SolutionWorkingSet` is orchestrating the command execution, handling things like the undo/redo stack and generates
instructions on what to render based on the result of the command execution (to avoid re-render the entire drawing).

Each command is responsible for carrying out the work needed and returning a result.

The command in our example, `ExtendFromPostCommand` was constructed using the alternative from `RuleEngine` as
previously  shown in `optionsForItem`. The replacement post and list of added items were simply given as constructor
arguments to the command, `ExtendFromPostCommand(alt.replacement, alt.added)`.

The implementation of the command execution is something like:

{{< code >}}
```
def execute(world) =
    // The command constructor was given the items to replace and add
    val newPost = this.added.posts.single()
    val wire = this.added.wires.single()
    
    // Connect the wires between the posts, this will set correct
    // positionings, etc.
    wire.wire(this.replacement, newPost) 
    
    return CommandExecutionResult(
        added = [newPost, wire],
        updated = [this.replacement]
        deleted = [])
```
{{< /code >}}

This is a simple command, the RuleEngine has already constructed all the items needed. The command only needs to connect
the wiring between the posts in order to set the positions and attach the wire to each post.


### Result of executing a command

All commands return a `CommandExecutionResult`, that contains:

- What was added to the world.
- What was updated in the world.
- The ID:s of any items that have been removed from the world.

This result is then used by the `SolutionWorkingSet` to update the world with any changes, and then to generate
rendering  instructions for the same set of changes. As we wanted to ensure that no command manipulates the world
directly we make  a clone of the world which is given to the command. Then, by updating the world with the execution
result we can be  certain that the result contains all updates made.

The render instructions are more or less view-model representations of the `CommandExecutionResult`, allowing our
renderer to only update the items that were changed.

I will not cover how the rendering was done in detail but the gist is that all items that are to be rendered have a
view-model representation. This is to ensure that the renderer cannot access or manipulate any model details directly,
as well as having a model optimized for rendering.

Each view model is represented by a shape, which in turn is constructed by a set of primitives such as line, rectangle,
arc, text, etc. These are laid out in a structure that makes it easy to index an item's shape by the item ID which
allows for fast rendering or removal of items.


## That is all there is

At a simplified level, this is it.

Let’s recap before we dive into the strengths and weaknesses of this design.

{{< fullwidth title="Step by step" source="/designer/step-by-step.png" >}}

The renderer and solution working set are unaware of the nitty-gritty details of posts and wires (i.e. the domain rules)
and only focus on showing available options and rendering a world of items.

The commands, world, and `RuleEngine` on the other hand are unaware of how options are presented or how actions are
triggered. Both the commands and rules are focused on the domain rules for a fixed number of situations or tasks.

All of this runs at the client side of the application, after a command has been executed, the server is updated with
the new version of the world for persistence.

### How did it turn out?

This article only used one use-case to illustrate the design. Imagine using the same design for a much more complex
domain, where different subdomains can have their own set of specialized commands.

Our project rendered the world in both 2D and 3D using multiple formats. We managed to reuse the items, view models and
shapes for all 2D rendering (web, PDF- and CAD-files). For 3D, we used separate models for web and STP-files, but the
domain model for items were the same regardless of presentation.

### Separation of concern

I think this software design fits these types of programs very well. It brings a clear separation of concern between
the application UI and the domain logic, we opted for dedicated view models and results instead of direct manipulation
which further cements this separation.

The interaction is naturally limited to be action based and one task at a time, making it easier to reason about the
effect of such a task (implemented as a command).

Once a command has been implemented, it is simple to generate different options and have the same command be executed
by a keyboard shortcut, a menu item or an option shown in the drawing (or why not all of them?).

Commands are also composable. When we were tasked to add support for building a long route of posts and wires we
implemented that as a separate command that executed multiple already existing commands to add one post at a time.

### Testability

Testability is great! As stated in the beginning of this article, everyone on the team are avid users of TDD so that
this design turned out to be highly testable was not by accident.

We have component level tests where items, view models or options are the input, and we can assert on details being
present or not in the UI. Commands execution are simple to test given their pure nature.

For the domain logic we have unit-tests for the domain model, such as getting the number of available connections from
a post given its state. Modifying or accessing the world is also simple to unit-test.

The domain rules have slightly more high-level tests that are executed via the `SolutionWorkingSet` to the `RuleEngine`
and back. These tests look something like this:

{{< code >}}
```
describe("extend form post"):
    beforeEach:
        articleRepository = ArticleRepository("fake-articles.json")
        workingSet = TestableWorkingSet()
       
    describe("single post"):
        beforeEach:
            post = workingSet.addPost("wood", { x: 0, y: 0, z: 0 })
           
        describe("when getting options":
            beforeEach:
                options = workingSet.getOptionsFor(post.id)
               
            test("should have extend options in all four directions"):
                actual = options.filter(_.command is ExtendFromPost)
                       .map(_.position)
                expect(actual).toEqual([
                    { x: 100, y: 0, z: 0 },  // East
                    { x: 0, y: 0, z: -100 }, // South
                    { x: -100, y: 0, z: 0 }, // West
                    { x: 0, y: 0, z: 100 },  // North
                ])
        
            describe("execute to the east"):
                beforeEach:
                    val command = option[0].command
                    workingSet.execute(command)
                
                test("should have added a post"):
                    var posts = workingSet.world.posts()
                    expect(posts.length).toEqual(2)
```
{{< /code >}}

Sure, in the real world there are slightly more details to these tests. I think that the use of options and commands
also benefit the test setup to be quite readable.


### Performance and waste

One challenge with this design is that there is some wasted processing when options are evaluated. For options that are
to be shown in the UI all variants are evaluated and options with commands are constructed for all items. This may lead
to quite some allocations which can cause stress on the browser's garbage collector. When debugging or logging, it is
harder to isolate the execution of a single option (without modifying the code) which makes debugging slightly harder.

Our approach here has been to identify where options are slow to generate or when actions are slow to execute and
tackle them case by case doing traditional performance work.

### Future improvements

When we started this project we kept a collaboration mode in the back of our mind. It is not yet realized, but I think
it fits well with the command execution model where the result of an execution can be distributed to other peers.

I think a simple conflict resolution scheme would be sufficient, one where the first to change an item wins. As pointed
out in [this article](https://jamsocket.com/blog/you-might-not-need-a-crdt){{< note 3 >}} there is a social lock
already. If you visualize where your collaborators are working there is a  natural tendency to not interfere with
others, effectively avoiding conflicts.
{{< sidenote >}}<a class="no-feedback" href="https://jamsocket.com/blog/you-might-not-need-a-crdt">3.You might not need a CRDT</a>{{< /sidenote >}}

As stated in the beginning of this long article, we wanted product data and rules to be managed as configuration and by
domain experts. Currently, domain experts can easily add or update project data, but they cannot change the business
rules. I believe there is room for improvement here. Maybe some of the rules in the `RuleEngine` can be expressed in
something more high-level that even the domain experts can be trained to work in, a small domain specific language,
perhaps. Considering how the gaming industry uses scripting languages in combinations with games engines, I think other
businesses could benefit from this way of adjusting the software, if you get it right it would surely amplify the
capabilities of your domain experts.

Thank you for reading! I hope this walkthrough gave you some inspiration or ideas. If you have any thoughts or
questions please [reach out](mailto:markus.eliasson@gmail.com).
