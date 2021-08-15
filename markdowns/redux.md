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

## Seperation of Concerns

One of the core principles of good software design is the separation of concerns. This means we shouldn't mix UI code with State logic code. Like this:

```js
const todos = []; // state

// Bad
function addTodo() {
    const input = document.getElementById('input').value;
    const description = input.value;

    todos.push(description); // State manipulation
    input.value = '';  // DOM manipulation

    localStorage.setItem("todos", JSON.stringify(todos)); // Secretely using the local storage. Not cool!
}
```

The `addTodo` function is bad because it does more than one thing. It updates the state, manipulates the DOM, and finally accesses the local storage. It doesn't really do one thing as the name specifies. This makes it hard to test, debug, and manage. To ensure we stick to this principle, we'll be sectioning our JS code into two parts.

```html
<script>
// State section

// ================== separating State and UI code =================
// UI section

</script>
```

State logic should never manipulate the UI (DOM) of an application.

Separating UI and State logic makes our code cleaner and more maintanable.

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
// State manipulation section
let state = {
    todos: [],
    books: [],
};
let counter = 0; // Just a useless variable
```

The `state` variable is called a **State Tree**. A state tree is an object that stores all our application data.

There should be only one State Tree in an application -- A single centralized place to contain the global state in our application.

## Store

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

Each item should have these fields:

- id: a unique number
- text: the text the user typed in
- completed: a boolean flag
