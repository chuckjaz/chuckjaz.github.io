---
layout: post
title:  "Linked List V: Revenge of the Type Parameter"
date:   2006-09-08 12:37:00
categories: [Programming]
permalink: post/2006/09/08/Linked-List-V-Revenge-of-the-Type-Parameter.aspx
---
<p>I honestly thought I was done with this topic but as it turns out I missed 
something. I stated firmly that "currently, in C#, we are stuck with the cast"
but I was wrong. One of the great things about working at Microsoft is the people you can interact with. In
back burner researching a definitive answer to Hallvard's
question to my post of why linked list <span class="code">LinkedList&lt;T&gt;</span> cannot be written,</p>
<pre>  <b>class</b> LinkedList&lt;T&gt;: LinkedList&lt;T, LinkedList&lt;T&gt;.Node&gt; { }</pre>
<p>I asked <a href="http://research.microsoft.com/~akenn/">Andrew Kennedy</a> to look at my page.
I like talking to Andrew; I learn at least one new thing every time we talk.
This was certainly no exception. He quickly pointed out something
that was staring me squarely in the face but I missed entirely, folding my implementation of
<span class="code">DoubleLinkedList&lt;T&gt;</span> with <span class="code">DoubleLinkedList&lt;T, N&gt;</span>
and closing the node type directly
will remove the cast. In other words, <span class="code">DoubleLinkedList&lt;T&gt;</span> can be written without
casts as,</p>
<pre>    <b>class</b> DoubleLinkedList&lt;T&gt;: LinkedList&lt;T, DoubleLinkedList&lt;T&gt;.DoubleNode&gt;,
        IDoubleLinkedList&lt;T&gt; {

        <b>public</b> <b>class</b> DoubleNode: Node {
            DoubleNode _previous;

            <b>public</b> <b>override</b> DoubleNode Next {
                <b>get</b> {
                    <b>return</b> <b>base</b>.Next;
                }
                <b>set</b> {
                    <b>base</b>.Next = value;
                    value._previous = <b>this</b>;
                }
            }

            <b>public</b> DoubleNode Previous { <b>get</b> { <b>return</b> _previous; } }
        }

        <b>public</b> IEnumerable&lt;T&gt; Backwards {
            get {
                <b>if</b> (Last != <b>null</b>) {
                    DoubleNode current = Last;
                    <b>do</b> {
                        <b>yield</b> <b>return</b> current.Data;
                        current = current.Previous;
                    } <b>while</b> (current != Last);
                }
            }
        }
    }</pre>

<p>which closes the node parameter instead of leaving it open. This is one of
those head-slapping, "how stupid of me" moments like when someone points out
that "you know, if you would have pushed the other pawn instead, you would have
had checkmate in three moves." You can see that this is a straight forward
application of the requires clause version (declaring it to be
<span class="code">DoubleNode</span> instead of requiring it). In my zest to introduce the concepts of the "this type" from LOOM and the
requires clause from Scala, I missed what should have been obvious.
Silly me. </p>
<p>This solution has the limitation that you cannot derive something like
<span class="code">TripleLinkedList&lt;T&gt;</span> from <span class="code">
DoubleLinkedList&lt;T&gt;</span> so my example still holds some water (<span class="code">TripleLinkedList&lt;T,
N&gt;</span> can be derived from <span class="code">DoubleLinkedList&lt;T, N&gt;</span>)
but it is not nearly so compelling. I still think that one of the two features
would be very useful but I need a much better motivating example.</p>

<p>While I still don't have a definitive answer to Hallvard's question, I picked
up this gem along the way. Not bad so far.</p>
