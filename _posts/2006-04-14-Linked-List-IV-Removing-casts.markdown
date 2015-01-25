---
layout: post
title:  "Linked List IV: Removing casts"
date:   2006-04-14 13:56:00
categories: [Programming]
permalink: post/2006/04/14/Linked-List-IV-Removing-casts.aspx
---
<p>In the last entry we successfully derived a double-linked list implementation 
from a single-linked list implementation, but we had to use casts to get it to
compile. What we want to do now is see if we can remove the casts.</p>
<p>We will first remove the cast in the <span class="code">Backwards</span>
method. </p>
<pre>        DoubleNode current = (DoubleNode)Last;</pre>
<p>The reason we need the cast is because the base class defines
<span class="code">Last</span> to be</p>
<pre>        <b>protected</b> Node Last { get { <b>return</b> _last; } }</pre>

<p>What we need is the type to be <span class="code">DoubleNode</span>, but this
property is defined in a descendent class. To allow us to change the type we
need to leave the type open, or, in other words,&nbsp; add the node type as a
type parameter. To accomplish this, I replaced all references
to the node type with a type parameter. The new <span class="code">LinkedList</span>
class looks
like,</p>
<pre>    <b>class</b> LinkedList&lt;T, NodeType&gt;: IEnumerable&lt;T&gt;, ILinkedList&lt;T&gt;
        <b>where</b> NodeType: LinkedList&lt;T, NodeType&gt;.Node, <b>new</b>() {

        <b>public</b> <b>class</b> Node {
            <b>private</b> T _data;
            <b>private</b> NodeType _next;

            <b>public</b> T Data {
                <b>get</b> { <b>return</b> _data; }
                <b>set</b> { _data = value; }
            }
            <b>public</b> <b>virtual</b> NodeType Next {
                <b>get</b> { <b>return</b> _next; }
                <b>set</b> { _next = value; }
            }
        }

        NodeType _last;

        <b>public</b> <b>void</b> Add(T value) {
            NodeType newNode = <b>new</b> NodeType();
            newNode.Data = value;
            <b>if</b> (_last == <b>null</b>) {
                _last = newNode;
                newNode.Next = newNode;
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
                NodeType current = _last;
                <b>do</b> {
                    current = current.Next;
                    <b>yield</b> <b>return</b> current.Data;
                } <b>while</b> (current != _last);
            }
        }

        IEnumerator IEnumerable.GetEnumerator() {
            <b>return</b> GetEnumerator();
        }

        <b>protected</b> NodeType Last {
            <b>get</b> { <b>return</b> _last; }
        }
    }</pre>
<p>The <b><span class="code">where</span></b> clause looks complicated but what
it says is that <span class="code">NodeType</span>'s actual parameter must
derive from the <span class="code">Node</span> type defined in the class. This
allows me to use the properties defined for the node class on variables of type
<span class="code">NodeType</span>. The <span class="code"><b>new</b>()</span> part says that the
actual parameter must also
have a default constructor. This allows us to remove the <span class="code">
CreateNode()</span> method.</p>
<p>Unfortunately, this definition is a bit difficult to use
because you need to define a node class to pass as a parameter. To make this
easier, I defined a single parameter version,</p>
<pre>    <b>class</b> LinkedList&lt;T&gt;: LinkedList&lt;T, LinkedList&lt;T&gt;.MyNode&gt; {
        <b>public</b> <b>class</b> MyNode: Node { }
    }</pre>
<p>I implemented the double-linked list as,</p>
<pre>    <b>class</b> DoubleLinkedList&lt;T, NodeType&gt;: LinkedList&lt;T, NodeType&gt;, IDoubleLinkedList&lt;T&gt;
        where NodeType: DoubleLinkedList&lt;T, NodeType&gt;.DoubleNode, <b>new</b>() {

        <b>public</b> <b>class</b> DoubleNode: Node {
            NodeType _previous;

            <b>public</b> <b>override</b> NodeType Next {
                <b>get</b> {
                    <b>return</b> <b>base</b>.Next;
                }
                <b>set</b> {
                    <b>base</b>.Next = value;
                    value._previous = (NodeType)<b>this</b>;
                }
            }

            <b>public</b> NodeType Previous { <b>get</b> { <b>return</b> _previous; } }
        }

        <b>public</b> IEnumerable&lt;T&gt; Backwards {
            get {
                <b>if</b> (Last != <b>null</b>) {
                    NodeType current = Last;
                    <b>do</b> {
                        <b>yield</b> <b>return</b> current.Data;
                        current = current.Previous;
                    } <b>while</b> (current != Last);
                }
            }
        }
    }</pre>
<p>This looks almost identical to the previous <span class="code">
DoubleLinkedList</span> that I posted last time but it no longer needs a cast in
<span class="code">Backwards</span>. This is achieved by constraining types
passed as <span class="code">NodeType</span> to descend from <span class="code">
DoubleNode</span> instead of just <span class="code">Node</span>. We can pass
<span class="code">NodeType</span> to <span class="code">LinkedList&lt;T, NodeType&gt;</span>
because the constrains on <span class="code">NodeType</span> are compatible with
the constraints for <span class="code">NodeType</span> in <span class="code">
LinkedList&lt;T, NodeType&gt;</span>. I require that <span class="code">NodeType</span>
derive from <span class="code">DoubleNode</span> which, itself, derives from
<span class="code">Node</span>. <span class="code">LinkedList&lt;T, NodeType&gt;</span>
requires <span class="code">NodeType</span> to derive from <span class="code">Node</span>; requiring it
derive from <span class="code">DoubleNode</span> ensures that is the case. I
will also make this easier to use, like <span class="code">LinkedList&lt;T, NodeType&gt;</span>
above, by creating <span class="code">DoubleLink&lt;T&gt;</span> from
<span class="code">DoubleLink&lt;T, NodeType&gt;</span>, as in,</p>
<pre>    <b>class</b> DoubleLinkedList&lt;T&gt;: DoubleLinkedList&lt;T, DoubleLinkedList&lt;T&gt;.MyNode&gt; {
        <b>public</b> <b>class</b> MyNode: DoubleNode { }
    }</pre>
<p>But, we still need a cast in the <span class="code">Next</span> property.
The reason we need it is, even though we know that <span class="code">NodeType</span>
derives from <span class="code">DoubleNode</span>, the compiler only is sure
that the type of the object itself is <span class="code">DoubleNode</span>. It
cannot prove that it must be a <span class="code">NodeType</span> and there is
no way in C# to declare that it must be. Currently, in C#, we are stuck with the
cast. A couple languages have tried to deal
with this problem. One such language is
<a href="http://www.cs.pomona.edu/~kim/README.html#Match">LOOM</a> by
<a href="http://www.cs.pomona.edu/~kim/">Kim Bruce</a> and others. LOOM allows
declaring that a variable must be the same type as <b><span class="code">
this</span></b>. What that might look like in our example is the following,</p>
<pre>        <b>public</b> <b>class</b> DoubleNode: Node {
            <b>this</b> _previous;

            <b>public</b> <b>override</b> NodeType Next {
                get {
                    <b>return</b> <b>base</b>.Next;
                }
                set {
                    <b>base</b>.Next = value;
                    value._previous = <b>this</b>;
                }
            }

            <b>public</b> <b>this</b> Previous { <b>get</b> { <b>return</b> _previous; } }
        }</pre>
<p>where I assume that the keyword <b><span class="code">this</span></b> is used
where the type is assumed to be the type of <b><span class="code">this</span></b>.
This concept also shows up in Eiffel as <span class="code"><b>like</b> Current</span>. Unfortunately, proving that <span class="code">NodeType</span>
and <span class="code">DoubleNode</span>'s <b><span class="code">this</span></b>
type are identitical is a non-trivial problem and has the side affect of adding
exact typing to the language or leaves the type system unsound as Eiffel's
<span class="code"><b>like</b> Current</span> does.</p>
<p>Another approach is to constrain the decedents of <span class="code">
DoubleNode</span> to also to be decedents of <span class="code">NodeType</span>.
The only requirement we have on <b><span class="code">this</span></b> is that it
be assignable to a variable of type <span class="code">NodeType</span>. If we
could express that all concrete implementations of <span class="code">DoubleNode</span>
are required to be decedents of <span class="code">NodeType</span> this
requirement would be met. This is the approach that
<a href="http://scala.epfl.ch/">Scala</a>, by
<a href="http://lampwww.epfl.ch/~odersky/">Martin Odersky</a> and others, took to the problem. What this might
look like in C# is,</p>
<pre>        <b>public</b> <b>abstract class</b> DoubleNode: Node <b>requires</b> NodeType {
            NodeType _previous;

            <b>public</b> <b>override</b> NodeType Next {
                get {
                    <b>return</b> <b>base</b>.Next;
                }
                set {
                    <b>base</b>.Next = value;
                    value._previous = <b>this</b>;
                }
            }

            <b>public</b> NodeType Previous { <b>get</b> { <b>return</b> _previous; } }
        }</pre>
<p>This states that descendants of <span class="code">DoubleNode</span> are required to also be decedents of type
<span class="code">NodeType</span> (or be <span class="code">NodeType</span>, as
is more likely the case). Knowing this we can then remove the cast of <b><span class="code">this</span></b>
because we are guaranteed it is assignment compatible with <span class="code">NodeType</span>. </p>
<p>Whether either of these extensions make it into C# is anyone's guess. It is
still open whether the solutions presented are really better than the problem. Is
requiring the cast really that bad? I will leave that up to you to answer. Also
I glossed over some tricky problems that would need to be addressed to implement
either option, not the least of which are the changes to the CLR.</p>
<p>This all started off innocently enough. We wanted to implement</p>
<pre>    <b>interface</b> ILinkedList&lt;T&gt;: IEnumerable&lt;T&gt; {
        <b>void</b> Add(T value);
        <b>void</b> Clear();
    }</pre>
<p>and,</p>
<pre>    <b>interface</b> IDoubleLinkedList&lt;T&gt;: ILinkedList&lt;T&gt; {
        IEnumerable&lt;T&gt; Backwards { <b>get</b>; }
    }</pre>
<p>but things got complicated when we wanted to share code and
remove casts. This innocent looking problem is devilishly hard to express in a
type-checked object-oriented language.</p>
