---
title: 'compiler created modules in F#'
id: 9
date: 2009-05-18 08:53:01
tags:
- 'f#'
---

Lately I've been learning F#. I have used SML before for little experiments. SML is one of F#'s parents, and a lot of the basic syntax is familiar. The things I've been learning and discovering the most about are the large-scale issues. How does C# call F#? F# call C#? Or even, F# call F#? I wasn't expecting that last one to be a challenge, but it was. Here's why...

I started in the way I start almost any project: code in one place, tests in another. My first goal is to get the tests calling the code. This means a trivial test like making sure that a function returns true, when it... returns true.

![screen-capture-4.png](http://twoplaces.files.wordpress.com/2009/05/screen-capture-4.png)

This is about the simplest F# solution I can picture. I'm already stumbling though, as evidenced by the red squiggly underneath always_true in TestsFile.fs. That squiggle is for a "The value or constructor 'always_true' is not defined." error. But it is defined, over in CodeFile.fs, and I have no idea how to be more specific because all my declarations are top-level. I haven't had to add any namespace or type names, which is nice. But as a result, I don't know which namespaces or types these items are part of.

Opening the assemblies in reflector might help:

![screen-capture-1.png](http://twoplaces.files.wordpress.com/2009/05/screen-capture-1.png)

There it is! My top-level declarations in CodeFile.fs are part of the CodeFile type in no namespace. always_true is there as a public static method, and my_true is there as a get-only public static property. There are three other internal types in there that have been created by the compiler.

F# will provide this automatic module even if an explicit module is declared in the file. For example, you might think that since always_true and my_true ended up as part of the CodeFile type, that the code in the screenshot above is equivalent to
<pre lang="ocaml">module CodeFile =
    let my_true = true
    let always_true () = my_true</pre>
Just to check, I'll compile that and open it up in Reflector again.

![screen-capture-2.png](http://twoplaces.files.wordpress.com/2009/05/screen-capture-2.png)

Instead of overwriting the automatic module, the explicitly declared module is nested inside of the automatic module. The public static type CodeFile, has a nested public static type CodeFile+CodeFile. Not what I was expecting.

The automatic module will continue to be produced unless I add a namespace declaration.
<pre lang="ocaml">namespace Code

module CodeFile =
    let my_true = true
    let always_true () = my_true</pre>
This ends up with a structure more familiar from C# work. CodeFile is a type within the Code namespace.

![screen-capture-3.png](http://twoplaces.files.wordpress.com/2009/05/screen-capture-3.png)

I now have a pretty good picture of how the automatic naming works, but there are still scenarios I'd like to try. First, what happens if I declare a namespace but don't include an explicit module declaration? I have a hunch, but I'll try it to confirm.

This surprised me! This is a compile-time error. There is no automatic module creation to make this work.
<pre lang="ocaml">namespace Code

let my_true = true // compile-time error: "Namespaces may not contain values.
                   // Consider using a module to hold your value declarations."

module CodeFile =
    let always_true () = my_true // ok</pre>
The second question I had in mind was about mixing a module declaration and top-level declarations in one file. This has been resolved by the previous findings, though. If my file has a namespace declaration, the top-level declarations will be illegal. If it does not have a namespace declaration, then an automatic module with the name of the file will be created.

This is a simpler result than I was expecting, because it means there is only a single rule at play:

1.  If an .fs file does not include a namespace declaration, a module will be created in the top-most namespace to contain the file's declarations.
Knowing this, I can return my code under test to how it looked before and get my tests working.

In CodeFile.fs, in the Code project:
<pre lang="ocaml">#light

let always_true () = true</pre>
In TestsFile.fs, in the Tests project:
<pre lang="ocaml">#light

open Xunit

[<Fact>] let true_is_true () = Assert.True(CodeFile.always_true ())</pre>
