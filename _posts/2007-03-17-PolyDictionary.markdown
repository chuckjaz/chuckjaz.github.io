---
layout: post
title:  "PolyDictionary"
date:   2007-03-17 13:30:00
categories: [Programming]
---
<p>CLR's standard <code>Dictionary&lt;K,T&gt;</code> allows you to express the 
type of the key and the type value so you don't have to cast on the way in
and out of a
dictionary. For example you can cache a bunch of strings based on some integer
key. I use this in the following program that takes an integer number and writes
out the English equivalent.</p>
<pre>    <b>class</b> Program {
        <b>static</b> Dictionary&lt;<b>int</b>, <b>string</b>&gt; names = <b>new</b> Dictionary&lt;<b>int</b>, <b>string</b>&gt;();

        <b>static</b> Program() {
            names[0] = "zero";
            names[1] = "one";
            names[2] = "two";
            names[3] = "three";
            names[4] = "four";
            names[5] = "five";
            names[6] = "six";
            names[7] = "seven";
            names[8] = "eight";
            names[9] = "nine";
            names[10] = "ten";
            names[11] = "eleven";
            names[12] = "twelve";
            names[13] = "thirteen";
            names[14] = "fourteen";
            names[15] = "fifteen";
            names[16] = "sixteen";
            names[17] = "seventeen";
            names[18] = "eighteen";
            names[19] = "nineteen";
            names[20] = "twenty";
            names[30] = "thirty";
            names[40] = "forty";
            names[50] = "fifty";
            names[60] = "sixty";
            names[70] = "seventy";
            names[80] = "eighty";
            names[90] = "ninety";
            names[100] = "hundred";
            names[1000] = "thousand";
            names[1000000] = "million";
            names[1000000000] = "billion";
        }

        <b>static</b> <b>int</b> WriteValue(<b>int</b> value, <b>int</b> place) {
            <b>if</b> (value &gt;= place) {
                <b>int</b> subValue = value / place;
                WriteValue(subValue);
                Console.Write(" {0} ", names[place]);
                value = value % place;
            }
            <b>return</b> value;
        }

        <b>static</b> <b>void</b> WriteValue(<b>int</b> value) {
            <b>int</b> originalValue = value;
            <b>if</b> (value &lt; 0) {
                value = -value;
                Console.WriteLine("negative ");
            }
            value = WriteValue(value, 1000000000);
            value = WriteValue(value, 1000000);
            value = WriteValue(value, 1000);
            value = WriteValue(value, 100);
            <b>if</b> (value &gt; 20) {
                <b>int</b> subValue = (value / 10) * 10;
                Console.Write(names[subValue]);
                value = value % 10;
                <b>if</b> (value == 0)
                    <b>return</b>;
                Console.Write("-");
            }
            <b>if</b> (value != 0 || originalValue == 0)
                Console.Write(names[value]);
        }

        <b>static</b> <b>void</b> Main(<b>string</b>[] args) {
            <b>while</b> (<b>true</b>) {
                Console.Write("Enter number: ");
                <b>string</b> line = Console.ReadLine();
                <b>if</b> (line == "exit" || <b>string</b>.IsNullOrEmpty(line))
                    <b>break</b>;
                <b>int</b> value;
                <b>if</b> (<b>int</b>.TryParse(line, <b>out</b> value)) {
                    WriteValue(value);
                    Console.WriteLine();
                }
                <b>else</b>
                    Console.WriteLine("Error: expected number");
            }
        }
    }</pre>

<p>This example doesn't use any casts. The expression <code>names[10]</code> is of type
<code><strong>string</strong></code> and the index is of type <code><strong>int</strong></code>.
This is great when the values of the dictionary are all homogeneous as <code>name</code> is
above. But what if I want to store values of varying types but still not use a
cast. In other words, I want a heterogeneous, type-checked dictionary. If I
don't mind casts, I can just use a <code>Hashtable</code> as in,</p>
<pre>    Hashtable table = <b>new</b> Hashtable();

    table.Add("k1", "one");
    table.Add("k2", 2);
    table.Add(3, 3.33);

    <b>string</b> v1 = (<b>string</b>)table["k1"];
    <b>int</b> v2 = (<b>int</b>)table["k2"];
    <b>double</b> v3 = (<b>double</b>)table[3];

    Console.WriteLine("v1 = {0}", v1);
    Console.WriteLine("v2 = {0}", v2);
    Console.WriteLine("v3 = {0}", v3);</pre>
<p>Unfortunately, if I later change the code to,</p>
<pre>    table.Add("k1", <span style="background-color:yellow">1</span>);
    table.Add("k2", 2);
    table.Add(3, 3.33);

    <b>string</b> v1 = (<b>string</b>)table["k1"];
    <b>int</b> v2 = (<b>int</b>)table["k2"];
    <b>double</b> v3 = (<b>double</b>)table[3];</pre>
<p>The compiler will dutifully compile the code and I will get a runtime error
at the first use of the dictionary when it tries to cast an integer to a string.
I would like the compiler to catch this type of error instead of having to wait
until my user does. </p>
<p>I have keys of varying type and values of varying type. How can I get
this and type verification as well? In other words, what I want is something
like,</p>
<pre>    dictionary.Add(k1, "one");
    dictionary.Add(k2, 2);
    dictionary.Add(k3, 3.33);

    <b>string</b> v1 = dictionary[k1];
    <b>int</b> v2 = dictionary[k2];
    <b>double</b> v3 = dictionary[k3];</pre>
<p>What I want to do is be able to infer the returns result of the expression
<code>dictionary[k1]</code> from the key. In the above example, there should be
something about the <code>k1</code> key that tells the compiler that the result will be a
<code>string</code> and when I change it later to an <code>int</code>, I want the
compiler to catch the assignment to <code>string</code> as a type error.</p>
<p>To do this I need a method that can infer the type of the return result from
a parameter. A simple example of such a method is,</p>
<pre>    T NoOp(T value) { <strong>return</strong> value; }</pre>
<p>which returns the same value of the same type as it retrieves for any value
of any type. The return type is inferred from the parameter <code>value</code>'s type.</p>
<p>This is not good enough yet. We don't want the key type to be the same as the value type;
we want the value type to be arbitrary. To do this we need to
encode the value's type in the key. One way is to have the
key's type include the value's type as part of the key's type definition. This can be done
through supplying a dummy type parameter, a type parameter that is not needed in the definition of
the type but the dummy type parameter is carried around by the type and can be
used by type inferencing. To do this we need
a method something like,</p>
<pre>    T Get&lt;T&gt;(Key&lt;T&gt; key) { ... }</pre>
<p>which takes a key and extracts the return type from the keys definition. This
requires a <code>Key</code> class definition that has one type parameter which is simply,</p>
<pre>    <b>class</b> Key&lt;T&gt; { <b>public</b> Key() { } }
</pre>
<p>Note we don't use <code>T</code> in <code>Key</code>, it is just there so type inferencing will find it. We can now
define some keys,</p>
<pre>    Key&lt;<b>string</b>&gt; k1 = <b>new</b> Key&lt;<b>string</b>&gt;();
    Key&lt;<b>int</b>&gt; k2 = <b>new</b> Key&lt;<b>int</b>&gt;();
    Key&lt;<b>double</b>&gt; k3 = <b>new</b> Key&lt;<b>double</b>&gt;();
</pre>
<p>The keys are just arbitrary instances. Type
inferencing will now be able to extract <code>T</code> from the key's type and use it as the
return result! This means that <code>Get(k1)</code> will be&nbsp; of type <code>string</code> and
<code>Get(k2)</code> will be of type
<code>int</code>.</p>
<p>Type inferencing also works the same way for other parameters of the method.
For the <code>Add()</code> method we can define it as,</p>
<pre>    <b>public</b> <b>void</b> Add&lt;T&gt;(Key&lt;T&gt; key, T value);</pre>
<p>With this&nbsp; <code>Add()</code> the code above will compile because the type of <code>value</code> is also
inferred from the key!</p>
<p>Unfortunately, the <code>this</code> property does not accept type parameters
so we cannot get the code, </p>
<pre>    <b>string</b> v1 = dictionary[k1];
    <b>int</b> v2 = dictionary[k2];
    <b>double</b> v3 = dictionary[k3];</pre>

<p>to compile; but if we replace the array indexers with a <code>Get()</code>
method we can get close, as in,</p>
<pre>    <b>string</b> v1 = dictionary.Get(k1);
    <b>int</b> v2 = dictionary.Get(k2);
    <b>double</b> v3 = dictionary.Get(k3);</pre>

<p>This works if dictionary has a <code>Get()</code> method like we
described above,</p>
<pre>    <b>public</b> T Get&lt;T&gt;(Key&lt;T&gt; key);</pre>
<p>There we have it! With the <code>Add()</code> and <code>Get()</code> methods we have the beginnings of
a dictionary that can hold any value of any type and retrieve the value back
without requiring our use of the dictionary to use a cast. A complete example is given below,</p>
<pre>    <b>class</b> Key&lt;T&gt; { <b>public</b> Key() { } }

    <b>class</b> PolyDictionary {
        <b>private</b> Dictionary&lt;<b>object</b>, <b>object</b>&gt; _table;

        <b>public</b> PolyDictionary() {
            _table = <b>new</b> Dictionary&lt;<b>object</b>, <b>object</b>&gt;();
        }

        <b>public</b> <b>void</b> Add&lt;T&gt;(Key&lt;T&gt; key, T value) {
            _table.Add(key, value);
        }

        <b>public</b> <b>bool</b> Contains&lt;T&gt;(Key&lt;T&gt; key) {
            <b>return</b> _table.ContainsKey(key);
        }

        <b>public</b> <b>void</b> Remove&lt;T&gt;(Key&lt;T&gt; key) {
            _table.Remove(key);
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

        <b>public</b> T Get&lt;T&gt;(Key&lt;T&gt; key) {
            T value;
            <b>if</b> (!TryGetValue(key, <b>out</b> value))
                <b>throw</b> <b>new</b> KeyNotFoundException();
            <b>return</b> value;
        }

        <b>public</b> <b>void</b> Set&lt;T&gt;(Key&lt;T&gt; key, T value) {
            _table[key] = value;
        }
    }</pre>
