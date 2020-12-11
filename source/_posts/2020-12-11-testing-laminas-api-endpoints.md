---
layout: post
title: "Testing Laminas API end points"
author: Chris Hartjes
date: 2020-12-11
---

## Testing Laminas API end points 

As I write this, I have been working for Unnamed Financial Services Company
for 3 months. It has been a good exercise in figuring out how to do a slow-but-steady
migration of a CodeIgniter 3.x application to a macro-service where we
are moving some things to an API and will move the other functionality
(which is mostly used by the customer service folks at UFSC) into a
different application.

For the API side of things I decided we should go with [Laminas API tools](https://api-tools.getlaminas.org/)
due to tooling and the flexibility that we can get from writing our own
glue code to solve particular problems. As much as I am an old
veteran of full-stack PHP frameworks, our architectural plans leave me
worried that I would end up fighting with the conventions of one of
those types of frameworks too much.

So, having used the admin UI to create an API end point, and organizing
our code that interacts with the database into some abstractions that
I think make sense and provide us with some much-needed structure,
I have a single-action controller that handles the API call:

```php
<?php
declare(strict_types=1);

namespace Lead\V1\Rpc\FindById;

use Application\Repository\DoctrineLeadRepository;
use Doctrine\ORM\EntityManagerInterface;
use Laminas\Mvc\Controller\AbstractActionController;
use Laminas\ServiceManager\ServiceLocatorInterface;

final class FindByIdController extends AbstractActionController
{
	private EntityManagerInterface $entityManager;

	public function __construct(ServiceLocatorInterface $serviceLocator)
	{
		$this->entityManager = $serviceLocator->get('doctrine.entitymanager.orm_default');
	}

	public function findByIdAction(): array
	{
		$leadId = (int) $this->getRequest()->getQuery('id');
		$lead = (new DoctrineLeadRepository($this->entityManager))->findById($leadId);

		if ($lead !== null) {
			return $lead->toArray();
		}

		return [];
	}
}
```

Now the discussion worth having: how do we test this thing?

For pragmatic reasons I decided that I would not go the path of creating test doubles
for everything and then using the [Laminas service manager](https://docs.laminas.dev/laminas-servicemanager/quick-start/)
to replace the existing dependencies with doubles. I'll just use the real database
that the `DoctrineLeadRepository` will be talking to and get on with a test.

So, we have two scenarios that we need to test:

* Does it behave correctly if it cannot find a lead in the database
* Does it behave correctly if it finds a lead and returns some response

Okay, so I get started with my test. The first step is creating a skeleton
that reads in the Laminas-specific configuration options for the main
Application module and then the module that contains our API controller action.

```php
<?php

namespace ApplicationTest\Lead\V1\Rpc\FindById;

use Laminas\Stdlib\ArrayUtils;
use Laminas\Test\PHPUnit\Controller\AbstractHttpControllerTestCase;

final class FindByIdControllerTest extends AbstractHttpControllerTestCase
{
	private \Laminas\ServiceManager\ServiceManager $serviceLocator;

	protected function setUp(): void
	{
		$this->setApplicationConfig(ArrayUtils::merge(
			include __DIR__ . '/../../../../../../../config/application.config.php',
			include __DIR__ . '/../../../../../../../module/Lead/config/module.config.php',
		));
		parent::setUp();
		$this->serviceLocator = $this->getApplicationServiceLocator();
	}
}
```

Okay, now a test for the first scenario:

```php
<?php

	// Addtional dependency...
	use Lead\V1\Rpc\FindById\FindByIdController;


	/** @test  */
	public function it_handles_missing_lead_correctly(): void
	{
		$response = (new FindByIdController($this->serviceLocator))->findByIdAction();
		self::assertEquals([], $response);
	}
```

This one passes since the code in the controller action behaves as follows:

* there is no value in the query for 'id'
* so when it tries to retrieve a `Lead` it will get back null
* so it returns an empty array

The next scenario was trickier. It was not obvious to me how I inject a
query parameter into the request. I was used to other frameworks where
I could add a parameter to the action of the controller and the framework
would automagically inject that corresponding HTTP query parameter.

Using an old tactic, I started digging around in the tests for the
[Laminas MVC package](https://github.com/laminas/laminas-mvc) to see
how *they* were testing things. It took a while and some trial and
error, but I did figure out. 

```php

// New dependencies added
use Laminas\Http\Request;
use Laminas\Mvc\MvcEvent;
use Laminas\Router\RouteMatch;
use Laminas\Stdlib\Parameters;


	/** @test */
	public function it_finds_something_that_looks_like_a_lead(): void
	{
		$controller = new FindByIdController($this->serviceLocator);

		// Create what route we want to execute
		$routeMatchParams = [
			'controller' => 'Lead\\V1\\Rpc\\FindById\\Controller',
			'action' => 'findById'
		];
		$routeMatch = new RouteMatch($routeMatchParams);

		// Build up the request that contains our lead ID
		$request = new Request();
		$request->setQuery(new Parameters(['id' => 2]));
		$request->setMethod('GET');
		$request->setUri('/lead/find');

		// Create an event that the app is listening for
		// and tell the controller to use it
		$event = new MvcEvent();
		$event->setRouteMatch($routeMatch);
		$event->setRequest($request);
		$controller->setEvent($event);

		// Get our response
		$result = $controller->dispatch($request);

		// Make sure that it is actually a lead
		self::assertEquals(2, $result['id']);
	}
```

While the test passes, in the next version I want to create a `Lead`
as part of the test, store it in the database, and then make sure
I retrieve the one I expect. Hard-coding is okay for the first
pass but should not be in the final version.

Hopefully this blog post helps you solve your own Laminas-related
problems faster than I did. ;)
