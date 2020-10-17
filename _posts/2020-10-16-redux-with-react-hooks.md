---
layout: post
title: "Redux with React hooks"
subtitle: "Getting started with Redux in a React application"
date: 2020-10-16 14:10:43 +0000
background: '/img/posts/2020-10-16/01.jpg'
---
What is Redux and why do you care?

> The short answer: It lets you define a regular javascript object that you use to store some global state so you can access it from any part of your app. That's effectively it, the rest is bells and whistles.

Before we start, if you learn better by seeing the whole thing working, the sample app code we build in both parts of this tutorial is [available here](https://github.com/jfemia/react-redux-hooks-example-app).

***

If you're reading this you probably heard about this Redux thing. Maybe you also heard about *reducers* and thought *"huh..."* - I certainly did. It took me a few passes over bits of Redux code before it fully clicked. Maybe I can save you some time.

But, rather than throw out-of-context explainations about Redux terminology at you, I'll introduce relevant concepts as we encounter them in a simple implementation. This is my fairly opinionated flavour of a Redux-based app. There are many pages of documentation and tutorials out there, all of them present their own conventions for structuring Redux code, and not many of them that I've encountered seem to make use of React Hooks.

So after some trial and error on my part I've found what works for me - clear and concise, hopefully.

# Get on with it

For this tutorial let's create a boilerplate React app called `react-redux-hooks` so we have something to work with:
```
npx create-react-app react-redux-hooks
```

We'll be building one of the most overused apps in tutorials, a "todo" app.

Firstly we need `redux` and `react-redux` packages
```
yarn add redux react-redux
```
(or, if you use npm)
```
npm i --save redux react-redux
```

# Reducers
While writing this tutorial I originally started by explaining the **Store** first, but I found that I couldn't give a satisfactory summary without the background information about what is reducer actually is.

Just like many things on the internet, there are many explainations of what a Redux reducer is, why it's called a reducer, and so on. If you're so inclined, i'll leave [this StackOverflow question](https://stackoverflow.com/questions/40599496/why-is-a-redux-reducer-called-a-reducer) here for you to read through at your leisure. For now I'll give you a short answer.

## What is a Reducer, then?
A reducer is simply a function that recieves two arguments: your state object, and some *action* to identify what it should do with the state. It returns a new state object to take the place of the one that was passed in.

Remember when I mentioned the regular javascript object that you use to store some global state? Well, that is the "state" object that a reducer function receives. We do this because our global state object is considered immutable - that is, we shouldn't change fields in it directly. A reducer is responsible for making state changes in a structured way. If we were just going to change fields directly on a global object, we wouldn't use Redux.

## Todo Reducer
Just to make clear what I mean, here is our empty reducer function for handling our todo list.

Let's go into the `src` folder of the React app we just made, and create `store/todo.js`. These are the lines we need:
```js
const initialState = [];

export default function(state = initialState, action) {
  return state;
}
```
Let's break this down.

The default export from this file is the reducer function. We can see it takes two arguments, and it returns some state. Right now it does not return a *new* state object, because it is not doing anything with the `action` argument and so can just pass the original state back, unmodified.

Notice that it also has a default value for the `state`, which is just an empty array - we don't have any items in our todo list yet. We do this to allow Redux to initialize the **Store**.


# Store
The central Redux concept, as it were. As we saw above, our reducer will be handling changes to our state, but we still need to create the "global state object". We do this by giving our reducer to Redux and instructing it to create this object for us. Don't worry, it should make sense in a minute.

Let's add another file - `src/store/index.js`:
```js
import { combineReducers, createStore } from 'redux';
import todo from './todo';

const reducers = { todo };

export default createStore(combineReducers(reducers));
```

As you can see this bit of plumbing does a few things.

The main bit of business is the `createStore` function call, which takes a single reducer and optionally some initial state to populate. This is the call that effectively returns us our global state object.

We create a `reducers` object with our todo reducer and use `combineReducers`, which is how you add multiple reducers. This is a bit of my personal preference, but my reasons are:
- Reducers can logically group distinct sections of the global state - we have todos now, maybe we also want to manage authentication or something, later.
- I don't think I've ever built a real app using Redux that didn't end up defining more than one reducer.
- Even if there's just one now, we don't have to change this code later, other than adding to the `reducers` object.

Right now, our call to `createStore` as written results in a javascript object that looks like this:
```js
{
  todo: []
}
```
When we called `createStore` with our reducer, it called our reducer with `undefined` as the state, and our default value - the empty array - was returned and put into the state.

# React plumbing
Before we go further with Redux, there's a couple of other bits of setup we need to do to connect React to our Redux state. Doing this now should help avoid distractions when we get to the next section.

In order to link the Redux store with our React app, the `react-redux` package gives us a `Provider` component, that we need to use to wrap the entire application so our store is available to our components.

We do this in `src/index.js`, which already exists, by adding some imports and the `Provider` component:
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux'; // add this import
import store from './store';            // and this one
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

// Wrap the <App /> line with a Provider using our store
ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);

// ... snip
```
> Note: you will see `//... snip` in several code blocks, this is just where I omitted some existing code from the example - it should remain untouched in your files.

This is enough for any of our components rendered by our app to be able to get data out of the state. Let's modify `src/App.js` to show how:
```jsx
import React from 'react';
import { useSelector } from 'react-redux'; //add this import
import './App.css';

function App() {
  // Define a variable to access our todo list
  const todos = useSelector(state => state.todo);

  // ...snip
}
```

Here we are using another function from `react-redux` - `useSelector`. This is a hook to connect to a specific piece of Redux state. Connecting in this way means that React will re-render if the specified part of the state changes, similar to `props`, or `useState`. 

I have seen a lot of tutorials and guides making use of `mapStateToProps` instead - this is also valid and is the more established way of doing it, but I find `useSelector` a lot more expressive.

Our `todos` variable now contains an array from the state - we can quickly update our `App.js` return value to include it (and to clear out some of the default React text). We aren't aiming to be UI wizards here, any display will work...
```jsx
return (
  <div className="App">
    <header className="App-header">
      <ul>
        {todos.map((t,i) => <li key={i}>{t.title}</li>)}
      </ul>
    </header>
  </div>
);
```

If we refresh the page, we can't see anything but the React-blue background. This is to be expected, we haven't added any todos yet. 

Also, what is `t.title`? We didn't actually define any structure for our todo items yet, but for this example app we'll just be using a title and a status.

So how do we add todos? We need to introduce the next Redux concept, **Actions**, and then define one for adding todos.

# Actions

An action in Redux is conventionally a Javascript object with `type` and `payload` fields.

The `type` identifies what the action does, usually a string, while `payload` is any other arbitrary data that the action needs, usually data that is going to be put into the Store.

Let's revisit our `store/todo.js` file, and define an action for adding items to our todo list.
```js
// Insert above the existing initialState line...
const ADD_TODO = "ADD_TODO";

export const addTodo = (title) => ({ type: ADD_TODO, payload: { title }});
```

We're doing two things here, and in Redux-world they're both optional - however I find this to be a good convention:
* `ADD_TODO` puts the action type into a variable we can refer to elsewhere (and avoid the risk of typos during refactoring)
* `addTodo` defines an "action builder" function - that is, a function that returns a new action object with a specified payload. This makes it easier for code that wants to trigger this action because it doesn't have to implement creation of the whole object, and it makes the call more expressive in my opinion.

The part of the code that handles the action and does something with its payload is the reducer function.

Right now our function does nothing but `return state`. Here's how we respond to our `ADD_TODO` action by adding a new object to our state:
```js
export default function(state = initialState, action) {
  if(action.type === ADD_TODO) {
    return [...state, { title: action.payload.title }];
  }
  else {
    return state;
  }
}
```
Remember that the Store is immutable, so we can't just do `state.push` as that would modify the existing object. What we need to do instead is return a copy of the state with our new item added. In our example we are doing that using ES6 spread syntax to unpack `state` into a new array and add a new todo item using the title in the `action` payload.

## Dispatching Actions

Now that we have an action to add a todo, we need a way of triggering it. In Redux we do this by "dispatching" an action. Dispatch is a function of Redux that receives an action and passes it through the reducers.

Let's quickly add a text input and Add button to `src/App.js` so we have a way of submitting a todo item:
```jsx
// modify the React import to include additional hook functions needed
import React, { useRef, useCallback } from 'react';
// ...
function App() {
  const todos = useSelector(state => state.todo);

  // add a ref so we can read the text input contents from a button click callback
  const inputRef = useRef();

  // add a callback for when the button is clicked
  const onAdd = useCallback(() => {
    // ... to be filled in
  }, []);

  // add the input and button elements to the rendered output...
  return (
    <div className="App">
      <header className="App-header">
        <input type="text" ref={inputRef} />
        <button onClick={onAdd}>Add</button>
        <ul>
          {todos.map((t,i) => <li key={i}>{t.title}</li>)}
        </ul>
      </header>
    </div>
  );
}
```

As with our list appearance, we're not aiming for a beautiful UI here, we just need enough to be able to add todo items.

As written, this button doesn't actually do anything yet. What we need to do is `dispatch` our `addTodo` action.

Most documentation and tutorials I have used in the past usually talk about `mapDispatchToProps`. Similar to `mapStateToProps`, this is a well-established way of using `react-redux`. However, as with `useSelector`, I prefer the hooks apprach - `useDispatch`.

`useDispatch` returns a dispatch function that can dispatch Redux actions. All we need to do is the following in `src/App.js`:
```jsx
// add useDispatch to our react-redux import line
import { useSelector, useDispatch } from 'react-redux';
// import our action-builder function
import { addTodo } from './store/todo';
// ...
function App() {
  const todos = useSelector(state => state.todo);

  // add a call to get a dispatch function
  const dispatch = useDispatch();

  const inputRef = useRef();
  
  const onAdd = useCallback(() => {
    // dispatch our addTodo action, using the <input> text
    dispatch(addTodo(inputRef.current.value));
  }, [inputRef, dispatch]);
```

Now our button callback can dispatch the new todo. Try it yourself! Enter some text and press "Add", and the item you added will appear in the list below.

# Now what?

So that's the basics, but what else can we do with this?

In part 2 we'll define a few additional actions for controlling the state of our items and also deleting them.