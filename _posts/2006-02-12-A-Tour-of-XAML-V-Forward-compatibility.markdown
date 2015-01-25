---
layout: post
title:  "A Tour of XAML V: Forward compatibility"
date:   2006-02-12 18:26:00
categories: [Programming, XAML]
permalink: post/2006/02/12/A-Tour-of-XAML-V-Forward-compatibility.aspx
---
As we saw last time, XAML supports backwards compatibility. XAML also
supports forward compatibility. Forward compatibility is easiest to understand from through a set of examples. Take the balloon
example before. Say we want to write a XAML document that says, when you have
the version 2 assembly, I want a dog shaped balloon. If, however, you only have
the version 1 assembly, a default shaped balloon is fine. In other words, I want
a document that contains some version 2 specific references in it but it is OK to just ignore
them when you have a version 1 assembly. In our example, we want to mark the Shape property some way that
the version 1 reader will know it can ignore the property. In XAML you can use
the markup compatibility namespace to accomplish this. You can write,</p>

<pre>    &lt;Balloon Color="Red" v2:Shape="Dog"
      xmlns="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"
      mc:Ignorable="v2" /&gt;</pre>
<p>This takes advantage of the Ignorable attribute of from the markup
compatibility namespace. mc:Ignorable="v2" says that any attribute or element
from the namespace
associated with the "v2" prefix can be ignored and you would still have a
document that would have an acceptable meaning. If a XAML reader only has the
version 1 assembly, it would interpreted the above document as,</p>
<pre>    &lt;Balloon Color="Red" xmlns="...assembly-v1-uri..."  /&gt;</pre>
<p>If the reader has the version 2 assembly, it would interpret it as,</p>
<pre>    &lt;Balloon Color="Red" Shape="Dog" xmlns="...assembly-v2-uri..." /&gt;</pre>
<p>mc:Ignorable can also applies to elements. To see example of this lets define a class, Party, that is
define as,</p>
<pre>    [ContentProperty("Balloons")]
    <b>public</b> <b>class</b> Party {
        List&lt;Balloon&gt; _balloons = <b>new</b> List&lt;Balloon&gt;();
        <b>public</b> List&lt;Balloon&gt; Balloons { <b>get</b> { return _balloons; }
    }</pre>
<p>This allows us to define a Party instance that contains balloons. For this
example, let's assume that Party was defined in the Version 1 assembly and is
unmodified in the Version 2 assembly. We can then
write a XAML document that looks like,</p>
<pre>    &lt;Party xmln="...assembly-v1-uri..."&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
    &lt;/Party&gt;</pre>
<p>We can now also write a document that will include a third dog shaped balloon
but only if we are using the version 2 assembly, and not include if we are still
using version 1. It looks like,</p>
<pre>    &lt;Party mc:Ignorable="v2"
      xmln="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;v2:Balloon Color="Green" Shape="Dog" /&gt;
    &lt;/Party&gt;</pre>

<p>This will be interpreted exactly as the prior document when reading it with
version 1. With version 2, however, it would be interpreted as if it was,</p>
<pre>    &lt;Party xmlns="...assembly-v2-uri..."&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;Balloon Color="Green" Shape="Dog" /&gt;
    &lt;/Party&gt;</pre>

<p>We can also add a new a new class, such as Favor, that can be used in a
document such as,</p>
<pre>    &lt;Party mc:Ignorable="v2"
      xmln="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;v2:Balloon Color="Green" Shape="Dog" /&gt;
        &lt;v2:Favor Kind="Kazoo" Quantity="10" /&gt;
    &lt;/Party&gt;</pre>

<p>This doesn't affect the version 1 interpretation of the document, but adds 10 kazoos to
the version 2 interpretation. If the author determines that the kazoos are
really necessary to the interpretation of either the reference to v1 can be
removed and the document rewritten to,</p>
<pre>    &lt;Party xmlns="...assembly-v2-uri..."&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;Balloon Color="Green" Shape="Dog" /&gt;
        &lt;Favor Kind="Kazoo" Quantity="10" /&gt;
    &lt;/Party&gt;</pre>

<p>or the the ignorable attribute can just be removed which amounts to an
equivalent document given subsumption described in the previous entry.</p>
