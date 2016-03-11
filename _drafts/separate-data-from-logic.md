---
layout: post
title:  "Separate Data From Logic"
date:   2016-03-09 14:57:18 +0300
categories: patterns generic javascript
---

Some time ago I was making one big mistake when writing code — I was coupling data with logic.
What does that mean?

Assume you have an application that stores all books you have read.
It stores some info about each book, such as title, author, description and your personal rating.
You can look at the books you have read and the ones in your reading list.
You can create new books, edit existing ones or remove some books at all.

Each book can be represented like this:

{% highlight javascript %}
let book = {
  id: 1,
  title: '2001: A Space Odyssey',
  author: 'Arthur C. Clarke',
  description: `A 1968 science fiction novel by Arthur C. Clarke.
          It was developed concurrently with Stanley Kubrick's film version and published after the release of the film.`,
  rating: 5
};
{% endhighlight %}

What was my mistake? I created a class named `Book` which contained all book data and methods to manipulate that data:

{% highlight js %}
class Book {
  constructor(data) {
    Object.assign(this, data);
  }

  get isHighestRating() {
    return this.rating === 5;
  }

  setRating(rating) {
    this.rating = rating;
  }
}
{% endhighlight %}

When I was receiving books from database, I first mapped all data to create `Book` instances:

{% highlight js %}
const books = response.map(book_data => new Book(book_data));
{% endhighlight %}

That was not so bad while `Book` class was being used in only one module, where it had beed declared.
But the project was growing and some time later we had got a lot of modules.

Some of them required one piece of `Book` class functionality. Others required another pieces.
For example, book form required `setRating` method.
Book list required `isHighestRating` getter to highlight books with highest rating.
The `Book` class grew up to hundreds of lines of code.
It became hard to use it in several modules, because it contained some dependencies that weren't available.
Also it became hard to serialize it.
I had to write special formatters in each module to run just before sending data to server.
Simple and trivial idea became hell.

What I would do now, when I know the drawbacks of coupling logic and data?
I would use plain old JavaScript objects for storing data, just as in the first code snippet:

{% highlight javascript %}
let book = {
  id: 1,
  title: '2001: A Space Odyssey',
  author: 'Arthur C. Clarke',
  description: `A 1968 science fiction novel by Arthur C. Clarke.
          It was developed concurrently with Stanley Kubrick's film version and published after the release of the film.`,
  rating: 5
};
{% endhighlight %}

I would treat each action related to book as first class citizen.
That means that actions will be freed from context.
They will be simple functions in appropriate modules:

{% highlight js %}
const BookHelpers = {
  isOfHighestRating: (book) => 
    book.rating === 5
};

const BookFormHelpers = {
  setRating: (book, rating) => {
    book.rating = rating;
    return book;
  }
};
{% endhighlight %}

With this approach we can access all necessary logic without extra dependencies.
Now we depend only on shape of `book` object. This dependency is now implicit.
But we can make it explicit if we describe its shape using interface:

{% highlight typescript %}
interface Book {
  id: number;
  title: string;
  author: string;
  description: string;
  rating: number;
}
{% endhighlight %}

Now all raised problems are solved:

* We're using only necessary functionality.
* All functions are context-agnostic and can be easily reused.
* All our dependencies are explicit.
* No need to create hundreds of instances of `Book` class.
* No need to create formatters that stringify `Book` class before saving to database.

----

The main intent of this post is to help you avoid this pitfall. It is very useful to keep your logic and data separate. It also allows another types of optimizations. We will cover them later.