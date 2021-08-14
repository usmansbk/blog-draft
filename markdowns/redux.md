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
Meta command pattern allows you to create macros of commands so that you can execute multiple commands at once.
A null object is useful when you don't have a meaningful object to return, and yet you want to remove the responsibility for handling null from the client.
Smart command objects implement the logic needed to carry out a request.
When you use the Command Pattern, you end up with a log of small classes.

Actions are request objects to a reciever/command -->

When you see the word Redux, the next thing that follows is probably React. But you don't need to know React in order to learn Redux. You can use Redux with any UI including basic HTML. Most people find Redux difficult because they don't understand what it's trying to solve, and how it was designed.

Like all well designed libraries and frameworks, Redux too is a combination of design patterns.

- Observer Pattern
- Command Pattern

Design Patterns are shared vocabulary. Understanding these design patterns will give us a good mental model of Redux. Don't worry if you don't know these patterns yet, we'll be learning each of them in this tutorial.

For a gentle intro to the Observer Pattern, check out my previous post, [To-Do List with Observer Pattern](https://dev.to/devusman/to-do-list-with-observer-pattern-1cl7). It'll help give you a good mental model of the pattern.

In this Redux tutorial, you will learn how redux works by building a simple to-do app step by step. Doing this will help you in demystifying the library.

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

One of the core principles of good software design is seperation of concern. What this means in the frond-end world is that we shouldn't mix our UI code with our App logic code. An example of a bad function is:

```js
const todos = []; // state

// Bad
function addTodo() {
    const input = document.getElementById('input').value; // UI
    const description = input.value;
    todos.push(description); // State manipulation
    input.value = '';  // UI manipulation

    localStorage.setItem("todos", JSON.stringify(todos)); // Secretely using the local storage. Not cool!
}
```

Why is this code bad? Because the `addTodo` does more than one thing. It updates the state, then tries to clear the input form, then finally save the todos in local storage. It doesn't really do one thing as the name specifies.
