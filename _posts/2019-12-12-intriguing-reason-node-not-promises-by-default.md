---
layout: default
title: "An intriguing reason why Node.js libraries aren't promises by default"
ShortDescription: "What is up with this .promise() method in the aws node sdk?"
---

A colleague came up to me the other day to ask me two very good questions:

> "Why does the AWS node sdk have this `.promise()` function that you can chain onto all its methods? Why doesn't it just return a promise by default?"

and

> "How do they do that? (support both callbacks by default *and* a `.promise()` method)"

I realised these were two excellent questions to trigger an explanation of why and where we are with promises and node.

On researching this topic I learnt about a fascinating issue with node and promises. So this one's for you, Ravish!


## Let's deal with the first question first:

"Why doesn't the AWS node sdk just return a promise by default?" A simplistic answer to this is it's because for a long time, before promises existed, the Node.js community settled on a convention that allows us to predictably deal with errors and return values in asynchronous functions. It's known as the "error-first callback" and if you've written much node you will probably be familiar with it:

## The error-first callback

By default (as Node.js is of course asynchronous by default) node has a [callback convention](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/) called the "error-first" callback, sometimes cutely named the "nodeback".

The convention is a form of [continuation passing style](https://en.wikipedia.org/wiki/Continuation-passing_style) that states:

> "The last argument you pass us will be a callback function that expects its first argument to be the error object and the rest of the arguments are the result."

> "You have to check the err object in your callback. If it's null, you can assume success, If it isn't you can now deal with the error scenario"

For example:

{% highlight javascript %}
fs.readFile('foo', (err, data) => {
  if (err) {
    // do something in the error scenario
    return
  }

  // err was null so you can now operate on data
})
{% endhighlight %}

This conventions allows us to accomplish what promises "promises" to do: allow us to get return values from async functions *and* deal with errors. With a little help (or boilerplate) from a library like [async](https://www.npmjs.com/package/async) you can do some functional composition with these error-first callback style async functions too.

## So what's the point of promises then?

There is an excellent and famous article entitled: ["You're missing the point of promises"](https://gist.github.com/domenic/3889970) Go have a read if you haven't as it's very interesting.

 > "The point of promises is to give us back functional composition and error bubbling in the async world. They do this by saying that your functions should return a promise, which can do one of two things:
 >
 > * Become fulfilled by a value
 > * Become rejected with an exception
 >
 > And, if you have a correctly implemented then function that follows promises/A, then fulfillment and rejection will compose just like their synchronous counterparts, with fulfillments flowing up a compositional chain, but being interrupted at any time by a rejection that is only handled by someone who declares they are ready to handle it."

However, every time I think of a way promises do something that error-first callbacks don't, there is always a workaround for the callback methodology. For instance, just so long as every async function has a try/catch around its own function and turns any throws into a `callback(err)` then we can deal with throwing. Just so long as we have a convention of passing an error back up the chain, we can deal with exceptions only being handled by someone who declares they want to handle them.

There is always a workaround. Just so long as everyone subscribes to the same conventions, all the compositional challenges of flowing values up the compositional call chain or interrupting the flow with exceptions can be dealt with.

So, promises are just a different convention, with their own conventions that everyone would have to sign up to as well. Of course, I would argue that the boilerplate involved in the convention of promises is much smaller and nicer, but that's largely subjective anyway.

I would argue that promises make it *easier* to combine compositional workflows between async and sync functions, but it's important to note that this is still *possible* with the callback methodology.

## It's also how our brain works

Probably the main reason we are moving towards promises (and even more so Async/await) is that it starts to make async code read like synchronous code. This was "possible" before using libraries like async, but a) there was no convention or standard and b) it was still more boilerplate and jumping through hoops.

As the v8 dev team say in this [https://v8.dev/blog/fast-async](blog):

> With async functions, the code becomes more succinct, and the control and data flow are a lot easier to follow, despite the fact that the execution is still asynchronous.

## Everyone is using promises nowadays

In my experience in building business code (not library code) this is largely true. It makes sense as well, promises cut down on the complexity and boilerplate required to deal with async code.

So, why the divide?

## Why **is** Node.js still "callbacks by default"?

Well here are *some* valid reasons:

 1. Node.js has years of libraries and production code already built on the convention of the error-first callback
 2. Node.js developers highly value simplicity and a basis of "just functions"
 3. Most of the reasons people say promises to be useful turned out to be solvable with callbacks
 4. For a while the Node.js community tried to deal with the problem of "throwing in async code" by Domains which was eventually deprecated

But none of them are objectively an absolute reason why it wasn't sensible to move to promises.

So I researched a little more and came up with an absolute doozy of a real objective problem with using promises.

## Core dumps and debugging

It turns out there is an objective serious issue baked into the promises spec. The Node.js team are serious about their engine. One of the things they know their serious customers need is the ability to do a post-mortem analysis of a Node.js process. This includes a reliable code dump of the complete state of that process at the time an exception occurs.

From the pull request for [adding --abort-on-uncaught-exception with promises](https://github.com/nodejs/node/issues/830
):

> For those new to these discussions, to summarize the problem again most concisely: post-mortem debugging with --abort-on-uncaught-exception collects the entire JavaScript and native state of the program (including all threads) in the precise context of the failure (so that stack state is preserved) with zero runtime cost until the failure happens, while promises explicitly specify that the program must continue executing after the point of the failure, meaning that state is necessarily mutated. Forking is the closest thing to having it both ways, but it has all of the drawbacks brought up here -- it's a heavy cost and it doesn't include all of the same state.

So, the problem is that promises require a program must continue executing after the point of failure. This is because it's impossible to know whether an exception will be caught later on. The only option was to fork the process at the time of an exception, in order to save the state.

Forking the exception specifically has some major drawbacks:

 1. It has a significant performance cost at the time an exception occurs *even if* that exception is caught later on
 2. Fork is not guaranteed to succeed because of potential memory limitations on the host
 3. It won't preserve the whole state in the core dump. Specifically: "State about thread pool threads or other threads created by libraries or other add-ons will be missing from the core file" (from the PR linked above)

## Ah-ha! Finally an absolute objective reason why libraries should NOT provide promises by default

So there we have it. If a library uses promises by default it removes the capability of a user of that library from getting a reliable usable core dump for doing postmortem debugging.

This is not an issue for the majority of application developers. But it's a significant and important issue for some users and an execution engine should absolute not limit this ability.

So what's the answer? Probably the state we have it today. Core libraries expose the error-first callback by default and if you, the user, choose to use Promises you can go ahead. It's trivial to "promisify" a callback function:

{% highlight javascript %}
const util = require('util')
const fs = require('fs')

const stat = util.promisify(fs.stat)
// done :)
{% endhighlight %}

## And back to the AWS library then?

Well my guess is that they have decided to follow the Node.js convention, possibly for the reason that they want to allow reliable postmortem analysis of Node.js processes that are using their library.

But they provide this handy `.promise()` method to satisfy the 99% of their users that don't know or care about this issue.

## References:

A list of places I went to research this article:

 * [https://v8.dev/blog/fast-async](https://v8.dev/blog/fast-async)
 * [https://github.com/nodejs/node/pull/15335](https://github.com/nodejs/node/pull/15335)
 * [https://github.com/nodejs/post-mortem/issues/45](https://github.com/nodejs/post-mortem/issues/45)
 * [https://github.com/nodejs/promises/issues/26](https://github.com/nodejs/promises/issues/26)
 * [https://github.com/nodejs/node/issues/830](https://github.com/nodejs/node/issues/830)
 * [https://gist.github.com/sunnycmf/b2ad4f80a3b627f04ff2](https://gist.github.com/sunnycmf/b2ad4f80a3b627f04ff2)
 * [https://stackoverflow.com/questions/40511513/why-does-node-prefer-error-first-callback](https://stackoverflow.com/questions/40511513/why-does-node-prefer-error-first-callback)
 * [http://callbackhell.com/](http://callbackhell.com/)
 * [https://www.joyent.com/node-js/production/design/errors](https://www.joyent.com/node-js/production/design/errors)
 * [https://github.com/maxogden/art-of-node#callbacks](https://github.com/maxogden/art-of-node#callbacks)
 * [https://gist.github.com/domenic/3889970](https://gist.github.com/domenic/3889970)
 * [http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/)
 * [https://stackoverflow.com/questions/46900782/what-is-the-overhead-of-javascript-async-functions](https://stackoverflow.com/questions/46900782/what-is-the-overhead-of-javascript-async-functions)

## Didn't you forget something?

Oh yes:

> "How does the AWS use the error-first callback mechanism by default but also provide a chained `.promise()` method for all those promise users"

I have no idea, but here's a naïve way of doing it just for fun. Obviously this would be abstracted in such a way that it could be reused across their SDK, but this is just an example. Also of interest is the way the `try {} catch {}` blocks are set up so the actual "do" method can just natively throw exception and the calling code will either turn them into the error callback or a promise rejection.

{% highlight javascript %}
const doS3GetObject = params => {
    if (someErrorHappens) {
      throw new Error('uh oh, we borked')
    }
    else {
      return 'we done good' // imagine we actually did the work!
    }
}

const s3 = {
  getObject: (params, callback) => {
    if (callback) {
      try {
        callback(null, doS3GetObject(params))
      }
      catch (e) {
        callback(e)
      }
    }
    return {
      promise: () => new Promise((resolve, reject) => {
          try {
            resolve(doS3GetObject(params));
          } catch (e) {
            reject(e)
          }
        }),
    }
  },
}
{% endhighlight %}

Thanks for reading and well done for getting through it! Please comment or let me know if you have anything to add or correct.

