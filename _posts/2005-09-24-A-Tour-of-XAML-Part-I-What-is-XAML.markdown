---
layout: post
title:  "A Tour of XAML - Part I: What is XAML?"
date:   2005-09-24 15:25:00
categories: [Programming, XAML]
---
Today I begin a tour of XAML. As <a href="http://www.simplegeek.com">Chris</a>
said, I have been part of development of XAML. I came fairly late to the game,
however, more closely to the end of
the Avalon development cycle than at the beginning, and XAML was already fairly
well defined when I arrived. I did work to push some changes into the language, primarily
driving consistency and regularity in the language as well as put developing
features that support versioning and extensibility. We will cover some of those
things in our tour, but lets start at the beginning.

The first question we will cover in our tour is what is XAML anyway? Simply,
XAML is an XML based object initialization language. Its job in life is to
create a tree of objects. For example, consider the following XAML file,

```
<Button Content="Hello, world!"
  xmlns="http://schemas.microsoft.com/winfx/avalon/2005" />
```

This creates an instance of `System.Windows.Controls.Button` and
sets the `Content1` property to the sting value
`"Hello, world!"`. The reason `System.Windows.Button`
is selected and not, say `System.Windows.Forms.Button`,
is because of the `xmlns` declaration in the
`Button` tag. The magic behind the URI
`"http://schemas.microsoft.com/winfx/avalon/2005"`
is hidden in the Avalon assemblies themselves. They contain an assembly
attribute that looks like,

```
[assembly: XmlnDefinition(
  "http://schemas.microsoft.com/winfx/avalon/2005",
  "System.Windows.Controls")]
```

It actually has several more definitions like this to associate several other
CLR namespaces with the Avalon XML namespace, but this is the one used above.
Because `PresentationFramework.dll` is in the GAC or referenced
by the project if the XAML is compiled into assembly, the Avalon
loader will find the `XmlnsDefinition` in the assembly
and it is able to determine that it is Avalon's button we want to create.

Now lets take a look at the phrase `Content="Hello, world!"`. This causes the
loader to look up the property called `Content` and
sets its value. The `Content` property is introduced in
`Button`'s `ContentControl` base class and is of type
`System.Object`. Since there is no applicable type
converter for `System.Object`, the loader uses the default type of an
attribute, string, and calls the setter method for `Content`.
Things are slightly more complicated than this, but this should give you the
basic idea. This gives us an instance of the `System.Windows.Controls.Button`
class with its `Content` property set to the value
`"Hello, world!"`, which is, hopefully, what we wanted.

If we change the XAML to,

```
<Button Content="Hello, world!" IsDefault="True"
  xmlns="http://schemas.microsoft.com/winfx/avalon/2005" />
```

adding the phrase `IsDefault="True"` which causes the loader to find the
`IsDefault` property introduced on the `Button` class itself. The `IsDefault`
property is of type `System.Boolean` which has a type converter,
`System.ComponentModel.BooleanConverter`, associated with it.
The loader will pass the string, `"True"`, to the type
converter associated with the property (or, as in this case, the property's
type) prior to setting the property's value; after which we will have an
instance of the `System.Windows.Controls.Button` class with its `Content`
property set to the value `"Hello, world!"`, and its `IsDefault`
property set to `True`.

This time we covered an _Object Element_, an XML element used to create
an object, and we used a _Property Attribute_, an XML element used to set
the value of an object instance's property. Next time we will get use a
_Property Element_ and take advantage of the _Content Property Attribute_.
