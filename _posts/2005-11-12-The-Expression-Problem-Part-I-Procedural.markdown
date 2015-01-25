---
layout: post
title:  "The Expression Problem: Part I - Procedural"
date:   2005-11-12 16:42:00
categories: [Programming]
permalink: post/2005/11/12/The-Expression-Problem-Part-I-Procedural.aspx
---
<div class="text"><p>Expressions are difficult to represent because they form a matrix of
operations and data types. Languages make it either easy to add new operations
but difficult to add data types or easy to add new data types but hard to add
new operations. This problem isn't limited to expressions, of course, but it is
a very clear and familiar example so I will use it to characterize the more
general problem. I will
present four ways to approach the problem, each with their particular strengths
and weaknesses. For my examples
I representing a simple expression language comprising of the following
expression forms,</p>
<blockquote>
<table border="0" id="table1">
<tbody><tr>
<td width="100"><i>number</i></td>
<td>Literal</td>
</tr>
<tr>
<td width="100"><i>expr</i> + <i>expr</i></td>
<td>Add</td>
</tr>
<tr>
<td width="75"><i>expr</i> - <i>expr</i></td>
<td>Subtract</td>
</tr>
<tr>
<td width="75"><i>expr</i> * <i>expr</i></td>
<td>Multiply</td>
</tr>
<tr>
<td width="75"><i>expr</i> / <i>expr</i></td>
<td>Divide</td>
</tr>
</tbody></table>
</blockquote>
<p>Some operations I want to perform on these expressions include
evaluating expressions and printing expressions. The operations and the expressions form a matrix
of functionality, expression kind by operation,</p>
<blockquote>
<table border="1" id="table3" style="border-collapse: collapse" width="321">
<tbody><tr>
<td width="72">&nbsp;</td>
<td>Evaluate</td>
<td width="102">Print</td>
</tr>
<tr>
<td width="72">Literal</td>
<td><i>evaluate literal</i></td>
<td width="102"><i>print literal</i></td>
</tr>
<tr>
<td width="72">Add</td>
<td><i>evaluate add</i></td>
<td width="102"><i>print add</i></td>
</tr>
<tr>
<td width="72">Subtract</td>
<td><i>evaluate subtract</i></td>
<td width="102"><i>print subtract</i></td>
</tr>
<tr>
<td width="72">Multiply</td>
<td><i>evaluate multiply</i></td>
<td width="102"><i>print multiply</i></td>
</tr>
<tr>
<td width="72">Divide</td>
<td><i>evaluate divide</i></td>
<td width="102"><i>print divide</i></td>
</tr>
</tbody></table>
</blockquote>
<p>Languages either make it easier to add new columns,
that is operations, or add new rows, that is new expression kinds. Procedural
and functional languages typically make it easy to add new operations.
Object-oriented languages typically make it easier to add new expression kinds
represented as new data types. </p>
<p>The first approach I will demonstrate is procedural. I define a data type,
<code>Expr</code>, that will hold the operator and the operands.</p>

{% highlight csharp %}
enum Oper { Add, Subtract, Multiply, Divide, Literal }

class Expr {
  public Oper Oper;
  public Expr Left;
  public Expr Right;
  public double Value;

  public Expr(Oper oper, Expr left, Expr right) { ... }
  public Expr(double value) { ... }
}
{% endhighlight %}<sup><a href="#note1" name="ref-note1">1</a></sup>
  <p>The expression evaluation operation for this data structure looks like,</p>

{% highlight csharp %}
static double Evaluate(Expr e) {
  switch (e.Oper) {
    case Oper.Add: return Evaluate(e.Left) + Evaluate(e.Right);
    case Oper.Subtract: return Evaluate(e.Left) - Evaluate(e.Right);
    case Oper.Multiply: return Evaluate(e.Left) * Evaluate(e.Right);
    case Oper.Divide: return Evaluate(e.Left) / Evaluate(e.Right);
    case Oper.Literal: return e.Value;
    default: Debug.Fail("Unknown operator"); return 0;
  }
}
{% endhighlight %}

    <p>The advantages to a procedural approach are,</p>
    <ol>
    <li>It is very simple to add additional operations (such as
      <code>Evaluate()</code>).</li>
      <li>All the logic for an operation is centralized in one place, and often in a single
      routine.</li>
      <li>New operations can be added without having the modify existing
      operations.</li>
      <li>New operations can be also be added dynamically without affecting or
      needing to recompile existing operations.</li>
      </ol>
      <p>To demonstrate some of the advantages of using a procedural approach we will
      add a print operation. It looks like,</p>

{% highlight csharp %}
static void Print(Expr e) {
  switch (e.Oper) {
    case Oper.Add: Print(e.Left); Console.Write(" + "); Print(e.Right); break;
    case Oper.Subtract: Print(e.Left); Console.Write(" - "); Print(e.Right); break;
    case Oper.Multiply: Print(e.Left); Console.Write(" * "); Print(e.Right); break;
    case Oper.Divide: Print(e.Left); Console.Write(" / "); Print(e.Right); break;
    case Oper.Literal: Console.Write(e.Value); break;
    default: Debug.Fail("Unknown operator"); break;
  }
}
{% endhighlight %}
<sup><a href="#note2" name="ref-note2">2</a></sup>

        <p>Adding the <code>Print</code> operation on the <code>Expr</code> type doesn't
        require any modifications to <code>Expr</code> or
        <code>Evaluate()</code>. </p>
        <p>Some disadvantages to a procedural approach are,</p>
        <ol>
        <li>It is difficult to add additional data types (in our example, each data
          type represents a different operator) because adding a new type affects
          all operations.</li>
          <li>Each operation must account for every data type.</li>
          <li>The internal representation of the data type must be accessible to every
          operation. This makes even minor changes to the data layout have far
          reaching effects and often leads to difficult to maintain implementations.</li>
          <li>It is very difficult, and requires careful planning, to dynamically add
          data types.</li>
          <li>Efficient storage of the data types requires variant record support in
          the language, as discussed in <a href="#note1">note 1</a> below, (which can
            be simulated though inheritance in <i>object-oriented</i> languages but I
            don't provide and example of that).</li>
            </ol>
            <p>To demonstrate the difficulty of adding a data type, let's add power support.
            </p>
            <blockquote>
            <table border="0" id="table2">
            <tbody><tr>
            <td width="100"><i>expr</i> ^ <i>expr</i></td>
            <td>Power</td>
            </tr>
            </tbody></table>
            </blockquote>
            <p>This represents the expression <i>x<sup>n</sup></i>. To implement this we first we need to modify the enumeration
            to add <code>Power</code>,</p>

{% highlight csharp %}
enum Oper { Add, Subtract, Multiply, Divide, Literal, Power }
{% endhighlight %}

            <p>Next we need to add an additional <i>case statement</i> to <code>Evaluate()</code>'s
            <i>switch statement</i>,</p>

{% highlight csharp %}
case Oper.Power: return Math.Pow(left, right);
{% endhighlight %}

            <p>An finally we need to also add another <i>case statement</i> to <code>Print</code>'s
            <i>switch statement </i>that looks like,</p>

{% highlight csharp %}
case Oper.Power: Print.(e.Left); Console.Write(" ^ "); Print(e.Right); break;
{% endhighlight %}

            <p>Note we had to modify both operations and the <code>Oper</code> enumeration. If we had several
            operations, we would have had
            to modify most, if not all, of them. Also, note that the compiler didn't help us
            find any of the places we need to modify since the code was legal after each of the above modifications.
            If we had made the changes to just <code>Evaluate()</code> for
            example, the compiler wouldn't have complained, but we would receive a runtime
            error if we called <code>Print()</code> with an expression tree
            that contains a <code>Power</code> node.</p>
            <p>To learn more about the expression problem and its history, see
            <a href="http://www.cs.pomona.edu/~kim/">Kim Bruce</a>'s excellent paper,
            <a href="http://www.cs.pomona.edu/~kim/README.html#Wood">Some Challenging Typing
            Issues in Object-Oriented Languages</a>. Be warned, it will be a bit of a
            spoiler for some of what I will cover.</p><hr>
            <p><a name="note1" href="#ref-note1">1</a> - A casual observer will notice that
            <code>Expr</code> is a bit wasteful in that it allocates
            room for a value for every node instead of only for literals. It also takes up
            space for <code>Left</code> and <code>Right</code> for literals. In C#, removing this waste is a bit awkward
            to express and makes the <code>Evaluate()</code> method more
            complicated, so I ignored the waste in favor of simplicity.</p>
            <p>In C++, this is a bit less awkward, and might look like,</p>

{% highlight c %}
struct Expr {
  Oper            oper;
  union {
    struct {
      Expr   *left;
      Expr   *right;
    };
    double      value;
  };
  Expr(Oper oper, Expr *left, Expr *right) { ... }
  Expr(double value) { ... }
};
{% endhighlight %}

              <p>with a very similar evaluation function, but for now we will stick with C#. <a href="#ref-note1">&lt;&lt; back &lt;&lt;</a></p>
              <p><a name="note2" href="#ref-note2">2</a> - For simplicity, I ignored operator precedence. Later I will present a version of
              this routine that handles precedence correctly. <a href="#ref-note2">&lt;&lt; back &lt;&lt;</a></p></div>
