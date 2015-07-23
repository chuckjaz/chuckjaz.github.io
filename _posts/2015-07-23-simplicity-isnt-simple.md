---
layout: post
title: "Simplicity isn't simple"
category: Programming 
tags: [Delphi,Design]
---
{% include JB/setup %}

    Button.Caption := 'OK';

This was first line of "Delphi" code ever written. As I mentioned in [Properties
on purpose][0] Anders wrote this on his whiteboard during one of our earliest
design discussion and we needed to get this to work. I already discussed what we
added to get this to work, but just as significant is what we removed.  

I have participated in, reviewed, read about, or heard about many things 
purporting to make programming easier. The few that actually did make things 
easier had one thing in common, they reduced the number of concepts the 
programmer had to keep in their head.

One of the primary influences to OWL (the ObjectWindows Library), the 
predecessor to the VCL, was a little known now, but wildly heralded at the time,
language called [Actor][1]. It was a [Smalltalk][2]-like language that made 
programming Windows a lot easier. However, one way it demonstrated that 
simplicity in its advertising was to show how you could write a one line editor 
application. That ad stuck with me, but not for the reason the marketing person
who came up with that ad intended. It lead me to this guiding principle,

* I don't mind requiring two lines of code to accomplish what could be 
accomplished in one line if I can understand both lines.

The line of code, through powerful, was completely incomprehensible and couldn't 
be reproduced independently without complete understanding of the underlying 
framework of [Actor][1]. Counting lines as a judge of simplicity is a fallacy. 
It is just an indicator, not an arbiter, of simplicity. The above principle is 
an informal observation of the fact that sometimes two lines are simpler than 
one.

Actor made programming in Windows much easier than its predecessors but being
able to utter a string hieroglyphics to produce an editor was *not* the way it
did it. It did it by reducing the number of concepts the user had to learn and
remember and it did it by reducing the bookkeeping which was, up till then, 
required.

The first line of "Delphi" code ever written as an appeal to a reduction in
concept count. If that was to be written in Turbo Pascal it would have had to
look like,

    Button^.SetCaption('OK');

The `^` operator, in Pascal, is a pointer dereference operator. First, 
linguistically, in this context, it is redundant. The member selection operator 
`.` is not legal on a pointer so the only reasonable interpretation of 
`Button.SetCaption` would be for `SetCaption` to be a member of the type 
referenced by the pointer. Second, it requires the understanding when and, when 
not to, use the `^` operator. We realized that removing the need for the `^` 
operator allowed us to defer the need for the user to understand pointers. If we 
allocated the button on the behalf of the user, they wouldn't need to know `New` 
and, later, if we disposed of the memory on their behalf, they wouldn't need to 
understand `Dispose` either. All this was accomplished simply by deleting one 
character from the line `Button^.SetCaption('OK')` to form 
`Button.SetCaption('OK')`; properties allowed us to further reduce this to 
`Button.Caption := 'OK'`.

> This actually brought us more in-line with the original [Object Pascal][6]
as defined by Apple in consultation with [Niklaus Wirth][7] which also did not
require the `^` in this context. 

In the VCL we took nearly every opportunity we could find to reduce the number 
of concepts the programmer was required to learn and/or maintain in their head 
when using Delphi as a [component user][3]. This showed up in everything from 
the way menus were defined to how Windows messages were routed.

Reducing concepts can also lead you into trouble. We always kept this maxim in
mind (commonly attributed to [Albert Einstein][4]):

* Everything should be made as simple as possible, but not simpler.

> I found it ironic that Microsoft chose the name "Einstein" for their, now
abandoned, user persona they used for a user who could handle a lot of 
complexity.

I always interpreted this, when applied to programming, as an appeal against 
[magic][5], like the Actor incantation for a one line application. The 
additional requirement of not only simplifying programming for component users, 
but also for component writers, kept us honest. Any magic we introduced for the 
component user needed to be fully understood by the component writer; which 
meant, in our case, magic only postponed, it did not not eliminate, complexity.

[0]: http://removingalldoubt.com/programming/2015/07/05/properties-on-purpose/
[1]: https://en.wikipedia.org/wiki/Actor_(programming_language)
[2]: https://en.wikipedia.org/wiki/Smalltalk
[3]: http://removingalldoubt.com/programming/2015/07/10/a-component-of-the-story/
[4]: https://en.wikipedia.org/wiki/Albert_Einstein
[5]: https://en.wikipedia.org/wiki/Magic_(programming)
[6]: https://en.wikipedia.org/wiki/Object_Pascal
[7]: https://en.wikipedia.org/wiki/Niklaus_Wirth

