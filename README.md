# 2 - React Patterns

## 2.1 - Container/Presentation Pattern

It's a design pattern in React that separates concerns by dividing components into two categories:

- Presentational Components: `Focus on UI, rendering, and styles`. They receive data and callbacks via props but don’t handle state or logic.
- Container Components: `Handle business logic, state, and data fetching`. They pass necessary data and functions to presentational components.

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

Modern Aproach - With Hooks (But Less Necessary*)

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

> * With React Hooks, the need for the Container/Presentational pattern has significantly decreased. The pattern was more common before Hooks, when class components required explicit separation of logic (state, lifecycle methods) from UI.

### 2.1.1 - Why Hooks Reduce the Need for the Pattern:
- State & Effects in Functional Components – `Hooks like useState and useEffect allow handling logic inside the same component without class-based lifecycles`.
- Encapsulation with Custom Hooks – Instead of separate container components, logic can be extracted into custom hooks, making components reusable and cleaner.

## 2.1.2 - Alternative with Hooks (No Need for Container Component)

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

> Why? No need for a separate container, since useState allows managing state within the same component.

### 2.1.3 - When Should We Still Use Container/Presentational?

- If the `UI component (Button) is used in multiple places with different logic`.
- If we want a `clear separation between UI and logic` (for better reusability in large apps).
- If the `logic is complex` (e.g., data fetching, context management).
