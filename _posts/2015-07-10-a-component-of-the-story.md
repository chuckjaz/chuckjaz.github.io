---
layout: post
title: "A component of the story"
description: ""
category: Programming
tags: [Delphi]
---
{% include JB/setup %}

The VCL, the Visual *Component* Library, is not an object oriented framework.
It never has been. In fact, when I designed it, I made sure it wasn't because
I had lost my taste for them having written two, Turbo Vision and OWL. I loved
Turbo Vision but, as Jeff Duntemann told us, we put the front door on the second
floor. You had to understand everything before you could understand anything. It 
was great at what it did but if you needed to learn and buy into every 
assumption we made or it would fight you the whole way.

In many ways, OWL (ObjectWindows Library) was even worse. We tried to learn from
our mistakes with Turbo Vision but we made too many new ones. OWL was intended 
as a thin wrapper on the Windows API that would simplify and highlight the 
underlying object oriented nature of Windows with language support. 
Unfortunately, approached that way, Windows is a terrible example of 
object-oriented design and all we ended up doing was making the flaws more 
apparent. The C++ version of OWL did a better job of hiding the flaws but, by
the time Borland started work on that, I was already in the middle of the VCL.

After two frameworks I saw the flaw in the very nature of frameworks. A 
framework is like a fill-in-the-blanks form with a lot of little places where
the framework designer allows the user to fill in code. If the framework 
resembles the finished application you want, you are golden. If it doesn't, pick
another framework. It was that all or nothing, the framework is king, model that
I found distasteful. The user of a framework extends the framework through 
inheritance and the blanks they fill in are virtual methods. If you think of the 
framework as a tree data structure, users are extending at the leaves, at the 
bottom of the tree. The framework is on top, the user extends at the bottom at 
whatever leaves the framework designer deemed worthy.

With the VCL we wanted to start fresh and that meant I was not doing a 
framework. I didn't want give the user a set of blanks to fill in, but,
rather, give them a toolbox full of useful tools and supplies they could fit 
together or ignore completely as they saw fit. I wanted the user writing 
the application to be in charge, not me. I wanted the user on top of the tree 
and the components at the leaves.

Some principles of the design of the VCL I came up with are,

 * The users should feel they are in control at all times.
 * Components should appear independent and self-contained.
 * Inheritance should not be required to construct an application.

To do this, in my mind I separated users into two camps; component writers and 
component users. Often they were the same person but it was a useful separation. 
Component users should be able to be successful constructing a full application 
with just the components they have and writing code to stitch them together. 
They should never have to know or care about object-oriented concepts like 
encapsulation, inheritance, or polymorphism.

The component writer, however, would care very deeply about object-oriented
programming. A component, after all, is a class that inherits from one of the
fundamental base classes. But once the component is complete, the inheritance 
it used for implementation should be a bit of interesting trivia, not the core 
of its being. From my perspective, the user of a component shouldn't need to 
know or care what its ancestor is, just what properties, methods and events it 
has. The fact it had an `OnMouseDown` event, just like the `OnMouseDown` event 
of some other component was a happy accident not because they both inherited 
from `TControl`. In reality, they both needed to inherit from `TControl` but 
that should only be important to the component writer, not the component user. 
The Delphi 1.0 documentation reflected this philosophy; something it lost over 
time, unfortunately.

The name of the VCL, Visual *Component* Library, is no accident. It is a 
library of *components* not a framework. It contains a framework, a framework to
enable writing components, but it is not a framework itself.

The focus on components also made the third-party ecosystem around Delphi very
clear. The plan was for users to be able to purchase third-party sets of 
components that would fit into Delphi like they were natives; which, in a way, 
they were. There is little difference between a first- and third-party component 
in Delphi as they are both built with the same tools. This was intentional from 
the very start and we believed would be the key to Delphi's success.