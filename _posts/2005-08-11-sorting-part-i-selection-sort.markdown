---
layout: post
title:  "Sorting Part I: Selection Sort"
date:   2005-08-11 23:59:00
categories: Programming
---
When I run across a particularly fast elegant algorithm I often find myself
asking, “Why it is fast?” How can I make my algorithms fast like the author of
this one did? I have found that the answer usually is found in how the algorithm
manages data. How does the algorithm preserve work and prevent recalculating
information? To step back a bit, there are only three ways to make some code go
faster, find a way to make what it is doing faster, find a way to do less, find
some other time to do it. The most elegant algorithms do the second well; they
do less. For a few entries I want to look at various algorithms to see what
makes them fast (or not so fast).

The first set of algorithms I want to investigate are some of the most
fundamental, sorting. The simplest sort to write is the selection sort. In C# it
looks like,

{% highlight csharp %}
static void SelectionSort(int[] a) {
  for (int i = 0; i < a.Length - 1; i++)
    for (int j = i + 1; j < a.Length; j++)
      if (a[i] > a[j])
  Swap(a, i, j);
}
{% endhighlight %}

where `Swap` looks like,

{% highlight csharp %}
static void Swap<T>(T[] a, int i, int j) {
  T v = a[i];
  a[i] = a[j];
  a[j] = v;
}
{% endhighlight %}

This sort finds the smallest item in the list and puts it first, and then it
finds the second smallest and puts it second, etc. Even though it is very easy
to write and relatively easy to verify, it isn’t very fast. We could improve
this by in-lining the `Swap()` method. Additionally, we can limit the number of
swaps to a.Length by only swapping the value we know is the smallest. A version
that includes both optimizations is,

{% highlight csharp %}
static void ImprovedSelectionSort(int[] a) {
  for (int i = 0; i < a.Length - 1; i++) {
    int min = i;
    for (int j = i + 1; j < a.Length; j++)
      if (a[min] > a[j])
        min = j;
    if (i == min) continue;
    int v = a[i];
    a[i] = a[min];
    a[min] = v;
  }
}
{% endhighlight %}

These optimizations are examples of making what we are doing faster.
Unfortunately, we still with do on the order of O(N<sup>2</sup>) comparisons
even though we reduced the number of moves to O(N) and removed any overhead of
doing a call in the middle of the loop. What we really want to do is less. To do
less we need to come up with information we are not using or information we are
throwing away.

Since we are trying to optimize the number of comparisons (since we got the
moves down to O(N)) we should examine them closely. We need to come up with a
way to make each comparison do more. One thing we should notice is the above
algorithm totally ignores the transitive nature of a compare. That is if A > B
and B > C then A > C. If we know that `a[10] > a[11]` and `a[11] > a[12]` we
don’t need to physically do the comparison between `a[10]` and `a[12]` to know
at `a[10] > a[12]`, but how do we take advantage of that? In my next "sorting"
post I will investigate an algorithm that takes advantage of this.
