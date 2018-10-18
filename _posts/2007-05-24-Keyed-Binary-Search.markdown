---
layout: post
title:  "Keyed Binary Search"
date:   2007-05-24 16:44:00
categories: [Programming]
---
<div class="text"><p>One thing that has always frustrated me about typical binary search routines
found in libraries is they always assume you can easily produce a copy of the
item you are searching for. What I mean is they typically follow this pattern,</p>
<pre><strong>int</strong> BinarySearch&lt;T&gt;(IList&lt;T&gt; list, T item);</pre>

<p>which takes a list of some type, left open here, and searches for it in the
list. This is great if the very next thing you do next is insert <code>item</code> into
the list. You already have <code>item</code> in hand so this signature is
perfect. But if you
don't have a ready <code>item</code>, and <code>item</code> is difficult to
produce, you are stuck. </p>
<p>This happened to me just recently in my code. I had a
sorted list of a data structure representing a span of source, similar to a <code>TextPointer</code> from WPF, called a
<code>TextRange</code>. My version can quickly return the
offset of the beginning of the span but, since <code>TextRange</code> tracks source changes,
creating a new <code>TextRange</code> is expensive. When text in the editor is
change, I receive the location of
the change in an event. How do I figure out which one of my <code>TextRange</code>s was modified? Since
the list is sorted, the obvious choice is to use a binary search which yields O(Log N)
performance. Since my <code>TextRange</code>s are store in a <code>List&lt;TextRange&gt;</code>,
obviously I should
call <code>List&lt;T&gt;.BinarySearch()</code>. Unfortunately, <code>TextRange</code>s are expensive to produce;
too expensive for a per-keystroke operation (I am glossing over some details for
the sake of brevity; you need to trust me this is true). I have one of two choices, write my
own <code>BinarySearch()</code> or create a fake <code>TextRange</code> that is a special
<code>TextRange</code> that
is less expensive to create and only used for searching. Neither choice is
pleasant. Hand-rolled binary searches are not hard to right but, for some
strange reason, I always get them wrong the first time.</p>
<p>What I needed is prebuilt and tested version of <code>BinarySearch()</code> that looks more like,</p>
<pre><strong>int</strong> BinarySearch&lt;T, K&gt;(IList&lt;T&gt; list, K key, Converter&lt;T, K&gt; converter);</pre>

<p>What this binary search routine does is find some key value in the list
given a function that can convert any list item into the key. In my example, I
would call it like,</p>
<pre><strong>int</strong> location = BinarySearch(textRangeList, location, TextRangeToLocation);</pre>

<p>where <code>TextRangeToLocation</code> looks something like,</p>
<pre><strong>int</strong> TextRangeToLocation(TextRange range) { <strong>return</strong> range.Location; }</pre>

<p>I don't need to produce a <code>TextRange</code>. I have the location I am looking for
handy, I just need to know what, if any, <code>TextRange</code> contains the
location. This method
would do just that. You can even use a lambda function in C# 3.0 to get this
all on one line,</p>
<pre><strong>int</strong> location = BinarySearch(textRangeList, location, (n) =&gt; n.Location);</pre>
<p>This is starting to look like a good idea. Let's code it up. Here is the
method I ended up with,</p>
<pre><b>public</b> <b>static</b> <b>int</b> BinarySearch&lt;T, K&gt;(IList&lt;T&gt; list, K value,
    Converter&lt;T, K&gt; convert, Comparison&lt;K&gt; compare) {
    <b>int</b> i = 0;
    <b>int</b> j = list.Count - 1;
    <b>while</b> (i &lt;= j) {
        <b>int</b> m = i + (j - i) / 2;
        <b>int</b> c = compare(convert(list[m]), value);
        <b>if</b> (c == 0)
            <b>return</b> m;
        <b>if</b> (c &lt; 0)
            i = m + 1;
        <b>else</b>
            j = m - 1;
    }
    <b>return</b> ~i;
}</pre>


<p>I added an extra parameter, <code>compare</code>. This is required because I
didn't put any limits on what <code>K</code> can be and I need a way to compare
one <code>K</code> with another. Many types, such as <code>int</code>, already know how to
compare values of their own type. Typically types that are comparable, such as
<code>int</code>,<code> string</code>, etc., implement
<code>IComparable&lt;T&gt;</code>. We can accommodate such types by creating an
overloaded version of <code>BinarySearch()</code> that looks like, </p>
<pre><b>public</b> <b>static</b> <b>int</b> BinarySearch&lt;T, K&gt;(IList&lt;T&gt; list, K value,
    Converter&lt;T, K&gt; convert) <strong>where</strong> K : IComparable&lt;K&gt; {
    <b>return</b> list.BinarySearch(value, convert, (n, m) =&gt; n.CompareTo(m));
}</pre>


<p>which uses a lambda function to create a <code>Comparison&lt;T&gt;</code>  for any
type that supports <code>IComparable&lt;T&gt;</code>. Beware that this doesn't
work well with reference types because it doesn't check for <strong><code>null</code></strong>.
You can make it safer by putting <strong><code>struct</code></strong> in the
<strong><code>where</code></strong> clause but that seemed overly restrictive to
me.</p>
<p>Now that I have a <code>BinarySearch()</code> that meets my needs better than the built-in
<code>BinarySearch()</code>, my next thought was I should evangelize the benefits my version
of <code>BinarySearch()</code> to the team responsible for maintaining the CLR
collections and, maybe, someday, I will have it as
a native part of the CLR; maybe, someday. Then I remember extensions methods. In
C# 3.0, with a slight modification to the above methods, they can appear as if
they were already part of the CLR.</p>
<pre><b>public</b> <b>static</b> <b>class</b> ListExtensionMethods {
    <b>public</b> <b>static</b> <b>int</b> BinarySearch&lt;T, K&gt;(<b>this</b> IList&lt;T&gt; list, K value,
        Converter&lt;T, K&gt; convert, Comparison&lt;K&gt; compare) {
        <b>int</b> i = 0;
        <b>int</b> j = list.Count - 1;
        <b>while</b> (i &lt;= j) {
            <b>int</b> m = i + (j - i) / 2;
            <b>int</b> c = compare(convert(list[m]), value);
            <b>if</b> (c == 0)
                <b>return</b> m;
            <b>if</b> (c &lt; 0)
                i = m + 1;
            <b>else</b>
                j = m - 1;
        }
        <b>return</b> ~i;
    }

    <b>public</b> <b>static</b> <b>int</b> BinarySearch&lt;T, K&gt;(<b>this</b> IList&lt;T&gt; list, K value,
        Converter&lt;T, K&gt; convert) where K : IComparable&lt;K&gt; {
        <b>return</b> list.BinarySearch(value, convert, (n, m) =&gt; n.CompareTo(m));
    }
}</pre>
<p>My version of <code>BinarySearch()</code> now
appears as part of the methods of any type that implements <code>IList&lt;T&gt;</code>.
This allows us to write,</p>
<pre><strong>int</strong> location = textRangeList.BinarySearch(location, (n) =&gt; n.Location);</pre>
<p>just like it was a native part of the CLR. No more hand-rolled <code>
BinarySearch()</code>s!</p>
<p>Now that we have a <code>ListExtensionMethods </code>class, what other missing methods
could we add? Here are a few that I thought would be useful,</p>
<pre><b>public</b> <b>static</b> <b>void</b> Sort&lt;T, K&gt;(<b>this</b> List&lt;T&gt; list,
    Converter&lt;T, K&gt; coverter, Comparison&lt;K&gt; compare) {
    list.Sort((n, m) =&gt; compare(convert(n), convert(m)));
}

<b>public</b> <b>static</b> <b>void</b> Sort&lt;T, K&gt;(<b>this</b> List&lt;T&gt; list,
    Converter&lt;T, K&gt;  converter) <b>where</b> K : IComparable&lt;K&gt; {
    list.Sort((n, m) =&gt; convert(n).CompareTo(convert(m)));
}

<b>public</b> <b>static</b> <b>void</b> Swap&lt;T&gt;(<b>this</b> IList&lt;T&gt; list, <b>int</b> index1,
    <b>int</b> index2) {
    var value = list[index1];
    list[index1] = list[index2];
    list[index2] = value;
}

<b>public</b> <b>static</b> <b>void</b> Shuffle&lt;T&gt;(<b>this</b> IList&lt;T&gt; list) {
    <b>int</b> count = list.Count;
    Random r = <b>new</b> Random();
    <b>for</b>(<b>int</b> i = count - 1; i &gt; 1; i--)
        list.Swap(i, r.Next(i - 1));
}</pre>
<p>The <code>Sort()</code> methods above are allow you to extract a key for
sorting using the same methods as above. These are not strictly necessary since
constructing the lambda function I use is not that complicated but I believe they
make the code more readable if you are consistent in how you extract keys from a
list, so these are convenient wrappers that allow you to use the same methods to
sort and search. <code>Swap()</code> and
<code>Shuffle()</code> are throw-ins. They are really only useful for demo code
or, in my case, test code. I used them to test the <code>Sort()</code> method
which I used to test the <code>BinarySearch()</code> method.</p>
</div>
