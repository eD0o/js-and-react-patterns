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

Statically imported modules are all included in the final bundle of our app, even components that don't need to be rendered right away. `Since all statically imported modules are included in the initial bundle, large files may slow down the first load`. Deferring imports until they're needed can improve performance by reducing the bundle size.

In many cases, `we can defer the import of modules until they're actually needed, which results in smaller bundles`.

Let's say for example that we have a Search input component. When a user clicks on the search input, we show a SearchPopup component that shows some popular locations.

```jsx
import React, { useState } from "react";
import SearchPopup from "./SearchPopup";

const SearchInput = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <input
        type="text"
        placeholder="Search..."
        onFocus={() => setIsOpen(true)}
      />
      {isOpen && <SearchPopup />}
    </div>
  );
};

export default SearchInput;
```

The SearchPopup component isn't instantly visible on the screen - or maybe won't even be visible at all if the user never clicks on the SearchInput. `We can dynamically import the SearchPopup component, which removes it from the initial bundle and instead loads it separately when needed`.

> User experience: If you're lazy-loading a component that's needed for the initial render, it may unnecessarily result in longer loading times. `Try to only lazy load components that aren't visible on the initial render`.

### 3.2.1 - Suspense

Thus, we can dynamically load a component by using React.Suspense with React.lazy:

```jsx
import React, { Suspense, lazy, useState } from "react";

const SearchPopup = lazy(() => import("./SearchPopup"));

const SearchInput = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <input
        type="text"
        placeholder="Search..."
        onFocus={() => setIsOpen(true)}
      />
      <Suspense fallback={<p>Loading search results...</p>}>
        {isOpen && <SearchPopup />}
      </Suspense>
    </div>
  );
};

export default SearchInput;
```

Here, SearchPopup `is loaded only when the user focuses on the search input`, and while it's loading, the fallback message "Loading search results..." is displayed.

Another example:

```jsx
import React, { Suspense, lazy } from "react";
import "./styles.css";

import { Card } from "./components/Card";
import Card1 from "./components/Card1";
import Card2 from "./components/Card2";
const Card3 = lazy(() =>
  import(/*webpackChunkName: "card3" */ "./components/Card3")
);
const Card4 = lazy(() =>
  import(/*webpackChunkName: "card4" */ "./components/Card4")
);

const App = () => {
  return (
    <div className="App">
      <Card1 />
      <Card2 />
      <DynamicCard component={Card3} name="Card3" />
      <DynamicCard component={Card4} name="Card4" />
    </div>
  );
};

function DynamicCard(props) {
  const [open, toggle] = React.useReducer((s) => !s, false);
  const Component = props.component;

  return (
    <Suspense fallback={<p id="loading">Loading...</p>}>
      {open ? (
        <Component />
      ) : (
        <Card rendered={false} onClick={toggle}>
          <p>
            Click here to dynamically import <code>{props.name}</code> component
          </p>
        </Card>
      )}
    </Suspense>
  );
}

export default App;
```

The `Suspense component provides a fallback, which is displayed while the dynamically imported component is being loaded`.

In this example, Card1 and Card2 are statically imported and included in the initial bundle. Card3 and Card4 however are dynamically loaded on user interaction.

### 3.2.2 - Import on Visibility

We can `dynamically import components based on their visibility within the viewport`. This is especially useful for `improving performance by lazy-loading` elements only when they are needed.

For example, in a listing page on smaller viewports, not all listings should be visible to the user immediately. Instead, `we can lazy-load the listings and only load them when they become visible as the user scrolls` down.

A common approach for dynamically importing components based on viewport visibility is using the Intersection Observer API. `React provides a hook called react-intersection-observer, which makes it easy to detect whether a component is visible in the viewport`.

Example:

```jsx
import { Suspense, lazy } from "react";
import { useInView } from "react-intersection-observer";

// Lazy load the Listing component
const Listing = lazy(() => import("./components/Listing"));

function ListingCard(props) {
  const { ref, inView } = useInView();

  return (
    <div ref={ref}>
      {/* Suspense with a loading indicator */}
      <Suspense fallback={<div>Loading...</div>}>
        {inView && <Listing />}
      </Suspense>
    </div>
  );
}
```

In this example:

- Lazy Loading: `Listing is lazily loaded using React.lazy(). This means the component is only fetched when it is needed`, improving initial page load time and resource usage.

- Intersection Observer API: The useInView hook from the `react-intersection-observer library detects whether the ListingCard component is visible in the viewport`. The component is only loaded when it enters the viewport, reducing unnecessary network requests.

- Suspense with Fallback UI: The `Suspense component handles the loading state by rendering a fallback UI (e.g., "Loading...") until the Listing component is fully loaded`. This ensures a smooth user experience.

## 3.2.3 - Route Based Splitting

If your application has `multiple pages, we can use dynamic imports to only load the resources that are needed for the current route`. Instead of the code for all the possible pages in the initial bundle, we can bundle-split based on routes. This approach allows us to defer loading the bundle until the user actually navigates to that page.

If you're `using react-router for navigation, you can wrap the Switch component in a React.Suspense, and import the routes using React.lazy`. This automatically enables route-based code splitting.

```jsx
import React, { lazy, Suspense } from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";

const App = lazy(() => import("./App"));
const About = lazy(() => import("./About"));
const Contact = lazy(() => import("./Contact"));

const root = ReactDOM.createRoot(document.getElementById("root"));

root.render(
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<App />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Suspense>
  </Router>
);
```
