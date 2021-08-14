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

When you see the word Redux, the next thing that follows is probably React. But you don't need to know React in order to learn Redux. You can use Redux with any UI including basic HTML. Most people find Redux difficult because they don't understand what it's trying to solve, and how it is designed. Instead they try to figure out how to make it work with React.

Like all well designed libraries and frameworks, Redux too is a combination of design patterns.

- Observer Pattern
- Command Pattern

Design Patterns are shared vocabulary. Understanding these design patterns will give us a good mental model of Redux. Don't worry if you don't know these patterns yet, we'll be learning them in this tutorial.

For a gentle intro to the Observer Pattern, check out my previous post, [To-Do List with Observer Pattern](https://dev.to/devusman/to-do-list-with-observer-pattern-1cl7). It'll help give you a good mental model of the pattern.

In this Redux tutorial, you will learn how redux works by building a simple redux clone step by step. Doing this will help you in demystifying the library.

## Structure

Our HTML will contain two kinds of `<ul>` list. The first is a normal Todo list that allows us to Add, Remove, and Toggle todos. The second is a goals list that allows us to Add and Remove our weekly goals.

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
            <h2>Goals</h2>
            <form id="add-goal">
                <input name="goal" required />
                <button type="submit">Add Goal</button>
            </form>
            <ul id="goals"></ul>
        </section>
    </body>
    <script>
        // All our JavaScript goes here
    </script>
</html>
```

## Seperation of Concern

One of the core principles of good software design is the separation of concerns. This means we shouldn't mix UI code with State logic code. Example:

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

The `addTodo` function is bad because it does more than one thing. It updates the state, manipulates the DOM, and finally accesses the local storage. It doesn't really do one thing as the name specifies. This makes it hard to test, debug, and manage. To ensure we stick to this principle, we'll be separating our JS code into two sections.

```html
<script>
// State section

// ================== separating State and UI code =================
// UI section

</script>
```

The State section will not manipulate the DOM or access the UI section, while the UI section will handle DOM manipulation. Since the UI needs to display and update the state, we need to create a way to access it from the UI section without breaking the "Separation of Concern" principle.

## State Tree

Our app will make use of two kinds of data. The `todos` and `goals`. A naive way of storing our application data would be declaring each as a variable.

```js
// State section
const todos = [];
const goals = [];
```

The problem with storing application data this way is that a team member could come along and do something like this:

```js
const todos = [];
const goals = [];
let counter = 0;
```

This makes it difficult to understand. Is the `counter` variable part of the application data or not?

A cleaner way of preventing this would be grouping our app data as a single unit. This makes it easier to differentiate application data from other variables.

```js
// State manipulation section
const state = {
    todos: [],
    goals: [],
};
```

The `state` variable is called a **State Tree**. A state tree is an object that stores all our application data.
