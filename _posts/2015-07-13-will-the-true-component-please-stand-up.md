---
layout: post
title: "Will the true component please stand up"
category: Programming
tags: [Delphi]
---
{% include JB/setup %}

There are only four words in computer science and component is one of them.

# Components

When we used the term component in VCL we meant a class that was primarily 
defined by its properties, its methods, and its events. I believe it was Zack
Urlocker I heard stated it this clearly first but once I heard it I knew it was 
true. That is exactly what we meant by component.

## Properties

The state of the component is surfaced exclusively through its properties. The 
some of the state of the properties should reflect the complete state of the 
component (that is, little to no hidden state). The properties can be changed 
directly and the state of the component should immediately reflect that state or 
it should throw an exception, rejecting the change; however, it should only do 
the latter infrequently. Ideally the type of the property should be the only 
filter on the values of the property.

All properties should be independent, or as independant as possible. Changing 
a property should rarely affect the value of another property. Sometimes it is
unavoidable but those cases should be minimized.

This principle is enforced by the serialization system. A Delphi form is the
serialized state of a set of components, serializing the values of their 
properties. They are deserialized by constructing an instance and setting the 
properties, one by one, from the form file. Any state not surfaced as a property
will not be serialized. Coupling of property values can cause the deserialized
version of the component to be different from the originally serialized version.

> Note this is not literally true as components, such as an image, have always
been able to store binary data not surfaced in a property but I am not one to
let the facts get in the way of a good story.

By state above, I mean specifically the initalization state, or the 
configuration state, of the component; not the running state. Components can and 
should use encapsulation, just like any other well designed object.

## Events 

An event is just a property with a method pointer type. It is called out 
independently because the way the user interact with it is much different that 
other properties, since it is how the component signals the user of important 
events, but, deep down, it is just a property. There are special principles 
around events that make them easier to use, however.

Events should be contract-less. This is hyperbole but, as demonstrated above,
I am a big fan of hyperbole. What I mean is that, in an event handler, the user
should be able to do anything they want; change focus, terminate the application,
destroy the component sending the event, anything. Any of these should never 
raise an exception or cause unexpected or undefined behavior. The event should
also never expect the user to do anything. That is, an empty event handler 
should always be legal. Logically, this means that event method pointers should
always be procedures, not functions (functions require a result, violating this
principle). An empty event handler must be treated semantically identically to 
having no event handler assigned. There are always going to be exceptions to 
these general principles (`OnFilter` being one obvious example) but any event
should try to adhere to these general principles.

This is the place where the contrast between a framework and a component library
is the most stark. A framework uses its virtual methods to ask questions the 
user is expected to supply answers to, such as, "What color would you like your 
hints today?". A component, on the other hand, just tells you about interesting 
things you might want to know about such as, "By the way, the user pressed the 
letter A. Just thought you would like to know." What you do with that 
information is strictly up to you.

## Methods

There is not much to say about methods, methods are verbs that tell the
component to do something. There is not much restriction on what that something
is.

# Learning after the fact

## Properties should be changeable

Sometimes you can really only learn what assumptions you are making when 
presented with a contrasting example. One assumption I had that I never wrote 
down or even consciously thought about was that the state of a component after a 
property change should be the same as had the component been born with that 
state. It is a bit subtle but important. For example, there should be no one-way 
trap-doors in a component's state. If you add a border to a control, should just 
as easily remove it and it should appear as if it never had one. This was 
enforced by the form designer. The form designer creates real instances of the 
controls you are editing, not fakes. If you violate this principle the control 
will have weird behavior in the form designer.

After leaving Borland I went to work at Microsoft on Avalon, later called the 
WPF (Windows Presentation Foundation) and even more recently, simply XAML. In 
contrast to the VCL, once a property is set in XAML, it is rarely ever changed. 
Once the view was created and displayed on the screen (that is, added to the
visual tree), properties would often become immutable. You can make changes in 
XAML that cannot be made at runtime. This made the framework perform well in a 
lot of situation but it made writing a designer for it, my job, hard. I 
stubbornly reported each one of these immutable properties as bugs and continued 
on my merry way writing the designer. It wasn't until later, when all my bugs 
were marked "working as intended" that I realized my mistake. They rightly 
rejected my attempt to slow WPF down to make my life easier. XAML is not the VCL 
and I shouldn't have treated it as if was.

## Delphi events are single-plex

Events are commonplace now and are seen in just about every user interface 
library and framework. It is only in Delphi, however, they are single-plex. That 
is, in Delphi an event has one and only one handler. That is because the type of 
the event is a method pointer, a pointer to one method.

After using multiplex events in several frameworks, I don't see them as an 
improvement. In Delphi, at least in 32-bit Delphi, a method pointer is 8 bytes, 
two pointers. It is a value, not an object. An just like other values is 
immutable. This allows the event to just be a property, as noted above. In 
systems with multi-plex events, they are treated quite differently from
properties. In .NET you have the `add_Event` and `remove_Event` methods (which 
is what C#'s `+=` and `-=` is translated into calling). In JavaScript you have
the `addEventHandler` or `on` or any number of patterns that mean the same 
thing. Each event must be able to handle a list of handlers and be able to 
dispatch to that list. In JavaScript, for example, there is an `EventEmitter` 
class whose sole job is to track these values and make emitting events 
tolerable. One thing all these ignore is that having more than one 
handler for an event is rare to the point of non-existence. All add significant 
overhead to the simple event handler just to handle this one case, dispatching
to multiple handlers. There are better ways to handle multi-plexing events than 
to make every event a boat anchor. Single-plex events have their own problems
but I still believe multi-plex events are not worth their cost.
