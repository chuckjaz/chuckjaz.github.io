---
layout: post
title: "The Biggest Bang"
category: Programming
tags: [Delphi,Design]
---
{% include JB/setup %}

Valentine's Day, February 14, 1995.

I was sitting the front row at the main hall in the Moscone Center. The lights 
had just dimmed and a hush fell over the crowd. Several thousand people suddenly 
in wrapped attention all looking at the stage; flashes of cameras going off, and 
murmurs of anticipation. Then Anders took the stage and started [the demo][1].

I had never experienced anything like it before or since. I was nervous and
excited at the same time. We had built up the expectations of everyone, I was
hoping we didn't disappoint them.

Apparently we didn't. It seemed that after every feature Anders showed people 
cheered and when Anders showed that Delphi itself was a Delphi app we received a 
standing ovation. I knew we had done something remarkable; something I will 
always be proud of.

From the [yes!][5] we received to that amazing night was about 2 years. That was 
the longest I had every spent working on one release and over a year longer than 
we had originally planned but we were finished and it felt very, very good.

> Well, almost finished. The first versions of Delphi were not due to hit the
streets until March 1st, 2 weeks after the 15th and we were in final stages of
signing-off the product but we were not finished just yet. There were a couple 
of "stop ship" bugs that needed to be fixed and, as I remember it, Marc 
Cousineau was the last man standing. He  missed that night. We missed him there 
but he took the last few bullets so the rest of us could enjoy ourselves. Thanks 
Marc!

How did we know the features we chose were the right ones? How did we know we
were done?

The biggest challenge of building a product is often not what to do, but 
figuring out what you don't have to do. Of all the features you can build, which
should you build? Which can wait and which cannot? Even in `95, 6 months before 
the earth shake that was Windows '95, the Windows API was large and complicated.
What parts of the API should we wrap, which should we ignore?

What we did was an application of [the 80/20 rule][2], or as I learned learned
later was called the [Pareto principle][2]. Applying this to something like the
Windows API means that roughly only 20% of the API is used over 80% of the time.
This meant we didn't have to wrap everything, we only had to figure out that 20%.
Or, as Anders likes to say, we needed to go for the biggest bang for the buck.

But what about the other 80% of the API that is used 20% of the time? We knew
two things. First, we would never have time to wrap it all. We didn't have the
people and we couldn't keep up producing wrappers for things from a company as 
large an prolific as Microsoft. Second, no application could be written that was
worth much that didn't, in some way, use part of that 80%. This lead to the 
principles,

* Make simple things simple.
* Make the hard things possible.

The most straightforward example of this principle in action is the `Graphics`
unit with `TCanvas` and its related classes. All of the `Graphics` classes that
wrap an underlying Windows API handle expose that handle through an aptly named 
`Handle` property. This was unique at the time for a high-level encapsulation. 
The user is allowed to do just about anything with the handle and the underlying 
object would adjust accordingly. Anders and I spend a considerable amount of 
time enabling this; making the easy things easy, and hard  things possible. 
`TCanvas` made using `DC`s easy, exposing the underlying handle we used made 
calling APIs we didn't wrap possible. You can see this principle applied 
throughout the VCL, the underlying handles are exposed to make the hard things 
possible, even though doing so wasn't exactly easy.

As I mentioned [previously][3], we intentionally made the VCL extensible. We 
didn't have the people to wrap everything and wanted our users to be able to 
build or buy the components they needed to create their applications. Allowing 
components to be written in the same language they used to write the rest of 
the application means we lowered the bar considerably for application specific 
encapsulations.

We began Delphi in early 1993 and we were locked and loaded for a release 
sometime in that winter. In fact, it didn't take us that long to get something 
that looked substantially like Delphi up and running. I remember a demo we did 
in July of 1993 to a group of engineers showing them what we had at the time. It 
included the [component palette][4], form designer, and a rudimentary editor and 
even included what we later called two-way editing.

> The editor was actually just a `TEdit` so very rudimentary! By July it wasn't 
just me and Anders anymore. We had decided to cancel work on what was going to 
be Borland Pascal 9.0 and focus completely on Delphi. Allen Bauer and Alex 
Shtaygrud began working on the debugger and editor (explaining why we were using 
`TEdit`) and Dave Scofield began making my chicken scratches of an IDE look 
presentable. Ironic that a developer that spent most of his career building UI 
libraries and frameworks cannot create a decent looking UI to save his life.

After the meeting, I remember one of the engineers saying we should ship it and 
ship it now! We were encouraged but knew we had a lot of work to get us to a 
shippable state. 

Then came Visual Basic 3.0 and with it the 80% changed dramatically.

[1]: http://edn.embarcadero.com/article/32977
[2]: https://en.wikipedia.org/wiki/Pareto_principle
[3]: http://removingalldoubt.com/programming/2015/07/10/a-component-of-the-story/
[4]: http://removingalldoubt.com/programming/2015/07/07/first-class-class/
[5]: http://removingalldoubt.com/programming/2015/07/08/the-delphi-event/
