---
title: Big O Notation
description: Big O Notation is used to describe how long an alogrithm will take to run in the worst case scenario.
---

## Alogrithm

Alogrithms are a collection or set of defined instructions for solving a specific problem. As we know there are many ways to solve a problem. So we get the correct solution but the way how it is solved may differ. For many people it would be enough to get the correct result.
So we can not evaluate the result of the alogirhtm because it needs to be correct, otherwise there would be already a problem with the alogrithm. So we have to evalute the **performance and efficiency (the time it will take for your algorithm to run/execute and the total amount of memory it will consume)** of the alogrithm.
This is very important for developers to find out how the alogirthm is going to performe, how it is going to scale and it helps the developer to write clean code.

And for that reason we have the Big O Notation.

## Big O Notation

The Big O Notation describes the worst case scenario of how complex our algorithm is. Big O defines how the performance is going to change if we are going to increase the input size for the alogrithm. But it does not tell you how fast the alogrithm is going to run. In the time complexity it can tell you about how long it is going to run.
Big O notation uses algebraic expressions to describe the complexity of the alogrithm. 
It measures the efficiency and performance of your algorithm using time and space complexity. So the space complexity is going to tell you how much memory is going to be used to execute the alogrithm. And the time complexity goint to tell you how long the alogrithm is going to run.

For my work and for many other developers it is important to have a look a the time complexity of their algorithms.

Nice StackOverFlow post that explains Big O notations really handy: https://stackoverflow.com/questions/2307283/what-does-olog-n-mean-exactly 

Here are some of the most important examples for Big O notations and there are many others, but these should be enough for the beginning.

### O(1) -> Constant Time

Algorithms described as O(1) have **no growth rate, meaning they don’t take longer the larger their input gets**. Their growth rate is classed as constant.
For example accessing an array and returing an element at a specific index, will always have the same constant time complexity independet of the array size (the amounts of entries in the array).
Another example would be the inserting of an element into the list or similar things.

**O(1)** would be the best case scenario for an alogrithm.

### O(n) -> Linear Time

Algorithms described as **O(n)** have a linear growth rate. They need to go through the whole data input in the worst case scenario. So the time to complete the alogrithm grows linear with the size of the input. An algorithm that has to go through an entire data set once before completing is likely to be classed as **O(n)**.

### O(log(n)) -> Logarithmic Time

This is similar to linear time complexity, except that the runtime does not depend on the input size but rather on half the input size. The time taken to complete only increases each time the input size doubles, which means as the input size grows substantially the algorithm’s time taken to complete only increases a little.

This method is the second best because your program runs for half the input size rather than the full size. After all, the input size decreases with each iteration.

A great example is binary search functions for sorted arrays. It begins by going to the middle of the array and deciding to take the upper or lower part of the array. In one iteration (run) it instantly knows it can ignore one half of the current array.

### O(n log(n)) -> Logarithmic Time

This is where many of the famous sorting algorithms land on the Big O spectrum. Algorithms defined as O(n log n) have a similar growth rate to **O(n)** except that it’s multiplied by the log of number of elements in the data set.

Example O(n log(n)) would be some sorting alogrithms like the famous **mergesort**. So lets take an alogrithm that loops through the whole array and for each iteration it does something with other elements in the array, that would be **O(n^2)**. But the alogrithm only does something we a specific selection of the array. The algorithm has some way of deciding about which elements to look at and which he currently does not need. And that would be then **O(n log(n)**.

### O(2^n) -> Exponential Time

Algorithms with running time O(2^N) are often recursive algorithms. So with each added item to the input the time to complete is going to double. For example the recursive algorithms are going to solve a problem of the input size by recursively solving two problems with a smaller input size than before until they reach the end.

For example the recursivly calculation of Fibonacci numbers.

### O(n^2) -> Quadratic Time

If a function loops over a dataset and with each item also does something with every other item, you’re looking at O(n^2). Because of how fast the time to complete can grow, this isn’t considered efficient. For example you have two nested loops. The inner loop has to run **n** times and the outer loop has to run **n** time for each iteration of the inner loop.
