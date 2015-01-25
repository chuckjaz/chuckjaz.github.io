---
layout: post
title:  "Sorting Part III - HeapSort"
date:   2005-08-18 09:55:00
categories: Programming
---
Last time we looked at the QuickSort and found that it was much better, in most
cases, than the selection sort, but there was still a pathological case where
the QuickSort wasn’t better. Can we guarantee we <em>are</em> better than the
O(N<sup>2</sup>) of the selection sort?
Can we guarantee O(N log N)? To do this we need to find something in QuickSort
we can do better. The Achilles’ heal for the QuickSort is selecting the median
value of the partition. Several techniques have been developed to avoid the
worst case scenarios (like selecting three values and taking the median; in
other words, median of three, or just picking a random item in the partition)
but none of these guarantee O(N log N). We need a more radical approach.

We need a way of preserving the comparisons; much like QuickSort does, but that
doesn’t use partitions. As is often the case, when you need a radical new
approach, you should try a different data structure (much like choosing a
different coordinate system in calculus). If we organize the array into a binary
tree, we can find the largest element in the array in O(N log N) comparisons.
This is because each element only needs to be compared with its parents, which
is O(log N) comparisons, and we need to do this for every element in the in the
array, O(N), giving us O(N log N). This sounds promising, but how do you
organize an array into a binary tree? This is easier than it sounds. If you
number an array from <em>1</em> to <em>n</em>, where each number is the index of
the elements in the array, you can treat the array as a binary tree by defining
the left node of any element <em>i</em> to be at index <em>2i</em> and the right
node to be at <em>2i+1</em> (or the other way around, if you must, but I like to
think of the odd children as right). To create the desired tree, we need to
ensure that for every element in the array at index <em>i</em>, it is the
greater than its children. In other words, for all elements of array <em>a</em>,
<em>a<sub>i</sub> ≥ a<sub>2i</sub></em> for <em>2i ≤ n</em> and
<em>a<sub>i</sub> ≥ a<sub>2i+1</sub></em> for <em>2i+1 ≤ n</em>. An array that
obeys this constraint is called a heap (hence the name HeapSort).

Forming a _heap_ is fairly straight forward and follows the informal description
given above; you swap each element, from _2_ to _n_, with its parent until you
come across a parent that is greater than the element. The algorithm for this
might look something like,

{% highlight csharp %}
static void FormHeap(int[] a) {
  // form the heap
  for (int i = 1; i < a.Length; i++) {
    int j = i;
    while (j > 0) {
      int k = (j + 1) / 2 - 1;
      if (a[k] < a[j])
        Swap(a, j, k);
      else
        break;
      j = k;
    }
  }
}
{% endhighlight %}

Note, that since C# numbers the array from _0_ to _n-1_ instead of from _1_ to
_n_, we need to subtract add one to the index _j_ and then subtract it again
while calculating the parent’s index.

Now that we have the have the greatest element in the array, we know which
element should be at the end of the array. We need only swap it with the last
element of the array to put it there, but doing so will violate the _heap_
constraint above since _a<sub>n</sub>_ is guaranteed to be less than or equal to
both _a<sub>2</sub>_ and _a<sub>3</sub>_. To restore order back to the _heap_ we
must push the new first element down in the heap until it is greater than or
equal to both of its children or it is childless. This can be done by selecting
the greatest child and swapping positions with it then repeating until it is
greater than the children or it has no children. Since we are following a single
line of succession, an element will only have, at most _log<sub>2</sub>n_
generations (or levels, as it is more traditional called) below it. This means
we can restore order to the _heap_ in O(log N) comparisons. This might look
like,

{% highlight csharp %}
static void ReformHeap(int[] a, int i) {
  int j = 0;
  while (true) {
    int k = (j + 1) * 2 - 1;
    if (k >= i)
      break;
    if (k + 1 >= i || a[k] > a[k + 1]) {
      if (a[j] > a[k])
        break;
      Swap(a, k, j);
      j = k;
    }
    else {
      if (a[j] > a[k + 1])
        break;
      Swap(a, k + 1, j);
      j = k + 1;
    }
  }
}
{% endhighlight %}

We now need to do this <em>n</em> times to completely sort the array, meaning that we are guaranteed to sort a <em>heap</em> in O(N log N) comparisons. This gives us O(N log N) operations to form a <em>heap</em> and O(N log N) operations to sort a <em>heap</em> which means that we can sort any array, using a <em>heap</em> in O(N log N) (since big O notation throws away the constant 2 in 2N log N). Putting this together gives us the <em>HeapSort</em> invented by J.W.J. Williams,

{% highlight csharp %}
static void HeapSort(int[] a) {
  // form the heap
  for (int i = 1; i < a.Length; i++) {
    int j = i;
    while (j > 0) {
      int k = (j + 1) / 2 - 1;
      if (a[k] < a[j])
        Swap(a, j, k);
      else
        break;
      j = k;
    }
  }

  // Pick the top of the heap and place it last, reform the
  // heap, then take the new top and place it next to last.
  for (int i = a.Length - 1; i > 0; i--) {

    // The top of the heap contains the highest value, swap
    // it the last member
    Swap(a, 0, i);

    // Reform the heap it. It contains a relatively low
    // value on top. Replace it with the greatest child.
    int j = 0;
    while (true) {
      int k = (j + 1) * 2 - 1;
      if (k >= i)
        break;
      if (k + 1 >= i || a[k] > a[k + 1]) {
        if (a[j] > a[k])
          break;
        Swap(a, k, j);
        j = k;
      }
      else {
        if (a[j] > a[k + 1])
          break;
        Swap(a, k + 1, j);
        j = k + 1;
      }
    }
  }
}
{% endhighlight %}

Here we have it. An algorithm that is guaranteed to perform on the order of O(N log N) operations to sort an array! And here, you are apt to say, “Why doesn’t everyone use a HeapSort instead of a QuickSort if HeapSort is so much better?” Good question, I am glad you asked. There are several reasons why, but I will save that for another time.
