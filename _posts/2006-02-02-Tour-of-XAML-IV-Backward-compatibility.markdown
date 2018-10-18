---
layout: post
title:  "Tour of XAML IV: Backward compatibility"
date:   2006-02-02 13:50:00
categories: [Programming, XAML]
---
<p>XAML has support for both forward and backward compatibility of XAML
documents. To understand backwards compatibility in a more formal sense, think
of a XAML reader combined with some set of assemblies as the predicate that
defines a set. If the reader with the given set of assemblies accepts a XAML
document
without error it is part of the set. If it rejects a document, it is not part of
the set. You can think of the XAML reader and assemblies as defining a set, <i>{r(s,x)|x}</i> where <i>x</i>
is the XAML file and <i>r(s,x)</i> is the XAML reader and <i>s</i> is a set of
assemblies. You can then describe the relationship between&nbsp; two sets of
assemblies with respect to XAML. The assemblies imply set schemas that the XAML
reader uses to interpret the XAML document. The schema implied by the WPF
assemblies states there is a Button element and a DockPanel element and the
Button element can be included in a DockPanel as one of it children, for
example. The a set of implied schemas&nbsp; <i>s<sub>2</sub></i> is backwards compatible
with <i>s<sub>1</sub></i> if <i>{r(s<sub>1</sub>, x)|x} ∩ {r(s<sub>2</sub>,
x)|x} = {r(s<sub>1</sub>, x)|x}</i> or, in other words, <i>s<sub>2</sub></i> is
backwards compatible with <i>s<sub>1</sub></i> if the reader will accept same XAML documents
with <i>s</i><sub><i>2</i></sub> as it accepts with
<i>s<sub>1</sub></i>.</p>
<p>XAML really doesn't have to do much to claim it supports
backwards compatibility because backwards compatibility is really a relationship
between two sets of XAML schemas (and the implying assemblies) than it is between XAML documents. XAML,
however, simplifies establishing this relationship by allowing one schema to declare it subsumes another
schema. That means that,
given both <i>s<sub>1</sub></i> and <i>s<sub>2</sub></i>, XAML allows <i>s<sub>2</sub></i> to declare that any reference elements of
<i>s<sub>1</sub></i> should be considered a reference to elements of <i>s<sub>2</sub></i>.</p>
<p>To get more concrete, suppose you had an assembly that declared a single type,</p>

```
public class Balloon {
    Color _color;

    public Color Color { get { return _color; } set { _color = value; } }
}
```

<p>A XAML reader will infer a schema that would allow you to write a XAML
document like,</p>

```
<Balloon Color="Red" xmlns="...assembly-v1-uri..." />;
```

<p>Now, lets say, in version 2 of the assembly you want to add a shape to the balloon
class to support different balloon shapes. In the version 2 assembly you can modify
the class to,</p>

```
public class Balloon {
    Color _color;
    Shape _shape;

    public Color Color { get { return _color; } set { _color = value; } }
    public Shape Shape { get { return _shape; } set { _shape = value; } }
}
```

<p>Now we can write a XAML document that refers to the new assembly by writing,</p>

```
<Balloon Color="Red" Shape="Dog" xmlns="...assembly-v2-uri..."/>;
```

<p>We, however, still want to read all XAML documents that refer to the previous
assembly as well using the new assembly. We can do this by declaring, in the new
assembly, that this assembly is compatible with the old assembly with a
XmlnsCompatibleWith("...assembly-v2-uri...","...assembly-v1-uri...") attribute.
When the XAML reader sees that attribute, it will silently interpret all references to "...assembly-v1-uri..." as if they where referring
to "...assembly-v2-uri...". A XAML reader, presented with the version 2
assembly, will treat,</p>

```
<Balloon Color="Red" xmlns="...assembly-v1-uri..." />
```

<p>exactly as if it was written as,</p>

```
<Balloon Color="Red" xmlns="...assembly-v2-uri..." />
```

<p>This allows all XAML documents referring to version 1 assembly to be readable
when you only have version 2 of the assembly. In fact, assuming you follow CLR
guidelines for evolving an assembly, any XAML file that was legal using the
version 1 assembly will be legal using the version 2 assembly.</p>

