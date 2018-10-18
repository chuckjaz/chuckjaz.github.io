---
layout: post
title:  "Zero the value is not"
date:   2008-10-31 06:32:00
categories: [Programming]
---
<div class="text"><p>One of my pet-peeves is code written like this,</p>
<pre class="code"><strong>if</strong> (0 != value) { ... }</pre>

<p>This reads backwards to me. I hear echoes of
<a href="http://en.wikipedia.org/wiki/Frank_Oz">Frank Oz</a> in my head, "if zero
the value is not". As cute as this dialect is coming from a pointy-eared muppet,
it still takes a bit to translate back into the more common English phrasing,
"if the value is not zero". It seems programmers who insist on writing in this style
have forgotten that their source should be readable as well as executable. Don't
force your readers to have to do grammar gymnastics as well as having to
decipher your algorithmic gynmastics.</p>
<p>Once upon a time, this was considered good style in order to avoid accidentally
writing something like,</p>
<pre class="code"><strong>if</strong> (value = 0) { ... }</pre>

<p>In C#, this is illegal. In most (all?) modern C and C++ compilers, this will
generate a warning. (You do compile with warnings reported as errors right?)
There ceases to be a good reason to write test expressions backwards. Let's let
this style rest in peace.</p>
<p>Even if you think the above form has some redeeming value, let's all agree
that any programmer that voluntarily writes something like,</p>
<pre class="code"><strong>if</strong> (<strong>false</strong> == value) { ... }</pre>

<p>should have an electric shock sent directly through their keyboard.</p>
</div>
