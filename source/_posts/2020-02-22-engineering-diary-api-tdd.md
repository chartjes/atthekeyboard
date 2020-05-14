---
layout: post
title: "Engineering Diary -- Building an API using TDD"
author: Chris Hartjes
date: 2020-02-22
---

## TL;DR - using Test-Driven Development to build an API 

(If you like the work I do on OpenCFP, please consider sponsoring me at
[https://github.com/sponsors/chartjes/](https://github.com/sponsors/chartjes/))

[OpenCFP Central](https://opencfpcentral.com) is a companion web application that
I run that works with [OpenCFP](https://github.com/opencfp/opencfp). Right now
people can sign up for an account at OpenCFP Central and get an key that can be
used to allow OpenCFP to use OpenCFP Central accounts as authentication. This
feature was one that was requested for a long time, and I'm happy I can offer it
to folks using a more recent versions of OpenCFP.

The next feature I am wanting to build is to add an API to OpenCFP Central that allows
talks to be stored there and then can be pulled and submitted to any OpenCFP application
that has been updated to include that ability. I'm already using OAuth so I can make
the API use that.

OpenCFP Central is currently a [Laravel 6](https://laravel.com/docs/6.x) application.
I'm using [Passport](https://laravel.com/docs/6.x/passport) for my OAuth needs. My
initial scan of the documentation seems to indicate that is straightforward (but not
necessarily easy) to do what I am trying to do.

### Models and Resources

This being the first time I've built anything related to an API with Laravel, it
looks like the path-of-least-resistance is to use as much of the built-in tools
that Laravel gives me. I settled on sending things back and forth as JSON. Looking
at the documentation it appears I should be using [resources](https://laravel.com/docs/6.x/eloquent-resources#generating-resources)
and resource collections. They will in turn become JSON responses via some code
behind the scenes.

What makes this appealing to me is that given the lack of time (basically one day a week)
I can give my side projects, I can have a lot of the plumbing for these API calls
already done.

So, I can use my existing `User` model and just add a relationship to say that "a
User can have one or more Talks"

```php
/**
* @return HasMany
*/
public function talks(): HasMany
{
    return $this->hasMany(Talk::class);
}
```

Next I have to go back and create my Talk model. But before that I need to create a
migration to add it to the database. I decided to keep things simple:

* an ID
* a UUID I will use for display purposes
* the ID of the user the talk belongs to 
* the details of the talk stored as JSON

I am using [PostgreSQL](https://www.postgresql.org/) as my database and it has a "JSONB"
field type.

```php
<?php
declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class NewTalksTableAsJsonb extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('talks', function (Blueprint $table) {
            $table->increments('id');
            $table->uuid('uuid');
            $table->integer('user_id');
            $table->jsonb('details');
            $table->datetime('created_at');
            $table->datetime('updated_at');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('talks');
    }
}
```

The `details` field needs to contain whatever is the data for an OpenCFP
talk that is not configurable by whomever is running it. I believe this
to be the following:

* title of the talk
* description of the talk
* and then any other details the speaker wishes to share

So I think a typical JSON payload for an individual talk will look something like this:

```json
{
  "user_id": 1,
  "uuid": "someuuidweusetodisplay",
  "title": "Test Title",
  "description": "Test Description",
  "other": "Things the speaker might want the organizers to know about"
}
```

With that design in place, time to write my first test. I am going to
start with "get one talk for a user. As I always do, I start off with
a skeleton of a test with just enough code in it to run and generate
a failing response.

```php
<?php
declare(strict_types=1);

use App\Talk;
use App\User;
use Tests\TestCase;

class GetTalkViaApiTest extends TestCase
{
    /**
     * @test
     */
    public function correctlyRetrievesTestAssociatedWithUser(): void
    {
        /**
         * Create a test user
         * Create a talk associated with that user
         * Save the talk
         */

        /**
         * Make an API call to /api/talk/user/{id}
         */

        /**
         * Assert that the JSON response matches what was created
         */
        $this->assertTrue(false);
    }
}
```

Those three comment blocks represent the three parts of the [Arrange Act Assert pattern](http://wiki.c2.com/?ArrangeActAssert)
that I like to use to organize my tests. As always with my tests, I did some prototyping
to figure out the best way to do the assertions of the JSON data. Once I figured
out the approach I wanted to use, I was ready to start writing the test

So now I need to create a user for the test. I have another test that creates users
for testing purposes, so I will use that.

```php
$user = factory(User::class)->create();
```

Then I create a talk associated with that test
and save it to the database

```php
$talkDetails = [
    'title' => 'Test title',
    'description' => 'Test description',
    'other' => 'Test other'
];
$talk = new Talk(); 
$talk->user_id = $talkDetails['user_id'];
$talk->uuid = Uuid::uuid5(Uuid::NAMESPACE_DNS, 'php.net');
$talk->details = json_encode($talkDetails, JSON_THROW_ON_ERROR);
$talk->save();
```

Next, I have to make a call to my API end point as a user who
is authenticated, using a Laravel helper

```php
/**
 * Make an API call to /api/talk/{id}
 */
$response = $this->actingAs($user, 'api')->get('/api/talk/' . $talk->id);
```	

Without adding any more assertions, I run the test to verify that *just*
the assertion I have there is not passing. I am seeing a failure because
I have not configured the API route that I need.


According to the documentation, I can set up the route and do the work
inside the closure. In this case I want to make sure my query makes sure
to return a response only if the user associated with the talk is correct.

This is what my route looks like:

```php
Route::middleware('auth:api')->get('/talk/{id}', function (Request $request, $id) {
    return \App\Talk::find(['id' => $id, 'user_id' => $request->user()->id]);
});
```

With that done, I now can add in the code I am using for my assertions.

```php
/**
 * Assert that the JSON response matches what was created
 */
$response->assertOk();
$talkViaApi = json_decode($response->content(), true, 512, JSON_THROW_ON_ERROR);
$this->assertEquals($expected['id'], $talkViaApi[0]['id']);
$this->assertEquals($expected['user_id'], $talkViaApi[0]['user_id']);
$this->assertEquals($expected['uuid'], $talkViaApi[0]['uuid']);

/**
 * Because we cannot guarantee the order of things in JSON, we have to look
 * inside the details field a little differently
 */
$talkDetailsFromApi = json_decode($talkViaApi[0]['details'], true, 512, JSON_THROW_ON_ERROR);
foreach ($talkDetails as $field => $value) {
    $this->assertArrayHasKey($field, $talkDetailsFromApi);
    $this->assertEquals($talkDetailsFromApi[$field], $value);
}
```

This test passes! Next test I need is one where I create talk associated with one
user but pass in the wrong user id. It should return a 404 to let me know that it
could not find it.

The test initially looks like this:

```php
    public function cannotAccessATalkNotBelongingToADifferentUser(): void
    {
        /**
         * Create a test user
         * Create a talk associated with that user
         * Save the talk
         */
        $user = factory(User::class)->create();
        $this->be($user);

        $talkDetails = [
            'title' => 'Test title',
            'description' => 'Test description',
            'other' => 'Test other'
        ];
        $talk = new Talk();
        $talk->user_id = $user->id - 1;
        $talk->uuid = Uuid::uuid5(Uuid::NAMESPACE_DNS, 'php.net');
        $talk->details = json_encode($talkDetails, JSON_THROW_ON_ERROR);
        $talk->save();

        /**
         * Make an API call to /api/talk/1
         */
        $response = $this->actingAs($user, 'api')->get('/api/talk/' . $talk->id);

        /**
         * Assert that we are getting back a 404
         */
        $response->assertNotFound();
    }
```

The test fails, telling me I am getting a 200 instead of a 404. Time to back
to our route and write some logic to generate the 404. So the code inside the
closure now needs to look like this:


```php
Route::middleware('auth:api')->get('/talk/{id}', function (Request $request, $id) {
    $talk = Talk::find($id);

    if ((int)$talk->user_id === (int)$request->user()->id) {
        return $talk;
    }

    abort(404);
});
```

I run the tests...and it fails, complaining that it can't find things at index 0
in the array that has my results in it. I see that just returning the talk means
I get one record instead of an array that contains one record. I fix the test
to reflect that change and now I have a completely passing test.

Here is the entire test and I hope this
blog post helps you get better at testing the API's you are building.

```php
<?php
declare(strict_types=1);

use App\Talk;
use App\User;
use Ramsey\Uuid\Uuid;
use Tests\TestCase;

class GetTalkViaApiTest extends TestCase
{
    /**
     * @test
     */
    public function correctlyRetrievesTestAssociatedWithUser(): void
    {
        /**
         * Create a test user
         * Create a talk associated with that user
         * Save the talk
         */
        $user = factory(User::class)->create();
        $this->be($user);

        $talkDetails = [
            'title' => 'Test title',
            'description' => 'Test description',
            'other' => 'Test other'
        ];
        $talk = new Talk();
        $talk->user_id = $user->id;
        $talk->uuid = Uuid::uuid5(Uuid::NAMESPACE_DNS, 'php.net');
        $talk->details = json_encode($talkDetails, JSON_THROW_ON_ERROR);
        $talk->save();

        $expected = [
            'id' => $talk->id,
            'user_id' => $user->id,
            'uuid' => $talk->uuid->toString(),
            'details' => $talk->details,
        ];

        /**
         * Make an API call to /api/talk/{id}
         */
        $response = $this->actingAs($user, 'api')->get('/api/talk/' . $talk->id);

        /**
         * Assert that the JSON response matches what was created
         */
        $response->assertOk();
        $talkViaApi = json_decode($response->content(), true, 512, JSON_THROW_ON_ERROR);

        $this->assertEquals($expected['id'], $talkViaApi['id']);
        $this->assertEquals($expected['user_id'], $talkViaApi['user_id']);
        $this->assertEquals($expected['uuid'], $talkViaApi['uuid']);

        /**
         * Because we cannot guarantee the order of things in JSON, we have to look
         * inside the details field a little differently
         */
        $talkDetailsFromApi = json_decode($talkViaApi['details'], true, 512, JSON_THROW_ON_ERROR);
        foreach ($talkDetails as $field => $value) {
            $this->assertArrayHasKey($field, $talkDetailsFromApi);
            $this->assertEquals($talkDetailsFromApi[$field], $value);
        }
    }

    /**
     * @test
     */
    public function cannotAccessATalkNotBelongingToADifferentUser(): void
    {
        /**
         * Create a test user
         * Create a talk associated with that user
         * Save the talk
         */
        $user = factory(User::class)->create();
        $this->be($user);

        $talkDetails = [
            'title' => 'Test title',
            'description' => 'Test description',
            'other' => 'Test other'
        ];
        $talk = new Talk();
        $talk->user_id = $user->id - 1;
        $talk->uuid = Uuid::uuid5(Uuid::NAMESPACE_DNS, 'php.net');
        $talk->details = json_encode($talkDetails, JSON_THROW_ON_ERROR);
        $talk->save();

        /**
         * Make an API call to /api/talk/1
         */
        $response = $this->actingAs($user, 'api')->get('/api/talk/' . $talk->id);

        /**
         * Assert that we are getting back a 404
         */
        $response->assertNotFound();
    }
}
```

