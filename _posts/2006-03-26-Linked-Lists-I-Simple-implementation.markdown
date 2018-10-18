---
layout: post
title:  "Linked Lists I: Simple implementation"
date:   2006-03-26 12:07:00
categories: [Programming]
---
<p>Last time I investigated several ways to implement an expression evaluator
using a procedural style and several object oriented styles. I used this to
demonstrate a data structure that doesn't lend itself well to being representing
in an object oriented style. Another problematic structure is a linked list. It
is not so much the linked list itself but writing it is such a way that is both
type safe and also supports code sharing with, say, a doubly-linked list. What I want to implement is a class
that implements the following interface.</p>

<pre class="code">    <b>interface</b> ILinkedList&lt;T&gt;: IEnumerable&lt;T&gt; {
        <b>void</b> Add(T value);
        <b>void</b> Clear();
    }</pre>

<p>And then a subclass of that class that implements,</p>

<pre class="code">    <b>interface</b> IDoubleLinkedList&lt;T&gt;: ILinkedList&lt;T&gt; {
        IEnumerable&lt;T&gt; Backwards { <b>get</b>; }
    }</pre>
<p>which inherits the implementation of <span class="code">Add()</span> and
<span class="code">Clear()</span> and efficiently
implements <span class="code">Backwards</span>. For now, I will ignore the generic parameter and just
implement a simple linked list. First, lets create a Node that will form the
nodes of the linked list.</p>
<pre>    <b>class</b> Node {
        <b>public</b> <b>object</b> Data;
        <b>public</b> Node Next;
    }</pre>
<p>A linked list consists of a list of these nodes.</p>
<pre>    <b>class</b> LinkedList: IEnumerable {
        Node _last;
    }</pre>
<p>When an item is added to the list we will create a new node to hold it and
create circular list where the <span class="code">_last</span> instance variable
will always point to the last node. This way we can quickly find both the
last node and the first node of the list because, since the list is circular,
the last node's <span class="code">Next</span> field points to the first node.
The reason I chose this particular way of implementing a linked list will be
obvious if you have watched
<a href="http://channel9.msdn.com/ShowPost.aspx?PostID=159952#159952">this video</a>.</p>
<pre>        <b>public</b> <b>void</b> Add(<b>object</b> value) {
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
        }</pre>
<p>Clearing the list is simple, just forget all the nodes.</p>
<pre>        <b>public</b> <b>void</b> Clear() {
            _last = <b>null</b>;
        }</pre>
<p>I implemented <span class="code">IEnumerable</span> interface by using the new
<b><span class="code">yield</span></b> feature of C# 2.0.
I start at the last node of the list, walk forward (to the first node) and yield an
element, each successive loop will yield another element until we have yielded the
last element (which is pointed to by <span class="code">_last</span>).</p>
<pre>        <b>public</b> IEnumerator GetEnumerator() {
            <b>if</b> (_last != <b>null</b>) {
                Node current = _last;
                <b>do</b> {
                    current = current.Next;
                    <b>yield</b> <b>return</b> current.Data;
                } <b>while</b> (current != _last);
            }
        }</pre>
<p>Now I have a reasonable implementation of a linked list. Next time we will
take several different approaches to implement the interfaces defined above.</p>
