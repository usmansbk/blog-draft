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

    // We add more codes here
  </script>
</html>
```

First, we populate the unordered list with items. Each item has a button that will serve as the drag handle.

```js
window.addEventListener("load", () => {
  const ul = document.querySelector("ul");
  items.forEach((item) => {
    const li = document.createElement("li");
    li.innerText = item.title;

    const dragHandle = document.createElement("button");
    dragHandle.innerText = "@";
    li.appendChild(dragHandle);

    ul.appendChild(li);
  });
});
```
