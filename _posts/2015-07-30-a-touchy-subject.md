---
layout: post
title: "A touchy subject"
category: Programming
tags: [Delphi, Design]
---
{% include JB/setup %}

[Visual Foo][0], the project that eventually led to Delphi, had one directive; 
create a development environment more visual than [Visual Basic][1]. We took 
that mandate very seriously. During the height of the project we produced some 
lovely prototypes that resembled the soon to be discontinued [Yahoo Pipes][2] or 
Microsoft's long since discontinued [Popfly][3].

What we learned, and I can only assume that Yahoo and Microsoft learned much 
later, was that such approaches are not better, they are just different. Making
something more "visual" doesn't, by itself, make it better it just makes it more
visual.

> I put Visual Foo and projects like it on a list of mistakes that every 
generation will make. I am convinced this model will come back with as much or
more fanfare than Pipes and Popfly did and will eventually die just as they did.
There are some mistakes that are so appealing they will be made over and over
again.
 
Delphi was a step back from the radical approaches that we experimented with
in [Visual Foo project][0]. It competed directly with [Visual Basic][1] instead
of trying to be more visual than Visual Basic. We believed Delphi brought a 
better compiler, a better language, and a system that was much easier to extend.

> I have since developed much more respect for the design of the Visual Basic
language than I had at the time. It is still not among my favorite languages but
I no longer run away screaming.

When you look closely at Visual Basic you realize that it is neither Visual nor
BASIC. It is not really BASIC; it is a long, long way from [Dartmouth BASIC][5] 
or even [Microsoft BASIC][6]. It shared some of the characteristics of BASIC and 
kept some of the syntax but a Visual Basic program looks much different than one 
in its older cousins. It also isn't Visual. The only "visual" part of Visual 
Basic is that it has a form designer. The Visual Basic form designer appeals 
more to [WYSIWYG][7] than visual programming. All the actual programming is done 
in a text editor, not the form designer. Delphi is the same.

One of the things that made Delphi's model easier was that you didn't have to 
visualize in your head where a thing like the file open dialog was, you could 
point to it, "It is right there next to the button that opens it." You can move 
it around; reach out and touch it so to speak. I never expect this to gain 
coinage, but this is why I sometimes refer to Delphi's programming model as 
tactile programming, not visual programming. Simplicity was helped more by the 
illusion that you could touch the component, moving it where you want it, not 
just that you could see it. To keep this illusion it should act as close to a 
physical object as was reasonable. It needed to feel like you could pick it up.

It is this concept that helps explains why I never created a "gutter" for 
non-visual components. Any restrictions on a component's movement breaks the 
illusion. This also explains why the Data Module looks and acts like a form; it 
helps reinforce the illusion. If the component can only appear "off screen" or 
in a restricted area, it no longer feels like a physical thing, the illusion 
breaks down. Even though non-visual components can get in the way of the forms
layout, I felt maintaining the illusion was more important than the irritation 
the real estate they consume causes.

Making something "visual" helps simplicity when it reinforces the conceptual 
module of the underlying system. It hurts when it tries to visualize something 
better left in code.

[0]: http://removingalldoubt.com/programming/2015/07/08/the-delphi-event/
[1]: https://en.wikipedia.org/wiki/Visual_Basic
[2]: https://pipes.yahoo.com/pipes/
[3]: https://www.cloudave.com/1843/microsoft-kills-yahoo-pipes-competitor/
[5]: https://en.wikipedia.org/wiki/Dartmouth_BASIC
[6]: https://en.wikipedia.org/wiki/Microsoft_BASIC
[7]: https://en.wikipedia.org/wiki/WYSIWYG
