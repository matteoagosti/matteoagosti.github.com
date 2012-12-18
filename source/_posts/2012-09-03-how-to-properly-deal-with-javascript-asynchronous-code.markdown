---
layout: post
title: "How to properly deal with Javascript asynchronous code"
date: 2012-09-03 12:15
comments: true
categories: [Development, JavaScript]
---

If you are a Javascript developer, you have to deal with asynchronous code. If you are a Javascript developer working with Node.js, asynchronous code is your all in a day's work. Let's see how to properly deal with asynchronous code, avoiding one of the most common mistake.

<!-- more-->

The instrinsic nature of asynchronous code is that its execution block can occur indipendently of the main program flow, thus providing a non blocking mechanism of executing actions. This means that, potentially, when the asynchronous code runs the status of any shared variables could be different than the one where the code was defined.

Consider the following example:

{%codeblock lang:javascript %}
var rankings = ['alice', 'bob', 'eve'];
for (var i = 0, len = rankings.length; i < len; i++) {
  setTimeout(function() {
    console.log(i, rankings[i]);
  }, i * 1000);
}
{%endcodeblock %}

The idea is to print every name from the `rankings` array together with its index. To make the code asynchronous every print is delayed by one second from its predecessor. The output we get is, however, different than what we expected it to be:

{%codeblock lang:javascript %}
3 undefined
3 undefined
3 undefined
{%endcodeblock %}

As you can see, by the time the asynchronous block is executed, the loop is already ended, so the value of variable `i` is the one that stops the loop (`3`). How to prevent this behavior? The answer are **closures**, one of the most powerful features of Javascript; a closure can be seen as a retained scope for Javascript, a function that can have its own variables together with an environment where those variables are binded.

Let's fix the previous example using closures:

{%codeblock lang:javascript %}
var rankings = ['alice', 'bob', 'eve'];
for (var i = 0, len = rankings.length; i < len; i++) {
  (function(i) {
    setTimeout(function() {
      console.log(i, rankings[i]);
    }, i * 1000);
  })(i);
}
{%endcodeblock %}

As you can see the `setTimeout` function is wrapped within a closure that has an `i` argument, whose value is passed at the end of closure definition and corresponds to the loop varialbe `i`. The output we get is the correct one:

{%codeblock lang:javascript %}
0 'alice'
1 'bob'
2 'eve'
{%endcodeblock %}

This is a really simple example and in this case detecting the error is fairly easy, however when you have to deal with nested async calls the execution flow gets more complex and it may become extremely painful to detect where the error is. Hopefully this helps, as it helped me in writing better code :)