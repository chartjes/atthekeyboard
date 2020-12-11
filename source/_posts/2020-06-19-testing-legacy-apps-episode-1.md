---
layout: post
title: "Testing Legacy Apps - Episode 1"
author: Chris Hartjes
date: 2020-06-19
---

(If you like the work I have done with testing PHP code and OpenCFP, please consider sponsoring me at
[https://github.com/sponsors/chartjes/](https://github.com/sponsors/chartjes/))

## Testing Legacy Apps - Episode 1 

I have an application that has been in use by the people who participate in my
[longest-running hobby](https://www.ibl.org) for more than a decade. It has
no tests. I have no excuses other than laziness. Time to change that.

In this continuing series of blog posts I am going to show you how to start from
having no tests and ending up with a test suite that covers the behaviour of your
application that matters the most. Along the way I am going to teach you what I
feel is a repeatable framework for approaching testing existing code.

I also want to emphasize that my approach doesn't depend on the framework you
are using. Most PHP web application frameworks include helpers to make testing
easier, and this is a good thing! But there is probably more PHP code out there
either built without a web application framework or one without helpers for testing
that we need to learn some new techniques for writing tests. 

For this whole series there are some constraints we are going to be dealing with:

* I'll be using PHPUnit to write the tests
* I'll be refactoring code to make it easier to test
* I won't be adding any new features

With those three conditions in place, let's pick something and get started.

### What Is The Application's Domain

The application we are writing tests for is for handling transactions and
managing rosters for a tabletop baseball simulation league. The most popular
game of this genre is [Strat-O-Matic Baseball](http://www.strat-o-matic.com/baseball-board-games/)
and the league I am in uses a game we created ourselves.

I'm sorry if some of the terms I end up using are ones you don't understand as
some domain knowledge is required to figure out tests.

I created a web application to manage all the roster stuff more than a 
decade ago, and slapped it together quickly because we needed something. Over
the years it's been tweaked but I have been lazy and justified not having any
tests for it because "I understand the domain well enough to manually test it".
Shockingly, I still manage to break things.

As with any long journey, it begins with a step. I'm going to pick something
that I constantly break and wrap some tests around it.


### What is our first testing scenario?

As part of this whole series, I also want to emphasize that I will be focussing
on not testing the code but coming up with tests for how the application is
supposed to behave. To figure this out, I need to identify what parts of the
application need to work all the time.

* making trades should not break
* signing free agents should not break
* entries in the transaction log should always be correct 

Those are the high level tests that need to be written. There are also some
tests at a lower level that need to happen to satisfy the conditions above.
I keep finding ways to break some functionality that deals with indicating
whether or not a player had a "card" in the game (basically, did we print
a card for that player for a specific year). Why does it keep breaking?
Because I did not create one centralized location that is the source of
truth for a player.

I've written tests for this functionality before as examples for my books
and presentations but I feel like it is time to take a different approach
and build something easier to work with.

When I am creating testing scenarios I like to use the language that
people who practice [Behavior-Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development)
tend to use. So here is a great testing scenario to start with:

*Given I am a Player*
*When I have no card for the current season*
*Then indicate my uncarded status*
*And indicate the season I was uncarded for*

Now, when I look at the code I have already I am doing this...but I don't like it.
I have created the idea of a Roster and created a 'model' for it and I just deleted
some tests I had for it. Instead, I think I want to drill a little deeper into this
problem and come up with a better fix.

A Roster is a collection consisting of one or more objects. They are "batters", "pitchers", and "draft picks".
Right now I am not making that distinction. Here's the code that I wrote that looks
at a player's "uncarded status" and figures out what additional information needs
to be displayed next to the player's name:

```php
    public function getBattersByIblTeam($ibl_team): array
    {
        $sql = "SELECT * FROM rosters r WHERE r.ibl_team = ? AND r.item_type = 2 ORDER BY r.tig_name";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute([$ibl_team]);
        $results = $stmt->fetchAll(PDO::FETCH_ASSOC);

        if (!$results) {
            return [];
        }

        $roster = [];

        foreach ($results as $row) {
            $displayName = trim($row['tig_name']);

            if ($row['uncarded'] === $this->previous_season || $row['uncarded'] === $this->current_season) {
                $displayName .= ' [UC' . $row['uncarded'] . ']';
            }

            $row['display_name'] = $displayName;
            $roster[] = $row;
        }

        return $roster;
    }
```

This is not bad code by any means -- it works. From a testing perspective there are all
sorts of problems with it due t
