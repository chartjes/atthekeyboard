<hr />

<p>layout: post</p>

<h2>title: Session handling</h2>

<p>At my day job, session handling was at one time the bane of our existance.  One poor server holding all the sessions for a large number of users.  We did lots of work to streamline them so that they take up far fewer resources than before, and are now going to be refactoring our code to remove all unnecessary info from the session.  Less data flowing back-and-forth, the better.</p>

<p>Over at <a href="http://www.littlehart.net/attheballpark">my other blog</a>, I was dealing with a session-related problem.  The sysadmin for the box had gotten sick of seeing all these files starting with ruby_ appearing in /tmp and told me to "fix it, damnit".  So off I went to the <a href="http://wiki.rubyonrails.com">Ruby wiki</a> and figured it out in short order.</p>

<p>So now, I have a Postgres-backed session handling option for the site...and no more files in /tmp that I have to go in and delete!</p>
