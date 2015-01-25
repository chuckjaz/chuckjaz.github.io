---
layout: post
title:  "Operators, Generics and Policies II: Adapter Policy"
date:   2007-07-31 22:27:00
categories: [Programming]
permalink: post/2007/07/31/Operators-Generics-and-Policies-II-Adapter-Policy.aspx
---
<div class="text"><p>The mixin technique in my
<a href="post/2007/07/25/Operators-Generics-and-Policies.asp">last post</a> was a bit awkward to use
with generic
methods. It is much less awkward to use for classes, however. For example, if
you want an expression evaluator much like I posted
<a href="post/2005/11/12/The-Expression-Problem-Part-I-Procedural.aspx">
here</a>, but you want to leave the result type open, you can use this technique
to do it.</p>
<p>First, I declared a policy interface for the operators I wanted to abstract
over,</p>
<pre><b>partial</b> <b>interface</b> IOperatorPolicy&lt;T&gt; {
    T Add(T a, T b);
    T Subtract(T a, T b);
    T Multiply(T a, T b);
    T Divide(T a, T b);
}</pre>

<p>Second, I created the operator policies for <code><strong>int</strong></code>
and <code><strong>double</strong></code>. I could have created more, <code>
<strong>decimal</strong></code>, <code><strong>float</strong></code>, etc. but
two is sufficient for demonstration.</p>
<pre><strong>partial</strong> <b>struct</b> IntOperatorPolicy : IOperatorPolicy&lt;<b>int</b>&gt; {
    <b>int</b> IOperatorPolicy&lt;<b>int</b>&gt;.Add(<b>int</b> a, <b>int</b> b) { <b>return</b> a + b; }
    <b>int</b> IOperatorPolicy&lt;<b>int</b>&gt;.Subtract(<b>int</b> a, <b>int</b> b) { <b>return</b> a - b; }
    <b>int</b> IOperatorPolicy&lt;<b>int</b>&gt;.Multiply(<b>int</b> a, <b>int</b> b) { <b>return</b> a * b; }
    <b>int</b> IOperatorPolicy&lt;<b>int</b>&gt;.Divide(<b>int</b> a, <b>int</b> b) { <b>return</b> a / b; }
}

<b>partial struct</b> DoubleOperatorPolicy : IOperatorPolicy&lt;<b>double</b>&gt; {
    <b>double</b> IOperatorPolicy&lt;<b>double</b>&gt;.Add(<b>double</b> a, <b>double</b> b) { <b>return</b> a + b;
    <b>double</b> IOperatorPolicy&lt;<b>double</b>&gt;.Subtract(<b>double</b> a, <b>double</b> b) { <b>return</b> a - b; }
    <b>double</b> IOperatorPolicy&lt;<b>double</b>&gt;.Multiply(<b>double</b> a, <b>double</b> b) { <b>return</b> a * b; }
    <b>double</b> IOperatorPolicy&lt;<b>double</b>&gt;.Divide(<b>double</b> a, <b>double</b> b) { <b>return</b> a / b; }
}</pre>

<p>Next are the expression nodes themselves. I wrap them in a generic class
that takes the parameters. This mitigates some of the clutter caused by the
policy.</p>
<pre><b>partial</b> <b>class</b> Expression&lt;T, OperatorPolicy&gt;
    <strong>where</strong> OperatorPolicy : <b>struct</b>, IOperatorPolicy&lt;T&gt; {

    <b>static</b> OperatorPolicy _operatorPolicy = <b>new</b> OperatorPolicy();

    <b>public</b> <b>abstract</b> <b>partial</b> <b>class</b> Expr {
        <b>public</b> <b>abstract</b> T Evaluate();
    }

    <b>public</b> <b>partial</b> <b>class</b> Literal : Expr {
        <b>private</b> T _literal;

        <b>public</b> Literal(T literal) { _literal = literal; }

        <b>public</b> <b>override</b> T Evaluate() {
            <b>return</b> _literal;
        }

        <b>public</b> <b>override</b> <b>string</b> ToString() {
            <b>return</b> _literal.ToString();
        }
    }

    <b>public</b> <b>abstract</b> <b>partial</b> <b>class</b> BinaryOp : Expr {
        <b>private</b> Expr _left;
        <b>private</b> Expr _right;

        <b>protected</b> BinaryOp(Expr left, Expr right) { _left = left; _right = right; }

        <b>public</b> <b>override</b> T Evaluate() {
            <b>return</b> EvaluateOp(_left.Evaluate(), _right.Evaluate());
        }

        <b>protected</b> <b>abstract</b> T EvaluateOp(T left, T right);
    }

    <b>public</b> <b>partial</b> <b>class</b> Add : BinaryOp {
        <b>public</b> Add(Expr left, Expr right) : <b>base</b>(left, right) { }
        <b>protected</b> <b>override</b> T EvaluateOp(T left, T right) {
            <b>return</b> _operatorPolicy.Add(left, right);
        }
    }

    <b>public</b> <b>partial</b> <b>class</b> Add : BinaryOp {
        <b>public</b> Add(Expr left, Expr right) : <b>base</b>(left, right) { }
        <b>protected</b> <b>override</b> T EvaluateOp(T left, T right) {
            <b>return</b> _operatorPolicy.Add(left, right);
        }
    }

    <b>public</b> <b>partial</b> <b>class</b> Subtract : BinaryOp {
        <b>public</b> Subtract(Expr left, Expr right) : <b>base</b>(left, right) { }
        <b>protected</b> <b>override</b> T EvaluateOp(T left, T right) {
            <b>return</b> _operatorPolicy.Subtract(left, right);
        }
    }

    <b>public</b> <b>partial</b> <b>class</b> Multiply : BinaryOp {
        <b>public</b> Multiply(Expr left, Expr right) : <b>base</b>(left, right) { }
        <b>protected</b> <b>override</b> T EvaluateOp(T left, T right) {
            <b>return</b> _operatorPolicy.Multiply(left, right);
        }
    }

    <b>public</b> <b>partial</b> <b>class</b> Divide : BinaryOp {
        <b>public</b> Divide(Expr left, Expr right) : <b>base</b>(left, right) { }
        <b>protected</b> <b>override</b> T EvaluateOp(T left, T right) {
            <b>return</b> _operatorPolicy.Divide(left, right);
        }
    }
}</pre>

<p>Here the <code>Expression</code> class acts as a container of a family of
classes. Once you have an instance of one of the <code>Expr</code> derived classes, calling
<code>Evaluate()</code> is straight-forward. Unfortunately, constructing one is a bit of a
challenge. To get a literal of the <code><strong>int</strong></code> expression you would need to do,</p>
<pre><strong>var</strong> i1 = <strong>new</strong> Expression&lt;<strong>int</strong>, IntOperatorPolicy&gt;.Literal(1);</pre>

<p>You can make this a little better by introducing a <code><strong>static</strong></code>
method into <code>Expression</code> like,</p>
<pre><b>public</b> <b>static</b> Expr L(T literal) {
    <b>return</b> <b>new</b> Literal(literal);
}</pre>

<p>then the above becomes,</p>
<pre><strong>var</strong> i1 = Expression&lt;<strong>int</strong>, IntOperatorPolicy&gt;.L(1);</pre>
<p>This still a bit awkward especially when you want to construct a more
complicated expression,</p>
<pre><strong>var</strong> e = <strong>new</strong> Expression&lt;<strong>int</strong>, IntOperatorPolicy&gt;.Add(i1, i1);</pre>
<p>This can be made significantly better by adding operator overloads to <code>Expr</code> as
in,</p>
<pre><b>public</b> <b>abstract</b> <strong>partial</strong> <b>class</b> Expr {
    <b>public</b> <b>abstract</b> T Evaluate();
    <b>public</b> <b>static</b> Expr <b>operator</b> +(Expr a, Expr b) {
        <b>return</b> <b>new</b> Add(a, b);
    }
    <b>public</b> <b>static</b> Expr <b>operator</b> -(Expr a, Expr b) {
        <b>return</b> <b>new</b> Subtract(a, b);
    }
    <b>public</b> <b>static</b> Expr <b>operator</b> *(Expr a, Expr b) {
        <b>return</b> <b>new</b> Multiply(a, b);
    }
    <b>public</b> <b>static</b> Expr <b>operator</b> /(Expr a, Expr b) {
        <b>return</b> <b>new</b> Divide(a, b);
    }
}</pre>
<p>now the <code>e</code> assignment can look like,</p>
<pre><strong>var</strong> e = i1 + i1;</pre>
<p>and finally, to make it look even better you can put the code to create the
expression in a static method of a class that derives from <code>Expression</code> and closes the
type parameters. Here are examples of one for <code><strong>int</strong></code> and
a similar one for <code><strong>double</strong></code>.</p>
<pre><b>class</b> TestInt : Expression&lt;<b>int</b>, IntOperatorPolicy&gt; {
    <b>public</b> <b>static</b> <b>void</b> Run() {
        <strong>var</strong> iv = L(1) + L(2) * L(3);
        Console.WriteLine(iv.Evaluate());
    }
}

<b>class</b> TestDouble : Expression&lt;<b>double</b>, DoubleOperatorPolicy&gt; {
    <b>public</b> <b>static</b> <b>void</b> Run() {
        <strong>var</strong> dv = L(1.2) + L(3.4) * L(5.6);
        Console.WriteLine(dv.Evaluate());
    }
}</pre>
<p>This shows taking advantage of the C# compiler's operator overloading to
produce an expression tree and the use of the <code>L()</code> method as a sort
of cast to change a literal of <code>T</code> into a <code>Literal&lt;T&gt;</code>.
The call to <code>Evaluate()</code> calculates the result of the expression as
an <code><strong>int</strong></code> or <code><strong>double</strong></code>.
The code generated, as discussed
<a href="post/2007/07/25/Operators-Generics-and-Policies.aspx">
last time</a>, is quite good when JIT'ing outside the debugger and using
optimized retail bits.</p>
<p>This use of a policy <code><strong>struct</strong></code> is what I refer to
as an <em>adapter policy</em>. The <code>Expression</code> class has an
abstraction it supports, represented by <code>IOperatorPolicy&lt;T&gt;</code>, but
this abstraction is not supported by any of the interesting types you would want
to pass as <code>Expression</code>'s <code>T</code>. This is solved by creating
an <em>adapter policy</em> and passing that along with <code>T</code>. This
allows a generic type to introduce an abstraction that is unknown to the types
the generic wants to range over, allow us, as above, to pass <code>int</code>
for <code>Expression</code>'s <code>T</code> even though <code><strong>int</strong></code>
doesn't implement <code>IOperatorPolicy&lt;T&gt;</code> itself.</p>
<p>Next time I will discuss another application of an <em>adapter policy</em>.</p>
</div>
