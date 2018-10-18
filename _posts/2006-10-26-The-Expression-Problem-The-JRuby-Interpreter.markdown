---
layout: post
title:  "The Expression Problem: The JRuby Interpreter"
date:   2006-10-26 14:23:00
categories: [Programming]
---
<p>I previously discussed the expression problem could be solved with various patterns 
including the
<a href="http://www.removingalldoubt.com/PermaLink.aspx/861feae8-2404-4044-b661-aa13d432b08d">
visitor pattern</a> and using
<a href="http://www.removingalldoubt.com/PermaLink.aspx/e6fb6ded-a66f-4707-97e7-0edaa04b30d9">
a procedural approach</a>. It is interesting to note that JRuby used to use a
visitor pattern but switched to a procedural approach to improve performance.
You can read about it
<a href="http://www.artima.com/forums/flat.jsp?forum=281&amp;thread=180325">here</a>.
This is a side note in the article since what they want to do is convert Ruby to
run as byte code similar to how
<a href="http://www.python.org/workshops/1997-10/proceedings/hugunin.html">
JPython</a> works.</p>
