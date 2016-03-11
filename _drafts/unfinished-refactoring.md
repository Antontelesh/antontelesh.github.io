---
layout: post
title:  "Unfinished Refactoring"
date:   2016-03-09 14:57:18 +0300
categories: refactoring complexity
---

I don't have any experience in writing Ruby but I know that this language has a great community.
This community is one of the greatest among developers.
It's because they put a lot of attention to make their code clean, easy to read and maintain.
That's why I like to learn from Ruby community.
Most of the principles they teach are universal and applicable to JavaScript world too.

One of my favorite speakers is Sandi Metz ([@sandimetz](https://twitter.com/sandimetz)).
She is a great teacher and has a lot of experience.
Each of her talks found on YouTube is very interesting.
I recommend wathing them all.

But now I want to speak about [one exact Sandi's talk](https://youtu.be/8bZh5LMaSmE) which is about refactoring.
In this talk Sandi takes some code, measures its complexity, then does some refactoring and again measures code complexity.
It's predictable that after all refactoring the code will be less complex.
That's the goal of refactoring.
But what's interesting is that *during* refactoring code complexity increases.

That can be explained by the fact that during refactoring new entities are added to a codebase but old ones are not yet removed.
So the code contains more entropy than it contained initially.

I can extract two lessons from Sandi's talk:

1. It's important to always finish refactoring.
2. Don't fear temporal increase of code complexity when you're doing refactoring.

### My experience

A year ago I faced exactly the same problem.
The project I was working on contained lots of code.
Each piece of code depended on another one, but all that dependencies were described only by AngularJS.
AngularJS provides a way to describe module dependencies, but not file dependencies.
That's why we had a json file, that contained paths to all necessary scripts.
That paths were put in order in which scripts should be included onto webpage.
Bundling was made by simple concatenation of those scripts.

At that time `webpack` became popular and solid.
It provided solutions to all described problems:

* Native dependency injection based on CommonJS or ES2015 imports.
* Universal approach to define all dependencies: own javascript, html and css as well as vendor code via `npm`.
* Cleaner dependency description. No more need to manually place scripts in order.
* Leads to framework-agnostic code.
* Leads to easier testing.

Having these benefits in mind we decided to move bundling of all our codebase to `webpack`.
As I said before, we had a lot of code with poor module boundaries.
Also there was no satisfactory solution to handle templates.

We have described project structure we wished to get and started moving code module by module.
Fortunately we already had code that was split onto modules.
Unfortunately those modules did not have strong api boundaries.

After some modules had been moved we had to pause refactoring and switch to implementing new features.
At that moment there were two ways to implement new code — using the old approach with concatenation and the new one with `webpack`.
Both worked in parallel, so our codebase remained stable.
But overall complexity of code increased.
New team members had to learn both ways of bundling.

Though we haven't finished that refactoring, we've got some benefits from it.
First, that lead us to define strong module boundaries.
Second, that allowed us to write new features using `webpack`, that is a lot more comfortable.
Third, we proved that good refactoring plan allows to pause huge refactoring and push intermediate results without fear of breaking application.

### What to do?

So now we know that it's extremely important to finish refactoring.
How to do that? I think the steps are the following:

1. Draw two states: the state you're now in and the state you wish to be in.
2. Draw a path between these two states.
3. Split that path onto small self-contained iterations.
   After each iteration your application should be in a stable, ready to production, state.
   Iterations should be as small as possible, so you can finish each of them quickly.
4. Complete iterations one by one.

----

We used these steps to make our migration plan.
That helped us to benefit even from unfinished refactoring.

I wish you to always finish what you start.
It's doable with right attitude.
