<hr />

<p>layout: post</p>

<h2>title: Pretty Please, Make Your Code Testable</h2>

<p>A lot of open source applications suffer from the twin demons of lack of extensive documentation and not enough tests.  Yes, I've mentioned many times in this blog how important I feel having tests built right into your application is.  Not only do you approach creating brand-new functionality differently when you write the tests before you write a line of code, you make it incredibly easy to extend existing code because when you change something, all you have to do is run the tests again.  CakePHP is aiming to have 100% coverage for tests.  It's a noble goal because I've come around to the idea that if you can't test your code automatically, you shouldn't be writing your code that way.</p>

<p>
Anyhow, back on topic.  I've been trying to write some tests for a component that is, well, a little alpha at this point.  Digging into the code has revealed that it is hopelessly tied into another component.  This is only a bad thing because it makes isolating the original component for testing purposes pretty much a hopeless task that I would delegate to my minions if I had any.  Which I don't.  Not yet anyway.</p>

<p>
I am by no means an expert tester as I have been known to be quite hypocritical at times with my own code in that I fail to create tests to verify that the crap I'm actually writing is working the way that it is supposed to.  But I can say that when I was working on the IPTV project I wrote all sorts of tests for the command-line scripts that I wrote and that the coverage was good.  I can quibble with CakePHP's choice of using <a href="http://www.lastcraft.com/simple_test.php">SimpleTest</a> for the built-in testing suite (I'm a <a href="http://www.phpunit.de">PHPUnit</a> guy myself) but at least they have testing built right in.  Bake even creates the shells of the tests for you.  But anyway, I am straying once again off-topic.</p>

<p>
What does code that is easy to test look like?  I think it boils down to two ideas:
<ul>
<li>It isn't tied into another component for it's existence</li>
<li>The code itself uses method, function and variable names that mean you don't have to actually read the comments to figure out what's going on</li>
</ul>
</p>

<p><i>It isn't tied into another component for it's existence</i>.  This is the number one problem I see in a lot of code that needs to be tested.  Component A relies on functions inside component B...and it's impossible to pretend to send info from component B to component A.  There's a concept I learned from my favourite programming book "The Pragmatic Programmer" called the <a href="http://en.wikipedia.org/wiki/Law_of_Demeter">Law of Demeter</a>.  Basically, it's the idea that a an object should assume as little as possible about the other objects that it has to interact with you.  The downside (as mentioned in the Wikipedia entry) is that you end up having to write a lot of wrapper code so you can keep those objects from knowing too much about each other.  But the upshot is that you will have to learn the awesome testing concept known as Mock Objects in order to make your tests work.  Here's an example of what I'm talking about:</p>

<p>
We have object A that uses object B, which is a database abstraction component.  Now, as long as Object A doesn't use Object B to access methods or information from Object C (think of object chaining: $this->A->B->foo() is bad for testing) then you can easily test Object A by creating a mock object that represents Object B (and controlling the responses from Object B under testing).  Otherwise, how can you simulate the results of your call to $this->A->B->foo()?  You can't.
</p>

<p><i>The code itself uses method, function and variable names that mean you don't have to actually read the comments to figure out what's going on</i>.  I'm of the programming school where you comment WHY you did something as opposed to WHAT you actually did.  If you're doing some weird of bit shifting followed by a little-used algorithm for figuring out the best path for travelling along 7 points, well, I expect the comments to say WHY you're doing what you've done and it will be up to me to decipher the code itself so I understand HOW you did it.  Luckily for me, CakePHP is filled with lots of well-named methods and variables and I'd like to think I've got enough programming experience that I can figure out HOW something is done but I often want to know WHY we're doing that.  I've had several Skype conversations with my fellow CakePHP developers asking why something was being done so I could figure out how to modify it without breaking stuff.  I added functionality to the <a href="http://api.cakephp.org/1.2/class_email_component.html">email  component</a> so it would actually send out stuff via SMTP (it currently doesn't do it) and found that the code itself was pretty readable and I didn't require a lot of extra work to figure out how to extend it the way I wanted to go.  That, my friends, is a good example of code that is easy to read and easy to understand.</p>

<p>Phew, that's a lot of stuff to dump out of my brain into the blog late on a Friday night.  So, to bring things around full circle I think it's very important to keep in mind that when you are writing code you also need to be thinking about how you would actually test that code.  Even more importantly, you need to think about what happens if someone other than you has to come along and has to change anything.  When your contributing to a project with the impact like CakePHP can have, well, you better damn well make sure that if you never contribute another line of code to the project that other people can take what you've done and keep building on it.  It's fun to do all sorts of fun tricks in code (some of the stuff I saw during a job interview once reminded me of why I don't like cute code tricks) when it's just your own stuff.  But to preserve my sanity, please make your code testable so that when I have to go and add something else to it I don't start screaming your name and pestering you via instant message, shaking a virtual fist at you.</p>
