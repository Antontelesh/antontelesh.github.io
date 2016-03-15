---
layout: post
title:  "Composition in Elm"
date:   2016-03-15 09:00:00 +0300
categories: elm
---

[Elm](http://elm-lang.org) is a functional programming language for the web.
It compiles to JavaScript to run in browser.
One of Elm's main advantages is strong type system combined with immutability.
For a developer it means that Elm's compiler catches all possible errors in code.
It provides safety that your code will work and not crash at runtime.

The language also provides [its own architecture](https://github.com/evancz/elm-architecture-tutorial), that became popular last year.
I became acquainted with this language several months ago, when tweets about its architecture started to appear, and [Redux](https://github.com/reactjs/redux) didn't yet exist.

Today we will speak not about the elm architecture but about the language itself.
Elm is very easy to learn because of its simplicity.
Unlike JavaScript it has very few ways to do one thing.
This kind of restriction makes the language easier to apprehend.

Functional programming languages pay a lot of attention to composition.
In JavaScript I'm used to write almost all my functions as composition of other ones.
Almost every time I want to create a new function I write:

{% highlight js %}
const functionName = compose(...);
{% endhighlight %}

compared to

{% highlight js %}
function functionName () { ... }
{% endhighlight %}

I use second syntax only for cases where there is nothing to compose.
Such cases are rare.

This `compose` function comes from some library like Lodash or Underscore.
This function is variadic.
It means that it can accept any number of arguments.
If you pass it two functions, it will compose two functions together.
If you pass five, it will compose all the five.

Elm does not support variadic functions.
That's why at first I was curious about how to use composition in Elm.

In fact, composition in Elm is a lot more advanced than `compose` function in JavaScript.
Elm provides three ways to compose functions:

* Backward function application
* Forward function application
* Function composition

Let's examine all of them.

### Backward Function Application

This type of composition has no equivalent in JavaScript and was created to bring some syntactic sugar to function application.
Let's see the problem it solves.

Assume we're implementing a calculator app in JavaScript.
We have created functions `add`, `subtract`, `multiply` and `divide`.

So to calculate `100 + 10 * 2` we can write:

{% highlight js %}
let result = add(100, multiply(10, 2));
{% endhighlight %}

And an equivalent in Elm:

{% highlight elm %}
result = add 100 (multiply 10 2)
{% endhighlight %}

See parentheses?
They are there for `multiply 10 2` to execute first, before calculating `add 100 20`.
These parentheses are necessary, without them the code will not compile.

Now assume we have lots of such parentheses:

{% highlight elm %}
result = add 100 (multiply 10 (subtract 5 3))
{% endhighlight %}

To clean up the code and remove parentheses without loosing readability we can use backward function application:

{% highlight elm %}
result = add 100 <| multiply 10 <| subtract 5 3
{% endhighlight %}

Look at the execution order:

{% highlight elm %}
result = add 100 <| multiply 10 <| subtract 5 3
result = add 100 <| multiply 10 <| 2
result = add 100 <| multiply 10 2
result = add 100 <| 20
result = add 100 20
result = 120
{% endhighlight %}

As you can see, backward function application applies functions instantly and cannot be used to create function from other functions.
But it's very useful for prettifying code.

### Forward function application

Forward function application is like backward function application, but it applies functions in reverse order.
At first it seems that there is no need for such diversity, but in fact forward function application is a lot more useful.
It provides ability to place our data first and then describe manipulations with it.

Let's see some code:

{% highlight elm %}

append : String -> String -> String
append a b =
  b ++ a

exclaim str =
  str
    |> String.toUpper
    |> append "!"
  
{% endhighlight %}

Now let's call `exclaim` from REPL:

{% highlight elm %}
> exclaim "hello world"
"HELLO WORLD!" : String
{% endhighlight %}

You can see that forward function application is useful when there are a lot of manipulations with data.
Without this pattern the code of `exclaim` will look like this:

{% highlight elm %}
exclaim str =
  append "!" (String.toUpper str)
{% endhighlight %}

A little shorter but less readable.

Forward function application applies functions instantly, like backward one.

### Function composition

And the last way to compose functions is traditional function composition.
This is my favorite one, because it allows to compose functions in a way I'm used to.

Let's implement our `exclaim` function with traditional function composition:

{% highlight elm %}
exclaim = append "!" << String.toUpper
{% endhighlight %}

Wow! Such short and readable, also poitfree.

But what if we want to compose more than two functions together?
That's simple and native. Just use more `<<` arrows:

{% highlight elm %}
magicFormula = add 100 << multiply 10 << subtract 5
{% endhighlight %}

Let's try this in REPL:

{% highlight elm %}
> magicFormula 3
120 : number
{% endhighlight %}

Works!

I use all three types of function composition in Elm, but the last one is my favorite,
because it allows to write reusable functions that can be composed together to get another functions.
I like point-free style of coding and use it even in JavaScript.

Probably I'll write another blog post covering point-free style of coding, its pros and cons.
Stay tuned.
