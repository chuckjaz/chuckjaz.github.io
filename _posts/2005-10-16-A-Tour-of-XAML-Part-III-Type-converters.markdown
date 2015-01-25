---
layout: post
title:  "A Tour of XAML - Part III - Type converters"
date:   2005-10-16 16:38:00
categories: [Programming, XAML]
permalink: post/2005/10/16/A-Tour-of-XAML-Part-III-Type-converters.aspx
---
[EDIT 01/19/2015: This content is out of date.]

<p>Fundamentally XAML is an object initialization language. It creates object
and sets values to properties. XAML, however, is not limit to creating reference types. You can also create instances of any type
that has an associated type converter. For example, XAML can create an instance of a brush
with the following code,</p>

{% highlight xml %}
<Brush>Red</Brush>
{% endhighlight %}

<p>which creates a red solid color brush by calling the type converter
associated with Brush. This means that,</p>

{% highlight xml %}
<Button>
  <Button.Background>
    <Brush>Red</Brush>
  <Button.Background>
</Button>
{% endhighlight %}

<p>is equivilent to,</p>

{% highlight xml %}
<Button Background="Red" />
{% endhighlight %}

<p>XAML can be used to create value types, such as an integer, through its type
converter as in, </p>

{% highlight xml %}
<s:Int32>27</s:Int32>
{% endhighlight %}

<p>or a string,</p>

{% highlight xml %}
<s:String>It is the time for all good men...</s:String>
{% endhighlight %}

<p>by using string's type converter. Note that XAML always refers to the CLR
name for the type, not a language specific name such as "int" in C# or "Integer"
in Pascal. Both of these types refer to the System.Int32 class in the CLR and
XAML uses the CLR name, Int32. Note also that neither the XAML nor the WPF (Avalon) namespaces refer to the System
namespace in mscorlib.dll where these types live. This mean that somewhere in the document
you need to declare the prefix. This might look something like:</p>

{% highlight xml %}
<?Mapping ClrNamespace="System" XmlNamespace="System"
    Assembly="mscorlib"?>
<Window
    xmlns="http://schemas.microsoft.com/winfx/avalon/2005"
    xmlns:s="System">
  <Button>
    <s:String>It is the time for all good men...</s:String>
  </Button>
</Window>
{% endhighlight %}

<p>Of course, for string, it is easier to write it,</p>

{% highlight xml %}
<Window xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <Button>It is the time for all good men...</Button>
</Window>
{% endhighlight %}

<p>but the string for is useful when you want multiple strings in a collection, say, for example, as the content of a list box,</p>

{% highlight xml %}
<?Mapping ClrNamespace="System" XmlNamespace="System"
    Assembly="mscorlib"?>
<Window
    xmlns="http://schemas.microsoft.com/winfx/avalon/2005"
    xmlns:s="System">
  <ListBox>
    <s:String>It is the time for all good men...</s:String>
    <s:String>to come to the aid of their country.</s:String>
  </ListBox>
</Window>
{% endhighlight %}

<p>Note that if this had been written,</p>

{% highlight xml %}
<Window xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <ListBox>
    It is the time for all good men...
    to come to the aid of their country.
  </ListBox>
</Window>
{% endhighlight %}

<p>the list box would only contain one string, not two. This is because all
continuous text content is considered one string and not two. If you load this
in WPF you will also notice that the string get treated as "It is time for all
good men... to come to the aid of their country" where the spaces and line
breaks have been collapsed. I will discuss the rules for whitespace
normalization (collapsing) in a subsequent post. For now, if you are familiar
with HTML, what XAML does shouldn't come as any surprise.</p>
