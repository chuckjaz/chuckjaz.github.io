---
layout: post
title:  "A Tour of XAML - Part II: Property Elements?"
date:   2005-09-28 20:36:00
categories: [Programming, XAML]
---
XAML provides two ways to set a property, either through a <i>Property
Attribute</i> or through a <i>Property Element</i>. We covered <i>Property
Attribute</i>s last time, this time we will focus on the
<i>Property Element</i>.

A <i>Property Element</i> is an element that, instead of creating an object
instance, set the value of a property. The following is equivalent to the
example given in Part I but uses <i>Property Element</i>s instead of <i>Property
Attribute</i>s.

```
<Button xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <Button.Content>
    Hello, World!
  </Button.Content>
</Button>
```

<p>The loader distinguishes a <i>Object Element</i> from a <i>Property Element</i>
by the presents of the '.' in the name. The first part of the name is a
reference to a type and the second part, after the dot, refers to the property
name (we will discuss exactly why this is when we discuss <i>Attached Properties</i>). Here we use Button in both <i>Property Element</i>s. <i>Property Element</i>s
are necessary when the value of the property cannot be expressed as a string.
For example, you can use a <i>Property Element</i> to set the content of a Button
to be another Button (not sure why you would want that, but suspend disbelieve for
  a second) as in,</p>

```
<Button xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <Button.Content>
    <Button Content="Hello, World!" />
  </Button.Content>
</Button>
```

<p>This can be written more explicitly as,</p>

```
<Button xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <Button.Content>
    <Button>
      <Button.Content>
        Hello, World!
      </Button.Content>
    </Button>
  </Button.Content>
</Button>
```

As you can see, using <i>Property Element</i>s can become a bit repetitive.
To mitigate this, one of the properties of a class can be designated as
the <i>Content Property</i> which tells the load to assign direct content of the element to
that property. In the case of Button (and all ContentControls)
the <i>Content Property</i> is, oddly enough, set to the property named
Content. This allows us to write something like,

```
<Button xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  Hello, World!
</Button>
```

for the first example above, and,

```
<Button xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <Button>Hello, World!</Button>
</Button>
```

  <p>for the second. Most Avalon classes have a <i>Content Property</i> defined.
  For Panel's, such as DockPanel, Grid, and Canvas, their <i>Content Property</i>
  is set to their Children collection. Any direct content of a Panel is assumed to
  be content intended for the Children property. This means the following,</p>

```
<StackPanel xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <Button>OK</Button>
  <Button>Cancel</Button>
</StackPanel>
```
  <p>is an abbreviated version of,</p>

```
<StackPanel xmlns="http://schemas.microsoft.com/winfx/avalon/2005">
  <StackPanel.Children>
    <Button>
      <Button.Content>
        OK
      </Button.Content>
    </Button>
  <Button>
    <Button.Content>
      Cancel
    </Button.Content>
  </Button>
  </StackPanel.Children>
</StackPanel>
```

  <p>Now, with <i>Property Element</i>s and <i>Content Property Attribute</i>s and<i>
  Object Element</i> and <i>Property Attribute</i> from Part I, we have covered
  the fundamentals of XAML. </p>
