---
layout: post
title:  "Linked Lists II: Simple Generics"
date:   2006-04-01 12:25:00
categories: [Programming]
---
<p>In the last entry we looked at implementing a collection class 
using a linked list data structure. This time I will show some ways to use that implementation
to implement the generic
interfaces introduced last time. The complete class looks like,</p>

<pre>    <b>class</b> LinkedList: IEnumerable {

        <b>private</b> <b>class</b> Node {
            <b>public</b> <b>object</b> Data;
            <b>public</b> Node Next;
        }

        <b>private</b> Node _last;

        <b>public</b> LinkedList() { }

        <b>public</b> <b>void</b> Add(<b>object</b> value) {
            Node newNode = <b>new</b> Node();
            newNode.Data = value;
            <b>if</b> (_last == <b>null</b>) {
                newNode.Next = newNode;
                _last = newNode;
            }
            <b>else</b> {
                newNode.Next = _last.Next;
                _last.Next = newNode;
                _last = newNode;
            }
        }

        <b>public</b> <b>void</b> Clear() {
            _last = <b>null</b>;
        }

        <b>public</b> IEnumerator GetEnumerator() {
            <b>if</b> (_last != <b>null</b>) {
                Node current = _last;
                <b>do</b> {
                    current = current.Next;
                    yield <b>return</b> current.Data;
                } <b>while</b> (current != _last);
            }
        }
    }</pre>
<p>The interface we want to implement is,</p>
<pre>    <b>interface</b> ILinkedList&lt;T&gt;: IEnumerable&lt;T&gt; {
        <b>void</b> Add(T value);
        <b>void</b> Clear();
    }</pre>
<p>One obvious, but bad, way to implement this interface is to
subclass <span class="code">LinkedList</span> like,</p>
<pre>    <b>class</b> LinkedList&lt;T&gt;: LinkedList, ILinkedList&lt;T&gt;, IEnumerable&lt;T&gt; {

        <b>public</b> <b>void</b> Add(T value) { <b>base</b>.Add(value); }

        IEnumerator&lt;T&gt; IEnumerable&lt;T&gt;.GetEnumerator() {
            <b>foreach</b> (T item <b>in</b> <b>this</b>)
                yield <b>return</b> item;
        }
    }</pre>
<p>The reason this is so bad is because given,</p>
<pre>    LinkedList&lt;<b>string</b>&gt; list = <b>new</b> LinkedList&lt;<b>string</b>&gt;();</pre>
<p>the following statements are both legal,</p>
<pre>    list.Add("some string");
    list.Add(1);</pre>
<p>The reason is because the first statement calls the <span class="code">
LinkedList&lt;T&gt;.Add(T value)</span> above, as expected, but the second will call
<span class="code">LinkedList.Add(<b>object</b> value)</span>, because the
generic <span class="code">Add</span> is overloaded with the the non-generic version. Even if it
was obscured by the new <span class="code">Add</span> method you could still access
<span class="code">Add(<b>object</b> value)</span> by,</p>
<pre>    LinkedList list2 = list;
    list2.Add(1);</pre>
<p>We wanted the compiler to be able to flag when we add a
non-string to the list as an error; this implementation doesn't do that. A slightly better approach is
to use delegation instead of
inheritance, as in,</p>
<pre>    <b>class</b> LinkedListD&lt;T&gt;: ILinkedList&lt;T&gt;, IEnumerable&lt;T&gt; {
        <b>private</b> LinkedList _list = <b>new</b> LinkedList();

        <b>public</b> <b>void</b> Add(T value) { _list.Add(value); }

        <b>public</b> <b>void</b> Clear() { _list.Clear(); }

        IEnumerator&lt;T&gt; IEnumerable&lt;T&gt;.GetEnumerator() {
            <b>foreach</b> (T item <b>in</b> _list)
                yield <b>return</b> item;
        }

        IEnumerator IEnumerable.GetEnumerator() {
            <b>return</b> _list.GetEnumerator();
        }
    }</pre>
<p>But this is still not good enough because, not only can we no longer inherit the implementation of <span class="code">
Clear()</span> and <span class="code">GetEnumerator()</span>, we also use
more memory by having to create an instance of <span class="code">LinkedList</span>
to wrap. Additionally, if we wanted to add an <span class="code">Insert()</span>
method, that put values at the beginning of the list instead of the end, we would have to
manually update <span class="code">LinkedListD</span>. A much better approach is to scrap the first implementation and
re-implement the class as a generic class, as in,</p>
<pre>    <b>class</b> LinkedList&lt;T&gt;: ILinkedList&lt;T&gt;, IEnumerable&lt;T&gt; {
        <b>private</b> <b>class</b> Node {
            <b>public</b> T Data;
            <b>public</b> Node Next;
        }

        <b>private</b> Node _last;

        <b>public</b> LinkedList() { }

        <b>public</b> <b>void</b> Add(T value) {
            Node newNode = <b>new</b> Node();
            newNode.Data = value;
            <b>if</b> (_last == <b>null</b>) {
                newNode.Next = newNode;
                _last = newNode;
            }
            <b>else</b> {
                newNode.Next = _last.Next;
                _last.Next = newNode;
                _last = newNode;
            }
        }

        <b>public</b> <b>void</b> Clear() {
            _last = <b>null</b>;
        }

        <b>public</b> IEnumerator&lt;T&gt; GetEnumerator() {
            <b>if</b> (_last != <b>null</b>) {
                Node current = _last;
                <b>do</b> {
                    current = current.Next;
                    yield <b>return</b> current.Data;
                } <b>while</b> (current != _last);
            }
        }

        IEnumerator IEnumerable.GetEnumerator() {
            <b>return</b> GetEnumerator();
        }
    }</pre>
<p>With this approach, if we really need a list with data type <b>
<span class="code">object</span></b>, we can always use <span class="code">LinkedList&lt;<b>object</b>&gt;</span>
which gives us a class essentially the same class as what we started with.&nbsp;
Sometimes it is just better to rewrite. Note that the BCL took a similar
approach. List&lt;T&gt; and Dictionary&lt;K,T&gt; do not inherit from ArrayList and
Hashtable.</p>
<p>Next time I will try a first crack at implementing <span class="code">
IDoubleLinkedList&lt;T&gt;</span>.</p>
