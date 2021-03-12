---
title: 'introducing a ChangeSet class to CruiseControl.net: shifting to supertypes'
id: 29
date: 2009-07-08 22:41:33
updated: 2009-07-08 22:41:33
tags:
- CruiseControl.net
- 'c#'
- introducing ChangeSet
- refactoring
---

When I refactor, I try to get the first piece of the change in with absolute minimal impact on the surrounding code. Once that first bit of the change is in, it often feels like the rest of the code will change itself to adapt. Those changes turn out better than what I could have designed up front. Of course, the code isn't changing itself -- the rest of the team is making the changes. Getting a bit of the change in to the actual system is the clearest way I know of communicating the intention to the rest of the team. Then if my idea makes any sense, the team will get ideas and make changes of their own.

I am taking the same approach with this refactoring. As a first stage, introduce ChangeSet with minimal impact on the rest of the code. Give time for the team to see some of the possibilities. Not removing any existing classes at this point.

What makes this different from refactorings I have done at my day job is the number of people involved. [Steve Trefethen](http://www.stevetrefethen.com/blog/) reminded me of this on the [mailing list](http://groups.google.com.au/group/ccnet-devel/msg/da8e7de5cb0c2add?hl=en) today. I had not even considered that someone may have subclassed [Modification](http://bitbucket.org/davcamer/ccnet/src/0375206b36d0/project/core/sourcecontrol/Modification.cs) for their own purposes. But they have. Steve has been on the receiving end of [painful breaking changes](http://groups.google.com.au/group/ccnet-devel/browse_thread/thread/542ce5a39d5a9859/e0c1762e10d77313?hl=en&lnk=gst&q=strefethen+processexecutor#e0c1762e10d77313) in the past, so I want to listen carefully to his objections and hopefully minimise his pain. Users sticking with old versions complicates support. And, although Steve is only one user there are probably dozens or hundreds of users experiencing the same issues he is raising.

Now what can be done to avoid breaking other's code. This whole refactoring centres around one method on the [ISourceControl](http://bitbucket.org/davcamer/ccnet/src/tip/project/core/sourcecontrol/ISourceControl.cs) interface:
<pre lang="csharp">public interface ISourceControl
{
	Modification[] GetModifications(IIntegrationResult from, IIntegrationResult to);</pre>
All the Modification objects in the system are originally returned by a call to this method. The capability to return ChangeSets needs to be introduced at this point. But, changing all the existing implementers of ISourceControl to return ChangeSets at the same time would be too much work. It would also require a lot of users to change their code, because ISourceControl is one of the natural extension points of the system. Fortunately, .net is on my side with this one.

A quick digression to sing the praises of .net. Every single type in .net derives from [Object](http://msdn.microsoft.com/en-us/library/system.object.aspx), including all the apparently built-in types like [arrays](http://msdn.microsoft.com/en-us/library/system.array.aspx). The compiler does work some magic around descendants of [ValueType](http://msdn.microsoft.com/en-us/library/system.valuetype.aspx) to pass them on the stack. That way small types like [int](http://msdn.microsoft.com/en-us/library/system.int32.aspx), [bool](http://msdn.microsoft.com/en-us/library/system.boolean.aspx) and [float](http://msdn.microsoft.com/en-us/library/system.single.aspx) work faster than they would otherwise. You can create your own ValueTypes, passed on the stack, using the [struct](http://msdn.microsoft.com/en-us/library/ah19swz4.aspx) keyword. All of these still derive from Object.

But getting back to it, every last type is part of the same hierarchy including our Modification[] return type.

![screen-capture-1.png](http://intwoplacesatonce.com/wp-content/uploads/2009/07/screen-capture-1.png)

Changing the return type to one of the supertypes of Modification[] would allow us to introduce a new class. Besides changing the method signature, existing code could remain the same and continue to return Modification[].

Using the type closest to the base of the hierarchy provides the most flexibility for implementing another class. I know from [previous experimentation](http://www.markhneedham.com/blog/2009/06/12/coding-dojo-17-refactoring-cruise-control-net/) that we can use IEnumerable&lt;Modification&gt; for everything except the tests. Linq can cover the test requirements. We have not previously included Linq in the CruiseControl.net codebase and I thought it was because of Mono concerns. I've found out today that Linq is included in Mono, so I can use it to update the tests. Time to go make [these changes](http://bitbucket.org/davcamer/ccnet/changeset/c6ae81ea556f/), on a branch.

<pre lang="csharp">public interface ISourceControl
{
	IEnumerable<Modfication> GetModifications(IIntegrationResult from, IIntegrationResult to);</pre>
