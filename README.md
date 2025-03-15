# 2 - React Patterns

## 2.1 - Container/Presentation Pattern

It's a design pattern in React that separates concerns by dividing components into two categories:

- **Presentational Components:** `Focus on UI, rendering, and styles`. They receive data and callbacks via props but don’t handle state or logic.
- **Container Components:** `Handle business logic, state, and data fetching`. They pass necessary data and functions to presentational components.

---

Classic Approach - Before Hooks

```jsx
// Presentational Component (Button.jsx)
import React from "react";

const Button = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

export default Button;
```

```jsx
// Container Component (ButtonContainer.jsx) - Before Hooks (Class-Based)
import React, { Component } from "react";
import Button from "./Button";

class ButtonContainer extends Component {
  state = { count: 0 };

  handleClick = () => {
    this.setState((prevState) => ({ count: prevState.count + 1 }));
  };

  render() {
    return (
      <div>
        <p>Clicked: {this.state.count} times</p>
        <Button label="Click me" onClick={this.handleClick} />
      </div>
    );
  }
}

export default ButtonContainer;
```

---

Modern Approach - With Hooks (But Less Necessary\*)

```jsx
// Presentational Component (Button.jsx)
import React from "react";

const Button = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

export default Button;
```

```jsx
// Container Component (ButtonContainer.jsx)
import React, { useState } from "react";
import Button from "./Button";

const ButtonContainer = () => {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount((prev) => prev + 1);
  };

  return (
    <div>
      <p>Clicked: {count} times</p>
      <Button label="Click me" onClick={handleClick} />
    </div>
  );
};

export default ButtonContainer;
```

> - With React Hooks, the need for the Container/Presentational pattern has significantly decreased. The pattern was more common before Hooks, when class components required explicit separation of logic (state, lifecycle methods) from UI.

---

### 2.1.1 - Why Hooks Reduce the Need for the Pattern:

- **State & Effects in Functional Components** – `Hooks like useState and useEffect allow handling logic inside the same component without class-based lifecycles.`
- **Encapsulation with Custom Hooks** – Instead of separate container components, logic can be extracted into custom hooks, making components reusable and cleaner.

---

### 2.1.2 - Alternative with Hooks (No Need for Container Component)

Instead of separating components into container/presentational, we can use Hooks directly inside a functional component:

```jsx
import { useState } from "react";

const ClickCounter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Clicked: {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
};

export default ClickCounter;
```

> **Why?** No need for a separate container, since `useState` allows managing state within the same component.

---

### 2.1.3 - Using a Custom Hook Instead of a Container Component

If we need to **reuse logic** across multiple components, a **custom hook** is a better alternative than a separate container component.

```jsx
// useCounter.js (Custom Hook)
import { useState } from "react";

const useCounter = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount((prev) => prev + 1);

  return { count, increment };
};

export default useCounter;
```

```jsx
// ButtonCounter.jsx (Using the Custom Hook)
import React from "react";
import useCounter from "./useCounter";

const ButtonCounter = () => {
  const { count, increment } = useCounter();

  return (
    <div>
      <p>Clicked: {count} times</p>
      <button onClick={increment}>Click me</button>
    </div>
  );
};

export default ButtonCounter;
```

> **Why use a Custom Hook?** It allows **reusing the counter logic** across different components without needing a separate container component.

---

### 2.1.4 - When Should We Still Use Container/Presentational?

Even though Hooks reduce the need for this pattern, there are cases where it's still useful:

- If the `UI component (Button) is used in multiple places with different logic`.
- If we want a `clear separation between UI and logic` (for better reusability in large apps).
- If the `logic is complex` (e.g., data fetching, context management).

## 2.2 - High-Order Components

A Higher-Order Component (HOC) is a `function that takes a component and enhances it by returning a new component with additional features`.

Example: You want to `add a style to multiple components without repeating the same CSS`.

```tsx
import React from "react";

// Higher-Order Component
const withBorder = (WrappedComponent: React.ComponentType) => {
  return (props: any) => (
    <div style={{ border: "2px solid red", padding: "10px" }}>
      <WrappedComponent {...props} />
    </div>
  );
};

// Base Component
const Message = ({ text }: { text: string }) => <p>{text}</p>;

// Enhanced Component
const MessageWithBorder = withBorder(Message);

export default function App() {
  return <MessageWithBorder text="Hello, world!" />;
}

// Message is a simple component that displays text.
// withBorder(Message) wraps it inside a div with a red border.
// Now, any component can be wrapped with withBorder to add the same effect.
```

> When to avoid HOCs ? If the enhancement involves state or effects, Hooks (useState, useEffect) are often a better choice.

# 2.3 Render Props Pattern in React

Allows components to `share logic by passing a function (render prop) as a prop`, which controls what to render.

Example: Mouse Position Tracker  

```tsx
import React, { useState } from "react";

// Component using Render Props
const MouseTracker = ({
  render,
}: {
  render: (position: { x: number; y: number }) => JSX.Element;
}) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (event: React.MouseEvent) => {
    setPosition({ x: event.clientX, y: event.clientY });
  };

  return (
    <div style={{ height: "200px" }} onMouseMove={handleMouseMove}>
      {render(position)}
    </div>
  );
};

// Component using MouseTracker
export default function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <p>
          Mouse position: ({x}, {y})
        </p>
      )}
    />
  );
}
```

> The Render Props pattern `does not require the prop to be named render—it can be anything, as long as it follows the same concept of passing a function as a prop` to control rendering.

Example 2: Temperature Conversion

```tsx
import React, { useState } from "react";

// Component using Render Props
function Input({
  renderKelvin,
  renderFahrenheit,
}: {
  renderKelvin: (data: { value: number }) => JSX.Element;
  renderFahrenheit: (data: { value: number }) => JSX.Element;
}) {
  const [value, setValue] = useState(0);

  return (
    <>
      <input
        type="number"
        value={value}
        onChange={(e) => setValue(Number(e.target.value))}
      />
      {renderKelvin({ value: value + 273.15 })}
      {renderFahrenheit({ value: (value * 9) / 5 + 32 })}
    </>
  );
}

// Component using Input
export default function App() {
  return (
    <Input
      renderKelvin={({ value }) => <div className="temp">{value}K</div>}
      renderFahrenheit={({ value }) => <div className="temp">{value}°F</div>}
    />
  );
}
```

Key Takeaways
  - ✅ The Render Props pattern allows components to `share logic without inheritance or HOCs`.
  - ✅ The function `prop can have any name, not just render`.
  - ✅ Use cases: reusable logic like mouse tracking, form validation, data fetching, etc.
  - ✅ Alternatives: `React Hooks (useState, useEffect) are often preferred` for newer projects.

