---
layout: post
title: "Properties on purpose"
tags: Language,Delphi,C#
---
{% include JB/setup %}

The development of the Visual Class Library started rather simple, it began with 
a few simple statements on the white board in Anders' office. The details are a
bit fuzzy after so much time but they looked something like this,

    Button.Caption := 'OK';

we then added,

    Button.OnClick := OkClickHandler;

to be able to be able to do something when the button was clicked.

Later we added things like,

    Button.Font.Name := 'Arial';

to change the font of the button.

We wanted this code to work. This is usually how like to design things. Start by
writting some code down that should work and then begin brainstorming ways it 
can; poking holes in those designs along the way. Anders and I began to do just 
that.

We wanted to avoid requiring an `Update`, `Realize`, or `Invalidate` method to 
be called to make the changes show up on the screen. That is, we wanted the 
caption of the button to change immediately when the assignment was made with no 
futher intervention.

> As a side note, we wanted to avoid the state problem we had introduced in OWL,
our previous class library. In OWL objects had several states they could
be in from constructed, to initialized, to having a window class, to having a 
window handle, etc. Each state enabled different operations, all of this was 
rather difficult to keep in your head. With the VCL, we wanted no states. We 
wanted the button object to be the button, not a remote control of a button. In
other words, the button's caption property was the caption of the button not a 
copy of it that will eventually be the caption of the window handle the object 
was wrapping.

This implied that the assignment had to have a side-effect; it would eventually 
need to call the `SetText()` Windows function. Side-effects are not generally 
considered kosher for assignemnts (other than the asignment itself); but we knew 
it would solve our problem. 

> I now know of several ways we could have avoided side-effects but either none 
occurred to us at the time or I just don't remember discussing them.

We then began brainstorming some syntax to represent it. We decided that we
would create a new object member called a property that would then have a 
read and a write method associated with it. It would look something like,

    property Caption: String read GetCaption write SetCaption;

where the `SetCaption` method would be called on assignment and the `GetCaption`
method would be called when retrieving the caption. This allowed us to call
a function to get the caption from the window handle intead of being forced to
keep a copy of the string just for the get method. Later, we relaxed the read
clause to take a field name if the data was actually in the object, such as,

    property Name: String read FName write SetName;

This allowed more consise read clauses as well as allowed us to put off 
inlining simple get methods which would have complicated the compiler.

Later we realized the serialization of the objects can be through their
properties; but that is propbably a story best left for another time.

In adding properties to Pascal, am sure we had some influences but don't recall 
any other language having a similar construct. The closest at the time was 
Visual Basics component properties, which is probably the genisis of the name. 
However, you could not declare them in Basic, you could only access them from 
Basic.
