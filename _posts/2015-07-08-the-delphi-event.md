---
layout: post
title: "The Delphi Event"
category: Programming
tags: [Delphi, Language]
---
{% include JB/setup %}

During the design of Delphi, one of the early lines of code we wrote on a 
whiteboard that we wanted to get working was,

    Button.OnClick := ClickHandler;

That seems natural enough now but Turbo Pascal didn't have method pointers
before Delphi; where did they come from? 

The advent of Visual Basic was quite a shot across the bow for Borland. At the
time, we were the hallmark of innovation and the champions of the developer.
Visual Basic changed that. It made what we were working on at the time, Turbo
Pascal for Windows, look clunky and dated. When TPW was release we received 
strong feedback from our users that Visual Basic was better and much easier to
use. It was back to the drawing board for us.

Our manager at the time, Rick Schell, gave us a mandate, "Create Visual Foo that
would be more Visual than Visual Basic!" and set both the C++ and Pascal teams
on that mission. We worked several months trying to come up with something that
would fit the bill but nothing really gelled. I learned a lot during that time
but after months of trying stuff that was more visual, certainly, but never 
easier to use than Visual Basic, we weren't left with much. What we did have
was a Pascal compiler that had method pointers because one of our prototypes
needed it.

I distinctly remember one day in early 1992, Anders and I were out in the 
parking lot behind Borland discussing the project and Visual Basic. We talked 
about how method pointers were all we needed to do everything that Visual 
Basic did. I listened as he sketched a design of something that would eventually 
be Delphi but we called it Visual Pascal at the time. We floated the idea around 
and Gary Whizen got very excited about, as I recall; but he came back from a 
meeting and was told, in no uncertain terms, that we should not build such a 
thing. He tried to float the idea several times after that and was shot down 
every time.

After that I got a bit frustrated with our progress and left the effort and
convinced Gary that we should put out another version of Turbo Pascal to help
pay the bills. He agreed and we started working on combining the DOS and Windows
versions of Turbo Pascal, plus the extended memory features from Borland C++, 
into what became Borland Pascal 7.0. Anders was still working on the Visual Foo 
project but I cajoled him into adding some features and fixing some bugs in the 
compiler; work which we kept hidden from everyone above Gary. I believe Borland
Pascal 7.0 was the best DOS development environment ever written. We released it
just in time for nobody to care as Windows became the only game in town.

In the spring of 1993 things changed. During a big layoff, Rick Schell left the 
company for a small startup company called Netscape and the Visual Foo mandate
left with him. In the midst of the shake-up, in a "get to know the new boss" 
meeting we asked the new VP if we could build Visual Pascal. He said yes! and 
off we went. We were not sure if he knew what we were asking but we didn't want 
to look a gift horse in the mouth.

As Anders surmised a year earlier, the method pointer, or, more correctly, the 
bound method pointer, was exactly what we needed to build Delphi. It wasn't the 
only thing we needed but was one of the fundamental changes we made to Turbo 
Pascal that enabled building the VCL.