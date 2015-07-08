---
layout: post
title: "First-class class"
category: Programming
tags: [Language,Delphi]
---
{% include JB/setup %}

When building the first prototypes of what became the Delphi form designer I
needed to solve a very simple problem, a palette of controls. When one of the 
palette items was selected and the form design was clicked, I needed to create 
control on the form in the location clicked. The problem was, how do you create 
such a list?

The solution we choose was to allow a reference to a class be treated as a 
first class value; a class reference. If you have a class called `A` then the
identifier `A` is considered a constant reference to the class of type 
`class of A`.

Class references follow inheritance substitution rules. For example, if `B` 
derives `A` then a reference to class `B` can be passed when a `class of A`
reference is expected. This allowed us to create `TComponent`, the base class
of all Delphi components, and the corresponding class reference type 
`TComponentClass`. 

With a class reference you can call any class method or constructor declared on 
that class. If the constructor or method is virtual, the most derived 
constructor or method is called. Given the `A` and `B` classes above, the 
statements:

    var
      SomeClass: class of A;
      SomeA: A;
    begin
      SomeClass := B;
      SomeA := SomeClass.Create();
    end;

create an instance of the `B` class which derives from `A`. This allowed us to
use an array of class references as the palette data structure. Something like,

    type
      TPaletteItems = array of TComponentClass;

    var
      PaletteItems: TPaletteItems = [TButton, TEdit, TCheckBox, TLabel];

Selecting an item in the palette was simply selecting an index into that array 
which selected the cooresponding class reference of which to construct an 
instance. 

The early prototypes and early pre-release versions of Delphi I just hard-coded 
the array of classes as I was the only one writting them at the time. Eventually 
we allowed the array to be modified by calling a `Register` procedure.

The class reference was also useful for form deserialization as it allowed us to
construct the correct class based on a registered class name but the genisis
of the class reference was the palette of the form designer.