---
title: 'in C# 3.5: interface + extension methods = mixin'
id: 81
date: 2009-12-06 11:34:21
updated: 2009-12-06 11:34:21
tags:
- 'c#'
- duplication
- extension methods
- mixin
- partial classes
- refactoring
- svcutil
---

On my current project, we have ended up with several classes that have the same, or nearly the same fields. The classes are generated from xsds that describe a set of SOAP services that we integrate with. We have tried avoiding the generation or tweaking the xsds to avoid the situation, but accepting the duplicate classes actually seemed to be the best way forward. So, we have code in a generated file like this:

<pre lang="csharp">namespace ServiceClients.Generated
{
	public partial class Address1
	{
		public string StreetNumber { get; set; }
		public string StreetName { get; set; }
		public string Suburb { get; set; }
		public string State { get; set; }
		public string PostCode { get; set; }
	}

	public partial class Address2
	{
		public string StreetNumber { get; set; }
		public string StreetName { get; set; }
		public string Suburb { get; set; }
		public string State { get; set; }
		public string PostCode { get; set; }
	}
}</pre>

Besides the obvious problem with duplication, this code is also difficult to extend. As just one example, we wanted to display addresses in a one-line format:
<pre lang="csharp">var address = new Address1
                  {
                      StreetNumber = "12A",
                      StreetName = "Spencer Street",
                      Suburb = "Melbourne",
                      State = "VIC",
                      PostCode = "3000",
                  };
Assert.AreEqual("12A Spencer Street, Melbourne, VIC 3000",
                address.ToOneLineFormat());</pre>

The implementation for this method is fairly simple, but where can we implement it so that we will only need to write it once? Ideally what we would like is a [mixin](http://en.wikipedia.org/wiki/Mixin): a way of adding new methods to a class without adding any fields, and without necessarily changing the type. Although C# does not have a language facility for mixins, we can get a similar effect by using an interface and an extension method.

<pre lang="csharp">namespace ServiceClients.Generated.Extensions
{
	public interface IAddress
	{
		string StreetNumber { get; }
		string StreetName { get; }
		string Suburb { get; }
		string State { get; }
		string PostCode { get; }
	}

	public static class AddressExtensions
	{
		public static string ToOneLineFormat(this IAddress address)
		{
			const string format = "{0} {1}, {2}, {3} {4}";
			return string.Format(format,
					address.StreetNumber,
					address.StreetName,
					address.Suburb,
					address.State,
					address.PostCode);
		}
	}
}</pre>

There is one more step. In C# the two concrete types need to explicitly implement the `IAddress` interface so that we can use the `ToOneLineFormat` method on them. I've never had much use for partial classes, but they were a lifesaver in this case. In another file away from the mammoth 40,000 line long svcutil generated file, the interface can be easily added to both classes.

<pre lang="csharp">namespace ServiceClients.Generated
{
	public partial class Address1 : IAddress {}
	public partial class Address2 : IAddress {}
}</pre>

And there it is: a mixin! The ToOneLineFormat method is defined in one place, can be used with either Address class, and there is no need to change the generated code or the inheritance hierarchy.

For a time I was quite sure I had heard that methods implemented directly on interfaces would be part of C# 4\. I must have been delusional though, because it is not on the list of new features. If it were, it seems it would just be syntax sugar for the above approach.
