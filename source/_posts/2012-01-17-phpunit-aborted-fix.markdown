---
layout: post
title: "PHPUnit aborted fix" 
author: Chris Hartjes
date: 2012-01-17
comments: true 
sharing: true 
---
A while back I ran into a problem on a server where PHPUnit kept [reporting
"aborted" no matter what tests were run](http://stackoverflow.com/questions/3065563/phpunit-reporting-aborted-no-matter-what-tests-are-run/8898712). 
That was a pretty annoying bug. I never did find out what the problem was
as I moved onto other problems and chalked that error up to some undiagnosed
weirdness on that particular server.

From time to time I would get asked on Twitter if I had ever solved the 
problem. My answer was always "no, and if *you* do solve it please let
met know how you fixed it." Today, my friends, was the day.

![](/images/phpunit-bug.png %)

Kudos to [Damian Katz](https://twitter.com/demiankatz) for coming up with a
solution. Without digging deeper into it, I would imagine that you could
alter the phpunit.xml file (if you are using one in your setup) to automatically
use the -dzend.enable\_gc=0 option every time you run a test.

How did he figure it out? He tracked down [this bug](https://bugs.php.net/bug.php?id=53976) that
seemed to be displaying the exact behaviour that I had reported way back when.
It seems odd that garbage collection would cause this problem, but perhaps it
was a perfect storm of lots of small tests causing over-active garbage collection
which in turn causes the PHP interpreter to abort whatever it is running.

This is another case where I would love someone with knowledge of PHP internals
to try and explain why it is happening.
