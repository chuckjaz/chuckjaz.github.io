---
layout: post
title:  "Scanners (not the movie)"
date:   2007-12-30 17:01:00
categories: [Programming]
permalink: 2007/12/30/Scanners-(not-the-movie).aspx
---
<div class="text"><p>In my previous post I introduced a program I wrote to that uses the method
described in
<a href="http://www.amazon.com/First-Order-Logic-Raymond-M-Smullyan/dp/0486683702/ref=sr_11_1/102-8843584-4849732?ie=UTF8&amp;qid=1192783059&amp;sr=11-1">First-Order Logic</a> by
<a href="http://en.wikipedia.org/wiki/Raymond_Smullyan">Raymond M. Smullyan</a>
to prove an arbitrary propositional logic expression is a tautology. In this
program I use some techniques that I want to describe. The first is the scanner.
When writing scanners I have always hand written the scanner. There are two
primary reasons for this, first I have never found scanners that difficult to
write and, second, they are very time critical, often taking more than 20% of
the overall parsing time, and auto-generated scanners are hard to optimize.
Tableaux uses a hand-written scanner. The code for the scanner is in Scanner.cs
in the zip file found
<a href="files/Tableaux.zip">here</a>.</p>
<p>A scanner is a routine that takes text input and classifies it into a stream
of tokens. Scanners typically give one token at a time. The scanner is the lowest level of the parsing process. Parsers
are built on top of scanners and use the scanner results to determine the
meaning of the source using some kind of grammar. The distinction between the
parser and the scanner is somewhat arbitrary. You could just consider each
character of the input as a token and build the parser up from there. Doing this
is typically slower than a traditional scanner, however, so most parsers use a
scanner. The line between the scanner and the parser is usually drawn at the
lexical elements such as identifiers, reserved words, numeric and string
constants, comments, etc. More formally, a lexical element is a string of
characters that can be matched and classified by a regular expression. Scanner
generators, such as <a href="http://en.wikipedia.org/wiki/Lex_programming_tool">
Lex</a>, use regular expressions to specify the scanner.</p>
<p>The trick to a fast scanner is to reduce the per-character cost of the
scanner. One way to do this is to make sure you are executing the fewest number of instructions
per-character-scanned as possible. A few of tricks I use to reduce the
per-character cost include,</p>
<ol>
	<li>Use the switch statement<ul>
		<li>That included switch or case statement typically heavily optimize
		the result, often generating jump tables.</li>
	</ul>
	</li>
	<li>Pull fields into local variables and only write them at the before
	returning.<ul>
		<li>Local variables can be mapped to registers during code generation.
		Fields never will be. Stores to global memory (i.e. a field of a object
		allocated from the heap) are very hard for code generators to optimize.
		Local variable are much easier.</li>
	</ul>
	</li>
	<li>Make checking for boundaries (end of file or end of buffer) appear as a
	character.<ul>
		<li>This merges the end-of-file check with character classification.</li>
		<li>In this case I use the character value 0 to indicate end of file. If
		I was willing to use unsafe code I would have taken advantage of the
		fact that the CLR ensures a 0 value at the end of every string.</li>
	</ul>
	</li>
</ol>
<p>One thing that looks slow that isn't is the GetChar() routine. Normally you
would see that and think I am paying for a CALL/RET pair but his gets inlined in
retail. I am paying for the end of buffer check every character but, as noted
above, this can be removed if I wanted to use unsafe code, definitely overkill
for a small program like tableaux.</p>
<p>I went a bit overboard, intentionally, with identifier. It handles C# style
identifiers, handling Unicode letter and number classifications. To avoid
calling the Unicode classification routines for every character I hard-coded the
ASCII part and only call the Unicode classification routines when I encounter a
non-ASCII potential identifier character. It is not that the classification
routines are slow, it is that every cycle counts in the scanner and avoiding the
CALL/RET and character table look is significant for file that contain mostly
ASCII characters. Note also that I hard-code some non-identifier characters in
the identifier internal loop. This avoids calling the Unicode classification
routines for those character, which are the most common characters you might
find after an identifier, which avoids calling the Unicode classification routines
at all for most input.</p>
</div>
