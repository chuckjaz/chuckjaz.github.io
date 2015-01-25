---
layout: post
title:  "C# Mixins - Sort of"
date:   2007-07-02 23:42:00
categories: [Programming]
permalink: post/2007/07/02/C-Mixins-Sort-of.aspx
---
<div class="text"><p>I recently needed to create a set of classes that was the combinatorial
expansion of several implementation flavors of two independent sets of methods. Why I needed
combinatorial expansion is not that interesting but the solution is. What I came
up with is a way of supporting a form of
<a href="http://en.wikipedia.org/wiki/Mixin">mixin</a>'s in C#.</p>
<p>Lets take the methods,</p>
<pre><strong>void</strong> Lock();
<strong>bool</strong> Unlock();
<strong>bool</strong> Locked { <strong>get</strong>; }</pre>

<p>If you call <code>Lock()</code>, <code>Locked</code>
returns <code><strong>true</strong></code>, until you call <code>Unlock()</code>. Also <code>Unlock()</code>
returns <code><strong>true</strong></code> if the object is now unlocked. There are several
implementations I can think of for these methods, one uses a Boolean value and
doesn't support nested calls to <code>Lock()</code>:</p>
<pre><strong>void</strong> Lock() {
  _locked = <strong>true</strong>;
}

<strong>bool</strong> Unlock() {
  _locked = <strong>false</strong>;
  <strong>return</strong> <strong>true</strong>;
}

<strong>bool</strong> Locked {
  <strong>get</strong> { <strong>return</strong> _locked; }
}</pre>

<p>Another uses an integer value to count the number of nested calls to <code>
Lock()</code> and it remains locked until a matching number of <code>Unlock()</code>
calls are made.</p>
<pre><strong>void</strong> Lock() {
  _count++;
}

<strong>void</strong> Unlock() {
  _count--;
  <strong>return</strong> _count == 0;
}

<strong>void</strong> Locked() {
  <strong>return</strong> _count &gt; 0;
}</pre>
<p>This isn't thread safe so we could provide a relatively thread-safe version
in,</p>
<pre><b>void</b> Lock() {
    System.Threading.Interlocked.Increment(<b>ref</b> _count);
}

<b>bool</b> Unlock() {
    <b>return</b> System.Threading.Interlocked.Decrement(<b>ref</b> _count) == 0;
}

<b>bool</b> Locked {
    get { <b>return</b> _count &gt; 0; }
}</pre>

<p>I am sure you have already thought of at least one more. Now I have four
possible implementations (including yours), which is the correct one? Unfortunately, that depends
on how the locking is going to be used. If I know I will never need nested
locks, only taking space for a Boolean makes sense. If I know I will need to
support nested locks then the <code><strong>int</strong></code> seems like a
good choice. If it is going to be used in a multi-threaded application I should
probably use the thread-safe version. I now have two choices, pick one
implementation or pass it the implementation as a parameter. Picking one
implementation is easy, just copy of of the above implementation into the class; but how do you efficiently pass an implementation as a
parameter? </p>
<p>From now on I will refer to each implementation as a locking policy. What I
want is an efficient way to pass the locking policy to a set of classes. Most obvious way is to
write a class for each policy and pass the policy
as a parameter to each instance. This might look something like,</p>
<pre><strong>abstract</strong> <strong>class</strong> LockingPolicy {
  <strong>public</strong> <strong>abstract</strong> <strong>void</strong> Lock();
  <strong>public</strong> <strong>abstract</strong> <strong>bool</strong> Unlock();
  <strong>public</strong> <strong>abstract</strong> <strong>bool</strong> Locked { <strong>get</strong>; }
}

<strong>class</strong> LockableObject {
    <strong>private</strong> LockingPolicy _lockingPolicy;

    <strong>public</strong> LockableObject(LockingPolicy lockingPolicy) {
        _lockingPolicy = lockingPolicy;
    }

    <b>public</b> <b>void</b> Lock() {
        _lockingPolicy.Lock();
    }

    <b>public</b> <b>bool</b> Unlock() {
        <b>return</b> _lockingPolicy.Unlock();
    }

    <b>public</b> <b>bool</b> Locked {
        get { <b>return</b> _lockingPolicy.Locked; }
    }
}</pre>

<p>Unfortunately, this has, on average, 36 bytes of overhead per instance on
a 32 bit machine (more on 64).&nbsp; This seems a bit much for such a policy and picking just one
implementation, even one too general for the average application, starts
looking like a better alternative. We must be able to do better than this.</p>
<p>Since passing the implementation as a parameter to the instances is
expensive, why not try to pass the implementation to the class instead? In order
to do that we need use a type parameter. What we want is something like,</p>
<pre><b>class</b> LockableObject&lt;LockingPolicy&gt; {
    <b>private</b> LockingPolicy _lockingPolicy = <b>new</b> LockingPolicy();

    <b>public</b> <b>void</b> Lock() {
        _lockingPolicy.Lock();
    }

    <b>public</b> <b>bool</b> Unlock() {
        <b>return</b> _lockingPolicy.Unlock();
    }

    <b>public</b> <b>bool</b> Locked {
        get { <b>return</b> _lockingPolicy.Locked; }
    }
}</pre>
<p>In other words, we need to passing a <code><strong>class</strong></code> and delegate the implementation to
that <code><strong>class</strong></code>. This does not look significantly better than the previous
implementation, however, because we are new'ing up an instance and assigning it
to an instance variable which, as before, consumes a minimum of 36 bytes per
instance. But if <code>LockingPolicy</code> is a <code><strong>struct</strong></code> instead
of a <code><strong>class</strong></code> then it will only add the size of the
<code><strong>struct</strong></code> to the <code>LockableObject</code> instance
size. In other words, we would only be taking space for either a <code><strong>bool</strong></code>
or an <code><strong>int</strong></code>. The size overhead looks good, now how
about the delegation? In order to call methods on <code>LockingPolicy</code> the
<code>LockingPolicy</code> <code><strong>struct</strong></code> must implement
an interface. It cannot be an <code><strong>abstract</strong></code>
<code><strong>class</strong></code> because all <code><strong>struct</strong></code>s
are sealed and cannot be inherited from. The interface for the locking policy is fairly straight
forward and looks like,</p>
<pre><b>interface</b> ILockingPolicy {
    <b>void</b> Lock();
    <b>bool</b> Unlock();
    <b>bool</b> Locked { get; }
}</pre>
<p>An implementation of the Boolean locking policy looks like,</p>
<pre><b>struct</b> SimpleLockPolicy : ILockingPolicy {
    <b>bool</b> _locked;

    <b>void</b> ILockingPolicy.Lock() {
        _locked = <b>true</b>;
    }
    <b>bool</b> ILockingPolicy.Unlock() {
        _locked = <b>false</b>;
        <b>return</b> <b>false</b>;
    }
    <b>bool</b> ILockingPolicy.Locked {
        get { <b>return</b> _locked; }
    }
}</pre>
<p>The counted locking policy looks like,</p>
<pre><b>struct</b> CountedLockPolicy : ILockingPolicy {
    <b>private</b> <b>int</b> _count;
    <b>void</b> ILockingPolicy.Lock() {
        _count++;
    }
    <b>bool</b> ILockingPolicy.Unlock() {
        _count--;
        <b>return</b> _count == 0;
    }
    <b>bool</b> ILockingPolicy.Locked {
        get { <b>return</b> _count &gt; 0; }
    }
}</pre>
<p>We now need to add a <code><strong>where</strong></code> clause to the <code>LockableObject</code>
class so we can call the methods provided by the <code><strong>struct</strong></code>'s methods. The modified <code>LockableObject</code>
class looks like,</p>
<pre><b>class</b> LockableObject&lt;LockingPolicy&gt;
    <strong>where</strong> LockingPolicy : <b>struct</b>, ILockingPolicy
{
    <b>private</b> LockingPolicy _lockingPolicy = <b>new</b> LockingPolicy();

    <b>public</b> <b>void</b> Lock() {
        _lockingPolicy.Lock();
    }

    <b>public</b> <b>bool</b> Unlock() {
        <b>return</b> _lockingPolicy.Unlock();
    }

    <b>public</b> <b>bool</b> Locked {
        get { <b>return</b> _lockingPolicy.Locked; }
    }
}</pre>
<p>The <code>LockableObject</code> can now be instantiated like,</p>
<pre><strong>var</strong> lockableObject = <b>new</b> LockableObject&lt;SimpleLockPolicy&gt;();</pre>
<p>for a lockable object that uses the Boolean implementation. To use the
counted locking policy you would construct the object like,</p>
<pre><strong>var</strong> lockableObject = <b>new</b> LockableObject&lt;CountedLockPolicy&gt;();</pre>

<p>The space consumption looks good but the delegation looks expensive it call
through an interface which is a virtual. But since we are using <code><strong>
struct</strong></code>s, which are <code><strong>sealed</strong></code>, the
call through the <code><strong>interface</strong></code> can be made static
and then be a candidate for inlining. In fact,
this is what happens in retail builds when not debugging. This is not entirely
free however since there the runtime doesn't inline twice. Implementing <code>
Lock()</code> directly instead of through delegation means the <code>Lock()</code>
method can be inlined, removing the call entirely, the delegated version inlines
the delegation but it still generates a call.</p>
<p>This mechanism for C# mixins is not as convenient as languages that support
multiple inheritance, because we have to manually delegate to the policy, but it
is just as efficient as we will see demonstrated more clearly in some of the
next entries.</p>
</div>
