---
layout: post
title:  "Simulating INumeric with policies"
date:   2009-03-06 19:22:00
categories: [Programming]
---
<div class="text"><p>After reading <a href="http://blogs.msdn.com/lucabol">Luca Bolognese's</a>
blog post regarding using
<a href="http://blogs.msdn.com/lucabol/archive/2009/02/05/simulating-inumeric-with-dynamic-in-c-4-0.aspx">
C# 4.0's dynamic keyword to simulate INumeric</a> I immediately thought that must be a way to express
this with generics instead. Generics, however, do not allow you to abstract over operators (which is what
is is needed to really do <code>INumeric</code>) but you can simulate it using adapters as described in
<a href="http://www.removingalldoubt.com/PermaLink.aspx/f712e487-a6f5-4a65-a718-5ce580f399a8">
this post</a>.</p>
<p>Say I want to write a general average routine that is generic over what type I am using to store the data.
That is, I want one routine that works equally for <code><strong>double</strong></code> as well as <code><strong>decimal</strong></code>.
What I want to be able to write is something like,</p>
<pre class="code"><strong>public</strong> <strong>static</strong> T Average(<strong>params</strong> T[] values) {
    T total = 0;

    <strong>foreach</strong> (<strong>var</strong> v <strong>in</strong> values)
        total = total + v;

    <strong>return</strong> total / values.Length;
}</pre>
<p>where the <code>T</code> is any type that supports adding and dividing. I can't get this code to
compile exactly but we can get surprisingly close! My final version looks like,</p>
<pre class="code"><strong>class</strong> Math&lt;T, A&gt; : PolicyUser&lt;T, A&gt; <strong>where</strong> A : <strong>struct</strong>, IOperatorPolicy&lt;T&gt; {
    <strong>public</strong> <strong>static</strong> T Average(<strong>params</strong> T[] values) {
        <strong>var</strong> total = L(0);

        <strong>foreach</strong> (<strong>var</strong> v <strong>in</strong> values)
            total = total + v;

        <strong>return</strong> total / values.Length;
    }
}</pre>
<p>which is much closer to the original than I thought originally could get!</p>
<p>I started by cloning the operator policy and added a <code>FromInt()</code> I will explain below,</p>
<pre class="code"><strong>interface</strong> IOperatorPolicy&lt;T&gt; {
    T Add(T a, T b);
    T Subtract(T a, T b);
    T Multiply(T a, T b);
    T Divide(T a, T b);
    T FromInt(<strong>int</strong> value);
}
</pre>
<p>Then I cloned the <code><strong>double</strong></code> policy struct and then added a <code><strong>decimal</strong></code> mainly by cut/paste/search/replace</p>
<pre class="code"><strong>struct</strong> DoubleOperatorPolicy : IOperatorPolicy&lt;<strong>double</strong>&gt; {
    <strong>double</strong> IOperatorPolicy&lt;<strong>double</strong>&gt;.Add(<strong>double</strong> a, <strong>double</strong> b) { <strong>return</strong> a + b; }
    <strong>double</strong> IOperatorPolicy&lt;<strong>double</strong>&gt;.Subtract(<strong>double</strong> a, <strong>double</strong> b) { <strong>return</strong> a - b; }
    <strong>double</strong> IOperatorPolicy&lt;<strong>double</strong>&gt;.Multiply(<strong>double</strong> a, <strong>double</strong> b) { <strong>return</strong> a * b; }
    <strong>double</strong> IOperatorPolicy&lt;<strong>double</strong>&gt;.Divide(<strong>double</strong> a, <strong>double</strong> b) { <strong>return</strong> a / b; }
    <strong>double</strong> IOperatorPolicy&lt;<strong>double</strong>&gt;.FromInt(<strong>int</strong> value) { <strong>return</strong> value; }
}

<strong>struct</strong> DecimalOperatorPolicy : IOperatorPolicy&lt;<strong>decimal</strong>&gt; {
    <strong>decimal</strong> IOperatorPolicy&lt;<strong>decimal</strong>&gt;.Add(<strong>decimal</strong> a, <strong>decimal</strong> b) { <strong>return</strong> a + b; }
    <strong>decimal</strong> IOperatorPolicy&lt;<strong>decimal</strong>&gt;.Subtract(<strong>decimal</strong> a, <strong>decimal</strong> b) { <strong>return</strong> a - b; }
    <strong>decimal</strong> IOperatorPolicy&lt;<strong>decimal</strong>&gt;.Multiply(<strong>decimal</strong> a, <strong>decimal</strong> b) { <strong>return</strong> a * b; }
    <strong>decimal</strong> IOperatorPolicy&lt;<strong>decimal</strong>&gt;.Divide(<strong>decimal</strong> a, <strong>decimal</strong> b) { <strong>return</strong> a / b; }
    <strong>decimal</strong> IOperatorPolicy&lt;<strong>decimal</strong>&gt;.FromInt(int value) { <strong>return</strong> value; }
}</pre>
<p>The cleaver bit is to define a base type that contains a <code>struct</code> that defines the operators
as overloads. The overloads use a policy <code><strong>struct</strong></code> to implement that overloaded operators. This <code>
<strong>struct</strong></code>
also defines implicit conversion operators to hide some (but not all) of the messiness introduced by using this
wrapper type. The <code>PolicyUser</code> class looks like,</p>
<pre class="code"><strong>class</strong> PolicyUser&lt;T, A&gt; <strong>where</strong> A: <strong>struct</strong>, IOperatorPolicy&lt;T&gt; {
    <strong>static</strong> A Policy = <strong>new</strong> A();

    <strong>protected</strong> <strong>struct</strong> Value {
        <strong>public</strong> T V;

        <strong>public</strong> Value(T v) { V = v; }

        <strong>public</strong> <strong>static</strong> Value <strong>operator</strong> +(Value a, Value b) { <strong>return</strong> <strong>new</strong> Value(Policy.Add(a.V, b.V)); }
        <strong>public</strong> <strong>static</strong> Value <strong>operator</strong> -(Value a, Value b) { <strong>return</strong> <strong>new</strong> Value(Policy.Subtract(a.V, b.V)); }
        <strong>public</strong> <strong>static</strong> Value <strong>operator</strong> *(Value a, Value b) { <strong>return</strong> <strong>new</strong> Value(Policy.Multiply(a.V, b.V)); }
        <strong>public</strong> <strong>static</strong> Value <strong>operator</strong> /(Value a, Value b) { <strong>return</strong> <strong>new</strong> Value(Policy.Divide(a.V, b.V)); }
        <strong>public</strong> <strong>static</strong> <strong>implicit</strong> <strong>operator</strong> Value(T v) { <strong>return</strong> <strong>new</strong> Value(v); }
        <strong>public</strong> <strong>static</strong> <strong>implicit</strong> <strong>operator</strong> Value(<strong>int</strong> v) { <strong>return</strong> <strong>new</strong> Value(Policy.FromInt(v)); }
        <strong>public</strong> <strong>static</strong> <strong>implicit</strong> <strong>operator</strong> T(Value v) { <strong>return</strong> v.V; }
    }

    <strong>protected</strong> <strong>static</strong> Value L(<strong>int</strong> value) { <strong>return</strong> <strong>new</strong> Value(Policy.FromInt(value)); }
}</pre>
<p>What the <code>Value</code> struct does is allows you to use the type <code>Value</code> in place of the type <code>T</code> whenever you want to
use operators. When operators are used you only need to ensure one sub-expression is promoted to <code>Value</code> and the implict operator will
take care of the rest.</p>
<p>One remaining oddness is using literals. Implicit operators have there limits which makes using literal a bit strange. This was also odd in the
<a href="http://www.removingalldoubt.com/PermaLink.aspx/f712e487-a6f5-4a65-a718-5ce580f399a8">Policy</a> post as well and I used the same trick to make it a little less cumbersome. I introduced
a static method <code>L()</code> that takes an integer and converts it to <code>Value</code> allowing you to use integer literals by calling <code>FromInt()</code> I introduced earlier. You can
extend to this to allow other types by repeating the pattern for, say <code><strong>double</strong></code>, allowing you to use <code><strong>double</strong></code> literals as well. I didn't
because I didn't need it for my example (but probably will if I try to implement something more complicated.
</p>
<p>To bind the actual data type to provide the data type and the policy. To make this simpler I created two create implementations of <code>Math</code>.</p>
<pre class="code">  <strong>class</strong> DoubleMath : Math&lt;<strong>double</strong>, DoubleOperatorPolicy&gt; { }
  <strong>class</strong> DecimalMath : Math&lt;<strong>decimal</strong>, DecimalOperatorPolicy&gt; { }
</pre>
<p>Calling the average method looks like,</p>
<pre class="code">    <strong>var</strong> resultDouble = DoubleMath.Average(1, 2, 3, 4, 5);
    <strong>var</strong> resultDecimal = DecimalMath.Average(1, 2, 3, 4, 5);
</pre>
<p>It should be straight forward to see how this could be adapted to a <code>Financial</code> class, as Luca was trying to build, and how the routines could
be made independent of the data type used in the calculations.</p>
<p>There you have it. Policies can be used to simulate <code>INumeric</code> without having to resort to using <code><strong>dynamic</strong></code>.</p>
<p>Edit: Mar 9, 2009: Fixed HTML formatting error
</p></div>
