---
layout: post
title:  "PolyDictionary III: Cast away"
date:   2007-03-26 19:02:00
categories: [Programming]
permalink: post/2007/03/26/PolyDictionary-III-Cast-away.aspx
---
<p>The astute among you realized right away that <code>PolyDictionary</code> 
does not really avoid a cast, it just hides it. The <code>TryGetValue()</code>
has a cast in it, highlighted below,</p>
<pre>        <b>public</b> <b>bool</b> TryGetValue&lt;K, V&gt;(Key&lt;K, V&gt; key, <b>out</b> V value) {
            <b>object</b> objValue;
            <b>if</b> (_table.TryGetValue(key, <b>out</b> objValue)) {
                value = <span style="background-color:#FFFF00">(V)objValue</span>;
                <b>return</b> <b>true</b>;
            }
            value = <b>default</b>(V);
            <b>return</b> <b>false</b>;
        }</pre>
<p>So, can we get rid of the this cast? Yes, but the results are not very
pleasant. One approach is to create one dictionary for each unique value type
and then find that dictionary by that type and index into it using the key,
instead of storing all the values in the same dictionary. This sounds similar to the
original problem! All we need is a <code>PolyDictionary</code> that maps <code>
T</code> to <code>Dictionary&lt;Key&lt;T&gt;, T&gt;</code>. We could code this but we
couldn't run it because we would be defining <code>PolyDictionary</code> in
terms of itself, consuming all of memory in the process. We need something like <code>
PolyDictionary</code> but isn't a <code>PolyDictionary</code>.</p>
<p>OK, things will now get ugly. Have your barf bags on the stand-by! And, for
the love of God, don't ever put this in a production application! </p>
<p>Now that is off my chest, here is one technique to avoid the cast. You can
create a nested class <code>SubDictionary</code> that has a static instance
variable which maps <code>PolyDictionaries</code> to <code>Dictionary&lt;Key&lt;T&gt;, T&gt;</code>.
We can then parameterize <code>SubDictionary</code> by <code>T</code> which will
create a unique dictionary for each <code>T</code> (I warned you it was ugly).
This is because the CLR treats every instantiation of <code>SubDictionary&lt;T&gt;</code> as a
unique type with unique static variables. Essentially we are tricking the CLR
into producing a dictionary of <code>T</code> to <code>Dictionary&lt;PolyDictionary,Dictionary&lt;Key&lt;T&gt;,T&gt;&gt;</code>.
Since the variable is static we need to find the version that is for the
instance we are using, hence the key of <code>PolyDictionary</code>.</p>
<p>Here is the example,</p>
<pre>    /* Never, Never use this code! */
    <b>class</b> PolyDictionary {

        <b>class</b> SubDictionaries&lt;T&gt; {
            <b>internal</b> <b>static</b> Dictionary&lt;PolyDictionary, Dictionary&lt;Key&lt;T&gt;, T&gt;&gt; Dictionaries =
                <b>new</b> Dictionary&lt;PolyDictionary,Dictionary&lt;Key&lt;T&gt;,T&gt;&gt;();
        }

        <b>public</b> PolyDictionary() { }

        <b>public</b> <b>void</b> Add&lt;T&gt;(Key&lt;T&gt; key, T value) {
            Dictionary&lt;Key&lt;T&gt;, T&gt; subDictionary = GetDictionary(key);
            subDictionary.Add(key, value);
        }

        <b>public</b> T Get&lt;T&gt;(Key&lt;T&gt; key) {
            Dictionary&lt;Key&lt;T&gt;, T&gt; subDictionary = FindDictionary(key);
            <b>if</b> (subDictionary != <b>null</b>)
                <b>return</b> subDictionary[key];
            <b>throw</b> <b>new</b> KeyNotFoundException();
        }

        <b>public</b> <b>bool</b> TryGetValue&lt;T&gt;(Key&lt;T&gt; key, <b>out</b> T value) {
            Dictionary&lt;Key&lt;T&gt;, T&gt; subDictionary = FindDictionary(key);
            <b>if</b> (subDictionary != <b>null</b>)
                <b>return</b> subDictionary.TryGetValue(key, <b>out</b> value);
            <b>else</b> {
                value = <b>default</b>(T);
                <b>return</b> <b>false</b>;
            }
        }

        Dictionary&lt;Key&lt;T&gt;, T&gt; FindDictionary&lt;T&gt;(Key&lt;T&gt; key) {
            Dictionary&lt;Key&lt;T&gt;, T&gt; subDictionary;
            <b>if</b> (!SubDictionaries&lt;T&gt;.Dictionaries.TryGetValue(<b>this</b>, <b>out</b> subDictionary))
                <b>return</b> <b>null</b>;
            <b>return</b> subDictionary;
        }

        Dictionary&lt;Key&lt;T&gt;, T&gt; GetDictionary&lt;T&gt;(Key&lt;T&gt; key) {
            Dictionary&lt;Key&lt;T&gt;, T&gt; subDictionary;
            <b>if</b> (!SubDictionaries&lt;T&gt;.Dictionaries.TryGetValue(<b>this</b>, <b>out</b> subDictionary)) {
                subDictionary = <b>new</b> Dictionary&lt;Key&lt;T&gt;, T&gt;();
                SubDictionaries&lt;T&gt;.Dictionaries.Add(<b>this</b>, subDictionary);
            }
            <b>return</b> subDictionary;
        }
    }</pre>

<p>This code has so many problems I don't know where to start. For one, it
never frees memory, the dictionaries are in memory and will never be
collected. I only include it to show that it can be done. As you can see,
sometimes it is better to have a cast. There might be better techniques that
eliminate some of the problems but my feeling is, why bother. The cast is not that
bad.</p>
