<hr />

<p>layout: post</p>

<h2>title: Looking Outside The Box</h2>

<p>As devoted as I am to the use of frameworks (although not THAT devoted that I would take the anti-lazy-programmer's route of porting an existing non-frameworked app to use a framework) I do enjoy seeing what frameworks in other languages are up to.  I'd like to share three with you that I've been following for a variety of reasons (mostly curiousity).</p>

<p>
I've mentioned <a href="http://erlyweb.org/">ErlyWeb</a> in this blog before.  Created by <a href="http://yarivsblog.com/">Yariv Sadin</a>, it's a web framework built to run on top of <a href="http://www.erlang.org/">Erlang</a>, which is a programming language created by Ericsson (the cellphone people) and open sourced a while back.  Why do I feel it's worth checking out?  Erlang is legendary in it's ability to supply a robust concurrent environment.  When you use it to run phone switches, it has to stay up!  99.999% uptime makes Erlang barely break a sweat.  Now, as web 2.0 morphs into Web Pi, there will be more demand for the type of interactive web apps that require statefull connections.  The most common "Ajax design pattern" that requires concurrency is <a href="http://en.wikipedia.org/wiki/Comet_(programming)">comet</a>, where you want to continually push information to the client without it having to request that information.  Given that much of the web is stateless, this is usually implemented as 'pull' with clients repeatedly polling data sources.  Anyway, that's a whole another discussion and I'd want to do lots more research before I get into that.  Anyway, Erlang is ideally suited for large-scale interactive applications that would be sending information back and forth.  Check out what Yariv's been talking about.</p>

<p>
My friend <a href="http://lazyweb.ca">Kevin</a> is a Perl hacker and he mentioned <a href="http://catalyst.perl.org">Catalyst</a> to me and then I heard a great podcast over at <a href="http://www.twit.tv/floss">FLOSS Weekly</a> about Catalyst.  Now, I'm not a Perl guy (I've used a little bit of it as part of the data-munging effort for the game the <a href="http://www.ibl.org">IBL</a> uses) but if you've made a big investment in Perl and want to move forward with some more modern web applications then I think Catalyst is a great option.  Listen to the podcast.
</p>

<p>
Finally, I took a very quick look at a web framework written in <a href="http://www.scala-lang.org">Scala</a> called <a href="http://liftweb.net/index.php/Main_Page">Lift</a>.  There's a <a href="http://http://radar.oreilly.com/archives/2007/05/liftscala_for_w.html">great write-up about it</a> by Tim O'Reilly, and I think I will look into it further.  Why should I care about Scala?  Well, it looks very similar to Ruby in it's syntax (and I have a smattering of Ruby experience) and it's multithreaded (which should theoretically scale better on a single machine I believe).  Besides, it never hurts to learn a new language.  Evolve or die, right?  I have a VPS that I can fool around on so why not see if Lift (and Scala) is worth investing some time into.
</p>
