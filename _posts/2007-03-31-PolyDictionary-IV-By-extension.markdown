---
layout: post
title:  "PolyDictionary IV: By extension"
date:   2007-03-31 23:06:00
categories: [Programming]
permalink: post/2007/03/31/PolyDictionary-IV-By-extension.aspx
---
<p>The technique we used to build <code>PolyDictionary</code> can be used in other 
places. For example, you can adapt a non-type-checked store to a
type-checked store by supplying your own <code>Get() </code>and <code>Set()</code>
methods. One place this might be useful is when you encounter something like
CodeDom's <code>UserData</code>. It is intended to allow users or producers
of a CodeDom to annotate a
CodeDom with their own information. Because it uses <code>IDictionary,</code>
using <code>UserData</code> is not type checked at compile
time. You can introduce type checking by using your own methods to access the
dictionary like,</p>
<pre>  <b>public</b> <b>static</b> T Get&lt;T&gt;(IDictionary dictionary, Key&lt;T&gt; key) {
      <b>return</b> (T)dictionary[key];
  }

  <b>public</b> <b>static</b> <b>void</b> Set&lt;T&gt;(IDictionary dictionary, Key&lt;T&gt; key, T value) {
      dictionary[key] = value;
  }</pre>
<p>which uses the <code>Key&lt;T&gt;</code> class from last time. This allows
the compiler to ensure that the type of the value you get out is the same as you
put in and vise versa.</p>
<p>With C# 3.0, however, you can get even more creative. You can create extension
methods for <code>IDictionary</code> that makes it appear that all <code>IDictionary</code>s have a type
checked <code>Get()</code> and <code>Set()</code> like,</p>
<pre>    <b>static</b> <b>class</b> IDictionaryExtensions {
        <b>public</b> <b>static</b> T Get&lt;T&gt;(<b>this</b> IDictionary dictionary, Key&lt;T&gt; key) {
            <b>return</b> (T)dictionary[key];
        }
        <b>public</b> <b>static</b> <b>void</b> Set&lt;T&gt;(<b>this</b> IDictionary dictionary, Key&lt;T&gt; key, T value) {
            dictionary[key] = value;
        }
    }</pre>
<p>This allows you to write code like,</p>
<pre>    Key&lt;<b>string</b>&gt; k1 = <b>new</b> Key&lt;<b>string</b>&gt;();
    Key&lt;<b>int</b>&gt; k2 = <b>new</b> Key&lt;<b>int</b>&gt;();
    Key&lt;<b>double</b>&gt; k3 = <b>new</b> Key&lt;<b>double</b>&gt;();

    IDictionary dictionary = <b>new</b> Hashtable();

    dictionary.Set(k1, "one");
    dictionary.Set(k2, 2);
    dictionary.Set(k3, 3.33);

    <b>string</b> v1 = dictionary.Get(k1);
    <b>int</b> v2 = dictionary.Get(k2);
    <b>double</b> v3 = dictionary.Get(k3);

    Console.WriteLine("v1 = {0}", v1);
    Console.WriteLine("v2 = {0}", v2);
    Console.WriteLine("v3 = {0}", v3);</pre>
<p>which allows us to use an <code>IDictionary</code> almost identically to a
<code>PolyDictionary</code>.&nbsp;</p>
<p>This works great for <code>Get()</code> and <code>Set()</code> but this
doesn't work for <code>Add()</code>, unfortunately.
The compiler accepts it but it doesn't do what we want. If we use a value
of an incorrect type for a particular key we want the compiler to generate and
error. In other words, if we change the <code>Set()</code> call for <code>k1</code> above to,</p>
<pre>  dictionary.<span style="background-color:#FFFF00">Add</span>(k1, <span style="background-color:#FFFF00">1</span>);</pre>
<p>we want the compiler to generate an error. This doesn't happen because of method overloading. The object version of
Add,
<code>Add(<strong>object</strong> value)</code> is called if the type is incorrect; this is not what we want.
We could add an <code>AddSafe()</code> method to the extension class but then we
would have to remember to call it instead of <code>Add()</code>, which defeats
the purpose in my opinion.</p>
