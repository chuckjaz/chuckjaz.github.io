---
layout: post
title:  "Operators, Generics and Policies"
date:   2007-07-25 22:46:00
categories: [Programming]
permalink: post/2007/07/25/Operators-Generics-and-Policies.aspx
---
<div class="text"><p>One of the limitations of C# generics is you cannot abstract over operators.
That is not completely true, you can if the base class used in <code><strong>where</strong></code> clause has
operators; but, since operators are more useful, and more common, on value types,
that is only marginally helpful. Also, this abstraction is provided by the base
class not by using generics. What I would like to do is declare that my type&nbsp;
parameter requires the type to implement a particular operator. For example, to create a generic <code>Add()</code>
method I would like to specify that the type parameter must implement the <code>
+</code> operator and then be able to use the <code>+</code> operator in my code
such as,</p>
<pre><span style="color: blue"><em>// Invalid C#</em></span>
<strong>static</strong> T Add&lt;T&gt;(T a, T b) <strong>where</strong> T: T <strong>operator</strong> +(T, T) {
    <strong>return</strong> a + b;
}</pre>

<p>C# doesn't support this but you can get somewhat close by supplying a policy
<code><strong>struct</strong></code> like I described in
<a href="post/2007/07/02/C-Mixins-Sort-of.aspx">
this post</a>. The policy <code><strong>interface</strong></code> and the <code><strong>int</strong></code> policy <code><strong>struct</strong></code> would
look like,</p>
<pre><b>interface</b> IAddPolicy&lt;T&gt; {
    T Add(T a, T b);
}

<b>struct</b> IntAddPolicy : IAddPolicy&lt;<b>int</b>&gt; {
    <b>public</b> <b>int</b> Add(<b>int</b> a, <b>int</b> b) { <b>return</b> a + b; }
}</pre>
<p>This allows you to write the following,</p>
<pre><b>static</b> T Add&lt;T, P&gt;(T a, T b) <strong>where</strong> P: <b>struct</b>, IAddPolicy&lt;T&gt; {
    <b>return</b> <b>new</b> P().Add(a, b);
}</pre>
<p>Unfortunately the policy <code><strong>struct</strong></code> cannot be
inferred from the parameters so you have to supply the type parameter
explicitly,</p>
<pre><strong>var</strong> i = Add&lt;<b>int</b>, IntAddPolicy&gt;(3, 4);</pre>
<p>which doesn't look pretty and generates less than efficient code for the <code>Add()</code>
method on an i386 architecture,</p>
<pre>00000000  sub         esp,8
00000003  xor         eax,eax
00000005  mov         dword ptr [esp],eax
00000008  mov         dword ptr [esp+4],eax
0000000c  mov         byte ptr [esp],0
00000010  movsx       eax,byte ptr [esp]
00000014  mov         byte ptr [esp+4],al
00000018  add         ecx,edx
0000001a  mov         eax,ecx
0000001c  add         esp,8
0000001f  ret              </pre>
<p>This generates code to initialize the policy
structure we don't actually use. We can get rid of that by recasting this a bit to save the dummy <code><strong>struct</strong></code> allocation by putting the
<code>Add()</code> method in a wrapping class that takes the type parameters.
The type would allocate the <code><strong>struct</strong></code> instead of the
method. This looks
like,</p>
<pre><b>class</b> Adder&lt;T, P&gt;
    <strong>where</strong> P: <b>struct</b>, IAddPolicy&lt;T&gt;
{
    <b>static</b> P AddPolicy = <b>new</b> P();
    <b>static</b> <b>public</b> T Add(T a, T b) { <b>return</b> AddPolicy.Add(a, b); }
}</pre>
<p>which generates the following code for <code>Add()</code>,</p>
<pre>00000000  add         ecx,edx
00000002  mov         eax,ecx
00000004  ret              </pre>
<p>This can be called like,</p>
<pre><strong>var</strong> i2 = Adder&lt;<b>int</b>, IntAddPolicy&gt;.Add(3, 4);</pre>
<p>Note that the code in <code>IntAddPolicy</code> gets inlined into the <code>
Add()</code> method. Even though this doesn't get inlined into the caller of
<code>Add(),</code> it will still be pretty quick with the branch prediction and
register aliasing of modern processors. This means that you are not paying much
(and in many cases nothing) in code quality for using an <code>AddPolicy</code> over writing out the <code>
Add()</code> method explicitly.</p>
<p>To use the <code>Add()</code> method above with another type, such as <code>
<strong>double</strong></code>, you need to create another
add policy <code><strong>struct</strong></code>. This would look like,</p>
<pre><b>struct</b> DoubleAddPolicy : IAddPolicy&lt;<b>double</b>&gt; {
    <b>public</b> <b>double</b> Add(<b>double</b> a, <b>double</b> b) { <b>return</b> a + b; }
}</pre>
<p>Using this <code><strong>struct</strong></code> to call <code>Add()</code>
will produce the following code for the <code>Add()</code> method,</p>
<pre>00000000  fld         qword ptr [esp+0Ch]
00000004  fadd        qword ptr [esp+4]
00000008  ret         10h  </pre>
<p>This is not quite as good as the code generated for <code><strong>int</strong></code>
because it uses the stack to pass the parameters but it is still pretty fast.</p>
<p>We now have a method that can add two values of any type <code>T</code> for
which an <code>AddPolicy</code> can be created. This is pretty close to, and
more general than, what I originally wanted. Unfortunately, even though the code
generated is good, calling this code is very awkward; we have to call a static
method of a parameterized class and the policy <code><strong>struct</strong></code>
prevents the compiler from being able to use type inferencing to supply the
parameters for us. Next time I will describe a more practical application of
this technique where the awkwardness is not as noticiable.</p>
</div>
