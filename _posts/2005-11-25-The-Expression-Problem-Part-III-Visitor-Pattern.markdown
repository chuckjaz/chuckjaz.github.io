---
layout: post
title:  "The Expression Problem: Part III - Visitor Pattern"
date:   2005-11-25 16:16:00
categories: [Programming]
---
<div class="text"><p>So far we have looked at two ways to represent the following expression
forms,</p>
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
<p>The first was procedural and the second object-oriented. Using the
procedural approach, it was easier to add operations than data types. Using an
object-oriented approach, the opposite was true. What I will demonstrate now is
a technique that balances some of the advantages and disadvantages of both into
a hybrid approach called the
<a href="http://en.wikipedia.org/wiki/Visitor_pattern">Visitor Pattern</a>.</p>
<p>There are several ways to apply the visitor problem to expressions, each with
their own unique set of advantages and disadvantages, but I will only cover one
of these approaches.</p>
<p>The visitor pattern is a modification of the the object-oriented approach
that allows an operation to be separated from the class hierarchy into
its own class. To demonstrate how a visitor pattern can be applied, I will first
created a visitor interface that will be implemented by all visitors. It looks like,</p>

<code><pre>
public interface IExprVisitor&lt;R&gt; {
  R Visit(Literal literal);
  R Visit(Add add);
  R Visit(Subtract subtract);
  R Visit(Multiply multiply);
  R Visit(Divide divide);
}
</pre></code>

<sup><a name="ref-note1" href="#note1">1</a></sup>

<p>Here I use a generic type to leave the type of the return result open. The
expression evaluator will use <code>double</code> but <code>double</code> isn't an appropriate return
result for printing, for example. Using a type parameter allows different visitors
to have their own result types.</p>
<p>Now I will create the class hierarchy that will be visited. I will first
create an
abstract class to represent the expressions, just as I did in the previous object
oriented example.</p>

<code><pre>
public abstract class Expr {
  public abstract R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor);
}
</pre></code>

<p>The method <code>Accept()</code> is key to the visitor pattern.
It accepts the visitor and, in turn, calls the correct <code>
Visit()</code>
method on the visitor, based on the type of the instance being visited. To do this, each descendant overrides
the <code>Accept()</code> method and calls the correct method. To
show how this is done I will implement the Literal expression form. </p>

<code><pre>
public class Literal: Expr {
  public double Value;

  public Literal(double value) { Value = value; }

  public override R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor) {
    return visitor.Visit(this);
  }
}
</pre></code>

<p>The above <code>Literal</code> class is very similar to the
original pure object oriented version with the addition of the <code>
Accept()</code> method. The implementation of the method is trivial, simply call
<code>visitor.Visit(this)</code>. The compiler will determine that
<code>IExprVisitor&lt;R&gt;.Visitor(Literal literal)</code> is the method
to bind to based on the type of the <code>this</code> parameter.</p>
<p>Next, as before, I introduce a base class to represent an abstraction of a
binary operator.</p>

<code><pre>
public abstract class BinaryOperator: Expr {
  public Expr Right;
  public Expr Left;

  public BinaryOperator(Expr left, Expr right) {
    Right = right;
    Left = left;
  }
}
</pre></code>

<p>Notice that I do not yet implement the <code>Accept()</code> method.
I could have, after adding <code>R Visit(BinaryOperator binop)</code>
to the <code>IExprVisitor</code> interface, but I didn't need it
for the visitors I wanted to create. I have found that it is usually best to only have
the visitors visit the concrete classes and avoid the abstract. A visitor can
always simulate visiting a abstract base class by calling a common routine in
each of the concrete class visitor methods (you will see this technique below in
the <code>Printer</code> visitor). This leaves the visitor less
cluttered when the abstract class doesn't need to be visited separately. In other
words, my <code>Accept()</code> methods only calls one <code>
Visit()</code> method for each instance, and that for the most derived type. This has the
additional advantage of allowing the <code>Visit()</code> method to
have a meaningful result, which wouldn't be possible if the <code>
Accept()</code> method called more than one <code>Visit()</code>
methods.</p>
<p>Now that I have a base class for binary operators, I can now implement the
binary operator expression forms.</p>

<code><pre>
public class Add: BinaryOperator {
  public Add(Expr left, Expr right) : base(left, right) { }

  public override R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor) {
    return visitor.Visit(this);
  }
}

public class Subtract: BinaryOperator {
  public Subtract(Expr left, Expr right) : base(left, right) { }

  public override R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor) {
    return visitor.Visit(this);
  }
}

public class Multiply: BinaryOperator {
  public Multiply(Expr left, Expr right) : base(left, right) { }

  public override R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor) {
    return visitor.Visit(this);
  }
}

public class Divide: BinaryOperator {
  public Divide(Expr left, Expr right) : base(left, right) { }

  public override R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor) {
    return visitor.Visit(this);
  }
}
</pre></code>

<p>The implementation of these should come as no surprise. They each
override the <code>Accept()</code> method and call the visitor
method associated with their type.</p>
<p>Now that we have the class hierarchy in place, I will now implement the
expression evaluator as a visitor.</p>

<code><pre>
public class Evaluator: IExprVisitor&lt;double&gt; {
  public Evaluator() { }

  public double Evaluate(Expr expr) {
    return expr.Accept(this);
  }

  double IExprVisitor&lt;double&gt;.Visit(Literal literal) {
    return literal.Value;
  }

  double IExprVisitor&lt;double&gt;.Visit(Add add) {
    return Evaluate(add.Left) + Evaluate(add.Right);
  }

  double IExprVisitor&lt;double&gt;.Visit(Subtract subtract) {
    return Evaluate(subtract.Left) - Evaluate(subtract.Right);
  }

  double IExprVisitor&lt;double&gt;.Visit(Multiply multiply) {
    return Evaluate(multiply.Left) * Evaluate(multiply.Right);
  }

  double IExprVisitor&lt;double&gt;.Visit(Divide divide) {
    return Evaluate(divide.Left) / Evaluate(divide.Right);
  }
}
</pre></code>

<p>The expression evaluator can be used by creating a new visitor and calling
the <code>Evaluate()</code> method of the evaluator, passing the root of the expression tree.
For example,</p>

<code><pre>
Expr expr = new Add(new Multiply(new Literal(10), new Literal(4)),
  new Literal(2));

Evalutor evaluator = new Evaluator();
double result = evaluator.Evaluate(expr);

Console.WriteLine("The answer is {0}", result);
</pre></code>

<p>The visitor pattern makes it easy to add new operations. To demonstrate this,
as before, I will add a print operation. This looks very much like the
expression evaluator above.</p>

<code><pre>
public class Printer: IExprVisitor&lt;object&gt; {
  public Printer() { }

  public void Print(Expr expr) {
    expr.Accept(this);
  }

  object IExprVisitor&lt;object&gt;.Visit(Literal literal) {
    Console.Write(literal.Value);
    return null;
  }

  void PrintBinaryOperator(BinaryOperator binOp, char opChar) {
    Print(binOp.Left);
    Console.Write(" {0} ", opChar);
    Print(binOp.Right);
  }

  object IExprVisitor&lt;object&gt;.Visit(Add add) {
    PrintBinaryOperator(add, '+');
    return null;
  }

  object IExprVisitor&lt;object&gt;.Visit(Subtract subtract) {
    PrintBinaryOperator(subtract, '-');
    return null;
  }

  object IExprVisitor&lt;object&gt;.Visit(Multiply multiply) {
    PrintBinaryOperator(multiply, '*');
    return null;
  }

  object IExprVisitor&lt;object&gt;.Visit(Divide divide) {
    PrintBinaryOperator(divide, '/');
    return null;
  }
}
</pre></code>

<p>As this shows, using a parameterized type for the visitor interface is awkward
when you don't have anything to return. What I would want to do is implement
<code>IExprVisitor&lt;void&gt;</code> but <code>void</code>
is not allowed as a type parameter so I use <code>object</code>
instead. This means I have to return something, so I always return
<code>null</code>. This should be seen as awkwardness introduced by
my choice of interface styles, not of the visitor pattern itself.</p>
<p>The visitor pattern shares some of the advantages of the procedural approach,
in that it is easy to add new operations and new operations can be added
dynamically, but we lose the natural ability to add new data types that is
normally associated with an object-oriented approach as well as lose
representation encapsulation since the instance fields need to be visible to the
visitors. We kept the natural size optimization and the completeness
verification by the compiler, however. To demonstrate the difficult of adding a
new type, I will again add Power. The first step is to add the new
<code>Power</code> type.</p>

<code><pre>
public class Power: BinaryOperator {
  public Power(Expr left, Expr right) : base(left, right) { }

  public override R Accept&lt;R&gt;(IExprVisitor&lt;R&gt; visitor) {
    return visitor.Visit(this);
  }
}
</pre></code>

<p>Now we need to add a method to <code>IExprVisitor&lt;R&gt;</code>
for the <code>Power</code> type that each visitor will need to implement. It looks like,</p>

<code><pre>
R Visit(Power power);
</pre></code>

<p>Next I need to add a method for in each visitor to handle the new class. For
<code>Evaluator</code> it is,</p>

<code><pre>
double IExprVisitor&lt;double&gt;.Visit(Power power) {
  return Math.Pow(Evaluate(power.Left), Evaluate(power.Right));
}
</pre></code>

<p>And and for the <code>Printer</code> class it looks like.</p>

<code><pre>
object IExprVisitor&lt;object&gt;.Visit(Power power) {
  PrintBinaryOperator(power, '^');
  return null;
}
</pre></code>

<p>This is just one way to apply the visitor pattern to the expression problem.
Other ways have their own strengths and weaknesses by trading off different
advantages and disadvantages of the the procedural and object-oriented styles.
Some visitor pattern applications have their own unique advantages, such as being able
to mesh functionality by executing several visitors concurrently such as
evaluating and printing.</p>
<p>Next time we will investigate a different modified object-oriented approach
that is sometimes dubbed groups but I will call layers.</p>
<hr>
<p><a name="note1" href="#ref-note1">1</a> - This application of the visitor
pattern is patterned after one from <a href="http://research.microsoft.com/~akenn/generics/gadtoop.pdf">Generalized Algebraic Data Types and Object-Oriented Programming</a>.
<a href="#ref-note1"><< back <<</a></p>
