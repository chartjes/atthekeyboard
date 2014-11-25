---
layout: post
title: "Have A 'Strategy'" 
author: Chris Hartjes
date: 2012-05-15
comments: true 
sharing: true 
---
Yesterday on Twitter I got into a nerdfight with a blogger who posted a
blog entry about [using the Strategy pattern to avoid switch-case](http://odino.org/use-the-strategy-to-avoid-the-switch-case-antipattern/).
At first glance, I was convinced that he was using [variable variables](http://ca.php.net/manual/en/language.variables.variable.php)
which I feel is a big [code smell](https://en.wikipedia.org/wiki/Code_smell).

So natually I started arguing. I was rude. He was rude, slapping me in the twitter face with a blog link that seemed
to indicate he thought I had no idea about variables in PHP. To be fair I earned that by not paying attention.
Once I realized what he was doing, I realized that the Strategy pattern was applicable in this case...but his
chosen example was dumb and one that I wouldn't have used to demonstrate things. As expected, he told me to supply
a sample of a better way. I did, telling him that the sample would be better if he didn't mash the logging level
together with the message.

Anyway, I'm not going to do that bit of work here. Instead, I want to show that it is not enough to teach an idea
to someone. You have to show them how to test their idea once it's been refactored to make sure nothing has changed.
First, let's start with a working example of his logging class, using the dreaded switch-case.

~~~
<?php

// Sample logger class that uses a switch-case
require_once 'LoggingException.php';

class LoggerSwitch
{
	public function logMessage($message = "CRITICAL::The system encountered a problem")
	{
		$parts = explode('::', $message);
		$level = strtolower($parts[0]);
		$logMessage = $parts[1];

		switch ($level) {
			case 'notice':
				return $this->_writeToLog($level, $logMessage);
			case 'critical':
				$this->_writeToLog($level, $logMessage);
				return $this->_emailOps($message);
			case 'catastrophe':
				$this->_writeToLog($level, $logMessage);
				$this->_emailOps($logMessage);
				return $this->_textCeo($message);
		}

		throw new LoggingException("Unknown logging level {$level}");
	}

	protected function _writeToLog($level, $logMessage)
	{
		return "Wrote {$logMessage} to {$level} file";
	}

	protected function _emailOps($logMessage)
	{
		return "Sent email to Ops";
	}

	protected function _textCeo($message)
	{
		return "Sent text to CEO";
	}
}
~~~

Here's the tests for it

~~~
<?php

// PHPUnit test for logger-switch.php

require '../logger-switch.php';

class LoggerSwitchTest extends PHPUnit_Framework_TestCase
{
	public $logger;

	public function setUp()
	{
		$this->logger = new LoggerSwitch();
	}
	
	/**
	 * Data provider for testing our logging object
	 */
	public function loggerTestingScenarios() 
	{
		return array(
			array(
				'NOTICE::test notice message',
				'Wrote test notice message to notice file'
			),
			array(
				'CRITICAL::test notice message',
				'Sent email to Ops'
			),
			array(
				'CATASTROPHE::test notice message',
				'Sent text to CEO'
			),
		);
	}

	/**
	 * @test
	 * @dataProvider loggerTestingScenarios
	 * @param $message          string
	 * @param $expectedResponse string
	 */
	public function testReturnsExpectedResponseBasedOnMessage(
		$message,
		$expectedResponse)
	{
		$response = $this->logger->logMessage($message);
		$this->assertEquals(
			$expectedResponse,
			$response,
			'Did not get expected response'
		);
	}

	/**
	 * @test
	 */ 
	public function testThrowsExceptionWhenUnknownLevelSubmitted()
	{
		$this->setExpectedException('LoggingException', 'Unknown logging level test');
		$response = $this->logger->logMessage('TEST:: test message');
	}
}
~~~

These tests give us 100% test coverage, in case you were wondering.

Okay, so the next step is to refactor our incredibly stupid logging class to
use the [Strategy pattern](https://en.wikipedia.org/wiki/Strategy_pattern).

~~~
<?php

// Logging class that uses the strategy pattern
require_once 'LoggingException.php';

class LoggerStrategy
{
	public function logMessage($message = "CRITICAL::The system encountered a problem")
	{
		$parts = explode('::', $message);
		$level = strtolower($parts[0]);
		$method = sprintf('_log%sMessage', ucfirst($level));

		if (!method_exists($this, $method)) {
			throw new LoggingException('Unknown logging level ' . $level);
		}

		return $this->$method($parts[1]);
	}	

	protected function _logNoticeMessage($message)
	{
		return $this->_writeToLog('notice', $message);
	}

	protected function _logCriticalMessage($message)
	{
		$this->_writeToLog('critical', $message);
		return $this->_emailOps($message);	
	}

	protected function _logCatastropheMessage($message)
	{
		$this->_writeToLog('catastrophe', $message);
		$this->_emailOps($message);
		return $this->_textCeo($message);
	}

	protected function _writeToLog($level, $logMessage)
	{
		return "Wrote {$logMessage} to {$level} file";
	}

	protected function _emailOps($logMessage)
	{
		return "Sent email to Ops";
	}

	protected function _textCeo($message)
	{
		return "Sent text to CEO";
	}
}
~~~

The thing that lept out at me was that this refactor worked, but resulted in
more code. The good characteristic is all the small methods without lots of
code in them. Even better is that we don't have to change any of our tests,
everything worked the same! All we had to do was change which object we
were using for $this->logger.

If you want to see the code for this blog post for yourself (it does work
and run using PHPUnit) you can grab it from [Github](https://github.com/chartjes/blog-strategy).
Also, feel free to take a look at the [code coverage report](http://www.grumpy-testing.com/strategy/tests/coverage/)
as proof that I did cover everything I should have.

