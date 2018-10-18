---
layout: post
title:  "To b | !b, that is the question"
date:   2007-12-19 17:10:00
categories: [Programming]
---
<div class="text"><p>When I was reading
<a href="http://www.amazon.com/First-Order-Logic-Raymond-M-Smullyan/dp/0486683702/ref=sr_11_1/102-8843584-4849732?ie=UTF8&amp;qid=1192783059&amp;sr=11-1">First-Order Logic</a> by
<a href="http://en.wikipedia.org/wiki/Raymond_Smullyan">Raymond M. Smullyan</a> I
was fascinated by
the second chapter in which he describes a method for proving a propositional
logic expression is a tautology. This method is called the Tableau Method. The Tableau Method is a
rather straight forward way to prove a statement is always going to be true by
demonstrating that asserting it is false will result in a contradiction. The
propositional logic expression language Dr. Smullyan uses is rather simple. I
present a modified version of it here. My version
contains a not operator (!), a conjunction operator (&amp;), a disjunction operator
(|) and an implication operator (→).<sup><a name="note1_1ref" href="#note1_1">1</a></sup> For example,</p>
<p style="margin-left: 40px;">p &amp; q</p>
<p>is true when both <em>p</em> and <em>q</em> are true; </p>
<p style="margin-left: 40px">p | q</p>
<p>is true when either <em>p</em> is true or <em>q</em> is true;</p>
<p style="margin-left: 40px">p <span class="style1">→ q</span></p>
<p>is true when<em> p </em>is false or <em>p</em> is true and <em>q</em> is also
true; and finally,</p>
<p style="margin-left: 40px">!p</p>
<p>is true only when <em>p</em> is false. This should be very familiar to any
programmer. A propositional logic expression is a tautology if it will always be
true regardless of the value of its propositional variables. An example of a
simple tautology is,</p>
<p style="margin-left: 40px">p | !p</p>
<p>which will be true if <em>p</em> is true or false. This statement is always
true. This can be proven by simply demonstrating that asserting it is false
results in a contradiction. For example, we assert a statement is false by
prefixing it with a capital F or true with a capital T. For the above expression
this would look like,</p>
<p style="margin-left: 40px">F p | !p</p>
<p>For this to be false both of the sub-expressions would also have to be
false. This would then look like</p>
<p style="margin-left: 40px">F p<br>F !p</p>
<p>For <em>!p</em> to be false <em>p</em> would have to be true, which looks
like,</p>
<p style="margin-left: 40px">T p</p>
<p>Now we have a contradiction, according to propositional logic, <em>p</em> cannot be both true and false so <em>
p | !p</em> must always be true. Using the Tableau Method this would look like,</p>
<p style="margin-left: 40px">(1) F p | !p<br>(2) F p (1)<br>(3) F !p (1)<br>(4) T p (3)<br>X</p>
<p>where the X indicates that the tableau ends in a contradiction (i.e. is
closed) and the
numbers to the left label the expressions and the numbers to the right indicate
which expression the assertion is derived from. To produce a tableau you first
assert the expression is false then use the following table to add assertions,</p>
<table style="width: 100%">
	<tbody><tr>
		<td><strong>Expression of the form</strong></td>
		<td><strong>Asserts</strong></td>
	</tr>
	<tr>
		<td style="width: 273px">T !p</td>
		<td>F p</td>
	</tr>
	<tr>
		<td style="width: 273px">F !p</td>
		<td>T p</td>
	</tr>
	<tr>
		<td style="width: 273px">T p &amp; q</td>
		<td>T p and T q</td>
	</tr>
	<tr>
		<td style="width: 273px">F p &amp; q</td>
		<td>F p or F q</td>
	</tr>
	<tr>
		<td style="width: 273px">T p | q</td>
		<td>T p or T q</td>
	</tr>
	<tr>
		<td style="width: 273px">F p | q</td>
		<td>F p and F q</td>
	</tr>
	<tr>
		<td style="width: 273px">T p → q</td>
		<td>F p or T q</td>
	</tr>
	<tr>
		<td style="width: 273px">F p → q</td>
		<td>T p and F q</td>
	</tr>
</tbody></table>
<p>When two expressions are asserted the table might branch. If the above table
says "and" then both statement are asserted directly, but if it contains an "or"
then both assertions must lead to a contradiction independently. For example,
consider wanting to prove the transitive property of the implication operator,
that is <em>(p→q &amp; q→r) → (p→r)</em>, this would start off with adding asserts
until we are required to branch,</p>
<p style="margin-left: 40px">
  (1) F (p→q &amp; q→r) → (p→r)<br>
  (2) T p→q &amp; q→r (1)<br>
  (3) F p→r (1)<br>
  (4) T p→q (2)<br>
  (5) T q→r (2)<br>
  (6) T p (3)<br>
  (7) F r (3)<br>
</p>
<p>We now have to handle (4) and (5) with require us to branch the table. The first branch is
the <em>F p</em> branch,</p>
<p style="margin-left: 40px">
  (8) F p (4)<br>
  X
</p>
<p>This lead immediately to a contradiction because of (6) so this branch of the tableau is closed. We now need
handle the <em>T q</em> branch,</p>
<p style="margin-left: 40px">
  (9) T q (4)
</p>
<p>This doesn't lead to an immediate contradiction so we need to branch again for (5). The first
branch for (5) looks like</p>
<p style="margin-left: 40px">
  (10) F q (5)<br>
  X
</p>
<p>This again leads to an immediate contradiction, we already asserted
that <em>q</em> must be true in (9). The second branch for (5) is the <em>T r</em> branch.</p>
<p style="margin-left: 40px">
  (11) T r (5)<br>
  X
</p>
<p>This also lead to an immediate contradiction because of (7). Since all
branches lead to a contradiction the original expression must always be true.</p>
<p>Putting this altogether we get,</p>
<table>
<tbody><tr>
<td>
<p>
  (1) F (p→q &amp; q→r) → (p→r)<br>
  (2) T p→q &amp; q→r (1)<br>
  (3) F p→r (1)<br>
  (4) T p→q (2)<br>
  (5) T q→r (2)<br>
  (6) T p (3)<br>
  (7) F r (3)<br>
</p>
</td>
</tr>
<tr>
<td>
<table style="text-align: center;">
<tbody><tr>
<td valign="top" style="border-right-color: black;
	border-right-style:solid;
	border-right-width:thin;
	padding-right: 10px">
<p>
  (8) F p (4)<br>
  X
</p>
</td>
<td style="padding-right:10px;">
<p>
  (9) T q (4)
</p>
<table>
<tbody><tr>
<td style="border-right-color: black;
	border-right-style:solid;
	border-right-width:thin;
	padding-right: 10px">
<p>
  (10) F q (5)<br>
  X
</p>
</td>
<td style="padding-right:10px;">
<p>
  (11) T r (5)<br>
  X
</p>
</td>
</tr>
</tbody></table>
</td>
</tr>
</tbody></table>
</td>
</tr>
</tbody></table>
<p>This is a simple and elegant way to demonstrate proof by contradiction. Of
course I just had to write a program to tell me whether an
arbitrary propositional logic expression is a tautology and automatically
generate the tableau.</p>
<p>You can find the source project that is works with the just released Visual
Studio 2008 <a href="/files/Tableaux.zip">here</a>. It requires VS 2008 because I am addicted to some of the new
C# features, especially the <b><code>var</code></b> keyword.</p>
<hr>
<p><a name="note1_1"><sup>1</sup></a> Dr. Sumullyan uses different characters for his operators
but his characters are not as commonly included in browser fonts and are not as easy to
type on a standard keyboard (using <span class="style3">-&gt;</span> as a synonym
of →) as the ones I use. <a href="#note1_1ref">&lt;&lt; back</a></p>
<p><a href="/files/Tableaux.zip">Tableaux.zip (9.88 kb)</a></p></div>
