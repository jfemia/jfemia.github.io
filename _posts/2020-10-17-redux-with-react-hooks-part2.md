---
layout: post
title: "Redux with React hooks (Part 2)"
subtitle: "Adding some extra features to our simple application"
date: 2020-10-17 13:06:14 +0000
background: '/img/posts/2020-10-16/01.jpg'
---

Welcome back. This article continues from [Part 1]({% post_url 2020-10-16-redux-with-react-hooks %}).

In part 1 we covered most of the main Redux concepts but we ended up with a pretty bare application that could only add todo items. For the purpose of a bit of practice, let's work through a few extra features of our app that require us to define some more actions.

We will do two things:
* Set todo items as "done"
* Delete todo items

# Dealing with existing items

We have some items in our todo list and we want to do something to them. Usually this type of data management (input, update, delete) would be expected to be persistent, driven by some kind of backend REST API and survive between visits to the site. 

We won't worry about that for this tutorial, however we do need to treat the data in our store in a similar way - namely we need to generate IDs to uniquely identify our todo items. Without unique IDs we will struggle to update our todo items accurately. Right now we only have the name to work with, and this might not be unique.

# Identifying todo items

In order to generate IDs for our todo items, we can simply update our reducer logic. When we process the `ADD_TODO` action, we can also generates an id for it, and insert it with the title. For simplicity, we'll just use a running counter, starting from 1.

When we last touched our reducer function in `src/store/todo.js`, it looked like this:
```js
const initialState = [];

export default function(state = initialState, action) {
  if(action.type === ADD_TODO) {
    return [...state, { title: action.payload.title }];
  }
  else {
    return state;
  }
}
```

In order to add IDs to our todos, we can just start counting and add the current value as a field on the todo object:
```js
const initialState = [];
let nextId = 1; // define a variable to hold the next id

export default function(state = initialState, action) {
  if(action.type === ADD_TODO) {
    // update our reducer to add the next id and increment it
    return [...state, { title: action.payload.title, id: nextId++ }];
  }
  else {
    return state;
  }
}
```

Now every time we dispatch `ADD_TODO`, the resulting todo item contains a unique number. (Unique enough for this example, at least...)

We can also update our list display in `src/App.js` to use the actual todo keys instead of the array index, so we display accurate information about the items:
```jsx
  //...

  // Update our map to use the unique ids as the React element key,
  // and also show it in the list
  return (
    <div className="App">
      <header className="App-header">
        <input type="text" ref={inputRef} />
        <button onClick={() => dispatch(addTodo(inputRef.current.value))}>Add</button>
        <ul>
          {todos.map(t => <li key={t.id}>
            ({t.id}) {t.title}
          </li>)}
        </ul>
      </header>
    </div>
  );
```

If we add a few items to our list in the UI now, we end up with some output like this:
> * (1) Some item
> * (2) Some other item

Now that we have a way to identify items, let's add a couple more actions for updating status and deleting, using the ID to find the item to update.

# Updating and Deleting

Remember how we defined our `ADD_TODO` action? Let's add a couple more in `src/store/todo.js`, starting with the name definitions:
```js
const UPDATE_TODO = "UPDATE_TODO";
const DELETE_TODO = "DELETE_TODO";
```

Next, the action builder functions:
```js
export const updateTodo = (id, status) => ({ type: UPDATE_TODO, payload: { id, status }});
export const deleteTodo = (id) => ({ type: DELETE_TODO, payload: { id }});
```

Notice that both of these functions take an id as a parameter - our reducer will need this in order to find the correct todo item in the state. The `updateTodo` function also takes a `status` parameter, which for the sake of this example will just be an arbitrary string.

Next let's update our reducer - Now that we have more than one possible action in our reducer, it becomes more readable to use a switch statement instead of a lot of `if`/`elseif`.

Let's start by changing what we currently have, before adding additional cases for the new actions - ending up with our reducer looking like this:
```js
export default function(state = initialState, action) {
  switch(action.type) {
    case ADD_TODO:
      return [...state, { title: action.payload.title, id: nextId++ }];
    default:
      return state;
  }
}
```

For updating, we need to find the todo in the state and provide it with a status field. But remember, we can't mutate the object directly - we need to create a new state array, and a new todo object to replace the one specified by the `id` parameter:
```js
switch(action.type) {
  // ... other case statements ...

  case UPDATE_TODO:
    return state.map(t => {
      if(t.id === action.payload.id) {
        return { ...t, status: action.payload.status };
      }
      else {
        return t;
      }
    });
}
```

In our example we can just use the array `map` function, to loop through the todo items and return them, resulting in a new array of todo items. Note that we don't need to copy the items that we don't change, so we return the unmodified todo items in most cases. If the item is the one requested, we instead return a new object containing the original values, and the additional status value.

Deleting is even simpler - just use `filter` to return a new array without the item we are deleting:
```js
switch(action.type) {
  // ... other case statements ...

  case DELETE_TODO:
    return state.filter(t => t.id !== action.payload.id);
}
```

# A bit more (ugly) UI
Let's add a bit more UI to make use of these new actions - namely two buttons, one to set the todo as "done", and another to delete the item.

All of the changes are to the list items we create in `todos.map` so I'll just include that part of the JSX here. Don't forget to `import` the new actions `updateTodo` and `deleteTodo`.
```jsx
<li key={t.id}>
  ({t.id}) {t.title} {t.status && `[${t.status}]`} 
  { t.status !== "done" && <button onClick={() => dispatch(updateTodo(t.id, "done"))}>Done</button>}
  <button onClick={() => dispatch(deleteTodo(t.id))}>X</button>
</li>
```

Not the most efficient or pretty, but that's not what we're here for.

Now if you add some todo items to the list, they will have extra buttons which should do something when pressed.

We also added the status display to the items, if present, meaning we just display the string we pass in to `updateTodo`, which in this case is just "done".

So that's some Redux basics, hopefully it makes sense. If it doesn't, don't worry - it will click with enough practice and varied reading.

As I mentioned at the start of part 1, you can find the whole working example app on GitHub, [available here](https://github.com/jfemia/react-redux-hooks-example-app).