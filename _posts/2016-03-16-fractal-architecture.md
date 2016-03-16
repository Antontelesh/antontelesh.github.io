---
layout: post
title:  "Fractal Architecture"
date:   2016-03-16 14:10:00 +0300
categories: architecture
---

Today in 2016 there are hundreds of frameworks and libraries that provide solutions to structure an application.
Many developers are even experiencing so-called JavaScript fatigue, caused by a diversity of tools existing today.
To prevent getting lost in this ocean of technologies, I prefer to find common patterns in all of them.
Tools are simply implementations of patterns.
If you understand patterns, you know all the tools.

### Theory

Some of the frameworks apply a pattern that I call "fractal architecture".
This type of architecture should be the most popular today.
To let you know what I mean by fractal architecture I will list some of its rules:

1. Application is a tree of components of the same API.
2. Each component is able to contain (use) other components.
3. Top component does not differ from other components.
4. Glue and application lay apart from each other.

### Examples

Now I'm going to show how these rules apply to some popular frameworks.

#### React/Angular components

Both React and Angular use the word "component" to name some of their entities.
Components in React and Angular are organized into trees.
They are composed together by APIs similar to XML.
And what they produce at the end are DOM trees.

Both Angular and React provide a way to bootstrap an application.
Bootstrap API is different from component API.

#### Elm

Elm in not a framework but language.
But there is [The Elm Architecture](https://github.com/evancz/elm-architecture-tutorial).
This architecture is also fractal.
An Elm application consists of components (you can call them modules).
Each component exposes three functions: `init`, `update` and `view`.
Each component can consume these functions from other components.
Bootstrapping is done by passing root component to a special function `main` with some preparations.


#### Cycle.js

In Cycle each component is a "pure" function (meaning it does not have any side effects) and a complete application.
It accepts some sort of observables and exposes another sort of observables.
Each component can use other components within itself, almost how a function calls other functions inside its body.

Wiring of components is done separately from components declaration.

### What's wrong with Redux

**There is nothing wrong with Redux itself.**

But sometimes I see how Redux is being used wrong.
It's necessary to notice that Redux has been created with influence of Elm and Flux.
Elm leads to fractal architecture. Some Redux applications not.

The case is that there is no concept of "Redux Application".
Redux itself is a library to manage data flow, nothing more.

Many people including myself do not pay much attention to application boundaries.
They write an application with a thought that there will be only one instance of it,
one instance of root component, that application won't be used inside another application.
This leads to non-scalable architecture.

#### How to fix that

Redux application scalability can be achieved by keeping it in mind while writing code.
This is possible by following the rules written at the top of this post.
With a context of Redux in mind, they can be interpreted as follows:

1. Make an application reusable.
2. Keep it apart from bootstrapping boilerplate.

How to make an application reusable?
By making it composable.
How to make it composable?
By treating an application as a module.
There are some requirements for module in redux application:

1. Module boundaries and API are well defined.
2. A module is not responsible for creating redux store or any other dependency that is meant to be a singleton.
3. A module accesses its state from its API. It's not responsible for knowing the full application state.
4. A module exports its reducer. It's not responsible for registering the reducer in the store.

#### Example

Let's apply these rules to a simple application that uses two counters.

First, let's define a `Counter` module.
Remember that each module can be used as an application.

{% highlight js %}
// Counter.js
function Counter () {
  
  const reducer = (state, action) => {
    if (action.type === 'INCREMENT') {
      return state++;
    }
    
    if (action.type === 'DECREMENT') {
      return state--;
    }
    
    return state;
  }
  
  const view = (dispatch, state) => {
    // some react component or HTML string
  };
  
  return {
    reducer,
    view
  };
  
}
{% endhighlight %}

As you see, our `Counter` is a factory.
It has no dependencies, so it could be a simple object.
But now it's a function that returns simple object.
The return value is our module's API.
I decided that the module in this case should return its reducer and a view function.
Reducer is the same as always.
View function takes dispatch function from redux store and current state of a counter.

As you can see, this module does not store any state itself.
Both `reducer` and `view` are pure functions with their dependencies.
Also it does not create redux store. It only accesses store's `dispatch` function via arguments.

Now let's define our `App` module, that will use just defined `Counter` module:

{% highlight js %}
// App.js
function App (Counter) {
  
  const reducer = combineReducers({
    top: Counter.reducer,
    bottom: Counter.reducer
  });
  
  const view = (dispatch, state) => {
    let top = Counter.view(dispatch, state.top);
    let bottom = Counter.view(dispatch, state.bottom);
    // use top and bottom views
  };
  
  return {
    reducer,
    view
  };
}
{% endhighlight %}

Our `App` module is also a factory that returns an object of the same shape as our `Counter` module.
This means that both modules have the same API.
Our `App` module accepts `Counter` via its arguments.
That's how we inject `Counter` module into an `App` module.

Again, our `App` module exports two functions: `reducer` and `view`.
They're both pure.

Let's wire those components together.

{% highlight js %}
// bootstrap.js
const counter = Counter();
const app = App(counter);
const store = createStore(app.reducer);

store.subscribe(state => {
  render(app.view(store.dispatch, state));
});
{% endhighlight %}

As you can see, we're finally creating redux store here and instantiating our app.
All bootstrapping boilerplate comes apart from our modules.
That allows us to use those modules again in another parts of our application or even in other applications.


### Advantages of fractal architecture

Fractal architecture applies one simple pattern across all the application.
If you know how one component works, it means you know how every of them work.

Another advantage is that each piece of an application is easily extractable into a separate application,
like each subtree is a valid tree.
That leads to easier refactoring, easier testing in isolation, and to code reusability.
