---
title: Simply Redux
published: false
description: Understanding Redux by building a Todo simple app
tags: Redux, Design Patterns
//cover_image: https://direct_url_to_image.jpg
---

In this tutorial, we're going to learn Redux by building a simple redux clone to manage the state of our application.

The objective is to understand core Redux. Hence we'll be using only HTML and vanilla JavaScript (No React).

Like all well-designed libraries and frameworks, Redux too makes use of design patterns (Observer Pattern) and follows the software design principles.

For a gentle introduction to the Observer Pattern, check out my post, [To-Do List with Observer Pattern](https://dev.to/devusman/to-do-list-with-observer-pattern-1cl7), to gain a good mental model of the pattern.

As per tradition, we'll be building a simple list application.

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

Conceptually, there are two kinds of data in our application: Todos item objects and Books item objects.

A naive way of storing our application data would be declaring them as separate state variables.

```js
let todos = [];
let books = [];
```

The problem with storing application data this way is that a team member could come along and do something like this:

```js
let todos = [];
let books = [];
let counter = 0;
```

This makes it difficult to understand. Is the `counter` variable part of the application state or not?

A cleaner way of preventing this would be grouping our application state as a single unit. This makes it easier to keep track of our state.

```js
let state = {
  todos: [],
  books: [],
};
let counter = 0; // Just a normal variable
```

The `state` variable is called a **State Tree**. A state tree is an object that stores all our application state.

There should be only one State Tree in an application ― A single centralized place to contain the global state of our application.

### Updating the State (Naive Approach)

Let's start adding Todo items to our State by connecting it to our UI.

```js
// UI section
window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
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

A naive approach of adding items would be directly updating the state tree in the UI code.

```js
// Bad
todoForm.addEventListener("submit", (event) => {
  event.preventDefault();
  const todoInput = todoForm.elements.todo;
  const text = todoInput.value;
  const todo = {
    id: Date.now(),
    text,
    completed: false,
  };
  state.todos.push(todo);
});
```

This approach is bad because our UI code knows too much about our state tree. Imagine we later decided to change our state tree to this:

```js
let state = {
  todos: {}, // No longer an array
  books: [],
};
```

This will break our UI because it depends on `state.todos` being an array. We can say our UI is tightly coupled to our state, and tight coupling leads to bad code.

One of the software design principles is striving for loosely coupled design between objects that interact. Two things are loosely coupled if they have very little knowledge of each other.

## Store

We can achieve loose coupling by hiding our state tree and only accessing it via public methods. A good analogy would be a TV. The electrical circuit is hidden inside the TV, and we only control it via the remote control (public method). This is called Data Encapsulation.

This is achieved by closing the state tree in a function and returning an object with public methods to access the state. The returned object is called a **store**.

```js
function createStore() {
  // hidden inside this function
  let state = {
    todos: [],
    books: [],
  };

  return {
    // store object
  };
}

const store = createStore();

// UI section
window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;
    const todo = {
      id: Date.now(),
      text,
      completed: false,
    };
  });
});
```

Doing this will prevent our UI code from directly accessing our state tree, and all interactions with the state will be done via the store. Keeping the state and UI loosely coupled.

### Updating the State (OOP Approach)

Now, we need away to update our state. Based on our application requirements, we should be able to do the following:

- Add a Todo item
- Delete a Todo item
- Toggle completed Todo

By following OOP practice, we can create methods to handle each action.

```js
function createStore() {
  let state = {
    todos: [],
    books: [],
  };

  const addTodo = (todo) => {
    state.todos.push(todo);
  };

  const deleteTodo = (id) => {
    state.todos = state.todos.filter((todo) => todo.id !== id);
  };

  const toggleTodo = (id) => {
    const todo = state.todos.find((todo) => todo.id === id);
    todo.completed = !todo.completed;
  };

  return {
    addTodo,
    deleteTodo,
    toggleTodo,
  };
}

const store = createStore();

// UI section
window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;
    const todo = {
      id: Date.now(),
      text,
      completed: false,
    };
    store.addTodo(todo);
  });
});
```

From the looks of it, It seems we'll be needing new methods for the book state later. In fact, we can expect more methods in the future to support new features.

Invoking many store methods in our UI code increases the dependency between the UI and store. Remember, we should always strive for loose coupling.

## Actions

Instead of calling different methods for different actions, we can create a single public method that takes an object as an argument.

This object will contain the type of action we would like to perform and any required data.

The method will then delegate our request to the right action handlers.

```js
function createStore() {
  let state = {
    todos: [],
    books: [],
  };

  const addTodo = (todo) => {
    state.todos.push(todo);
  };

  const deleteTodo = (id) => {
    state.todos = state.todos.filter((todo) => todo.id !== id);
  };

  const toggleTodo = (id) => {
    const todo = state.todos.find((todo) => todo.id === id);
    todo.completed = !todo.completed;
  };

  const dispatch = (action) => {
    // call the right function to handle action
    if (action.type === "ADD_TODO") {
      addTodo(action.todo);
    } else if (action.type === "DELETE_TODO") {
      deleteTodo(action.id);
    } else if (action.type === "TOGGLE_TODO") {
      toggleTodo(action.id);
    }
  };

  return {
    dispatch,
  };
}

const store = createStore();

// UI section
window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;
    const todo = {
      id: Date.now(),
      text,
      completed: false,
    };

    const action = {
      type: "ADD_TODO",
      todo,
    };
    store.dispatch(action);
  });
});
```

An action encapsulates a request to do something (like add or delete todo) on the state tree. Our UI code doesn't have any idea what method is used behind the scenes, it just knows how to talk to the store using actions.

The next problem is, the more methods we add to our store function, the more it will grow. Making it harder to manage.

We can solve this by following another design principle ― Separation of concerns. The idea is to move the job of delegating action handlers to a different function outside the store function.

## Updating the state (Reducer approach)

Let's start by abstracting the Todo action handlers from the store function into a new function.

```js
function todoReducer(state, action) {
  const addTodo = (todo) => {
    state.todos.push(todo);
  };

  const deleteTodo = (id) => {
    state.todos = state.todos.filter((todo) => todo.id !== id);
  };

  const toggleTodo = (id) => {
    const todo = state.todos.find((todo) => todo.id === id);
    todo.completed = !todo.completed;
  };

  if (action.type === "ADD_TODO") {
    addTodo(action.todo);
  } else if (action.type === "DELETE_TODO") {
    deleteTodo(action.id);
  } else if (action.type === "TOGGLE_TODO") {
    toggleTodo(action.id);
  }
}

function createStore() {
  let state = {
    todos: [],
    books: [],
  };

  const dispatch = (action) => {
    // We now delegate action to the function
    todoReducer(state, action);
  };

  return {
    dispatch,
  };
}
```

We call this new function a **Reducer** because it takes our state and action, and reduces it to a new state, similar to how array `reduce` method works.

Also, since our arrow functions are short, we can move their bodies to their respective `if` blocks.

```js
function todoReducer(state, action) {
  if (action.type === "ADD_TODO") {
    state.todos.push(action.todo);
  } else if (action.type === "DELETE_TODO") {
    state.todos = state.todos.filter((todo) => todo.id !== action.id);
  } else if (action.type === "TOGGLE_TODO") {
    const todo = state.todos.find((todo) => todo.id === action.id);
    todo.completed = !todo.completed;
  }
}
```

Now, we've made our `createStore` function more maintainable by separating concerns, but we've also introduced an old problem again.

Our reducer knows too much about our state ― Tight coupling. This makes it easy to introduce bugs like this:

```js
// Fire spitting bug
if (action.type === "DELETE_TODO") {
  // overwriting the books state instead of todos
  state.books = state.todos.filter((todo) => todo.id !== id);
}
```

We can prevent this by passing only the todos state as an argument to the reducer function.

```js
function todoReducer(todos, action) {
  if (action.type === "ADD_TODO") {
    todos.push(action.todo);
  } else if (action.type === "DELETE_TODO") {
    todos.filter((todo) => todo.id !== action.id);
  } else if (action.type === "TOGGLE_TODO") {
    const todo = todos.find((todo) => todo.id === action.id);
    todo.completed = !todo.completed;
  }
}

// ...Inside the createStore function
const dispatch = (action) => {
  // We pass only the todos state
  todoReducer(state.todos, action);
};
```

We've made our state tree and reducer loosely coupled. But we introduced another problem in the process. Let's take a look our `DELETE_TODO` action.

```js
todos.filter((todo) => todo.id !== action.id);
```

The `ADD_TODO` and `TOGGLE_TODO` actions both modify the existing `todos` state while the `DELETE_TODO` doesn't.

Unlike the array `push` method, the `filter` method doesn't modify an array. Instead, it returns a new array of items that meet a certain criteria.

A naive way of fixing this would be to return the modified or new state whenever an action is performed, and assign it to `state.todos`.

```js
// returning the modified and new state
function todoReducer(todos, action) {
  if (action.type === "ADD_TODO") {
    todos.push(action.todo);

    return todos;
  } else if (action.type === "DELETE_TODO") {
    return todos.filter((todo) => todo.id !== action.id);
  } else if (action.type === "TOGGLE_TODO") {
    const todo = todos.find((todo) => todo.id === action.id);
    todo.completed = !todo.completed;

    return todos;
  }
}

// ...Inside the createStore function
const dispatch = (action) => {
  // We set todos state to the updated state
  state.todos = todoReducer(state.todos, action);
};
```

But doing this makes the reducer function difficult to predict.

Let's understand what this means with the help of a few examples:

```js
function square(num) {
  return num * num;
}

const sameNumber = 2;
console.log(square(sameNumber)); // => 4
console.log(square(sameNumber)); // => 4
console.log(square(sameNumber)); // => 4
```

Calling the `square` function with the same argument will always return the same result. This makes the function predictable because we know the result will never change given the same argument is passed into it.

Let's look at a more complex example.

```js
// This function will return an array of square of numbers
function squares(numbers) {
  let result = [];

  for (let i = 0; i < numbers.length; i++) {
    const num = numbers[i];
    result[i] = num * num;
  }

  return result;
}

const arrOfNumbers = [1, 2, 3];
console.log(squares(arrOfNumbers)); // => [1, 4, 9]
console.log(squares(arrOfNumbers)); // => [1, 4, 9]
console.log(squares(arrOfNumbers)); // => [1, 4, 9]
```

The `squares` function will also return the same result if the same arguments are passed to it.

Now, let's look at a different implementation of the `squares` function.

```js
// This function will return an array of square of numbers
function squares2(numbers) {
  for (let i = 0; i < numbers.length; i++) {
    const num = numbers[i];
    numbers[i] = num * num; // modifying the existing array
  }

  return numbers;
}

const arrOfNumbers = [1, 2, 3];
console.log(squares2(arrOfNumbers)); // => [ 1, 4, 9]
console.log(squares2(arrOfNumbers)); // => [ 1, 16, 81]
console.log(squares2(arrOfNumbers)); // => [ 1, 256, 6561]
```

This version returns a different result every time we call it with the same argument.

Can you quickly predict the result after calling the function with the same argument a thousand times?

In functional programming, `square` and `squares` are called **pure functions**.

### Pure Functions

By definition, pure functions:

- Return the same result if the same arguments are passed
- Depend solely on the arguments passed into them
- Do not produce any side effects, such as mutating external data or making API requests

Our reducer is not pure because it modifies the state each time it is called. We need our application to be as predictable as possible.

Let's go ahead and convert our reducer into a pure function by returning a new state each time an action is performed.

```js
function todoReducer(todos, action) {
  if (action.type === "ADD_TODO") {
    return [...todos, action.todo];
  } else if (action.type === "DELETE_TODO") {
    return todos.filter((todo) => todo.id !== action.id);
  } else if (action.type === "TOGGLE_TODO") {
    const index = todos.findIndex((todo) => todo.id === action.id);
    const todo = todos[index];

    const newState = [...todos];
    newState[index] = {
      ...todo,
      completed: !todo.completed,
    };

    return newState;
  }

  // return exisiting state if nothing changed
  return todos;
}
```

So far, this is what our code looks like:

```js
function todoReducer(todos, action) {
  if (action.type === "ADD_TODO") {
    return [...todos, action.todo];
  } else if (action.type === "DELETE_TODO") {
    return todos.filter((todo) => todo.id !== action.id);
  } else if (action.type === "TOGGLE_TODO") {
    const index = todos.findIndex((todo) => todo.id === action.id);

    if (index !== -1) {
      const todo = todos[index];
      const newState = [...todos];
      newState[index] = {
        ...todo,
        completed: !todo.completed,
      };
      return newState;
    }
  }

  return todos;
}

function createStore() {
  let state = {
    todos: [],
    books: [],
  };

  const dispatch = (action) => {
    state.todos = todoReducer(state.todos, action);
  };

  return {
    dispatch,
  };
}

const store = createStore();

// UI section
window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;
    const todo = {
      id: Date.now(),
      text,
      completed: false,
    };

    const action = {
      type: "ADD_TODO",
      todo,
    };
    store.dispatch(action);
  });
});
```

Currently, the store only allows us to update the state using the dispatch method.

In order to display the items on the screen, we need a way to get the state from the store.

Now, we need to provide a method to get access to the state.

```js
function createStore() {
  let state = {
    todos: [],
    books: [],
  };

  const getState = () => state;

  const dispatch = (action) => {
    state.todos = todoReducer(state.todos, action);
  };

  return {
    dispatch,
    getState,
  };
}
```

Now, we can use our `getState` method to log our state whenever we add a Todo item.

```js
// UI section
window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;
    const todo = {
      id: Date.now(),
      text,
      completed: false,
    };

    const action = {
      type: "ADD_TODO",
      todo,
    };
    store.dispatch(action);
    console.log(store.getState());
  });
});
```

Try adding a to-do item, and you'll see it being logged in the console.

![getState](./imgs/get-state.png)

## Updating UI

Now that we've provided a way to get the state, let's create a function that will add the items to the DOM.

```js
// UI section
function renderTodos(todos) {
  const ul = document.getElementById("todos");

  ul.innerHTML = ""; // Remove previous state items from DOM

  // Add new state items to DOM
  todos.forEach((todo) => {
    const li = document.createElement("li");

    const removeButton = document.createElement("button");
    removeButton.innerText = "Remove";

    li.appendChild(document.createTextNode(todo.text));
    li.appendChild(removeButton);

    ul.appendChild(li);
  });
}

window.addEventListener("load", () => {
  const todoForm = document.getElementById("add-todo");

  todoForm.addEventListener("submit", (event) => {
    event.preventDefault();
    const todoInput = todoForm.elements.todo;
    const text = todoInput.value;
    const todo = {
      id: Date.now(),
      text,
      completed: false,
    };

    const action = {
      type: "ADD_TODO",
      todo,
    };
    store.dispatch(action);
    renderTodos(store.getState());
  });
});
```
