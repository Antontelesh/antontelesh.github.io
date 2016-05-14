---
layout: post
title:  "Make Your Intention Explicit"
date:   2016-05-14 19:00:00 +0300
categories: architecture
---

What differentiates a good developer from an average one?
One of the main developer's professional skills is their ability to see through the problem and find its root.
Each problem is based on some initial intention.
The ability to see that intention and to keep it in mind is very valuable.

Look at yourself.
Do you like what you do?
Do you feel pleasure while doing your job?
Do new solutions come natural to your mind or you fight with your old ones?
How frequently do the previous results of your work get in your way?
**If you find yourself struggling with your code, you're doing it wrong.**

How to do it right?
Always ask yourself the following questions:

* Why am I doing it right now?
* Can I simply get rid of this?
* Is this solution explicit?
* Will a newcomer understand what's going on there?
* How can I avoid creating this entity?

Do you see a pattern here?
Each of these questions really asks you to think before adding something new to the codebase.
Always strive to delete more code than to write, because unwritten code has no bugs.
It does not take time to write it, maintain it and test it.
Here is the rule: **no code — no problems**.
By writing more code you create more problems. Why?

Having more code to maintain requires more time and money, creates additional complexity and slows down development.

Having less code to maintain saves you money and time and speeds up development. All the remaining code becomes more readable, straightforward and thus more elegant.

### Let me show you a small example.

Imagine you've just come to a new project, you clone the repository, run build script and it fails with an error like

> Module 'a' not found. File 'a.js' does not exist.

You spend few hours trying to investigate the reason, you ask your colleagues. But no way. They don't know the reason too.

Finally, at the end of the day you find out that file `a.js` should have been copied to your build directory from another one, but it didn't. The reason for this is that the script used for copying the files is too weak — it can't create a directory if the latter does not exist.

### What would be a bad solution?

Maybe, creating that directory manually as all your colleagues have done long time ago and already forgot they did.
This solution would help. The file would be copied and the build would pass.

But what can happen a year later?
You will forget about your solution and won't help a newcomer to solve this the next time.
The newcomer will have to spend a whole day to investigate the issue as you did a year ago.

### What would be a better solution?

If you looked through the problem and asked yourself those questions I mentioned before, you would decide to replace the copying tool with a more powerful one.
That will solve the problem for you and for your followers.

### Can we do even better?

Maybe.
It depends on situation, but we should consider removing the copying step completely. Remember? Less code leads to less problems.

____
That's all for today.
Remember to constantly ask yourself those questions.
This is useful not only for the development.
