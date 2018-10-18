---
layout: post
title:  "Sorting Part II: QuickSort"
date:   2005-08-13 22:53:00
categories: Programming
---
As we noted in Part I, a selection sort is an easy sort to write but
unfortunately it performs on the order of
<em>n<span style="font-size: 8pt;"><sup>2</sup></span></em> comparisons.
To produce an improved version we need to find a piece of information that the
selection sort is throwing away and take advantage of it.

Selection sort will compare a value of the array with all of the other values
of the array to determine which is the smallest. It doesn’t take advantage to
the fact that comparing say,
<em>a<span style="font-size: 8pt;"><sub>10</sub></span></em> to
<em>a<span style="font-size: 8pt;"><sub>20</sub></span></em> and comparing
<em>a<span style="font-size: 8pt;"><sub>20</sub></span></em> to, say,
<em>a<span style="font-size: 8pt;"><sub>40</sub></span></em> could tell us
something about the relationship between
<em>a<span style="font-size: 8pt;"><sub>10</sub></span></em> and
<em>a<span style="font-size: 8pt;"><sub>40</sub></span></em>. What we could do
is take some element of <em>a</em> and compare it to every other element in
<em>a</em> and partition the array into two pieces, one containing all the
elements less than the selected element and all elements greater than or equal
to the selected element. By transitivity, we would know that each element in the
first partition is less than all elements in the second partition even if we
don’t compare them directly. We could then partition the partitions again and
again, in Xeno-esque fashion, until the array only contains partitions of one
element in length. Since each partition will be less than its successor, the
array will be sorted.

An algorithm that takes this very approach is the QuickSort, invented by C.A.R.
Hoare. A version of which looks like the following,

```
public class Sorts {

  public static void QuickSort(int[] a) {
    QSort(a, 0, a.Length - 1);
  }

  private static void QSort(int[] a, int l, int r) {
    // Partition the array into two parts, one with all values
    // greater than a certain value in the array, one with all
    // less.
    int v = a[(l + r) / 2];
    int i = l;
    int j = r;
    while (i < j) {
      // Find the first values that don't belong and swap
      // them. Repeat until all values are in the right
      // partitions.
      while (a[i] < v)
        i++;
      while (a[j] > v)
        j--;
      if (i if (j > i)
        Swap(a, i, j);
      i++;
      j--;
    }
  }

  // Recursively partition the array until we can't any
  // longer. When this happens, the array is sorted.
  if (j > l)
    QSort(a, l, j);
  if (r > i)
    QSort(a, i, r);
}
```

If we get lucky and always guess the median value of each of the partitions we
will only perform O(N log N) comparisons. The worst thing we can do is select a
value that, given an <em>n</em> length partition will give us two partitions one
of length <em>n-1</em> and the other of length 1. This would put us back where
we started; with the selection sort. This would happen if we selected the first
(or last) element in the partition and the array was already sorted. To avoid
this, the above code selects an element from the middle of the partition, so, if
the array is sorted, it will be the median. If the array is in random order, it
is not better or worse than the first element.

<p>This, however, doesn’t guarantee we will get the elusive O(N log N)
performance. Many cases we will be less than O(N log N), those cases where we
don’t magically pick the median. In some pathological cases, we could arrive
very close to O(N<span style="font-size: 8pt;"><sup>2</sup></span>). An example
of which is an array ordered 1..<em>m</em>,1..<em>m</em> where <em>m</em> is
<em>n/2</em>. Next we will examine an algorithm that guarantees O(N log N)
performance.
