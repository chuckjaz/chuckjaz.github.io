---
layout: post
title:  "BeginInvoke's last parameter: What's in a name?"
date:   2008-06-16 13:21:00
categories: [Programming]
permalink: post/2008/06/16/BeginInvokes-last-parameter-Whats-in-a-name.aspx
---
<p>I have been playing around with asynchronous programming lately and it 
bothers me that the last parameter to the <em>begin invoke</em> pattern doesn't
have a name that everyone agrees on. The <em>begin invoke</em> pattern is the
asynchronous calling pattern were the <code>BeginXxxx()</code>, such as <code>
Stream.BeginRead()</code>, takes a set of parameters, a callback method, and
some, well, thing, at the end that is carried along with the asynchronous
operation that eventually finds it way into the <code>IAsyncResult</code>. The
problem is what do we call that thing? In an informal survey of the methods in
the BCL that implement this pattern I have found a wide variation. Here is a
partial list in somewhat order of popularity,</p>
<ul>
	<li>object</li>
	<li>stateObject</li>
	<li>state</li>
	<li>asyncState</li>
	<li>extraData</li>
	<li>data</li>
</ul>
<p>There seems to be little agreement about what to call it. We could pick the
most prevalent but the name <code>
object</code> occurs the most often because every
delegate gets a <code>BeginInvoke()</code> method created for it and in this
automatically generated code the parameter is called <code>object</code>. We can't standardize on <code>object</code> because it is a reserved word in several languages so either it would be impossible to
specify or awkward (i.e. <code>@object</code> in C#). What I would like is a
name that we can all use and we can all agree on.</p>
<p>To name the thing, we first must understand why it it is there at all. It
exists to hold some state on behalf of the caller. Having the state object makes
programming against the <em>begin invoke</em> pattern easier in languages that
do not have anonymous methods that capture local variables. Consider the
following program (which intentionally ignores errors because it is only an
example),</p>
<pre><b>const</b> <b>string</b> uriFormat =
    "http://messenger.services.live.com/users/{0}@apps.messenger.live.com/presence";
<b>const</b> <b>string</b> responsePattern =
    @"""statusText""\s*:\s*""(?&lt;status&gt;\w+).*""displayName""\s*:\s*""(?&lt;name&gt;\w+)""";

<b>static</b> <b>void</b> Main(<b>string</b>[] args) {
    <b>var</b> uri = <b>string</b>.Format(uriFormat, args[0]);
    <b>var</b> request = WebRequest.Create(uri);
    <b>var</b> result = request.BeginGetResponse(PresenceCallback, request);

    // Other interesting work....

    result.AsyncWaitHandle.WaitOne();
}

<b>private</b> <b>static</b> <b>void</b> PresenceCallback(IAsyncResult result) {
    <b>var</b> request = result.AsyncState <b>as</b> WebRequest;
    <b>var</b> response = request.EndGetResponse(result);
    <b>var</b> s = <b>new</b> StreamReader(response.GetResponseStream()).ReadToEnd();
    <b>var</b> search = <b>new</b> Regex(responsePattern);
    <b>var</b> match = search.Match(s);
    <b>if</b> (match.Success)
        Console.WriteLine("{0} is {1}",
           match.Groups["name"].Captures[0].Value,
           match.Groups["status"].Captures[0].Value);
    <b>else</b>
        Console.WriteLine("Unexpected response");
}
</pre>


<p>This will get the online status of a Windows Live Messanger account given the
live ID account number. For example, passing 1afa695addc07e5 as an argument to
the above will tell whether or not I am online. In this case I am using last
parameter of the <code>BeginGetResponse()</code> method to pass the request
itself. This then is cast back to <code>WebRequest</code> in the <code>
PresenceCallback()</code> method so I can call <code>EndGetResponse()</code> to
retrieve the actual response. As far at the <code>BeginGetResponse()</code> call
is concerned, this value is opaque. It ignores it completely and just supplies
it blindly in the <code>IAsyncResult</code>. It makes no assumptions about the
data at all, it is just something the caller just wants carried around. If I was using
anonymous delegates in C# this would look a lot better as,</p>
<pre><b>static</b> <b>void</b> Main(<b>string</b>[] args) {
    <b>var</b> uri = <b>string</b>.Format(uriFormat, args[0]);
    <b>var</b> request = WebRequest.Create(uri);
    request.BeginGetResponse(result =&gt; {
        <b>var</b> response = request.EndGetResponse(result);
        <b>var</b> s = <b>new</b> StreamReader(response.GetResponseStream()).ReadToEnd();
        <b>var</b> search = <b>new</b> Regex(responsePattern);
        <b>var</b> match = search.Match(s);
        <b>if</b> (match.Success)
            Console.WriteLine("{0} is {1}",
               match.Groups["name"].Captures[0].Value,
               match.Groups["status"].Captures[0].Value);
        <b>else</b>
            Console.WriteLine("Unexpected response");
        }
    }, <b>null</b>);

    // Other interesting work....

    result.AsyncWaitHandle.WaitOne();
}</pre>

<p>Here the request local variable <code>request</code> is captured
automatically by the C# compiler and placed into a compiler generated class as a
field. The compiler generated class also contains the code I supplied in the
lambda as an instance method. When I refer to response in the lambda the
reference is automatically translated into a field lookup in the generated
class. Since <code>request</code> is already accessible in the lambda I don't
need the last parameter to carry anything useful so I pass <code><strong>null</strong></code>.</p>
<p>Anonymous methods makes using callbacks much easier but since not all
languages support anonymous methods or lambdas the BCL standardized on a method
pattern for <em>begin invoke</em> that can easily be used by those languages. If the calling pattern did not have a caller state
object then the work performed automatically by the C# compiler would have to be
repeated manually by the programmer in these, then, second class languages. The
.NET team did not want such languages to be second class citizen (especially since neither C# nor VB.NET supported anonymous methods initially)
so they required the presence of the caller state object parameter.</p>
<p>Now we know why it is there, what to do we call it? I like <code>state</code>
because the parameter represents state the caller want's to preserve. I don't
like <code>object</code> because it is a common reserved word. I don't like
<code>stateObject</code> because a symbol's type should not be repeated in the
name, we already know it is an <code><b>object</b></code> by its type. <code>
asyncState</code> is acceptable, especially since that is the name it is given
by <code>IAsyncResult</code>, but it is a bit redundant, we already know, by
context, it is asynchronous state. Plus we should avoid abbreviation, like "async",
in symbol names (though it is very common, and asynchronous is very very long,
so not that bad). <code>data</code> seems fine to me, it is a synonym to state,
but it is overused. <code>extraData</code> I cannot, for the life of me, figure
out. Extra for whom? Extra as opposed to what? Unfortunately, this is the name
given to the parameter by <code>IHttpAsyncHandler</code> (see, I told you "async"
was common).&nbsp; Its name tells me nothing about what I should do with it. It
is very unclear that this value should be mapped to <code>AsyncState</code> in <code>IAsyncResult</code>.</p>
<p>I propose we call it <code>state</code>, with an acceptable variation of
<code>asyncState</code>.</p>
<p>Now, if changing a parameter name was not a breaking change...</p>
<hr>
<p>Trivia: </p>
<ul>
	<li>The above uses the <a href="http://live.com">Live Services</a> that you
	can find more about <a href="http://dev.live.com">here</a>.</li>
	<li>The IM presence service returns JSON which I parse using a fancy looking
	<code>Regex</code> instance. I recommend that production code use <code>
	DataContractJsonSerializer() </code>instead.</li>
</ul>
