<hr />

<p>layout: post</p>

<h2>title: More Awesome Deployment Using Phing</h2>

<p>In a previous blog post I showed some very basic techniques for using Phing for doing deployment.  As with any good deployment process, once you let people know that there is a flexible deployment tool other people start asking you to new things.</p>

<p>
"Hey Chris, can you set things up so that whenever someone runs one of those deployment scripts, we send an email to our support mailbox of all the files that were pushed up since the last deploy?"  After pausing for a minute to figure out how I would do it (the quick answer was "no idea") I said "Sure, I'll see what I can do."
</p>

<p>
So I stood in front of my whiteboard and proceeded to sketch out an idea of how I would want to do such a thing.  Luckily I could cheat and look at how Capistrano did it.  Because the way that they do it makes so much sense, so I duplicated it quick-and-dirty style.  A big thanks goes out to my friend <a href="http://www.derekmartin.ca/">Derek</a> for pointing me in the right direction.
</p>

<p>
So, the way to make this happen is as follows:
<ol>
<li>Read in a file inside your current deployment that contains the date of your last deployment</li>
<li>Check your code out of your version control system into <app ROOT>/releases/<date format></date></app></li>
<li>Symlink <app ROOT>/releases/<date format> to <app ROOT>/current</app></date></app></li>
<li>Write a file into <app ROOT>/current that contains the date of the last deploy.
</app></li></ol>
</p>

<p>Here's what it looks like as a Phing script</p>

<pre><code>&lt;?xml version="1.0"?&gt;                                                                               
&lt;project name="build" default="main"&gt;
    &lt;property name="exportdir"  value="/var/www/app" /&gt;
    &lt;property name="svnpath" value="/usr/bin/svn" /&gt;
    &lt;property name="repo" value="svn+ssh://path/to/repo" /&gt;
    &lt;tstamp&gt;
        &lt;format property="build.time" pattern="%Y%m%d%H%I%S" /&gt;
        &lt;format property="newdeploy.date" pattern="%Y-%m-%d %H:%I:%S" /&gt;
    &lt;/tstamp&gt;
    &lt;property file="${exportdir}/current/deploy.properties" /&gt;
    &lt;target name="main" depends="svnexport"&gt;
        &lt;exec command="rm -f ${exportdir}/current" escape="false" /&gt;
        &lt;exec command="ln -s ${exportdir}/releases/${build.time} ${exportdir}/current" escape="false" /&gt;
        &lt;exec command="chmod -R 777 ${exportdir}/current/tmp" escape="false" /&gt;
        &lt;exec command="chown -R scriptkiddie:company ${exportdir}/current" escape="false" /&gt;
        &lt;exec command="chmod -R ug+rw ${exportdir}/current" escape="false" /&gt;
        &lt;exec command="echo 'deploy.date=${newdeploy.date}' &gt;&gt; ${exportdir}/current/deploy.properties" /&gt;
        &lt;exec command="svn log -r '{${deploy.date}}':'{${newdeploy.date}}' ${repo} | mail -s 'Deployment Of FizzBuzz in production' support@fizzbuzz" /&gt;
    &lt;/exec&gt;&lt;/target&gt;
    &lt;target name="svnexport"&gt;
        &lt;svnexport svnpath="${svnpath}"
            username="${username}"
            password="${password}"
            nocache="true"
            repositoryurl="${repo}"
            todir="${exportdir}/releases/${build.time}"/&gt;
    &lt;/target&gt;
&lt;/project&gt;
</code></pre>

<p>The key was figuring out the best way to send SVN log information via email.  Leave it to the UNIX way of piping output to provide with the best solution totally within Phing.
</p>
