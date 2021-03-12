---
title: the other thing about "using" blocks
id: 145
date: 2011-09-01 13:39:57
updated: 2011-09-01 13:39:57
tags:
- blocks
- 'c#'
- compiler
- dispose
- lean on the compiler
- object instantiation
- resource acquisition
- scope
- using
---

C#'s `using` statement is well recognized for one thing: calling dispose on objects so that you don't have to. That bit is wonderful:

<pre lang="csharp">string contents;
using (var f = File.OpenText("/path/to/file") {
	contents = f.ReadToEnd();
}</pre>

This is much simpler than the fully spelled out alternative:

<pre lang="csharp">string contents;
var f = File.OpenText("/path/to/file");
try {
	contents = f.ReadToEnd();
} finally {
	f.Dispose();
}</pre>

And even this longer form actually misses one of the more interesting aspects of the using statement…

<pre lang="csharp">string contents;
var f = File.OpenText("/path/to/file");
try {
	contents = f.ReadToEnd();
} finally {
	f.Dispose();
}
f.ReadToEnd(); // oops! ObjectDisposedException at runtime
f = null; // you could set it to null
f.ReadToEnd(); // but now you have a NullReferenceException, even more mysterious</pre>

The using statement on the other hand, creates a scope, so its variable can't be referenced at all after it is disposed:

<pre lang="csharp">string contents;
using (var f = File.OpenText("/path/ro/file") {
	contents = f.ReadToEnd();
}
f.ReadToEnd(); // unknown identifier error at compile-time</pre>

An entirely equivalent bit of code can be written, using an anonymous scope, but it starts to look quite baroque:

<pre lang="csharp">string contents;
{
	var f = File.OpenText("/path/ro/file");
	try {
		contents = f.ReadToEnd();
	} finally {
		f.Dispose();
	}
}
f.ReadToEnd(); // Unknown identifier at compile-time again, but 2x the lines and 2x the scopes!</pre>

using provides the try-finally-dispose structure, and also provides a scope. The scope means a whole class of errors where resources are accessed after being released is transformed from run-time to compile-time errors. Dealing with errors at compile-time is quicker, and with the right tools to highlight problems, the compile problems can be seen directly in the code as it is being edited.
