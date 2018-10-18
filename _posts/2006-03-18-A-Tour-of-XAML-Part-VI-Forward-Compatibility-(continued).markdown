---
layout: post
title:  "A Tour of XAML - Part VI - Forward Compatibility (continued)"
date:   2006-03-18 16:26:00
categories: [Programming, XAML]
---
<p>Returning to the Party example we where using, lets say you want to author a
document that has 10 kazoos when the version 2 assembly is present but replaces
them with 4 extra balloons if we only have the version 1 assembly. This can be
written using a markup compatibility AlternateContent block. This might look
like,</p>
<pre>    &lt;Party mc:Ignorable="v2"
      xmln="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;mc:AlternateContent&gt;
            &lt;mc:Choice Requires="v2"&gt;
                &lt;Balloon Color="Green" Shape="Dog" /&gt;
                &lt;v2:Favor Kind="Kazoo" Quantity="10" /&gt;
            &lt;/mc:Choice&gt;
            &lt;mc:Fallback&gt;
                &lt;Balloon Color="Green" /&gt;
                &lt;Balloon Color="Violet" /&gt;
                &lt;Balloon Color="Orange" /&gt;
                &lt;Balloon Color="Yellow" /&gt;
            &lt;/mc:Fallback&gt;
        &lt;/mc:AlternateContent&gt;
    &lt;/Party&gt;</pre>

<p>When the version 1 assembly is present this is interpreted as,</p>
<pre>    &lt;Party xmln="...assembly-v1-uri..."&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;Balloon Color="Green" /&gt;
        &lt;Balloon Color="Violet" /&gt;
        &lt;Balloon Color="Orange" /&gt;
        &lt;Balloon Color="Yellow" /&gt;
    &lt;/Party&gt;</pre>

<p>but in version 2 it is interpreted as,</p>
<pre>    &lt;Party xmlns="...assembly-v2-uri..."&gt;
        &lt;Balloon Color="Red" /&gt;
        &lt;Balloon Color="Blue" /&gt;
        &lt;Balloon Color="Green" Shape="Dog" /&gt;
        &lt;Favor Kind="Kazoo" Quantity="10" /&gt;
    &lt;/Party&gt;</pre>

<p>The content of the Choice element is used when the
v2 namespace is available because it is mentioned in the Requires attribute.
This corresponds to when the version 2 assembly is in use. If v2 is not
available, the content of the Fallback element is used. AlternateContent
supports an arbitrary list of Choice elements and the Requires can be an
arbitrary list of prefix names. </p>
<p>Of course exchanging balloons for kazoos is not a very compelling example. For a more practical example, let's
assume we have following classes in version 1 of an assembly,</p>
<pre>    [ContentProperty("Content")]
    <b>class</b> Paragraph {
        <b>private</b> List&lt;Inline&gt; _content = new List&lt;Inline&gt;();
        <b>public</b> List&lt;Inline&gt; Content { <b>get</b> { <b>return</b> _content; } }
    }

    <b>abstract</b> <b>class</b> Inline { }

    [ContentProperty("Text")]
    <b>class</b> Run : Inline {
        <b>string</b> _text;
        <b>public</b> <b>string</b> Text { <b>get</b> { <b>return</b> _text; } <b>set</b> { _text = value; } }
    }

    <b>class</b> Image : Inline {
        <b>private</b> Uri _source;
        <b>public</b> Uri Source { <b>get</b> { <b>return</b> _source; } <b>set</b> { _source = value; } }
    }</pre>

<p>Now in the version 2 assembly we add a new class that looks like,</p>
<pre>    <b>class</b> Graph : Inline {
        <b>private</b> List&lt;Point&gt; _points = <b>new</b> List&lt;Point&gt;();
        <b>public</b> List&lt;Point&gt; Points { <b>get</b> { <b>return</b> _points; } }
    }</pre>

<p>Now we want to write a paragraph that uses the new Graph class when version 2
of the assembly is present, because, for example, it scales better, but in
version 1 we will just use a bitmap image of the graph. This might
look like,</p>
<pre>    &lt;Paragraph mc:Ignorable="v2"
      xmln="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"&gt;
        &lt;Run&gt;The sales projects for the next quarter are&lt;/Run&gt;
        &lt;AlternateContent&gt;
            &lt;mc:Choice Requires="v2"&gt;
                &lt;v2:Graph Data="1,2 2,3 3,5" /&gt;
            &lt;/mc:Choice&gt;
            &lt;mc:Fallback&gt;
                 &lt;Image Source="SalesData.jpg" /&gt;
            &lt;/mc:Fallback&gt;
        &lt;/mc:AlternateContent&gt;
    &lt;/Paragraph&gt;</pre>

<p>When the version 2 assembly is present, this is interpreted as,</p>

<pre>    &lt;Paragraph xmlns="...assembly-v2-uri..."&gt;
        &lt;Run&gt;The sales projects for the next quarter are&lt;/Run&gt;
        &lt;Graph Data="1,2 2,3 3,5" /&gt;
    &lt;/Paragraph&gt;</pre>

<p>but, when only version 1 available, it is interpreted as,</p>
<pre>    &lt;Paragraph xmln="...assembly-v1-uri..."&gt;
        &lt;Run&gt;The sales projects for the next quarter are&lt;/Run&gt;
        &lt;Image Source="SalesData.jpg" /&gt;
    &lt;/Paragraph&gt;</pre>

<p>using the bitmap instead of of the Graph class. This allows a document author
to produce a file that takes advantage of a new feature, such as Graph with its
better scaling properties, but still allows older versions to see a the graph at
a lower fidelity.</p>

<p>These classes make a better example for another feature of markup
compatibility, the ProcessContent attribute. Let's add another class in the
version 2 assembly,</p>
<pre>    [ContentProperty("Content")]
    <b>class</b> Glow : Inline {
        <b>private</b> List&lt;Inline&gt; _content = new List&lt;Inline&gt;();
        <b>public</b> List&lt;Inline&gt; Content { <b>get</b> { <b>return</b> _content; } }
    }</pre>

<p>This class allows us to make the content to appear to glow when drawn. This
allows us to write a document that looks like,</p>
<pre>    &lt;Paragraph mc:Ignorable="v2"
      xmln="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"&gt;
        &lt;Run&gt;The word&lt;/Run&gt;
        &lt;v2:Glow&gt;&lt;Run&gt;glow&lt;/Run&gt;&lt;/v2:Glow&gt;
        &lt;Run&gt;will appear to glow when it viewed with version 2.&lt;/Run&gt;
    &lt;/Paragraph&gt;</pre>

<p>Unfortunately, with version 1, this is interpreted as,</p>
<pre>    &lt;Paragraph xmln="...assembly-v1-uri..."&gt;
        &lt;Run&gt;The word&lt;/Run&gt;
        &lt;Run&gt;will appear to glow when it viewed with version 2.&lt;/Run&gt;
    &lt;/Paragraph&gt;</pre>
<p>The word "glow" disappeared because, by default, when an element is ignored its
content is ignored as well. This default can be modified by using the ProcessContent attribute. If the original document is modified to be,</p>
<pre>    &lt;Paragraph
      mc:Ignorable="v2"
      mc:ProcessContent="v2:Glow"
      xmln="...assembly-v1-uri..."
      xmlns:v2="...assembly-v2-uri..."
      xmlns:mc="http://schemas.micrsoft.com/winfx/2006/markup-compatibility"&gt;
        &lt;Run&gt;The word&lt;/Run&gt;
        &lt;v2:Glow&gt;&lt;Run&gt;glow&lt;/Run&gt;&lt;/v2:Glow&gt;
        &lt;Run&gt;will appear to glow when it viewed with version 2.&lt;/Run&gt;
    &lt;/Paragraph&gt;</pre>
<p>This will tell the reader to just ignore the tags for Glow if it doesn't
understand it but leave the content intact. This is then interpreted as,</p>
<pre>    &lt;Paragraph xmln="...assembly-v1-uri..."&gt;
        &lt;Run&gt;The word&lt;/Run&gt;
        &lt;Run&gt;glow&lt;/Run&gt;
        &lt;Run&gt;will appear to glow when it viewed with version 2.&lt;/Run&gt;
    &lt;/Paragraph&gt;</pre>

<p>for version 1, and will be interpreted as,</p>
<pre>    &lt;Paragraph xmlns:v2="...assembly-v2-uri..."&gt;
        &lt;Run&gt;The word&lt;/Run&gt;
        &lt;Glow&gt;&lt;Run&gt;glow&lt;/Run&gt;&lt;/Glow&gt;
        &lt;Run&gt;will appear to glow when it viewed with version 2.&lt;/Run&gt;
    &lt;/Paragraph&gt;</pre>

<p>when version 2 is present which is exactly what we wanted.</p>
<p>In general, forward compatibility allows previous versions of a document
reader to read a document intended for later versions and render the content at
a lower fidelity. This helps immensely when deploying a newer version of an
assembly. During initial deployment it might not be that wide spread. You can
have your users start taking advantage of the new features immediately because
older versions will still be able to read newer documents, at a lower fidelity,
even when they don't have the new assembly.</p>
