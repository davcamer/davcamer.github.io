---
title: sleeping in batch files
id: 73
date: 2009-11-22 22:48:52
updated: 2009-11-22 22:48:52
tags:
- CruiseControl.net
- automation
- batch files
- cmd scripts
- debugging
---

I was recently debugging a situation involving CruiseControl.net where one [&lt;exec&gt; task](http://confluence.public.thoughtworks.org/display/CCNET/Executable+Task) would not complete until a second &lt;exec&gt; task in an unrelated project completed.

When it was originally [described on the mailing list](http://groups.google.com/group/ccnet-devel/browse_thread/thread/fdbc5228726a122d), I thought it might be some interaction between batch files and the parallel task in the [ProcessExecutor class](http://ccnet.svn.sourceforge.net/viewvc/ccnet/trunk/project/core/util/ProcessExecutor.cs?revision=6352&view=markup). ProcessExecutor encapsulates the concurrency aspect of running an external process and if there is a concurrency problem with processes, it's probably in that class.

In any case, at that point I was seriously misunderstanding the problem, but it led me to some interesting debugging. For the debugging, I wanted to create a few long running batch files to run in parallel. But what is the batch equivalent of Thread.Sleep?

The `pause` command came to mind. It has no capabilities beyond displaying a "press any key" prompt and waiting for input. No parameters or switches at all. There is no [`sleep`](http://bashcurescancer.com/man/cmd/sleep) command, as there is in bash. `wait` came to mind, but it is not a command either. I went to google...

And met [with success](http://www.velocityreviews.com/forums/t172496-bat-file-pauses.html). The `choice` command can be used as a timed delay. The forum I found it in made it sound as if  `choice` is not available on every platform, but my Windows Server 2003 development image has it. The exact syntax of the command does seem to vary between versions, as some example on the web do not work on my system. For me, a command to wait five seconds is

<pre lang="cmd">choice /M:"Waiting for 5 seconds" /T:5 /D:Y /C:Y</pre>

A simple one-line batch file that waits for a given number of seconds is

<pre lang="cmd">@choice /M:"Waiting for %1 seconds" /T:%1 /D:Y /C:Y</pre>

As with many command-prompt commands, you can see more information about the command including the exact syntax for your version with

<pre lang="cmd">choice /?</pre>

If you are writing a cmd or bat script and need to wait or sleep for a short time, `choice` is well, choice.
