---
title: To-do list with Observer Pattern
published: false
description: Using the Observer Pattern to manage the state of a simple todo list app.
tags: 
//cover_image: https://direct_url_to_image.jpg
--- 

In this post, we'll be learning about the Observer Pattern by creating a simple To-do application.

In a nutshell, the Observer Pattern is similar to Twitter's _Followers_ feature. When you post a tweet, all your followers get notified, and they decide whether to read your tweet or not. We can say our _Followers_ are observing our tweets.

The Observer Pattern has only two components. The _Subject_ and the _Observers_. The _Observers_ only want to know when we update the _Subject_. They don't care when it happens.

Going back to our Twitter analogy, our Tweet is the Subject, while our Followers are the Observers.

So, how does it relate to our Todo list application? We'll uncover the answer while we build the app, but first, we need to know the features of our app.

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

In this HTML, we have an unordered list element which will hold our todo items, a form element to add a todo item to our list, and finally a `script` element to hold our JavaScript code.

The _Subject_ will be our todo items. So we create an array list to store our todos.

```html
<script>
  let todos = []; // Subject
</script>
```

Next we create a list of Observers. (Functions that will make use of the list).

```html
<script>
  let todos = []; // Subject
  let observers = [];
</script>
```

Then, we implement the add todo functionality. Each todo needs to be uniquely identified, so assign each item with an ID.

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

When you try running the app, you'll notice nothing is being displayed on screen. That's because we haven't hooked up our `todos` array to our HTML unordered list element.

Our HTML `ul` element is interested in our `todos` array. It wants to observe our array list so that it can display it on the screen. So it wants to be an Observer. Let's implement a function that will display our list.

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

Now, we register this function as an _Observer_ by adding it to our list of `observers`. To do that we create a helper function to `register` new observers.

```js
function registerObserver(observer) {
  // The observers array is basically an array of functions
  observers.push(observer);
}

registerObserver(displayTodos);
```

Despite registering as an observer, nothing is being displayed. That's because our `todos` array hasn't notified the observers.
We create a `notifyObservers` function that will loop through our `observers` array and call each `observer` function to know an update has happened.

```js
function notifyObservers() {
  observers.forEach((observer) => observer());
}
```

Then, we call the `notifyObservers` function whenever we change the _Subject_.

```js
function addTodo(item) {
  todos.push(item);
  notifyObservers(); // Add this line
}
```

Now, run the app in your browser and see your todos being added to the list.

## Congratulations on your first bug 🥳

You've probably noticed that our list doubles every time we add a new item. We can fix that by clearing it first.

```js
// Inside the displayTodos function

function displayTodos() {
    const ul = document.querySelector('ul');
    ul.innerHTML = ''; // Add this line
```

Now that we have "add" functionality working, it's time to remove todos. First, we add a remove `button` to every `li` element.

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

Then, we create a `removeTodo` function that will handle removing to-dos by their ID.

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

The final step is to save our list in local storage and load it when we reload the page. We want our local storage to be an observer, and save the list whenever it is notified.

```js
function persistData() {
  localStorage.setItem("saved-todos", JSON.stringify(todos));
}

registerObserver(persistData);
```

Then, we load the saved todos on page load.

```js
function loadTodos(todoList) {
  todos = todoList;
  notifyObservers();
}

window.addEventListener("load", () => {
  const savedTodos = localStorage.getItem("saved-todos");
  if (savedTodos) {
    loadTodos(JSON.parse(savedTodos));
  }
});
```

## The Clean Code

Our code is working. It meets the minimum requirements, but it's not elegant. If you follow closely, you'll notice there are 2 kinds of code. Those that manipulate the unordered list element and those that manipulate the `todos` array list. We are mixing UI logic and State logic, which is an attribute of a messy code.

Let us start by wrapping our state logic in a function and exposing the `register`, `add`, `remove`, and `load` functions as methods. This is called _Abstraction_.
Our `todos` array is no longer visible to the UI logic code. So we create the `getTodos` method for accessing the `todos`. This is called _Encapsulation_. The art of hiding internal state and exposing it via a method.

```js
function createSuject() {
    let todos = [];
    let observers = [];

    function registerObserver(observer) {
      observers.push(observer);
    }

    function notifyObservers() {
      observers.forEach((observer) => observer());
    }

    function addTodo(item) {
        todos.push(item);
        notifyObservers();
    }

    function removeTodo(id) {
      todos = todos.filter((todo) => todo.id !== id);
      notifyObservers();
    }

    function loadTodos(todoList) {
        todos = todoList;
        notifyObservers();
    }

    function getState() {
        return todos;
    }

    return {
        registerObserver,
        addTodo,
        removeTodo,
        loadTodos,
        getState,
    }
}
```

Next we use the `createSubject` to create a todos subject.

```js
const subject = createSubject();

function displayTodos() {
    const ul = document.querySelector("ul");
    ul.innerHTML = "";
    todos.forEach((todo) => {
        const li = document.createElement("li");
        li.innerText = todo.description;

        const button = document.createElement("button");
        button.innerText = "Remove";
        button.addEventListener("click", () => {
            subject.removeTodo(todo.id);
        });
        li.appendChild(button);

        ul.appendChild(li);
    });
}

subject.registerObserver(displayTodos)

subject.registerObserver(() => {
    localStorage.setItem("saved-todos", JSON.stringify(todos));
});

window.addEventListener("load", () => {
    const savedTodos = localStorage.getItem("saved-todos");
    if (savedTodos) {
        subject.loadTodos(JSON.parse(savedTodos));
    }

    const form = document.querySelector("form");
    form.addEventListener("submit", (event) => {
        event.preventDefault();
        const input = form.elements[0];
        const item = {
            id: Date.now(),
            description: input.value,
        };
        subject.addTodo(item);
        input.value = "";
    });
});
```

The `createSubject` function adheres to the Observer Pattern design. We subscribe to the todos by registering as observers. What about if we no longer want to be notified?
It's quite simple. We can return a function in the `registerObserver` method.

```js
function registerObserver(observer) {
    observers.push(observer);

    return function () {
        observers = observers.filter((currentObserver) => !== observer);
    }
}
```

Then, we can save the return value after registering and call it later to unregister.

```js
const unregisterDisplayTodos = subject.registerObserver(displayTodos)

// later when we want to unregister
unregisterDisplayTodos(); // displayTodos will no longer be notified
```

## FIN

Redux is a popular JavaScript library that uses the _Observer Pattern_. In the next post, we'll demystify redux by creating our own small redux library.

Happy Coding!
