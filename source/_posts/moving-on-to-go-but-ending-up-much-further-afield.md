---
title: 'moving on to go, but ending up much further afield'
id: 90
date: 2010-01-15 22:51:11
updated: 2010-01-15 22:51:11
tags:
- broken code
- 'c#'
- code on bitbucket
- composition over inheritance
- duplication
- go
- refactoring
---

While I was preparing my last blog post about mixins in C#, I was also reading about go. From looking at go's syntax, I thought I would be able to replace the C# code one-for-one with go code and end up with a valid program. I thought [this would be the code](http://bitbucket.org/davcamer/csharp_to_go/changeset/cd1b05edfddb/):

<pre lang="none">// Not actually valid go!

package main

import "fmt"

type IAddress interface {
	StreetNumber string;
	StreetName string;
}
func (a IAddress) ToOneLineFormat() string {
	return a.StreetNumber() + " " + a.StreetName()
}

type Address1 struct {
	StreetNumber, StreetName string
}
type Address2 struct {
	StreetNumber, StreetName string
}

func main() {
	address1 := &Address1{"12A", "Spencer Street"};
	fmt.Println(address1.ToOneLineFormat());

	address2 := &Address2{"12A", "Spencer Street"};
	fmt.Println(address2.ToOneLineFormat())
}</pre>

I liked this code. It's slightly more lightweight than the equivalent C# because the interfaces don't need to be explicitly declared on the implementing classes. Otherwise it's quite similar. Declaring funcs away from types seemed a natural analogue to the interface + extension methods approach I described in the last post.

But this is not valid go code. Why not?

The first point is, that I've confused C#'s concept of properties with both fields and methods in my go code. The declarations in the structs can remain as fields, but the declarations in the interface must [change to be methods](http://bitbucket.org/davcamer/csharp_to_go/changeset/38a29a5faf45/). My interface needs to be:

<pre lang="none">type IAddress interface {
	StreetNumber() string;
	StreetName() string;
}</pre>

Now, to conform to the interface the two Address types need to have [methods that correspond to the interface](http://bitbucket.org/davcamer/csharp_to_go/changeset/512eecb12230/). Not fields.

<pre lang="none">type Address1 struct {
	streetNumber, streetName string
}
func (a Address1) StreetNumber() { return a.streetNumber }
func (a Address1) StreetName() { return a.streetName }
type Address2 struct {
	streetNumber, streetName string
}
func (a Address2) StreetNumber() { return a.streetNumber }
func (a Address2) StreetName() { return a.streetName }</pre>

Address1 and Address2 now both conform to the IAddress interface, though at the price of duplicate property/accessor/getter code. Accessors like this aren't particularly idiomatic for go, so there is no syntactic sugar to support them. Members are intended to either be fields, possibly public, or methods implementing significant behaviour.

The next problem arises because in go methods cannot be defined on interfaces. The syntax would seem to allow it, but it is simply illegal. The receive of a method must be a pointer to a named type or a named type itself. No interfaces. And also none of the familiar basic types like int, float and so on because they are unnamed types. A particular named type that aliases a basic type can have methods defined on it however. Coming back to this experiment, ToOneLineFormat needs to [have a concrete receiver](http://bitbucket.org/davcamer/csharp_to_go/changeset/c364216640fc/):

<pre lang="none">func (a Address1) ToOneLineFormat() string {
	return a.StreetNumber() + " " + a.StreetName()
}
func (a Address2) ToOneLineFormat() string {
	return a.StreetNumber() + " " + a.StreetName()
}</pre>

At this point I have brought back all the duplication that I was hoping to eliminate. On the up side, I have working go code.

Go has its own mechanism to reduce duplication. Its based on composing new types from existing types. A type can have an unnamed field of another type. The properties of the second, contained type can be accessed as if they were properties of the containing type. Address1 and Address2 could be [defined in terms of](http://bitbucket.org/davcamer/csharp_to_go/changeset/34e541d9a3cf/) a BaseAddress type.

<pre lang="none">type BaseAddress struct {
	streetNumber, streetName string
}
type Address1 struct {
	BaseAddress
}
type Address2 struct {
	BaseAddress
}</pre>

These new versions of Address1 and 2 will have exactly the same fields as the old type. An object composed like this can also receive methods as if it were an object of the anonymous field's type. This allows us to move the  ToOneLineFormat method on to BaseAddress directly. Also, since StreetNumber() and StreetName() simply return the value of fields which are available on BaseAddress, we can remove them. This in turn means IAddress is no longer useful. The complete code for Address1 and Address2 is significantly more compact. Note that the initialisation expression does need to change now, to recognise the anonymous BaseAddress field.

<pre lang="none">type BaseAddress struct {
	streetNumber, streetName string
}
func (BaseAddress a) ToOneLineFormat() {
	return a.streetNumber + " " + a.streetName
}
type Address1 struct {
	BaseAddress
}
type Address2 struct {
	BaseAddress
}

func main() {
	address1 := &Address1{BaseAddress{"12A", "Spencer Street", "Melbourne", "VIC", "3000"}};
	fmt.Println(address1.ToOneLineFormat());

	address2 := &Address2{BaseAddress{"12A", "Spencer Street", "Melbourne", "VIC", "3000"}};
	fmt.Println(address2.ToOneLineFormat())
}</pre>

Address1 and Address2 themselves are looking redundant now. Having a BaseAddress with two classes that "inherit" from it seems to clash strongly with the ideas of go. Based on this exercise, I believe an anonymous field still needs to capture some freestanding meaning of its own. The two types are a somewhat artificial constraint anyway. I'll leave them here, as they were the two classes that motivated this experiment originally.

Hopefully in go, you won't often end up in the same situation we faced in C#, needing two structurally identical types.
