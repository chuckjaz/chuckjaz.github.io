---
layout: post
title:  "Linked List VI: Recursion - see recursion"
date:   2006-09-25 10:05:00
categories: [Programming]
---
<p>I finally have a definitive answer to the question of why <span class="code">MyNode</span> is needed.
Consider the following, simplified, version of the <span class="code">LinkedList</span> classes I posed,</p>
<pre>  <b>class</b> C&lt;X&gt; { <b>class</b> N { } }
  <b>class</b> D: C&lt;D.N&gt; { }</pre>
<p>This is really an obfuscated form of</p>
<pre>  <b>class</b> N&lt;X&gt; { }
  <b>class</b> C&lt;X&gt; { }
  <b>class</b> D: C&lt;N&lt;?&gt;&gt; { }</pre>
<p>where ? needs to be replaced by an infinite list if <span class="code">N&lt;N&lt;...&gt;&gt;</span>. In other
words, I am trying to define a type directly in terms of itself, a
<span class="code">T</span> where <span class="code">T=N&lt;T&gt;</span>. This is not
allowed. An additional type is required to break the cycle, such as
<span class="code">M</span> in,</p>

<pre>  <b>class</b> M: N&lt;M&gt; { }</pre>

<p>Then I can write,</p>

<pre>  <b>class</b> D: C&lt;M&gt; { }</pre>

<p>So, there we have it, <span class="code">MyNode</span> is necessary to avoid defining
a type directly in terms
of itself. The unobfuscated version makes this much more clear.
<span class="code">MyNode</span> serves the same role as <span class="code">M</span>
does above.</p>
<p>Thanks again to <a href="http://research.microsoft.com/~akenn/">Andrew</a>
for taking the time to beat this into my thick skull.</p>
