---
title: Simply Redux
published: false
description: Understanding Redux by building a Todo simple app
tags: Redux, Design Patterns
//cover_image: https://direct_url_to_image.jpg
---

<!-- Redux is a library for managing global application state.
Redux is typically used with React

Redux uses a "one-way data flow" app structure

Command Pattern allows you to decouple the requester of an action from the object that actually performs the action.

The client is responsible for creating the command object. The command object consist of a set of actions on a reciever.

The command object encapsulates the actions and can be called to invoke the actions on the Receiver.

The command pattern encapsulates a request as an object, thereby letting you parameterize other other objects with different requests.

The command pattern aims to encapsulate method invocation, requests or operations into a single object and gives us the ability to both parameterize and pass method calls around that can be executed at our discretion.

Meta command pattern allows you to create macros of commands so that you can execute multiple commands at once.

A null object is useful when you don't have a meaningful object to return, and yet you want to remove the responsibility for handling null from the client.
Smart command objects implement the logic needed to carry out a request.
When you use the Command Pattern, you end up with a log of small classes.

Actions are request objects to a reciever/command -->

When you see the word Redux, the next thing that follows is probably React. But you don't need to know React in order to learn Redux. You can use Redux with any UI including basic HTML. Most people find Redux difficult because they don't understand what it's trying to solve, and how it is designed.

Like all well designed libraries and frameworks, Redux too is an implementation of design patterns.

- Observer Pattern
- Command Pattern

Design Patterns are shared vocabulary. Understanding these design patterns will give you a good mental model of Redux. Don't worry if you don't know these patterns yet, we'll be learning them in this tutorial.

For a gentle intro to the Observer Pattern, check out my previous post, [To-Do List with Observer Pattern](https://dev.to/devusman/to-do-list-with-observer-pattern-1cl7). It'll help give you a good mental model of the pattern.

In this tutorial, we're going to learn how redux works by building a simple list application.

### Requirements

- The UI should consist of two main sections: Todos and Books
- Each section should consist of 3 parts:
  - An input box to let the user type in the new item
  - A button to add the item
  - A list of all the exisiting items
- Each item should have a unique ID value
- We should be able to delete items
- Clicking a Todo item should mark it as completed

The final app should look like this:

[screenshot](./app_screenshot.png)

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Simply Redux</title>
    </head>
    <body>
        <section>
            <h2>Todos</h2>
            <form id="add-todo">
                <input name="todo" required />
                <button type="submit">Add To-Do</button>
            </form>
            <ul id="todos"></ul>
        </section>

        <section>
            <h2>Books</h2>
            <form id="add-book">
                <input name="book" required />
                <button type="submit">Add Book</button>
            </form>
            <ul id="books"></ul>
        </section>
    </body>
    <script>
        // All our JavaScript goes here
    </script>
</html>
```

## State Tree

Conceptually, there are two kinds of data in our application. Two arrays of todo item objects and books item objects. A naive way of storing our application data would be declaring them as variables.

```js
// State section
let todos = [];
let books = [];
```

The problem with storing application data this way is that a team member could come along and do something like this:

```js
let todos = [];
let books = [];
let counter = 0;
```

This makes it difficult to understand. Is the `counter` variable part of the application data or not?

A cleaner way of preventing this would be grouping our app data as a single unit. This makes it easier to keep track of our application state.

```js
// State section
let state = {
    todos: [],
    books: []
};
let counter = 0; // Just a useless variable
```

The `state` variable is called a **State Tree**. A state tree is an object that stores all our application data.

There should be only one State Tree in an application ― A single centralized place to contain the global state in our application.

### Updating the State (Naive Approach)

Let's start adding Todo items to our State by connecting it to our UI.

```js
// UI section
window.addEventListener('load', () => {
    const todoForm = document.getElementById('add-todo');

    todoForm.addEventListener('submit', (event) => {
        event.preventDefault();
        const todoInput = todoForm.elements.todo;
        const text = todoInput.value;
    });
});
```

Based on the requirements, each todo item should have these fields:

- id: a unique number
- text: the text the user typed in
- completed: a boolean flag

A naive approach of adding items would be directly doing it in the UI code.

```js
// Bad
todoForm.addEventListener('submit', (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;

    const todo = {
        id: Date.now(),
        text,
        completed: false
    };
    state.todos.push(todo);
});
```

This approach is bad because our `submit` handler knows too much about our state tree. Imagine we later decided to change our state tree to this:

```js
// state section 
let state = {
    todos: {}, // Now an object literal
    books: []
};
```

This will break our UI because it depends on `state.todos` being an Array object. We can say our UI is tightly coupled to our state. Tight coupling is a sign of bad code.

One of the software design principles is striving for loosely coupled design between objects that interact. Two things are loosely coupled if they have very little knowledge of each other. Minimizing the interdependency between the two of them.

## Store

We can achieve loose coupling by hiding our state tree and only accessing it via public methods. A good analogy is a TV. The electrical circuit is hidden inside the TV and we only control it via public remote control. This is called **Data Encapsulation**.

This is achieved by closing the state in a function and returning an object with public methods to access the data. The returned object is called a store.

```js
// State section
function createStore() {
    let state = {
        todos: [],
        books: []
    };

    return { // store object

    };
}

const store = createStore();

// UI section
window.addEventListener('load', () => {
    const todoForm = document.getElementById('add-todo');

    todoForm.addEventListener('submit', (event) => {
        event.preventDefault();
        const todoInput = todoForm.elements.todo;
        const text = todoInput.value;
    });
});
```

Doing this will prevent our UI code from directly accessing our state tree. All interactions with the state will be done via the store object methods. Keeping the state and view code loosely coupled.

### Updating the State (OOP Approach)

Now, we need away to update our state. Based on our application requirements, we should be able to do the following:

- Add a Todo item
- Delete an Item
- Toggle completed todo

By following good OOP practice, we can create a method for each action.

```js
// State section
function createStore() {
    let state = {
        todos: [],
        books: []
    };

    const addTodo = (text) => {
        const todo = {
            id: Date.now(),
            text,
            completed: false
        };
        state.todos.push(todo);
    };

    const deleteTodo = (id) => {
        state.todos = state.todos.filter((todo) => todo.id !== id);
    };

    const toggleTodo = (id) => {
        const todo = state.todos.find((todo) => todo.id === id);
        if (todo) {
            todo.completed = !todo.completed
        }
    };

    return {
        addTodo,
        deleteTodo,
        toggleTodo
    };
}

const store = createStore();

// UI section
window.addEventListener('load', () => {
    const todoForm = document.getElementById('add-todo');

    todoForm.addEventListener('submit', (event) => {
        event.preventDefault();
        const todoInput = todoForm.elements.todo;
        const text = todoInput.value;
        store.addTodo(text);
    });
});
```

It seems we will need new methods for the book state later. In fact, we can expect more methods in the future to support new features (pun intended).

Invoking many store methods in our UI code increases the dependency between the view and store. Remember, we should always strive for loose coupling.

## Actions

Instead of calling different methods for different actions. We could invoke a single method, passing it a request object containing the type of action we would like to perform and the required data.

```js
// State section
function createStore() {
    let state = {
        todos: [],
        books: []
    };

    const addTodo = (text) => {
        const todo = {
            id: Date.now(),
            text,
            completed: false
        };
        state.todos.push(todo);
    };

    const deleteTodo = (id) => {
        state.todos = state.todos.filter((todo) => todo.id !== id);
    };

    const toggleTodo = (id) => {
        const todo = state.todos.find((todo) => todo.id === id);
        if (todo) {
            todo.completed = !todo.completed
        }
    };

    const dispatch = (action) => {
        // call the right function to handle action
        if (action.type === 'ADD_TODO') {
            addTodo(action.text);
        } else if (action.type === 'DELETE_TODO') {
            deleteTodo(action.id);
        } else if (action.type === 'TOGGLE_TODO') {
            toggleTodo(action.id);
        }
    };

    return {
        dispatch
    };
}

const store = createStore();

// UI section
window.addEventListener('load', () => {
    const todoForm = document.getElementById('add-todo');

    todoForm.addEventListener('submit', (event) => {
        event.preventDefault();
        const todoInput = todoForm.elements.todo;
        const action = {
            type: 'ADD_TODO',
            text: todoInput.value
        };
        store.dispatch(action);
    });
});
```

An action encapsulates a request to do something (like add or delete todo) on the state tree. Our UI code doesn't have any idea what method is used behind the scenes, it just knows how to talk to the store using action.

This only solves half of our problem. Notice that our `createStore` function will keep growing longer the more methods we add.

We can solve this using another design principle ― Separation of concerns. By abstracting the todos state logic, we keep our `createStore` function simple and short, making it maintainable.

## Updating the state (Reducer approach)

Lets move the todos logic outside the store function.

```js
// State section
function handleTodos(todos, logic) {
    const addTodo = (text) => {
        const todo = {
            id: Date.now(),
            text,
            completed: false
        };
        todos.push(todo);
    };

    const deleteTodo = (id) => {
        todos = todos.filter((todo) => todo.id !== id);
    };

    const toggleTodo = (id) => {
        const todo = todos.find((todo) => todo.id === id);
        if (todo) {
            todo.completed = !todo.completed
        }
    };

    if (action.type === 'ADD_TODO') {
        addTodo(action.text);
    } else if (action.type === 'DELETE_TODO') {
        deleteTodo(action.id);
    } else if (action.type === 'TOGGLE_TODO') {
        toggleTodo(action.id);
    }
}

function createStore() {
    let state = {
        todos: [],
        books: []
    };

    const dispatch = (action) => {
        // We now delegate action to the function
        handleTodos(state.todos, action);
    };

    return {
        dispatch
    };
}
```
