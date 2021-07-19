---
title: To-do list with Observer Pattern
published: false
description: Using the Observer Pattern to manage the state of a simple todo list app.
tags: 
//cover_image: https://direct_url_to_image.jpg
--- 

In this post, we'll learn about the Observer Pattern by creating a simple To-do application.

In a nutshell, the Observer Pattern is similar to Twitter's _Followers_ feature. When you post a tweet, all your followers get notified, and they decide whether to read the tweet or not. We can say our _Followers_ are observing our tweets.

The Observer Pattern has only two components. The _Subject_ and the _Observers_. The _Observers_ only want to know when we update the _Subject_. They don't care when it happens.

Going back to our Twitter analogy, Our _Tweet_ is the _Subject_, while our _Followers_ are the _Observers_. We notify our followers whenever we post a new tweet.

So, how does it relate to our Todo list application? We'll uncover the answer while we build the app, but first we need to know the features of our app.

- We want to be able to add a unique todo to our list
- We want to be able to remove a todo from our list
- We want to persist our list on page reload

Let's create the HTML of our Todo app.

```html
<!DOCTYPE html>
<html>
<head>
 <meta charset="utf-8">
 <title>Observer Pattern</title>
</head>
<body>
 <ul></ul>
 <form>
  <input required type="text" />
  <button type="submit">Add</button>
 </form>
</body>
<script>
// We'll add all our code here
</script>
</html>
```

In this HTML, we have an un-ordered list element which will hold our todo items, and a form element to add a todo item to our list.

The _Subject_ will be our todo items. Let's create an array list to store our todos.

```html
<script>
  let todos = []; // Subject
</script>
```

Next we create a list of Observers. (Basically a list of our Todo followers).

```html
<script>
  let todos = []; // Subject
  let observers = [];
</script>
```

Then we implement add todo functionality. Each todo needs to be unique, so we'll be using the current timestamp to uniquely identify them.

```js
const form = document.querySelector("form");
form.addEventListener('submit', (event) => {
  event.preventDefault();
  const input = form.elements[0];
  const item = {
    id: Date.now(),
    description: input.value,
  };
  addTodo(item);
  input.value = ''; // Clear text input
});

function addTodo(item) {
  todos.push(item);
}
```

## Introducing our first observer

When you try running our simple app, you'll notice nothing is being displayed on screen. That's because we haven't hooked up our `todos` array to our HTML un-ordered list.

Our HTML `ul` element is interested in our `todos` array. It wants to observe our array list, so that it can display it on screen. Lets implement a function that will display our list.

```js
function displayTodos() {
   const ul = document.querySelector('ul');
   todos.forEach((todo) => {
     const li = document.createElement('li');
     li.innerText = todo.description;
     ul.appendChild(li);
  });
}
```

Now, to register as an observer is simple. We just have to add the `displayTodos` to our list of `observers`. To do that we create a function to `register` new observers.

```js
function registerObserver(observer) {
  // The observers array is basically an array of functions
  observers.push(observer);
}

registerObserver(displayTodos);
```

Despite registering as an observer, nothing is being displayed on the screen. That's because our `todos` array hasn't notified the observers.
We create a `notifyObservers` function that will loop through our `observers` array and call each `observer` function.

```js
function notifyObservers() {
  observers.forEach((observer) => observer());
}
```

Then, we call the `notifyObservers` function in the `addTodo` function we created earlier.

```js
function addTodo(item) {
  todos.push(item);
  notifyObservers(); // Add this line
}
```

## Congratulations on your first bug 🥳

You've probably noticed that our list doubles every time we add a new item. That's because we keep appending to our `ul` element without clearing the previous items. Here's the code to fix that:

```js
    // Inside the displayTodos function

    function displayTodos() {
      const ul = document.querySelector('ul');
      ul.innerHTML = ''; // Add this line
```

Now that we have the add functionality working, it's time to remove todos.
First, we give every single todo `li` element a remove `button`.

```js
function displayTodos() {
  const ul = document.querySelector('ul');
  ul.innerHTML = '';
  todos.forEach((todo) => {
    const li = document.createElement('li');
    li.innerText = todo.description;

    // Add these lines
    const button = document.createElement('button');
    button.innerText = 'Remove';
    li.appendChild(button);

    ul.appendChild(li);
  });
}
```

Then, we create a `removeTodo` function that will remove a single todo with a given `id` from the `todos` list.

```js
function removeTodo(id) {
  todos = todos.filter((todo) => todo.id !== id);
  notifyObservers();
}
```

Then we attach a `click` event listener to the remove button, that will call the `removeTodo` function.

```js
// Inside the displayTodos function

const button = document.createElement('button');
button.innerText = 'Remove';
// Attach an event listener here
button.addEventListener('click', () => {
   removeTodo(todo.id);
});
li.appendChild(button)
```

## Introducing the second observer

With the add/remove features working, the final step is to prevent our todos from clearing on page reload. To do that, we save our list in local storage any time we modify the list, and load the saved list any time we reload the page.
We want our local storage to be an observer, and save the list whenever it is notified.

```js
function persistData() {
  localStorage.setItem("saved-todos", JSON.stringify(todos));
}

registerObserver(persistData);
```

Finally, we load the saved todos on page load.

```js
function loadTodos() {
  const savedTodos = localStorage.getItem("saved-todos");
  if (savedTodos) {
    todos = JSON.parse(savedTodos);
    notifyObservers();
  }
}

window.addEventListener("load", () => {
  loadTodos();
});
```
