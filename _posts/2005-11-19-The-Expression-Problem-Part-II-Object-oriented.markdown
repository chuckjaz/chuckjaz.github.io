---
layout: post
title:  "The Expression Problem: Part II - Object-oriented"
date:   2005-11-19 12:49:00
categories: [Programming]
permalink: post/2005/11/19/The-Expression-Problem-Part-II-Object-oriented.aspx
---
<p>Last time we took a procedural approach to solve the <i>expression problem</i>.
This time I will demonstrate an object-oriented approach. As a
reminder, we want to model the following forms of expressions,</p>
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
<p>The most natural thing to do in an object-oriented language is to make each of the above
expression forms its own class. First, I will introduce a base class to provide an expression abstraction.</p>

{% highlight csharp %}
abstract class Expr {
}
{% endhighlight %}

<p>Now I will now create a sub-class for each expression.</p>

{% highlight csharp %}
class Literal: Expr {
  protected double Value;

  public Literal(double value) {
    Value = value;
  }
}

abstract class BinaryExpr: Expr {
  protected Expr Left;
  protected Expr Right;

  public BinaryExpr(Expr left, Expr right) {
    Left = left;
    Right = right;
  }
}

class Add: BinaryExpr {
  public Add(Expr left, Expr right) : base(left, right) { }
}

class Subtract: BinaryExpr {
  public Subtract(Expr left, Expr right) : base(left, right) { }
}

class Multiply: BinaryExpr {
  public Multiply(Expr left, Expr right) : base(left, right) { }
}

class Divide: BinaryExpr {
  public Divide(Expr left, Expr right) : base(left, right) { }
}
{% endhighlight %}

  <p>I created a base class for the binary expressions which I will take
  advantage of below. </p>
  <p>To evaluate the expression I will add an abstract virtual method,
  <code>Evaluate()</code>, to <code>Expr</code>. </p>

{% highlight csharp %}
abstract class Expr {
    public abstract double Evaluate();
}
{% endhighlight %}

<p>Then I will override the <code>Evaluate()</code> method for
each of the above classes. Evaluating a literal is simply returning the value of
the literal so I modified the <code>Literal</code> class to,</p>

{% highlight csharp %}
class Literal: Expr {
  protected double Value;

  public Literal(double value) {
    Value = value;
  }

  public override double Evaluate() {
    return Value;
  }
}
{% endhighlight %}

<p>All binary expressions need to evaluate their left-hand expression and
right-hand expression and then perform their operation on the results. To represent this, I
overrode <code>Evaluate()</code> to evaluate both <code>Left</code> and <code>Right</code>
and then call a newly introduced <code>EvaluateOp()</code>
method. I sealed the <code>Evaluate()</code> method because I want
descendants to override <code>EvaluateOp()</code> not
<code>Evaluate()</code>.</p>

{% highlight csharp %}
abstract class BinaryExpr: Expr {
  protected Expr Left;
  protected Expr Right;

  public BinaryExpr(Expr left, Expr right) {
    Left = left;
    Right = right;
  }

  public sealed override double Evaluate() {
    return EvaluateOp(Left.Evaluate(), Right.Evaluate());
  }

  protected abstract double EvaluateOp(double left, double right);
}
{% endhighlight %}

<p>Now I can implemented the concrete descendents of <code>BinaryExpr</code>.</p>

{% highlight csharp %}
class Add: BinaryExpr {
  public Add(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left + right;
  }
}

class Subtract: BinaryExpr {
  public Subtract(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left - right;
  }
}

class Multiply: BinaryExpr {
  public Multiply(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left * right;
  }
}

class Divide: BinaryExpr {
  public Divide(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left / right;
  }
}
{% endhighlight %}
<p>The advantages an object-oriented approach are,</p>
<ol>
<li>New data types can be added without affecting any of the other data
types. </li>
<li>Each data type is encapsulated, only the data type needs to have access
the instance variables.</li>
<li>All the operations affecting the instance variables are in one place,
the methods of instance variable's class.</li>
<li>New operations can be added as abstract methods. The compiler will then
generate an error if an operation is not implemented for one of the leaf
classes.</li>
<li>Efficient storage is natural. Adding field to one of the expression
types do not affect the others.</li>
</ol>
<p>To demonstrate how easy it is to add a new data type I will again add support
for Power.</p>
<blockquote>
<table border="0" id="table2">
<tbody><tr>
<td width="100"><i>expr</i> ^ <i>expr</i></td>
<td>Power</td>
</tr>
</tbody></table>
</blockquote>
<p> To do this I will add a class to represent the Power expression form.
This looks like,</p>

{% highlight csharp %}
class Power: BinaryExpr {
  public Power(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return Math.Pow(left, right);
  }
  }
{% endhighlight %}
<p>Note that the <code>Power</code> data type can be added without
modifying the other classes. </p>
<p>The disadvantages to an object-oriented approach are,</p>
<ol>
<li>It is difficult to add new operations because adding an operation
requires modifying all the classes.</li>
<li>The logic for each operation is spread over multiple classes and often
multiple files. This makes it cumbersome to get a complete picture what the
operation does.</li>
<li>It is very difficult to dynamically add operations and can't be done
without careful planning.</li>
</ol>
<p>To demonstrate some of the difficulties of adding a new operation, I will add
the Print operation. First I will modify the base <code>Expr</code>
class to add an abstract <code>Print()</code> method.</p>

{% highlight csharp %}
abstract class Expr {
  public abstract double Evaluate();
  public abstract void Print();
}
{% endhighlight %}
<p>Now I will override this method in each class similar to the way I did for
the <code>Evaluate()</code> method.</p>

{% highlight csharp %}
class Literal: Expr {
  protected double Value;

  public Literal(double value) {
    Value = value;
  }

  public override double Evaluate() {
    return Value;
  }

  public override void Print() {
    Console.Write(Value);
  }
}

abstract class BinaryExpr: Expr {
  protected Expr Left;
  protected Expr Right;

  public BinaryExpr(Expr left, Expr right) {
    Left = left;
    Right = right;
  }

  public sealed override double Evaluate() {
    return EvaluateOp(Left.Evaluate(), Right.Evaluate());
  }

  protected abstract double EvaluateOp(double left, double right);

  public sealed override void Print() {
    Left.Print();
    PrintOp();
    Right.Print();
  }

  protected abstract void PrintOp();
}

class Add: BinaryExpr {
  public Add(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left + right;
  }

  protected override void PrintOp() {
    Console.Write(" + ");
  }
}

class Subtract: BinaryExpr {
  public Subtract(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left - right;
  }

  protected override void PrintOp() {
    Console.Write(" - ");
  }
}

class Multiply: BinaryExpr {
  public Multiply(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left * right;
  }

  protected override void PrintOp() {
    Console.Write(" * ");
  }
}

class Divide: BinaryExpr {
  public Divide(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return left / right;
  }

  protected override void PrintOp() {
    Console.Write(" / ");
  }
}

class Power: BinaryExpr {
  public Power(Expr left, Expr right) : base(left, right) { }

  protected override double EvaluateOp(double left, double right) {
    return Math.Pow(left, right);
  }

  protected override void PrintOp() {
    Console.Write(" ^ ");
  }
}
{% endhighlight %}
<p>As you can see, each of the classes needed to be modified to implement
<code>Print()</code>. You can also note that the compiler
complained until each of the classes implemented the <code>Print()</code>
operation. This is because we used an abstract method in the <code>
Expr</code> class.</p>
<p>If you compare the procedural approach vs. the object-oriented approach you
will notice that they are pretty much opposites. It is easy to add operations in
the procedural approach but difficult using an object-oriented approach. Adding
data types is difficult using in a procedural approach, but easy in an
object-oriented approach. Data needs to be public in procedural, but can be
private in object oriented. When deciding which approach to use you need to try
an predict what will occur more often, adding new data types or adding new
operations. What I will present next are some compromise solutions that balance
the advantages and disadvantages.</p>
