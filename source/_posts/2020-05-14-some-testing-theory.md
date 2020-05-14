---
layout: post
title: "Some Testing Theory"
author: Chris Hartjes
date: 2020-05-14
---

## TL;DR - Focus on testing behaviours instead of testing code 

(If you like the work I have done with testing PHP code and OpenCFP, please consider sponsoring me at
[https://github.com/sponsors/chartjes/](https://github.com/sponsors/chartjes/))

## Some Testing Theory 

While doomscrolling on Twitter, I saw internet-friend [Snipe](https://twitter.com/snipeyhead)
ask the following [question](https://twitter.com/snipeyhead/status/1260814028500119552?s=20):

> When writing unit tests, do you typically write a test to check if the model is saved (i.e. create it via factory, check it passes built-in validation)? Feels a bit too much like testing the framework (or the factory) to me

To which I [responded](https://twitter.com/grmpyprogrammer/status/1260883985670057984?s=20)

> Personally I am writing unit tests to verify behaviour, and if it requires making sure the model was saved then I will check

Snipe [followed-up-with](https://twitter.com/snipeyhead/status/1260884210736234497?s=20):

> In acceptance or functional tests, sure - just seems weird to have them in unit tests.

Her response got me to thinking about some testing theory ideas that have changed how
I approach the tests I write and how I categorize a particular test. As always, there
are multiple approaches to solving these problems -- "trust, but verify" is a good
practice.

### Test Behaviours, Not Code

No matter what type of test I am writing (more on that later) I always ask myself "what
test will prove this code is behaving as expected?". This is a different approach from
"what parts of the code am I going to test". My experience has been that when you focus
on testing behaviours, you end up writing fewer tests but with the same level of coverage
of the code under test. 

Focussing on behaviour also means you do not have to make context switches when you start
thinking about tests of different types.

### Test Types

Commonly these have been referred to as unit vs integration vs acceptance. Labels can change over time
and right now I have settled on three types of tests:

#### Microtests

Microtests are tests that are verifying the behaviour of a single object in isolation. Calling
them unit tests works here too. The additional pressure being applied here is whether or not
you will use real versions of the dependencies the code you are testing requires or if you
will create test doubles. 

Both approaches have benefits and drawbacks. Tests with doubles tend to run quickly but have
the maintenance overhead of needing to be updated if the behaviour of the doubles no longer
matches the dependency. Tests what use real dependencies are slower and can have the maintenance
overhead of needing databases or services to be made available and updated on a regular basis.

These are typically written a testing framework and can be automated via CLI tools.

#### Integration Tests

Integration tests are tests that verify that the behaviour of two objects, when interacting with
each other, is as we expect. These tests should almost always use real dependencies unless
there is a really good reason not too. Maybe something like an API with sandbox access because
the API you are using in production is rate limited or charges per use. These things should
be exceptions rather than a common practice.

These are also typically written with a testing framework and can be automated via CLI tools.
The goal of this level of tests is to act as a filter that catches any bugs that your first
layer of microtests missed.

##### Acceptance Tests

Acceptance tests are tests that verify that the behaviour of the application is correct, meaning
that multiple objects will be interacting with each other using real dependencies. These sort
of tests are usually conducted manually or built using some kind of automation framework that
can drive a client application (usually a web browser).

Just like the integration tests, this layer should be catching any bugs your microtests and
integration tests didn't find. Tests are usually written by humans, so there are some scenarios
and edge cases that were not considered when the tests were written. All you can do is write
code as defensively as possible, carefully consider your testing scenarios, and hope that
nothing goes horribly wrong in production.

### Back to Snipe's Question

So the original question is "should my unit tests be checking that data is saved?". The answer,
in my mind, is that if the behaviour you are testing requires you to verify that data that was
just created is saved and contains data you expect, then you will need to use real models with 
a real database connection. 

I emphasize there is no wrong answer to Snipe's question! It is a matter of deciding on an approach
and dealing with the associated technical debt.
