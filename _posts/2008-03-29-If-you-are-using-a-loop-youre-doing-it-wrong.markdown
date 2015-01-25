---
layout: post
title:  "If you are using a loop, you're doing it wrong"
date:   2008-03-29 16:55:00
categories: [Programming]
permalink: post/2008/03/29/If-you-are-using-a-loop-youre-doing-it-wrong.aspx
---
<div class="text"><p>"If you are using a loop, you're doing it wrong." </p>
<p>That is the advice one of my college professors told us when he was teaching us
<a href="http://en.wikipedia.org/wiki/APL_(programming_language)">APL</a>. APL
was designed to perform vector and matrix operations. Programming in APL is an
exercise in stringing operators together to perform strange and wonderful
things. Using a loop is just wrong and slows things down.</p>
<p>It is similar with LINQ, if you are using a loop you are doing it wrong. I
find myself
doing a lot of prototyping lately and I am forcing myself to use LINQ; not
because I don't like it, far from it, I really like LINQ, but using loops is so
ingrained into my psyche that I have to stop myself and force myself to think in
LINQ. Every time I am tempted to write a loop that involves a collection or an
array I ask myself, could I use LINQ instead? Programmers with more of a
database background seem to take to LINQ like a duck to water. They think in
sets and vectors, I don't, but I am getting there.</p>
<p>For example sometimes I find myself needing to return an <code>IEnumerable&lt;T&gt;</code> when
I have a <code>List&lt;T&gt;</code> but of a different <code>T</code>. This often happens when I want to keep
internal implementation details private. My internal <code>List&lt;T&gt;</code> might have the
actual class but I need to return an enumerable for some interface. Before I
would simply write a loop using the C# 2.0's <code><strong>yield</strong></code> syntax,</p>

  <pre class="code">List&lt;Foo&gt; foos = <strong>new</strong> List&lt;Foo&gt;();

...

<strong>public</strong> IEnumerable&lt;IFoo&gt; Foos {
    <strong>get</strong> {
        <strong>foreach</strong> (Foo f <strong>in</strong> foos)
            <strong>yield return</strong> f;
    }
}</pre>

<p>This loop involves a collection, can I use LINQ? Sure! By using LINQ's <code>Cast&lt;T&gt;()</code> method this can be replaced with,</p>

  <pre class="code"><strong>public</strong> IEnumerable&lt;IFoo&gt; Foos {
    <strong>get</strong> { <strong>return</strong> foos.Cast&lt;IFoo&gt;(); }
}</pre>

<p>If you are trying to find if a list contains an object by some name, you
could write a loop like,</p>
<pre class="code"><strong>public</strong> <strong>bool</strong> Contains(<strong>string</strong> value) {
    <strong>foreach</strong> (Foo foo <strong>in</strong> foos)
        <strong>if</strong> (foo.Name == value)
            <strong>return</strong> <strong>true</strong>;
    <strong>return</strong> <strong>false</strong>;
}
</pre>
<p>Using LINQ this might look like,</p>
<pre class="code"><strong>public</strong> <strong>bool</strong> Contains(<strong>string</strong> value) {
    <strong>return</strong> (<strong>from</strong> foo <strong>in</strong> foos <strong>where</strong> foo.Name == value <strong>select</strong> foo).Any();
}
</pre>
<p>A nice thing about LINQ is you can perform complicated queries in pieces.
With deferred execution of enumerators, this is fairly efficient as well. This
is really helpful in the debugger. In one chunk of code I was writing I needed to
coalesce adjacent ranges that are marked with the same text value. Assume you
have a structure called Range that looks like,</p>
<pre class="code"><strong>struct</strong> Range {
    <strong>public int</strong> Start;
    <strong>public int</strong> Length;
}</pre>
<p>and another <code><strong>struct</strong></code> that labels the ranges with names,</p>
<pre class="code"><strong>struct</strong> NamedRange {
    <strong>public string</strong> Name;
    <strong>public</strong> Range Range;
}
</pre>
<p>Now lets have a routine that calculates the range information over some
stream,</p>
<pre class="code"><strong>public</strong> IEnumerable&lt;NamedRange&gt; GetNamedRanges(Stream stream) {
    ...
}</pre>
<p>Lets assume that name ranges are some things like "whtespace", "reserved",
"identifier", "number" "string", etc. as you might expect to receive from a
lexical scanner like found in this
<a href="http://www.removingalldoubt.com/PermaLink.aspx/f615fab5-28e7-4560-8d9e-db90502a4c5e">
post</a>.</p>
<p>What I want to do with these ranges is to convert the names into styles such
as you might find referencing a CSS style sheet. So, in effect, I am mapping
NameRange values to StyledRange values where StyleRange would look something
like,</p>
<pre class="code"><strong>struct</strong> StyledRange {
    <strong>public</strong> Style Style;
    <strong>public</strong> Range Range;
}</pre>
<p>Lets create a dictionary that maps range names to styles such as,</p>
<pre cla="code">styleMap["number"] = <strong>new</strong> Style() { Name = "green" };
styleMap["reserved"] = <strong>new</strong> Style() { Name = "bold" };</pre>
<p>I only wanted to highlight numbers and reserved words, for everything
else I will use the default style,</p>
<pre class="code">defaultStyle = <strong>new</strong> Style { Name = "normal" };</pre>
<p>We can translate our named ranges directly into styled ranges by using a
LINQ query expression such as,</p>
<pre class="code"><strong>var</strong> ranges = GetNamedRanges(stream);
<strong>var</strong> styledRanges = <strong>from</strong> range <strong>in</strong> ranges
                  <strong> select new</strong> StyledRange() {
                       Style = styleMap.MapOrDefault(range.Name)
                       Range = range.Range
                   };
</pre>
<p>where <code>MapOrDefault()</code> is an extension method for <code>
IDictionary&lt;TKey, TValue&gt;</code> that looks like,</p>
<pre class="code"><strong>public static</strong> TValue MapOrDefault&lt;TKey, TValue&gt;(
      <strong>this</strong> IDictionary<TKey, TValue> dictionary, TKey key) {
    TValue result;
    <strong>if</strong> (dictionary.TryGetValue(key, <strong>out</strong> result))
        <strong>return</strong> result;
    <strong>return</strong> <strong>default</strong>(TValue);
}</pre>
<p>which is patterned after the existing LINQ methods for <code>IEnumerable&lt;T&gt;</code>,
<code>FirstOrDefault()</code> and <code>LastOrDefault()</code>.</p>
<p>Since many of the ranges that have different names will have the same style, it
would be nice to coalesce adjacent styles together so no two adjacent ranges
have the same style. In other words, we only want a new styled range when the
style changes. The above query expression just produces a one-to-one
mapping of named range to styled range. What we need is something that will
merge adjacent ranges. Do do this I will introduce another extension method,
<code>Reduce&lt;T&gt;()</code> for <code>IEnumerable&lt;T&gt;</code>,</p>
<pre class="code"><strong>public static</strong> IEnumerable&lt;T&gt; Reduce&lt;T&gt;(<strong>this</strong> IEnumerable&lt;T&gt; e,
    Func&lt;T, T, <strong>bool</strong>&gt; match,
    Func&lt;T, T, T&gt; reduce) {
    <strong>var</strong> en = e.GetEnumerator();
    T last;
    <strong>if</strong> (en.MoveNext()) {
        last = en.Current;
        <strong>while</strong> (en.MoveNext()) {
            <strong>if</strong> (!match(last, en.Current)) {
                <strong>yield return</strong> last;
                last = en.Current;
            }
            <strong>else</strong>
                last = reduce(last, en.Current);
        }
        <strong>yield return</strong> last;
    }
}</pre>
<p>What this method does is if two adjacent elements match (as defined by the <code>match</code>
delegate returning <code><strong>true</strong></code>) they will be reduced into one element by calling the <code>reduce</code>
delegate. For example, the <code>Sum&lt;T&gt;()</code>standard extension method could be
implemented using <code>Reduce&lt;T&gt;()</code> as,</p>
<pre class="code"><strong>var</strong> sum = numbers.Reduce((a, b) =&gt; true, (a, b) =&gt; a + b).First();</pre>
<p>Now that we have <code>Reduce&lt;T&gt;()</code>, lets reduce the list of styled
ranges to coalesce the adjacent ranges with the same style. This can be done by,</p>
<pre class="code">styledRanges = styledRanges.Reduce(
    (r1, r2) =&gt; r1.Style == r2.Style,
    (r1, r2) =&gt; <strong>new</strong> StyledRange() {
        Style = r1.Style,
        Range = MergeRanges(r1.Range, r2.Range)});</pre>
<p><code>MergeRanges()</code> referenced above, is,</p>
<pre class="code">Range MergeRanges(Range r1, Range r2) {
    <strong>return new</strong> Range() { Start = r1.Start, Length = r2.Start - r1.Start + r2.Length };
}</pre>
<p>In my example, this took 18 ranges for a typical line of C# source down to 7
ranges. Pretty good. But I noticed that some of those ranges were styling
whitespace as "normal". This seems like a waste; why switch back from
green to black text for writing whitespace? Why not combine those with
adjacent ranges instead? A simple approach to this is to add the following code
prior to mapping the styles,</p>
<pre class="code">ranges = ranges.Reduce(
    (r1, r2) =&gt; r1.Name == "whitespace" || r2.Name == "whitespace",
    (r1, r2) =&gt; <strong>new</strong> NamedRange() {
        Name = r1.Name == "whitespace" ? r2.Name : r1.Name,
        Range = MergeRanges(r1.Range, r2.Range)
    });</pre>
<p>This says to merge whitespace ranges with adjacent non-whitespace ranges.
This reduces the range count from 7 to 4. Not bad from the original 18 and that
was just for one line of source. This savings adds up quickly over an entire file.</p>
<p>The complete example, made as a function, looks like,</p>
<pre class="code">IEnumerable&lt;StyledRange&gt; StyleRanges(IEnumerable&lt;NamedRange&gt; ranges) {

    <em>// Merge whitespace ranges with adjacent non-whitespace ranges</em>
    ranges = ranges.Reduce(
        (r1, r2) =&gt; r1.Name == "whitespace" || r2.Name == "whitespace",
        (r1, r2) =&gt; new NamedRange() {
            Name = r1.Name == "whitespace" ? r2.Name : r1.Name,
            Range = MergeRanges(r1.Range, r2.Range)});

    <em>// Map named ranges to styles.</em>
    <strong>var</strong> styledRanges = <strong>from</strong> range <strong>in</strong> ranges
                       <strong>select new</strong> StyledRange() {
                           Style = styleMap.MapOrDefault(range.Name)
                           Range = range.Range };

    <em>// Merge adjacent ranges with the same style.</em>
    styledRanges = styledRanges.Reduce(
        (r1, r2) =&gt; r1.Style == r2.Style,
        (r1, r2) =&gt; <strong>new</strong> StyledRange() {
            Style = r1.Style,
            Range = MergeRanges(r1.Range, r2.Range)});

    <strong>return</strong> styledRanges;
}</pre>

<p>There are a few nice things about this function. First, it builds up an <code>
IEnumerable&lt;T&gt;</code> but this enumerable doesn't execute until one of its
enumerators is enumerated. This is due to the deferred execution of enumerators.
Second, even though deferred execution makes single step debugging challenging,
you can get a picture of what the function will do by using <code>ToArray()</code>
in the watch windows on the intermediate results. This allows you to inspect the
intermediate result to see if the mapping or reducing is what you expected.
Third, this routine has no loops. It operates on each enumerable as a set. Now
some of you will rightly say there is a loop buried in the <code>Reduce&lt;T&gt;()</code>
method. True; <code>Reduce&lt;T&gt;()</code> is just like many other LINQ extension
methods, they contain loops implied by the returned enumerator. But LINQ allows me to write code that communicates
what I am trying to do without it being obscured by the details of how it is done.
I think of LINQ not so much as a set of features in C# but a way of programming.
A way of programming that, if you are using loops, you are doing it wrong.</p>
</div>
