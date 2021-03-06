<hr />

<p>layout: post</p>

<h2>title: Extraction vs. Design</h2>

<p>For those not familiar with the history of the <a href="http://www.rubyonrails.org">Ruby on Rails</a> framework, it's interesting (well, at least to me) to note that the framework grew from extractions from working on <a href="http://www.basecamphq.com/">Basecamp</a>, an online project management / collaboration tool.  Basically, they took functionality from Basecamp (the "extraction") and used it as components for Ruby on Rails.  This is probably a gross oversimplification, but this is not a blog posting about how Ruby on Rails was created.</p>

<p>
<a href="http://www.loudthinking.com">David Heinemeier Hansson</a> (the driving force behind Ruby on Rails) is a huge believer in using extractions as a method to add functionality to your projects, typically in the context of a framework.  You need some particular functionality, you write it, and if it's good enough you extract it from it's original context and make it generic enough that it can be used by other people.  You could also call this "building stuff you need" instead of "building stuff you think you might need".  It's part of the whole <a href="http://en.wikipedia.org/wiki/YAGNI">YAGNI</a> philosophy of programming, I guess.</p>

<p>
So, the flipside to this idea of how you get new features is doing it by design.  This is where you sit down and decide "I'm going to write a component to interface with Frammastat's Whoozinator API because I think some people would like to have that".  So, you sit down and you bang out the code so that users of GrumpyFramework can now talk to Frammastat's Whoozinator via a component instead of writing their own.  It's like what I did when I helped write code for Zend Framework to talk to Last.fm's Audioscrobbler service.  I didn't need it, but a friend thought it was a good idea and it was a chance to learn how to use Zend Framework.  I received a patch request for it the other day, so maybe I should go and take a look at it... :)</p>

<p>
Anyway, where was I?  Oh yeah</p>

<p>
In this case, a decision was made to build something specifically for a project, whether it was required as part of my current work or not.  Did I need that Frammastat interface for GrumpyFramework?  No, but somebody else might.  And that, in my opinion, leads down the road to premature optimization and feature creep.  Just because you *might* need something, doesn't mean you should spend the time and effort to write some code for it.  Often times, design makes sense when you know that the vast majority of users of your project will need something.  But for a lot of things, wait until you actually need them.</p>

<p>
I have found that the best bits and pieces of code that I have used with <a href="http://www.cakephp.org">CakePHP</a> have come from people extracting ideas they were using for their own personal projects and then putting a good spin on it so others could use it. Of course, it helps that CakePHP lets you create components and behaviors (that talk to controllers and models respectively) so that you can extend functionality without rewriting any of the core, but I remain firmly convinced that the extraction of ideas from current projects is the best method of adding new functionality to an existing application.
</p>
