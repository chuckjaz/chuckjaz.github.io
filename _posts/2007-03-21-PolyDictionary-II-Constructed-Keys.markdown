---
layout: post
title:  "PolyDictionary II: Constructed Keys"
date:   2007-03-21 15:47:00
categories: [Programming]
---
<p>In the last entry we defined a dictionary that can contain a value of any 
type and extra that value out of the dictionary without our code using a cast.
The dictionary itself used a cast (which I show a "way" to remove it next time) but our code
didn't. One problem it had, however, is it used declared keys instead of
constructed keys. A declared key is a key can only be referenced by knowing the
variable it is assigned to; a unique object. A constructed key, on the other
hand, uses object equality instead of object identity for the key. String keys
are constructed keys; you can read them from a file, calculate them or use a
literal value. The dictionary will eventually compare the strings
for equality. In other words, constructed keys can be constructed from data
where declared keys cannot.</p>
<p>Can we modify <code>PolyDictionary</code> to use constructed keys? One way to
do it is to create a new <code>Key</code> type that takes the constructed key
type as a
parameter. One such type is, </p>
<pre>    <b>struct</b> Key&lt;K, V&gt; {
        <b>public</b> K Value;
        <b>public</b> Key(K value) {
            <b>this</b>.Value = value;
        }
        <b>public</b> <b>static</b> <b>implicit</b> <b>operator</b> Key&lt;K, V&gt;(K value) {
            <b>return</b> <b>new</b> Key&lt;K, V&gt;(value);
        }
    }</pre>
<p>This <code><strong>struct</strong></code> just wraps the key value. I use a
<code><strong>struct</strong></code> instead of a <code><strong>class</strong></code>
because structs will do a value comparison for equality by default, classes use
reference equality.</p>
<p>The existing methods of <code>PolyDictionary</code>
do not take this kind of key; but, we can add methods to <code>PolyDictionary</code> that do. For
example, we can create an <code>Add()</code> method that looks like,</p>
<pre>    <b>public</b> <b>void</b> Add&lt;K, V&gt;(Key&lt;K, V&gt; key, V value);</pre>
<p>We can then create a <code>Get()</code> method that looks like,</p>
<pre>    <b>public</b> V Get&lt;K, V&gt;(Key&lt;K, V&gt; key);</pre>
<p>All these methods can be added directly to <code>PolyDictionary</code>
because, through method overloading, they don't interfere with the original,
declared key versions.</p>
<p>Using this dictionary looks like,</p>
<pre>    Key&lt;<b>string</b>, <b>string</b>&gt; k1 = "k1";
    Key&lt;<b>string</b>, <b>int</b>&gt; k2 = "k2";
    Key&lt;<b>int</b>, <b>double</b>&gt; k3 = 3;

    PolyDictionary dictionary = <b>new</b> PolyDictionary();

    dictionary.Add(k1, "one");
    dictionary.Add(k2, 2);
    dictionary.Add(k3, 3.33);

    <b>string</b> v1 = dictionary.Get(k1);
    <b>int</b> v2 = dictionary.Get(k2);
    <b>double</b> v3 = dictionary.Get(k3);

    Console.WriteLine("v1 = {0}", v1);
    Console.WriteLine("v2 = {0}", v2);
    Console.WriteLine("v3 = {0}", v3);
</pre>
<p>The complete example of <code>PolyDictionary</code> follows,</p>
<pre>    <b>class</b> PolyDictionary {
        <b>private</b> Dictionary&lt;<b>object</b>, <b>object</b>&gt; _table;

        <b>public</b> PolyDictionary() {
            _table = <b>new</b> Dictionary&lt;<b>object</b>, <b>object</b>&gt;();
        }

        <b>public</b> <b>void</b> Add&lt;K, V&gt;(Key&lt;K, V&gt; key, V value) {
            _table.Add(key, value);
        }

        <b>public</b> <b>void</b> Add&lt;T&gt;(Key&lt;T&gt; key, T value) {
            _table.Add(key, value);
        }

        <b>public</b> <b>bool</b> Contains&lt;K, V&gt;(Key&lt;K, V&gt; key) {
            <b>return</b> _table.ContainsKey(key);
        }

        <b>public</b> <b>bool</b> Contains&lt;T&gt;(Key&lt;T&gt; key) {
            <b>return</b> _table.ContainsKey(key);
        }

        <b>public</b> <b>void</b> Remove&lt;K, V&gt;(Key&lt;K, V&gt; key) {
            _table.Remove(key);
        }

        <b>public</b> <b>void</b> Remove&lt;T&gt;(Key&lt;T&gt; key) {
            _table.Remove(key);
        }

        <b>public</b> <b>bool</b> TryGetValue&lt;K, V&gt;(Key&lt;K, V&gt; key, <b>out</b> V value) {
            <b>object</b> objValue;
            <b>if</b> (_table.TryGetValue(key, <b>out</b> objValue)) {
                value = (V)objValue;
                <b>return</b> <b>true</b>;
            }
            value = <b>default</b>(V);
            <b>return</b> <b>false</b>;
        }

        <b>public</b> <b>bool</b> TryGetValue&lt;T&gt;(Key&lt;T&gt; key, <b>out</b> T value) {
            <b>object</b> objValue;
            <b>if</b> (_table.TryGetValue(key, <b>out</b> objValue)) {
                value = (T)objValue;
                <b>return</b> <b>true</b>;
            }
            value = <b>default</b>(T);
            <b>return</b> <b>false</b>;
        }

        <b>public</b> V Get&lt;K, V&gt;(Key&lt;K, V&gt; key) {
            <b>return</b> (V)_table[key];
        }

        <b>public</b> T Get&lt;T&gt;(Key&lt;T&gt; key) {
            <b>return</b> (T)_table[key];
        }

        <b>public</b> <b>void</b> Set&lt;K, V&gt;(Key&lt;K, V&gt; key, V value) {
            _table[key] = value;
        }

        <b>public</b> <b>void</b> Set&lt;T&gt;(Key&lt;T&gt; key, T value) {
            _table[key] = value;
        }
    }</pre>
<p>Note: the implementation changed a bit from the previous example. The <code>
Get()</code> methods now use the <code><strong>this</strong></code> property
instead of calling <code>TryGetValue()</code>. This allows the <code>Get()
</code>methods to be inlined at the cost of an additional cast or two in the
code.</p>
