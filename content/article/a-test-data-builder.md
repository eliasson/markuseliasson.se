---
title: "A test-data builder"
date: 2025-02-26T09:00:00+02:00
description:
  Something catchy.
---
The other day I was tasked to produce fake data for a particular scenario of the project I work on.
I needed to create realistic data for our system of relational data. In order to do this I reused
a test-data builder we have for our automated tests which made the task a lot easier. This article
describes an example of how such a test-data builder can be built.

In the system I currently work on we do a lot of automated testing. Both unit-tests, system-tests,
integration-tests, end-to-end tests, etc. When doing system level tests and integration-tests we
are faced with setting up test data for many different situations. The test-data must also be
correct / valid, as there are plenty of business rules that are validated by the system. This
can easily result in complex and verbose test-setup.

A complex test-setup in combination with a system where we have thousands of tests is not a good
combination! One of the ways to mitigate this is to provide a test-data builder.

## A relational model

Before we dive into the art of test-data, let us set the scene by going through the example domain
used in this article.

As I enjoy listening to music and collecting vinyl records I will base the rest of this article on
a small domain model around that.

**Add diagram of Artist, Album, Track, Rating**

Besides the aggregates, the system has Users and some sort of Configuration.

## Our system design

The way we have designed our system is neither the only way nor the required way to design a system
to benefit from the test-data builder described here. There are however some aspects of it that the
builder needs to support.

- **Domain model**, we use a rich domain model that contains most of the business logic.
- **Repositories**, we use repositories for all reading and writing of aggregates.
- **Services**, we use services for other IO or for processing that is not part of the domain
model.
- **Asynchronous**, all IO operations are asynchronous and test-data generation must therefore also
be.


## A first test

Now, imagine that we want to write a test for a service where a user can mark a _track_ as a
favourite.

```
public class WhenGettingFavouriteTracksForUser
{
  private TestCaseBuilder _tc = null!;

  [OneTimeSetUp]
  public async Task SetUp()
  {
    _tc = await new TestCaseBuilder()
      .WithDefaultConfiguration()
      .WithUser()
      .WithArtist()
      .WithAlbum()
      .WithTrack(name: TrackOne)
      .WithTrack(name: TrackTwo, configure: t => t.Title = "Pictures of you")
      .BuildAsync();

    var user = await _tc.UserOrThrow();
    var track = await _tc.TrackOrThrow(TrackTwo)
    _favouriteTracks = _tc.Service.GetFavouriteTracksAsync(user.Id, CancellationToken.None);
  }

  [Test]
  public void It_should_have_one_favourite_track()
  {
    var favourites = _favouriteTracks.Select(t => t.Title);
    Assert.That(favourites, Is.EqualTo(new [] { "Pictures of you" }));
  }
}
```
