---
layout: post
title:  "My Life as a GUI Builder Builder"
date:   2005-07-30 23:57:00
categories: Programming
---
Frankly I am not sure how to respond to
Beware The GUI Builder. I have spent nearly all my
programming career building what this article describes
as GUI builder (I have always called them designers, but
that's me, I hate the term GUI). It is not often you
come across an opinion piece that condemns nearly your
entire body of work as worse than worthless (unless you
are a lawyer or politician, that is). Let me see if I
can find myself and respond.

At this point, if you haven't already, read the
referenced article, if just for the clever poem.
My response will make more sense that way (well,
hopefully, anyway), and I apologize for my lack
of a poem (well, not really; be glad I didn't try
one).

## "major failings of this application genre"
Don't be fooled (OK, you can be fooled if you want but
I don't have to like it). This article is really just
a diatribe about the short-coming of a particular
unnamed Java based SWT GUI builder with passing
references to Swing builders. Many of the flaws of
"GUI Builders" it points out are not really faults
of the entire genre, but, rather, flaws in the
combination of Java, SWT, and this unnamed GUI
builder. I wish the author would have named it, at least
I could then verify the claims made in the piece, but
I will trudge ahead anyway, blissful in my ignorance.
Many of the flaws are common to a wide range of
designers, but not all, and certainly not mine, of
course ;-).

## Reliance on Static Code Analysis
Not all designers rely on "static code analysis".
Many, such as all the ones I have worked on, store
the result of the designer in as a serialized image of
the form. This is what code generating form designers
do as well, such as WinForms, but the format of the
serialization of the form is in code. I also have to
quibble with the description of how the loading of the
form is done, "By 'static code analysis', I refer to
those aspects of the code's behavior that can be
determined from consideration of it's structure
alone, without regard to it's runtime behavior."
I am not sure what this means. All the code-generating
form designers I am familiar with have a language
interpreter that can execute a subset of the
serialization language. The subset is usually restricted
from executing flow control logic, but not always as
in Visual dBASE. The reasons for these limitations is
to simplify translating gestures in the designer to
changes in the code that was generated. In an example
given in the article, it would be difficult for a
designer to figure out the mapping between the array,

{% highlight csharp %}
string[] colors = { "Red", "Orange", "Yellow",  "Green", "Blue",
  "Indigo", "Violet" };
{% endhighlight %}
and the value passed to the setText() method in the for loop (which isn't actually there in the example, but should be). But, in my mind, this is no excuse, if the language allows it the designer should support it. My solution in the past is to use a language that only allows what the designer supports. In the case of Delphi it was the .DFM file format, and currently, in my "new" job, it is XAML. In these cases, the language is designed in such a way that it is limited to supporting only that which the designer can figure out. This is one of the several reasons I don't like the code generator style of serialization, programmers are tempted to futz with it and the will always use a feature of the serialization language the designer doesn't support. The authors has a point, Java does not make very good serialization language (nor does any other general purpose language).

Going back to the example used in this section (instead of skipping ahead as above), localization and handling dynamic values are typically where designers excel, not fall down. If you separate the logic of the form from the initialization state, you can have localizers just edit the initialization information, not your code (good reason not to code-gen, by the way). The article refers to resource bundles in "capable" GUI builders but it goes beyond that. The entire initialization state should be localizable, not just strings or what the developer might feel generous about allowing the poor localizer to fiddle with. The localizer might need to entirely reorient the form for a right-to-left language, reorganize the form in a more culturally friendly order, etc. If layout is done in code this is notoriously difficult, if not impossible, for localization teams to deal with. Most localization teams I have had the pleasure to interact with require the framework to have a form designer for these very reasons. Don't layout a form in code. Just don't. What assumption you have about how the form will look is culturally biased. Writing layout in code just makes the bias impossible to remove. Localization is a tough, thankless job. Don't make it harder.

As for dynamic values based on "male" and "female", is easy to do a one-liner, but data binding is what is really needed here. A form should follow good MVC (Model-View-Controller) principles, where appropriate, and this example screams model-view. If you have a model that controls changes the salutation options, a good class library will have a data-bound version of a radio group that will update accordingly. This is not a GUI builder problem, it is a framework/data-binding problem.

## Masking Understanding and Inhibiting Learning
This whole section is arguing against a false premise. No designer, ever, was written with the idea that the user would never have to code again. (Well, I have to admit, the marketing teams for them often have said that, but they were lying). Good designers are written to smooth and facilitate the transition between designing a form and writing code, allowing the user to do those things that are better done in a designer in the designer, and leave those things better done in code, done in code. They also where never intended be the only way you learn the framework (although they do help a lot) or take the place of good framework knowledge. Their job is to make using the framework easier, period. The author has a good point, most of your time is not spent in the designer, you spend most of you time in code. That is when the designer really shines, when you don't have to use it much to get the job done.

> How do you visually represent these infinitely variable sets of constraints amongst components, and in such a way that they can be configured with mouse clicks and keyboard operations? The answer is 'With great difficulty'.

This is a criticism of frameworks, not designers. The framework should present a model where this is straight forward. The designer should make using it easy to use and easily discoverable. Also, remember, doing this in code might be easier for you, it just makes other people's jobs impossible. You are not the only one that needs to control the layout of the form. A localizer might have a different idea of where the OK button should go. And, to be blunt, if your code has "infinitely variable sets of constraints" on anything, you should consider a different line of work.

## Missed Opportunities for Reuse
I covered most of this above, use data-binding; understand data-binding; love data-binding. The second part, the address form example, is a job for nested forms or data templates. Each of the address should be a an instance of a single nested form or data template (depending on the framework you choose). This is where data-binding shines again since moving data in an out of controls is what data-binding does. This is a case where you shouldn't need to write much code. The code is shared if you use the sharing techniques provided by the framework and the designer.

## Dependency Upon Another Tool
Ideally the framework and the designer come from the same company and are rev'ed at the same time. If you get your designer from two different places, the author has a point. In my opinion, the designer and the framework should be developed together with full knowledge of each other. In Java, these are typically provided separately for reasons I will never understand.

## Signs That Your Predecessors Used A GUI Builder
Most of these are either artifacts of code generation, covered above, or just signs that your predecessor didn't leave because he wanted to. You can write bad code in any language and and using any tool. These are common problems, granted, but well written code is not common. As a retort, here are some signs that your predecessor didn't use a GUI designer,

- The form looks hideously ugly; something only a Unix user would love.
- You are considering rewriting it because you cannot change it without breaking something in the layout.
- There are dead spots, places where the controls are obviously missing on the form. This happened because it was just easier to hide the control than figure out how to re-layout the form or because the predecessor forgot to save the "prototype".
- Your not sure the form you are looking at in the UI or what code creates it.
- The localization team curses when your predecessor's name is mentioned.

OK, so mine are not as witty...

### Conclusion

> They have some utility for mocking up interfaces. You can create static "snapshots" of your interface and show them to users, perhaps employing printouts of them as paper prototypes.

This is truly damning with faint praise. If this is all you use a designer for, go get a better designer. Seriously. They exist and hopefully you now know what to look for in order to find a good one.
