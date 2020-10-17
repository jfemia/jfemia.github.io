---
layout: post
title: "Redux with React hooks (Part 2)"
subtitle: "Adding some extra features to our simple todo application"
date: 2020-10-17 13:06:14 +0000
background: '/img/posts/2020-10-16/01.jpg'
---

Welcome back. This article continues from [Part 1]({% post_url 2020-10-16-redux-with-react-hooks %}).

In part 1 we covered most of the main Redux concepts but we ended up with a pretty bare application that could only add todo items. For the purpose of a bit of practice, let's work through a few extra features of our app that require us to define some more actions.

We will do two things:
* Set todo items as "done"
* Delete todo items

Usually this type of data management (input, update, delete) would be driven by some kind of persistent data storage. We won't worry about that just yet, however we do need to treat the data similarly - namely we need to generate IDs to identify our todo items.
Without this, we will struggle to update items as we only have the name to work with, and this might now be unique.

We can simply update our reducer, so that when it adds a new todo, it also generates an id for it. Since this is an example, we'll just use a running counter from 1.

We can also update our list display to use the actual todo keys instead of the array index.

Now let's add a couple more actions for updating status, and deleting, using the ID to find the item to update.

Now that we have more than one possible action in our reducer, it becomes more readable to use a switch statement

Let's add a bit more UI to make use of these new actions

There we go, some rough buttons to set a todo as "done", or delete one. 