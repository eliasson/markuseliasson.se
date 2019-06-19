+++
Categories = ["Code"]
Description = "Writing clear, consice and maintainable tests can be quite challenging regardless of tool and language. Following the practices discussed here will hopefully help you achieve this if you are using ScalaTest."
Tags = ["Scala"]
date = "2019-06-13T14:35:00+02:00"
title = "Best practices for ScalaTest"
draft = false
+++
Like many things in the Scala ecosystem, [ScalaTest](http://www.scalatest.org) does not come short
on features or alternatives to use when writing tests. This is seen as a strength and is advertised on their webpage as:

> ScalaTest does not try to impose a testing philosophy on you, but rather is designed from the philosophy that the tool should get out of your way and let you work the way you find most productive.

The last two years I have been working a fair share with ScalaTest, both writing new tests as well as extending and modifying existing tests. Working with different code bases, where different features of ScalaTest are being used can be quite frustrating given the vast number of features and alternatives ScalaTest offers. The number of features can sometimes be overwhelming when writing test, and I would argue that it increases the cognitive load when reading code as well (such as reviewing Pull Requests).

This led me to define a core list of conventions and practices of the features that I find most useful and that I think gives clear, readable and maintainable tests.

## Use FreeSpec

I prefer to use `FreeSpec` as the basis for my tests for the following reasons:

* Low ceremony
* No specific keywords such as `test` or `describe`
* Tests can be nested as deep as you want

```
import org.scalatest.FreeSpec

class UserRepositoryTest extends FreeSpec {
    "UserRepository" - {
        "after initial setup" - {
            "there should be zero users" in {
                // assertion
            }
        }
    }

    // helpers
}
```

I tend to write hierarchical tests as I think it is a great way to structure tests both when writing and later when I need
to find the corresponding tests for a given piece of functionality.

The tests are often structured into _Component_ / _State_ / _Behaviour_ and while reading the path should make sense, it does not have to be a well-formed sentence, and does not have
to start using _"should"_ or _"test"_ either.

## Use fixture-context objects

I am a firm believer in that a test only should test one thing, and multiple assertions in a single test should be avoided whenever possible. In order to achieve this, some sort of test setup is needed.

A common way of structuring tests and data is to use separate setup methods (such as `beforeEach` and `afterEach`) - these can be very useful but my biggest concern is that they often lead to a disconnect between setup and test, which obviously is coherent.

ScalaTest has a feature named [fixture-context objects](http://www.scalatest.org/user_guide/sharing_fixtures) that helps keeping the test data setup next to the test, it basically instantiates a class and makes its members available in the scope of the test, e.g.:

```
import org.scalatest.{FreeSpec, Matchers}
import scala.concurrent.Future

class UserRepository {
    def count: Long = 0
}

class UserRepositoryTest extends FreeSpec
    with Matchers {

    "UserRepository" - {
        "initial setup" - {
            "has zero users" in new TestData(new UserRepository()) {
                repository.count should equal (0)
            }
        }
    }

    class TestData(val repository: UserRepository)
}
```

Here the `TestData` is parameterized via constructor arguments for the specific test and it stays close to the test code. Compared to using `beforeEach` which cannot be parameterized and often relies on mutable variables that are modified inside the test. Not to mention that the test and the `beforeEach` sometimes are separated by hundreds of lines.

You might have noticed that ScalaTest offers another implementation of `FreeSpec`, namely `org.scalatest.fixture.FreeSpec` that
offers similar functionality. It is the _officially recommended solution_ to use when many tests share the same fixture, and even
so I would recommend against using due to:

* You must supply an internal case class named `FixtureParam`
* The setup is done at a distance (not close to the test)
* It's cumbersome as you have to call the test with the fixture

```
class UserRepositoryTestSyncWithFixture extends org.scalatest.fixture.FreeSpec
  with Matchers {

  case class FixtureParam(repository: UserRepository)

  override def withFixture(test: OneArgTest): Outcome = {
    val theFixture = FixtureParam(new UserRepository())
    withFixture(test.toNoArgTest(theFixture))
  }

  "UserRepository" - {
    "when loaded" - {
      "has three active users" in { fixture =>
        fixture.repository.activeUsers.size should equal (3)
      }
    }
  }
}
```

## Be async

Working with Scala, sooner or later you will need to test functions that returns `Future[T]`, and there are basically two options.

Mix in the trait `ScalaFutures` in your test class and use `whenReady` to handle your future responses, like this:

```
import org.scalatest.{FreeSpec, Matchers}
import org.scalatest.concurrent.ScalaFutures
import scala.concurrent.Future

class UserRepositoryTest extends FreeSpec
    with Matchers
    with ScalaFutures {

    "UserRepository" - {
        "initial setup" - {
            "has zero users" in new TestData(new UserRepository()) {
                whenReady(repository.countAsync) { count =>
                    count should equal (0)
                }
            }
        }
    }

    class TestData(val repository: UserRepository)
}
```

`whenReady` will keep polling the future until it is ready, or the (implicit) timeout has expired. It is blocking, which enables a flatter structure (similar to `await` in other languages) so that the following snippet is valid:

```
whenReady(firstFuture) { () }
whenReady(secondFuture) { somethingOfInterest =>
  somethingOfInterest.value should be (42)
}
```

The alternative is to use the trait `AsyncFreeSpec`, which then expect each test to return a `Future[Assertion]`. Unfortunately, _async specs_ does not mix with _fixture-context object_ out-of-the-box as the fixture needs to be created asynchronous as well.

This [thread on StackOverflow](https://stackoverflow.com/questions/46801118/how-to-use-fixture-context-objects-with-async-specs-in-scalatest) describes the situation, based on that thread and some helper functions we get the following test code:

```
import org.scalatest.{Assertion, AsyncFreeSpec, FreeSpec, Matchers}
import scala.concurrent.Future

class UserRepositoryTestAsyncOther extends AsyncFreeSpec
    with Matchers {

    "UserRepository" - {
        "initial setup" - {
            "has zero users" in async(new TestData(new UserRepository())) { testData =>
                testData.repository.countAsync.map { count =>
                    count should equal (0)
                }
            }
        }
    }

    class TestData(val repository: UserRepository)

    def async(data: TestData)(fn: TestData => Future[Assertion]) = fn(data)
}
```

Note, that by default ScalaTest will run the test in a single suite sequentially and not in parallel unless the test mixin `ParallelTestExecution` (rationale is available in the [documentation](http://www.scalatest.org/user_guide/async_testing)). Even though the performance wins might be small in parallel testing, I would recommend turning this on - as it helps

I would consider running tests in parallel even if there is little to no performance win. Running tests sequential might hide concurrency related bugs in your code - parallel and random execution order will give you the best coverage.

## Be eventual

When writing tests for an eventual consistent system you might need to test an asynchronous behavior that _does not_ return a `Future`. ScalaTest comes
with the trait `Eventually` which fills this need so that you don't have to sleep.

```
import org.scalatest.{FreeSpec, Matchers}
import org.scalatest.concurrent.Eventually

class TestEventualConsistency extends FreeSpec
  with Matchers
  with Eventually {

  "Trigger a change" - {
    "should eventually be processed" in new TestData() {
      fireEvent()

      eventually {
        eventProcessed() should be (true)
      }
    }
  }

  case class TestData() {
    def fireEvent(): Unit = ???
    def eventProcessed(): Boolean = ???
  }
}
```

In this example, the assertion will be periodically evaluated using a timeout. If the assertion fails after the specified timeout the test will fail.

For how long and at what interval the evaluation should be performed is specified using a _patience_. The default is tuned for unit-testing, but there is a handy trait named `IntegrationPatience` that increase the timeout to 15 seconds instead of 150 ms which is the default.

Based on your need you can defined your own patience, it's just an implicit parameter to `eventually`, see the [Scaladoc](http://doc.scalatest.org/3.0.0/index.html#org.scalatest.concurrent.Eventually).

## Use beforeEach carefully

If your tests are using test-fixtures there are less of a need to run code before and after each test. But if it still is needed the test class needs to mix-in the trait `BeforeAndAfterEach`.

```
import org.scalatest.{FreeSpec, Matchers, BeforeAndAfterEach}

class UserRepositoryTest extends FreeSpec
    with Matchers
    with BeforeAndAfterEach {

    "UserRepository" - {
        println("describe 1 st level")

        "initial setup" - {
            println("describe 2 nd level")

            "has zero users" in {
                println("the test")
                1 should equal(1)
            }
        }
    }

    override def beforeEach() = {
        println("beforeEach")
    }
}
```

The call to `beforeEach` will literally come just before the test, so the output the test above will be:

```
describe 1 st level (during discovery phase)
describe 2 nd level (during discovery phase)
beforeEach (during test execution)
the test (during test execution)
```

Note that both mix-ins mentioned above are synchronous so you cannot do any async setup in them without blocking (e.g. using `Await.result`).

It is quite easy to end up putting setup code that gets executed in the discovery phase which often lead to subtle bugs and non-deterministic behavior. Only put constants here, or preferably, nothing at all.

Sharing test setup is much easier done using _fixture-context objects_ described earlier.


## Define a focused tag

In many unit-testing frameworks it is common to be able to focus on a specific test or suite - e.g. using `fit` in Jasmine (JavaScript).

And while your IDE might support running individual tests, I often find that when changing existing code, it is very handy to be able
to highlight a few individual tests that does not necessarily exist in the same hierarchy.

ScalaTest handles this a bit differently, where you tag your test using symbols you define yourself and then filter tags when you run the tests using the parameter `-n focused`

```
import org.scalatest.{FreeSpec, Matchers, Tag}

object Focused extends Tag("focused")

class UserRepositoryTestSync extends FreeSpec
  with Matchers {

  "UserRepository" - {
    "initial setup" - {
      "has zero users" taggedAs(Focused) in new TestData(new UserRepository()) {
        repository.count should equal (0)
      }

      "has zero users again" in new TestData(new UserRepository()) {
        repository.count should equal (0)
      }
    }
  }

  class TestData(val repository: UserRepository)
}
```

While this is slightly more cumbersome than just using `fit` it comes with the benefit that the test-runner controls what is executed without changes to code, which of course is great to use in a CI pipeline to run integration-tests, nightly, etc.

My recommendation here is to define your tags in a [top-level package object](https://docs.scala-lang.org/tour/package-objects.html) so that they are available in all your test automatically:

```
package foo

import org.scalatest.Tag

package object util {
  object Focused extends Tag("focused")
}
```

## Keep your matchers few

Here is a glance on the available flavours of matchers in ScalaTest:

```
repository.activeUsers.size should equal (3)  // Must have parens
repository.activeUsers.size shouldEqual 3     // May have parens
repository.activeUsers.size should === (3)
repository.activeUsers.size should be (3)
repository.activeUsers.size shouldBe 3
repository.activeUsers should have size 3     // An implicit Length[T] exists
repository.activeUsers shouldNot be ('empty)  // ' is a shorthand for parameterless methods returning Boolean
```

More matchers and descriptions available in the [documentation](http://www.scalatest.org/user_guide/using_matchers).

I tend to use the `equal` matcher as much as possible. Even though ScalaTest comes with quite a lot of matchers I stick to basic ones - there are two reasons for that;

First, I don’t find them intuitive, so I have to search for which one to use when writing a test, and then often once again whenever I review some code later on. Some require parenthesis, others don't - it is quite confusing.

Second, using familiar Scala constructs makes the test more readable and I prefer to process the data prior to any assertion which often let me use simpler matchers.

```
case class User(username: String, fullName: String)

class UserRepository {
  def activeUsers: Seq[User] = Seq(User("John", "John doe"), User("Jane", "Jane Doe"), User("janet", "Janet Doe"))
}

class UserRepositoryTestSync extends FreeSpec
  with Matchers
  with BeforeAndAfterEach {

  "UserRepository" - {
    "when loaded" - {
      "has three active users (3)" in new TestData(new UserRepository()) {
        var usernames = repository.activeUsers.map(_.username).map(_.toLowerCase)
        usernames should equal (Seq("john", "jane", "janet"))
      }
    }
  }

  class TestData(val repository: UserRepository)
}
```

This test uses a pattern called _Transform to Identity_, it was coined by my co-worker [@provegard](https://twitter.com/provegard) in his talk [Supercharge your automated tests to fail better](https://www.youtube.com/watch?v=jJRgSy2vVF8). The assertion is made simple while reading and it is expressive when failing, win-win.


## Other considerations

When composing this list, I identified a few things that is unrelated to ScalaTest, or at least not dependent of its features. But there are a few things that I have started to apply more and more regardless of language and framework when I write unit-tests.

### Don’t name things SUT

Stay away from using the name `sut` such as in variables. While it makes perfect sense when writing the test, I find that when I come back later, or when reviewing an isolated delta it is easy to make mistakes on what is being tested. It might be that the test file is too large, but I still think spelling out what it is you are testing improves readability.

### Put your test first

Do not put class member declarations, test-fixtures, helper functions, `beforeEach/afterEach` at the top of the class body. A future developer, or future you, are primarily interested in reading the test and not boilerplate and setup. Put the most relevant first and everything else at the end of the class / file.

And for shared helpers I find Scala package object to be a great container.

```
package foo

import org.scalatest.Tag

package object util {
  def randomPort(): Integer = 4000 + scala.util.Random.nextInt(6999)
}
```

Will make the method `randomPort` available in your tests in the same package.

## In summary

* Use FreeSpec
* Use fixture-context objects
* Be async
* Be eventual
* Use beforeEach and beforeAll carefully
* Define a focused tag
* Keep your matchers few
* Don’t name things SUT
* Put your test first

I find this list helpful when I write my tests, and even if you don’t agree on all the practices described you have hopefully picked up something useful.

As shown I use only a limited set of features from ScalaTest, I don’t think that is in conflict with their design goals, rather the opposite - using ScalaTest a team can cherry pick features and have it done their way - just like Scala itself.

Oh, even though it goes without saying, when adding a new single test to an existing spec adhering to the existing style - whatever it is - is outweighs the recommendations here.

Happy hacking!
