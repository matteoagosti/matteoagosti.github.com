---
layout: post
title: "Rate limiting function calls in JavaScript"
date: 2013-01-22 00:23
comments: true
categories: [Development, JavaScript]
---

I have recently struggled against the annoying problem of rate limitations. I was trying to connect to a service that prevents you to do more than 5 calls every 5 seconds. I didn't want to limit calls at the origin, but instead have some sort of mechanism that could handle limitations automatically by queuing calls and allowing bursts. With this article I'd like to share the code that helped me achieve this result.

<!-- more-->

The solution to the problem is a [FIFO](http://en.wikipedia.org/wiki/FIFO "FIFO") queue that gets drained according to the current execution rate. Any operation that needs to be executed is added to the queue once the rate is exhausted and as soon as a slot is available it will get shifted out with the queue entrance priority (first in, first out). In addition to that, I also wanted to introduce a mechanism for handling bursts: there are services that limit your rate using the simple formula `maxOperations / interval`, but others simply check that you don't overpass the maximum number of operations within the allowed interval so that bursts are possible.

I came up with a simple class that you can reuse in your projects whenever you need a mechanism for controlling the rate of function calls. First let's look at the full code and then stop on relevant parts:

{% codeblock lang:javascript %}
var RateLimit = (function() {
  var RateLimit = function(maxOps, interval, allowBursts) {
    this._maxRate = allowBursts ? maxOps : maxOps / interval;
    this._interval = interval;
    this._allowBursts = allowBursts;

    this._numOps = 0;
    this._start = new Date().getTime();
    this._queue = [];
  };

  RateLimit.prototype.schedule = function(fn) {
    var that = this,
        rate = 0,
        now = new Date().getTime(),
        elapsed = now - this._start;

    if (elapsed > this._interval) {
      this._numOps = 0;
      this._start = now;
    }

    rate = this._numOps / (this._allowBursts ? 1 : elapsed);

    if (rate < this._maxRate) {
      if (this._queue.length === 0) {
        this._numOps++;
        fn();
      }
      else {
        if (fn) this._queue.push(fn);

        this._numOps++;
        this._queue.shift()();
      }
    }
    else {
      if (fn) this._queue.push(fn);

      setTimeout(function() {
        that.schedule();
      }, 1 / this._maxRate);
    }
  };

  return RateLimit;
})();
{% endcodeblock %}

You may notice from the class constructor that we have three parameters (I implicitly assume their type and haven't added any type check):

* `maxOps (int)`, maximum number of operations allowed
* `interval (int)`, timespan (in milliseconds) where `maxOps` should be calculated
* `allowBursts (boolean)`, whether or not bursts are allowed

The prototype method `schedule` is the core function of the class and it accepts a function `fn` that will be executed according to rate limits. Further improvements can be done on this as, for example, setting out the context where the function is being called. However, for this small article I tried to keep it as simple as possible.

The actual flow of the schedule algorithm can be described as follows (line numbers from the previous code block):

* `18-20`: first it checks if we are still in the timespan, resetting rate counters otherwise
* `23`: then it calculates the current rate based on bursts allowance, equal to the number of operations executed until now when true, or to the frequency of operations executed until now otherwise
* `25-36`: if the rate is not exhausted it will execute the current operation immediately when the queue is empty, otherwise it will add the current operation to the queue and execute the first operation that was inserted in the queue (here is the FIFO strategy)
* `37-34`: if the rate is exhausted it will add the operation to the queue and postpone the execution of the scheduler on the next available timespan.

You'll notice that on lines `31` and `38` a check on `fn` is being done. This is due to what happens on line `41`: when I postpone the execution of the `schedule` method I do not pass any operation to execute as I want it to process the queue.

Now a working example you can play with to test out the `RateLimit` class:

{% codeblock lang:javascript %}
var rateLimit = new RateLimit(5, 2000, false),
    delay = 0,
    i;

for (var i = 0; i < 20; i++) {
  delay += Math.floor(Math.random() * 10);
  (function(numOperation, delay) {
    setTimeout(function() {
      rateLimit.schedule(function() {
        console.log('Operation %d with delay %d', numOperation, delay);
      });
    }, delay);
  })(i, delay);
}
{% endcodeblock %}

The example creates 20 operations and simulates their arrival in a sequence where items are delayed randomly between 0 and 10 milliseconds (if you wonder why I used a closure for the `setInterval` you should check my article on [How to Properly Deal With Javascript Asynchronous Code](http://www.matteoagosti.com/blog/2012/09/03/how-to-properly-deal-with-javascript-asynchronous-code/ "How to Properly Deal With Javascript Asynchronous Code")). Every operation, a simple console log of operation number and the delay of arrival), is passed to an instance of `RateLimit` configured to allow a maximum number of 5 operations every 2 seconds and without bursts, this means an operation every 400 milliseconds. You can modify the example and see what happens when allowing bursts.

Hopefully this could be any help for you. The code is far from being complete, but it can be considered as a good starting point to develop your own rate limiting strategy. Happy coding!