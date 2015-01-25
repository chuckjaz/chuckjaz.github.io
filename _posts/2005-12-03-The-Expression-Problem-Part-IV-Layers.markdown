---
layout: post
title:  "The Expression Problem: Part IV - Layers"
date:   2005-12-03 17:48:00
categories: [Programming]
permalink: post/2005/12/03/The-Expression-Problem-Part-IV-Layers.aspx
---
<p>So far I have presented three different approaches to solving the expression
problem, procedural, pure object-oriented, and a visitor pattern. This time I
will present a technique I will refer to as layers. Like the visitor pattern, it is a modification of the
pure object-oriented approach that takes advantage of partial classes. The
solution begins very similarly to the other object-oriented approaches, I create an abstract
class to represent an abstraction of expressions,</p>


{% highlight csharp %}
abstract partial class Expr { }
{% endhighlight %}

<p>Notice the addition of the <code>partial</code> modifier. I will
take advantage of that shortly. For now I will just flesh out the rest of the
hierarchy in a way that should, by now, be very familiar. I created a class to
represent literals,</p>

{% highlight csharp %}
partial class Literal: Expr {
  protected double Value;

  public Literal(double value) { Value = value; }
}
{% endhighlight %}

<p>Notice that, as with the pure object-oriented approach, I can keep the field
used to hold the value as protected. This is also true for the class to
represent an abstraction of binary operators.</p>

{% highlight csharp %}
abstract partial class BinaryOperator: Expr {
  protected Expr Left;
  protected Expr Right;

  public BinaryOperator(Expr left, Expr right) {
    Left = left;
    Right = right;
  }
}
{% endhighlight %}
<p>The actual binary operators are just descendants of <code>
BinaryOperator</code> with no additional data.</p>

{% highlight csharp %}
partial class Add: BinaryOperator {
  public Add(Expr left, Expr right) : base(left, right) { }
}

partial class Subtract: BinaryOperator {
  public Subtract(Expr left, Expr right) : base(left, right) { }
}

partial class Multiply: BinaryOperator {
  public Multiply(Expr left, Expr right) : base(left, right) { }
}

partial class Divide: BinaryOperator {
  public Divide(Expr left, Expr right) : base(left, right) { }
}
{% endhighlight %}

<p>So far, this is nearly identical to the pure object-oriented approach. Where
it differs is in how new operations are added.&nbsp; To add the
<code>Evaluate()</code> method, instead of modifying each class to add the methods, I can take advantage of the fact I left them
as partial classes and create additional parts that contain the methods instead.
First I created a new part of the <code>Expr</code> class to
introduce the virtual method <code>Evaluate()</code>.</p>

{% highlight csharp %}
abstract partial class Expr {
  public abstract double Evaluate();
}
{% endhighlight %}

<p>Compiling now will generate several errors indicating that the other classes
do not implement the <code>Evaluate()</code> method.&nbsp; This is
want I wanted. Using <code>abstract</code> in the above class part lets the compiler help me
determine when I have completed the implementation of the evaluate operation.
The rest of the class parts for <code>Evaluate()</code> looks like,</p>

{% highlight csharp %}
partial class Literal {
  public override double Evaluate() {
    return Value;
  }
}

abstract partial class BinaryOperator {
  public sealed override double Evaluate() {
    return EvaluateOp(Left.Evaluate(), Right.Evaluate());
  }

  protected abstract double EvaluateOp(double left, double right);
}

partial class Add {
  protected override double EvaluateOp(double left, double right) {
    return left + right;
  }
}

partial class Subtract {
  protected override double EvaluateOp(double left, double right) {
    return left - right;
  }
}

partial class Multiply {
  protected override double EvaluateOp(double left, double right) {
    return left * right;
  }
}

partial class Divide {
  protected override double EvaluateOp(double left, double right) {
    return left / right;
  }
}
{% endhighlight %}

<p>I call each of the collections of class parts a layer. You can think
of them much like a transparency teachers uses on overhead projectors. You can
build up a class by combining layers of functionality. I usually keep each layer
in its own file named for the layer. In this case, Evaluator.cs might be a good
choice.</p>
<p>To demonstrate building up a class with through layers I
will create a printing layer.</p>

{% highlight csharp %}
abstract partial class Expr {
  public abstract void Print();
}

partial class Literal {
  public override void Print() {
    Console.Write(Value);
  }
}

abstract partial class BinaryOperator {
  public sealed override void Print() {
    Left.Print();
    PrintOp();
    Right.Print();
  }

  protected abstract void PrintOp();
}

partial class Add {
  protected override void PrintOp() {
    Console.Write(" + ");
  }
}

partial class Subtract {
  protected override void PrintOp() {
    Console.Write(" - ");
  }
}

partial class Multiply {
  protected override void PrintOp() {
    Console.Write(" * ");
  }
}

partial class Divide {
  protected override void PrintOp() {
    Console.Write(" / ");
  }
}
{% endhighlight %}

<p>As in the other approaches, I will demonstrate how to add support for the
Power expression form. I have two choices, I can add it just as we did in the pure object-oriented approach,</p>

{% highlight csharp %}
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

<p>or I could divide it into multiple parts that can be co-located with the other
similar parts. The data part would look like,</p>

{% highlight csharp %}
partial class Power: BinaryExpr {
  public Power(Expr left, Expr right) : base(left, right) { }
}
{% endhighlight %}

<p>The part for the expression layer would look like,</p>

{% highlight csharp %}
partial class Power: BinaryExpr {
  protected override double EvaluateOp(double left, double right) {
    return Math.Pow(left, right);
  }
}
{% endhighlight %}

<p>An the printing layer part would look like,</p>

{% highlight csharp %}
partial class Power: BinaryExpr {
  protected override void PrintOp() {
    Console.Write(" ^ ");
  }
}
{% endhighlight %}

<p>It is this flexibility that makes layers so appealing. You can decide, case
by case, which is the right way to modify the hierarchy.</p>
