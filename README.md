# 3 - Performance Patterns

## 3.1 - Static Import

We can import React components from other files.

```jsx
import Header from "./Header";
import Content from "./Content";

export default function BlogPost({ post }) {
  return (
    <div>
      <Header title={post.title} />
      <Content body={post.body} />
      <Footer />
    </div>
  );
}
```

The `Header and Content modules are evaluated as soon as they are imported`. Their top-level code runs immediately, `but their exported functions or components are only executed when used`.

When a module is statically imported, a `bundler analyzes the entire dependency tree, collecting all referenced modules and bundling them together into a single output file` (or multiple files, depending on the configuration).

Let's say we have three files in index.js:

```js
import module1 from "./module1";
import module2 from "./module2";
import module3 from "./module3";
```

module1.js exports a function named module1. module2.js imports module1 and exports a function named module2. module3.js imports module2 and exports a function named module3.

This pattern continues up to index.js, which ultimately imports module3 and logs its value.

![](https://i.imgur.com/d6Qivmy.png)

### 3.1.1 - Tree Shaking

It's a process where `unused code is removed from the final bundle`.

Since `static imports are analyzed at build time, bundlers like Webpack or ESBuild can eliminate unused exports`, optimizing performance.

```jsx
// input.js
export function validateInput(input) {
  const isValid = input.length > 10;
  return isValid;
}

export function formatInput(input) {
  const formattedInput = input.toLowerCase();
  return formattedInput;
}
```

```jsx
// index.js
import { validateInput } from "./input";

const input = document.getElementById("input");
const btn = document.getElementById("btn");

btn.addEventListener("click", () => {
  validateInput(input.value);
});
```

After tree shaking, the `final bundle won't include the formatInput function as it's not referenced` in the code.

![](https://i.imgur.com/69cr5L6.png)

### 3.1.2 - Hoisting

ES module imports are hoisted, meaning `they are resolved first, before any code runs`.

Because of this behavior, `all imported bindings are available throughout the module, even if they appear later` in the file.

> Can Static Imports Be Anywhere? No, `static imports must be at the top level of the module`. This is different from function hoisting, where you can define functions later but still call them earlier. Also, You `cannot place static imports inside functions or conditionals`. Finally, keeping imports at the top ensures readability and maintainability.

```jsx
console.log(myFunction); // ✅ No error, import is processed before this line runs

import { myFunction } from "./utils.js";

console.log(myFunction()); // ✅ Runs correctly
```

```jsx
// index.js
import { validateInput } from "./input";

const input = document.getElementById("input");
const btn = document.getElementById("btn");

btn.addEventListener("click", () => {
  validateInput(input.value);
});
```

### 3.1.3 - Cannot Be Conditional

Since static imports are processed at the start of module execution, `they cannot be placed inside blocks or conditionals`.

If `conditional loading is required, use dynamic imports` (import()) instead.

```jsx
if (someCondition) {
  import myModule from "./myModule"; // ❌ SyntaxError
}
```

## 3.2 - Dynamic Import



### 3.2.1 - Import on Visibility
