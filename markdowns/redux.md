---
title: Simply Redux
published: false
description: Understanding Redux by building a Todo simple app
tags: Redux, Design Patterns
//cover_image: https://direct_url_to_image.jpg
---

When you see the word Redux, the next thing that follows is probably React. But you don't need to know React in order to learn Redux. You can use Redux with any UI including basic HTML. Most people find Redux difficult because they don't understand what it's trying to solve, and how it was designed.

Like all well designed libraries and frameworks, Redux too is a combination of design patterns.

- Observer Pattern
- Command Pattern
- State Pattern
- Mediator Pattern

Design Patterns are shared vocabulary. By understanding these design patterns we will finally understand the thought process of Redux creator. Don't worry if you don't know these patterns, as we'll learn them in this tutorial.

For a gentle intro to the Observer Pattern, check out my previous post, [To-Do List with Observer Pattern](https://dev.to/devusman/to-do-list-with-observer-pattern-1cl7). It'll help give you a good mental model of the pattern.

In this Redux tutorial, you will learn how redux works by building a simple to-do app step by step. Doing this will help you in demystifying the library.

## Structure

We will try to keep the project as simple. Our app will contain two kinds of list. The first is a normal Todo list that allows us to Add, Remove, and Toggle todos. The second is a goals list that allows us to Add and Remove our weekly goals.

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Simply Redux</title>
    </head>
    <body>
        <section>
            <h2>To-Do</h2>
            <ul id="todos"></ul>
            <form id="add-todo">
                <input name="todo" required />
                <button type="submit">Add To-Do</button>
            </form>
        </section>

        <section>
            <h2>Goals</h2>
            <ul id="goals"></ul>
            <form id="add-goal">
                <input name="goal" required />
                <button type="submit">Add Goal</button>
            </form>
        </section>
    </body>
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

    // more code that manipulates the ui and state
}
```

Why is this code bad? Because the `addTodo` function does two things - Manipulating the State, and the UI
