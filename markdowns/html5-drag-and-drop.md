---
title: Sorting with Drag & Drop API
published: false
description: Understanding the HTML5 Drag & Drop API
tags:
cover_image: https://user-images.githubusercontent.com/10219539/126219879-543edda3-088b-4314-bf6a-0783b86d44cb.png
---

The drag-and-drop API allows us to drag items and drop them anywhere in the browser and other applications. In this article, we'll learn how to use it to sort a list of items.

Let's start simple. We'll structure the UI according to the header image. We only want the bare minimum required, so we won't be implementing the "add items" and "checkbox" features.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Sorting with Drag & Drop</title>
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/skeleton/2.0.4/skeleton.min.css"
    />
  </head>
  <body>
    <ul></ul>
  </body>
  <script>
    const items = [
      {
        index: 0,
        title: "1 - One",
      },
      {
        index: 1,
        title: "2 - Two",
      },
      {
        index: 2,
        title: "3 - Three",
      },
    ];

    // We add more code here
  </script>
</html>
```

First, we populate the unordered list with items. Each item has a button that will serve as the drag handle.

```js
function displayItems() {
  const ul = document.querySelector("ul");
  ul.innerHTML = "";
  items.forEach((item) => {
    const li = document.createElement("li");
    li.innerText = item.title;

    const dragHandle = document.createElement("button");
    dragHandle.innerText = "@";
    li.appendChild(dragHandle);

    ul.appendChild(li);
  });
}

window.addEventListener("load", () => {
  displayItems();
});
```

To start dragging an item, we click and hold on a drag handle. The DOM event equivalent for that is the `mousedown` event.

```js
function displayItems() {
  const ul = document.querySelector("ul");
  items.forEach((item) => {
    const li = document.createElement("li");
    li.innerText = item.title;

    const dragHandle = document.createElement("button");
    dragHandle.innerText = "@";
    li.appendChild(dragHandle);

    // Equal to Click and Hold
    dragHandle.addEventListener("mousedown", () => {
      console.log("holding");
    });

    ul.appendChild(li);
  });
}
```

If you try dragging an item, you'll notice there is no indication of it being dragged around. By default, all links, text nodes, and image elements are draggable. For others, we tell the browser that an element can be dragged around by setting a `draggable` attribute.

```js
dragHandle.addEventListener("mousedown", () => {
  li.setAttribute("draggable", true);
});
```

Now, try dragging an item around, and you'll see it highlighted. Our browser knows we're dragging a `li` element, but it doesn't know what item of the array. To tell the browser what item we're dragging, we can use the `dataTransfer` object. The `dataTransfer` object allows us to communicate with the browser when dragging and dropping.

```js
// After the mousedown listener
li.addEventListener("dragstart", (event) => {
  event.dataTransfer.setData("index", item.index);
});
```

Why do we even need to tell the browser? The answer is simply because the item we're dropping on needs to know what is being dropped on it, and the only way for it to know is through the browser's `dataTransfer` object.

So, how does an item know something is being dropped on it? Every draggable element can listen for a `drop` event. The browser fires a `drop` event whenever we drop an item. For instance, when we drag item-1 and drop it on item-3, item-1 will listen for `dragstart` event, while item-2 will listen for a `drop` event.

```js
// After the dragstart listener
li.addEventListener("drop", (event) => {
  const droppedIndex = event.dataTransfer.getData("index");
  console.log("current item index", item.index);
  console.log("dropped item index", droppedIndex);
});
```

Now, try dragging and dropping an item. Check your console. If you're lucky, you'll see an output. But if you're unlucky like me, then allow me to explain.

When you grab an item and drag it into a different location without dropping it, it means you are dragging over. The browser locks every element we drag over to. Preventing us from dropping anything.

Any element that wants to allow the dropping of dragged-over items needs to prevent the browser's default behavior.

```js
// After the drop listener
li.addEventListener("dragover", (event) => {
  event.preventDefault();
});
```
