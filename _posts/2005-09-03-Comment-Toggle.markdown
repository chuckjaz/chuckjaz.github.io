---
layout: post
title:  "Comment Toggle"
date:   2005-09-03 10:47:00
categories: Programming
---
When writing a prototype for a syntax highlighter,
more years ago than I care to admit, I came across an interesting property of
the two character comment delimiters found in the C family of
languages as well as Pascal. I have shown this to several people and many had never seen it before
so I thought I would share it more widely.

Take C# for example, you can quickly toggle between
two blocks of code by using the three characters `/*/`
in the middle. These three characters act as a comment toggle. If you are in a
block comment, it will terminate the comment, if not, it starts one. This combined
with the unconditional comment terminator sequence `/* */` which ensures that the characters follow are not in a block
comment, but is still a legal sequence of characters outside a comment, you can toggle between
blocks of code. For example,

{% highlight csharp %}
/* */
Console.WriteLine("Option 1");
/*/
Console.WriteLine("Option 2");
/* */
{% endhighlight %}

This will print “Option 1” to the console. The second
`WriteLine()` is commented out. With a simple
one character change, removing the trailing “/” in the first comment, will cause
“Option 2” to print instead.

{% highlight csharp %}
/* *
Console.WriteLine("Option 1");
/*/
Console.WriteLine("Option 2");
/* */
{% endhighlight %}

The first `WriteLine()` is commented out and the
second is now not. I use this sometimes to quickly switch between two optional
implementations of routines when I am rewriting code, or when constructing test
applications.

You can use the unconditional comment terminator to alternate between several
versions such as,

{% highlight csharp %}
/* */
Console.WriteLine("Option 1");
/* *
Console.WriteLine("Option 2");
/* *
Console.WriteLine("Option 3");
/* */
{% endhighlight %}

To select a different option you need to change two characters, such as removing the
trailing "/" of the first comment and adding one to the third, as in,


{% highlight csharp %}
/* *
Console.WriteLine("Option 1");
/* *
Console.WriteLine("Option 2");
/* */
Console.WriteLine("Option 3");
/* */
{% endhighlight %}

which will cause "Option 3" to print instead of "Option 1".

Pascal can do the same thing with the its block comment as
in,

{% highlight pascal %}
(* *)
Writeln('Option 1');
(*)
Writeln('Option 2');
(* *)
{% endhighlight %}

Note that some implementations of C and C++ support nested
comments and, therefore, the sequence `/*/`
always starts a comment instead of toggling, so this sequence is not as useful.

The reason this was important to my prototype is a bit hard
to explain and is not nearly as interesting as the toggle itself.
