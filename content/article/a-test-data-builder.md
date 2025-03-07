---
title: "A test-data builder"
date: 2025-02-26T09:00:00+02:00
description:
  A while back I built a test-data builder for one of the projects I am working on. Now, after
  using if for some time I really think it simplifies and improves our tests, while also being
  faster to write. This article will guide you through how to implement one for yourself.
---
In the system I currently work on we do a lot of automated testing. Both unit-tests, system-tests,
integration-tests, end-to-end tests, etc. When doing system level tests and integration-tests we
are faced with setting up test data for many different situations. The test-data must also be
correct / valid, as there are plenty of business rules that are validated by the system. This
can easily result in complex and verbose test-setup.

A complex test-setup in combination with a system where we have thousands of tests is not a good
combination! One of the ways to mitigate this is to provide a test-data builder.

This article will go through the implementation of such a test-data builder.

## A relational model

Before we dive into the art of test-data, let us set the scene by going through the example domain
used in this article.

As I enjoy listening to (and collecting) music, so today's domain will be albums and
tracks. The model is greatly simplified to only contain what is needed for two illustrative tests.

Think of system where a user enters music albums, their tracks and the performing artist. The user
can mark tracks as favourite tracks. And that is all there is.

![Our simplified model](/test-data-builder/test-data-model-white.png)

A User is a cross-cutting concern in this system. All persisted data will be separated by user.

- Each Artist has a reference to the owning User.
- Each Album has a reference to an Artist.
- Each Track has a reference to the Album it belongs to.

Here is the code, in C#{{< note 1 >}}.
{{< sidenote >}}
1. The examples here will be in C#, but the concepts is transferable to most languages. At least
object-oriented languages.
{{< /sidenote >}}

{{< code >}}
```csharp
public interface IAggregate
{
    // As this is an example, use Guid instead of, better, a type-safe ID.
    public Guid Id { get; }
}

public class User : IAggregate
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Username { get; set; } = string.Empty;
}

public class Artist : IAggreagate
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Name { get; set; } = string.Empty;
}

public class Album : IAggreagate
{
    // As this is an example, let's limit each album to one artist.
    public Guid ArtistId { get; set; } = Guid.NewGuid();
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Name { get; set; } = string.Empty;
}

public class Track : IAggregate
{
    public Guid AlbumId { get; set; } = Guid.NewGuid();
    public Guid Id { get; set; } = Guid.NewGuid();
    public string Title { get; set; } = string.Empty;
    public bool IsFavorite { get; private set;  } = false;
    
    public void MarkAsFavourite() => IsFavorite = true;
    public void UnmarkAsFavourite() => IsFavorite = false;
}
```
{{< /code >}}

The example used something called an _Aggregate_. If you are not familiar with the term, you can think
of it as a coherent entity with a unique ID. It is not that important{{< note 2 >}} for the rest of this
article.
{{< sidenote >}}
2. Aggregate is an important concept in Domain Driven Design, however. And I think DDD is a valuable tool
for modelling complex domains.
{{< /sidenote >}}

## Our system design

While being a very small example, I have built it using patterns from Domain Driven Design{{< note 3>}}, just
like the system that where I originally created this test-data builder.
It is however not necessary to use these patterns to get the benefits of a test-data builder. The concepts used
in the example are fairly unorthodox and hopefully easy enough to understand even if you are not familiar with
them in the first place. 
{{< sidenote >}}
3. Here is this domain thing again! Could it be something interesting? 🤔
{{< /sidenote >}}

Concepts used:
- **Domain model**, a rich domain model that contains most of the business logic. Free of I/O and other effects.
- **Repositories**, are used for all reading and writing of aggregates, lots of I/O.
- **Services**, are the point of entry for any actions, orchestrates domain model and other processing. The examples use it very sparsely.
- **Asynchronous**, all I/O operations are asynchronous and test-data generation must therefore support it.

## A first test

Now, imagine that we want to write a test for a service where a user can mark a _track_ as a
favourite.

With no test-helper such a test might look something like.
{{< code >}}
```csharp
public class WithoutTheBuilder
{
    private UserRepository _userRepository = null!;
    private Repository<Artist> _artistRepository = null!;
    private Repository<Album> _albumRepository = null!;
    private Repository<Track> _trackRepository = null!;

    private User _actingUser = null!;
    private FavouriteService _service = null!;

    [OneTimeSetUp]
    public async Task MarkTrackAsFavourite()
    {
        // First, we need to set up the infra to store our stuff.
        // Due to validation rules we must create the correct graph of aggregates.
        _userRepository = new UserRepository();
        _artistRepository = new Repository<Artist>();
        _albumRepository = new Repository<Album>();
        _trackRepository = new Repository<Track>();

        // Create the acting user for our test
        _actingUser = new User { Username = "emma.goldman" };
        await _userRepository.SaveAsync(_actingUser, CancellationToken.None);

        var artist = new Artist { UserId = _actingUser.Id, Name = "Kite" };
        await _artistRepository.SaveAsync(_actingUser.Id, artist, CancellationToken.None);

        var album = new Album { ArtistId = artist.Id, Name = "VII" };
        await _albumRepository.SaveAsync(_actingUser.Id, album, CancellationToken.None);

        var track = new Track { AlbumId = album.Id, Title = "Glassy Eyes" };
        await _trackRepository.SaveAsync(_actingUser.Id, track, CancellationToken.None);

        // Create the service we are writing our test of.
        _service = new FavouriteService(_userRepository, _trackRepository);

        // Perform the action that we are testing
        await _service.SetFavouriteTracksAsync(_actingUser.Id, track.Id, CancellationToken.None);
    }

    [Test]
    public async Task It_should_have_one_favourite_track()
    {
        var favouriteTracks = await _service
            .GetFavouriteTracksAsync(_actingUser.Id, CancellationToken.None)
            .ToListAsync();

        var favourites = favouriteTracks.Select(t => t.Title);

        Assert.That(favourites, Is.EqualTo(new [] { "Glassy Eyes" }));
    }
}
```
{{< /code >}}

> Hey, there is a lot of stuff being set up which is never used!

Well, indirectly it is being used. If you have a complex domain you most definitely want to enforce the business rules,
have a consistent model, etc. This is typically done both in the domain model and in the services. In the
example above, there can be constraints at the persistence layer, that a track cannot be saved unless
its album reference also exists.

Now, think about your own system. Are your entities as simple as set up as the `Artist` and `Album` above? No, I did
not think so.

Combine this with having thousands of tests. Where a lot of setup is repeated.

## An alternative

Now, let us see how this might look using a test-data builder.

{{< code >}}
```csharp
public class WhenGettingFavouriteTracksForUser
{
    private FavouriteTestCase _tc = null!;

    [OneTimeSetUp]
    public async Task MarkTrackAsFavourite()
    {
        _tc = await new FavouriteTestCase()
            .WithUser()
            .WithArtist()
            .WithAlbum()
            .WithTrack(configure: t => t.Title = "Glassy Eyes")
            .BuildAsync();

        var user = await _tc.UserOrThrowAsync();
        var track = await _tc.TrackOrThrowAsync();

        await _tc.FavouriteService.SetFavouriteTracksAsync(user.Id, track.Id, CancellationToken.None);
    }

    [Test]
    public async Task It_should_have_one_favourite_track()
    {
        var user = await _tc.UserOrThrowAsync();

        var favouriteTracks = await _tc.FavouriteService
            .GetFavouriteTracksAsync(user.Id, CancellationToken.None)
            .ToListAsync();

        var favourites = favouriteTracks.Select(t => t.Title);

        Assert.That(favourites, Is.EqualTo(new [] { "Glassy Eyes" }));
    }
}
```
{{< /code >}}

This alternative is a lot more concise. The "irrelevant" details, such as the username of the acting user, details
about the artist and album is omitted.

What is left is the details _that do matter_. The name of the track is configured, since we use that later in the
test when asserting the result.

But there are a lot of things going on under the hood. Let us go through piece by piece.

## Features of the builder

### Synchronous setup - asynchronous construction

Previously I wrote that all construction and I/O has to be asynchronous, didn't' I? So how come we add
user with `WithUser`, this does not look very asynchronous{{< note 5>}}.
{{< sidenote >}}
4. In C# asynchronous methods returns a `Task` (similar to JavaScript's `Promise` or `Future` in Scala). The convention
is to name such methods using a `Async` suffix.
{{< /sidenote >}}

Looking at the implementation of the `WithUser` shows us the answer.

```csharp
public TestCaseBuilder WithUser(
    UserName? name = null,
    Action<User>? configure = null)
{
    // Kick off the construction on some thread, to run at some point.
    var task = Task.Run(() => {
        var user = new User { Username = name?.Name ?? "emma.goldman" };
        configure?.Invoke(user);
        await UserRepository.SaveAsync(user, CancellationToken.None);
    
        // Use the given name or create a default name to use for this
        // test-data instance.
        var testName = name ?? NextUserName();
        
        // Store details about the constructed user and its test-data
        // name so it can be retrieved from tests. 
        _constructedUsers.Enqueue(new ConstructedUser(user, testName));
    });
    
    // Keep track of the task so that we can wait for its completion at
    // a later stage.
    _constructionOfUsers.Add(task);
    
    // This is a builder alright!
    return this;
}
```

Aha! The user _is_ created asynchronously, we just do not wait for it until we return!

What takes place is the following:

1. Kick of the construction to run as a `Task` on the thread pool that runs tasks. The execution
will not wait for this complete.
2. Store the result of this operation as `task`, so that it can be added to a list of all the tasks of _"created users"_.
3. Once the code passed to `Task.Run` is executed, create an arbitrary instance of `User`.
4. Since arbitrary data only gets you so far, there is an option for the caller to configure the instance.
5. When the code passed to `Task.Run` is run, it can be async, which 
6. After the user is saved via the repository it is added to a list of constructed users.
There is an option for the caller to provide a name for this instance, if none is given a default name is created

Being synchronous allows for the builder notation, where method calls are chained like `WithUser().WithArtist()`, even
thought the construction itself is asynchronous.

Providing an _optional_ argument where the caller of the function (the test) can configure the test-data is quite
nice. The basis of the instance will be using the default data used in the builder, and the test can override the
significant details from the test, like setting the track name: 

```csharp
.WithTrack(configure: t => t.Title = "Glassy Eyes")`
```

### Building

Building the test-case on the other hand is an asynchronous task.

### Acting on test data

- TODO: Describe how to access constructed instances
- TODO: Expose a service with everything setup
- TODO: Inheritance

### Asserting

- Again, accessing test data
- Not needed for the test-data builder, asserting on identity is a lot better than asserting on count.

### Future stuff


## A full example

The code examples from this article is also available in this [repository](https://github.com/eliasson/test-data-builder/)
at GitHub. It might be that the example code looks slightly different from the examples above since we explored the
builder step by step in the article.

The support for the `Album` aggregate is the one with guide comments:
[TestCaseBuilder.Album.cs](https://github.com/eliasson/test-data-builder/blob/main/Code.Tests/TestCaseBuilder.Album.cs).