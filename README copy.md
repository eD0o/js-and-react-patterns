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

# 2.4 - Hooks Pattern

React Hooks are special functions that allow you to:

- `Add state` to functional components.
- `Reuse stateful logic` across multiple components.
- `Manage the lifecycle` a component without using class components.

Hooks unify previously introduced patterns (Container-Presentation, HOC, and Render Props) into a more streamlined approach.

Besides built-in hooks, such as useState, useEffect, and useReducer, we `can create custom hooks to easiliy share stateful logic across multiple components`.

> To create a custom hook, its name `must start with "use" so that React recognizes` it as a hook.

With Hooks, we `no longer have to wrap Presentational components in Container components to pass data. Instead, we can use hooks directly inside presentational components`.

Example: Custom Hook (useHover)

```jsx
export function useHover() {
  const [isHovering, setIsHovering] = React.useState(false);
  const ref = React.useRef(null);

  const handleMouseOver = () => setIsHovering(true);
  const handleMouseOut = () => setIsHovering(false);

  React.useEffect(() => {
    const node = ref.current;
    if (node) {
      node.addEventListener("mouseover", handleMouseOver);
      node.addEventListener("mouseout", handleMouseOut);
      return () => {
        node.removeEventListener("mouseover", handleMouseOver);
        node.removeEventListener("mouseout", handleMouseOut);
      };
    }
  }, [ref.current]);

  return [ref, isHovering];
}
```

```jsx
// Using useHover in a Component
import { useHover } from "../hooks/useHover";

export function Listing() {
  const [ref, isHovering] = useHover();

  React.useEffect(() => {
    if (isHovering) {
      // Add logic here
    }
  }, [isHovering]);

  return (
    <div ref={ref}>
      <ListingCard />
    </div>
  );
}
```

Con: Hooks require certain rules to be followed. `Without a linter plugin (eslint-plugin-react-hooks), it can be difficult to detect rule violations`, and it's easy to misuse hooks, such as calling them conditionally or inside loops.

Some other example hooks: https://usehooks.com/

# 2.5 - Provider Pattern

It `utilizes React's Context API`, which allows for easy data sharing between components.

A Provider `is a higher-order component provided by the Context object. We can create a Context object using the createContext` method that React provides.

```tsx
import React, { createContext, useState } from "react";

export const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

```tsx
import { ThemeProvider, ThemeContext } from "../context";

const LandingPage = () => {
  return (
    <ThemeProvider>
      <TopNav />
      <Main />
    </ThemeProvider>
  );
};

const TopNav = () => {
  return (
    <ThemeContext.Consumer>
      {({ theme }) => (
        <div style={{ backgroundColor: theme === "light" ? "#fff" : "#000" }}>
          ...
        </div>
      )}
    </ThemeContext.Consumer>
  );
};

const Toggle = () => {
  return (
    <ThemeContext.Consumer>
      {({ theme, setTheme }) => (
        <button
          onClick={() => setTheme(theme === "light" ? "dark" : "light")}
          style={{
            backgroundColor: theme === "light" ? "#fff" : "#000",
            color: theme === "light" ? "#000" : "#fff",
          }}
        >
          Use {theme === "light" ? "Dark" : "Light"} Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
};
```

## 2.5.1 - Multiple Providers in an App

A Provider `does not have to be used only in App.tsx`. `Any component can define its own Provider`, allowing for scoped contexts.

```tsx
const Layout = () => {
  return (
    <ThemeProvider>
      <UserProvider>
        <Navbar />
        <MainContent />
      </UserProvider>
    </ThemeProvider>
  );
};

// Here, ThemeProvider manages the theme state, while UserProvider manages authentication.
```

Component-Specific Providers: A component `can wrap a specific section with its own Provider, overriding the global context`.

```tsx
const Section = () => {
  return (
    <ThemeProvider>
      <SubSection />
    </ThemeProvider>
  );
};
```

In this case, SubSection and its children will use the new ThemeProvider instead of inheriting from a global provider.

Nested Providers for User and Theme: We `can nest multiple providers to manage separate state logic` while keeping components independent.

```tsx
import React, { createContext, useState, useContext } from "react";

// Theme Context
const ThemeContext = createContext(null);

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// User Context
const UserContext = createContext(null);

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

// Component consuming both contexts
const Dashboard = () => {
  const { theme, setTheme } = useContext(ThemeContext);
  const { user, setUser } = useContext(UserContext);

  return (
    <div
      style={{
        background: theme === "light" ? "#fff" : "#000",
        color: theme === "light" ? "#000" : "#fff",
      }}
    >
      <h1>Welcome, {user ? user.name : "Guest"}!</h1>
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
      <button onClick={() => setUser(user ? null : { name: "John" })}>
        {user ? "Logout" : "Login"}
      </button>
    </div>
  );
};

// App Component with Nested Providers
const App = () => {
  return (
    <ThemeProvider>
      <UserProvider>
        <Dashboard />
      </UserProvider>
    </ThemeProvider>
  );
};
```

This example demonstrates:

- Independent providers for theme and user state.

- Consuming multiple contexts inside the Dashboard component.

- Scoped providers so different parts of the app can manage their own context values.

# 2.6 - Compound Component Pattern

The Compound Component Pattern allows components to work together as a cohesive unit by exposing multiple subcomponents inside a single parent component. Instead of passing multiple props, this pattern enables implicit communication between components using React Context.

Difference Between Provider Pattern and Compound Pattern

| Feature        | Provider Pattern                                                 | Compound Component Pattern                                                |
| -------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Purpose        | Shares data across components globally or in a section           | Groups related components together into a `single reusable unit`          |
| Implementation | Uses createContext and provides a Provider for state management  | Uses `createContext inside a parent component to manage state` internally |
| Usage          | Any component within the provider’s tree can consume the context | Subcomponents are `explicitly structured` within the main component       |
| Best for       | Theming, Authentication, Global State                            | Toggles, Tabs, Accordions, Menus                                          |

---

Example: Toggle Component with Compound Pattern

`Instead of passing multiple props to a single component, we define subcomponents inside a parent` Toggle component.

```tsx
import React, { createContext, useState, useContext } from "react";

const ToggleContext = createContext(null);

const Toggle = ({ children }) => {
  const [on, setOn] = useState(false);
  return (
    <ToggleContext.Provider value={{ on, setOn }}>
      {children}
    </ToggleContext.Provider>
  );
};

const ToggleButton = () => {
  const { on, setOn } = useContext(ToggleContext);
  return (
    <button onClick={() => setOn(!on)}>{on ? "Turn Off" : "Turn On"}</button>
  );
};

const ToggleStatus = () => {
  const { on } = useContext(ToggleContext);
  return <p>Status: {on ? "ON" : "OFF"}</p>;
};

// Usage of Toggle with subcomponents
const App = () => {
  return (
    <Toggle>
      <ToggleButton />
      <ToggleStatus />
    </Toggle>
  );
};

export default App;
```

What Makes This a Compound Component?

Toggle acts as the parent component, managing state.

ToggleButton and ToggleStatus are subcomponents that consume the context but don't require direct props.

The API is `more intuitive and declarative than passing multiple props to a single component`.

Example: Tabs Component Using Compound Pattern

Tabs are a common UI pattern where `different sections of content are displayed based on the active tab`.

```tsx
import React, { createContext, useState, useContext } from "react";

const TabsContext = createContext(null);

const Tabs = ({ children }) => {
  const [activeTab, setActiveTab] = useState(0);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
};

const TabList = ({ children }) => <div>{children}</div>;

const Tab = ({ index, children }) => {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  return (
    <button
      style={{ fontWeight: activeTab === index ? "bold" : "normal" }}
      onClick={() => setActiveTab(index)}
    >
      {children}
    </button>
  );
};

const TabPanel = ({ index, children }) => {
  const { activeTab } = useContext(TabsContext);
  return activeTab === index ? <div>{children}</div> : null;
};

// Using Tabs with subcomponents
const App = () => {
  return (
    <Tabs>
      <TabList>
        <Tab index={0}>Tab 1</Tab>
        <Tab index={1}>Tab 2</Tab>
      </TabList>
      <TabPanel index={0}>Content of Tab 1</TabPanel>
      <TabPanel index={1}>Content of Tab 2</TabPanel>
    </Tabs>
  );
};

export default App;
```

Why use the Compound Pattern for Tabs?

Keeps the Tabs API clean and readable.

`Avoids passing props like activeTab and setActiveTab manually to each component`.

`Automatically associates tab buttons with their content` without prop-drilling.

When to Use the Compound Component Pattern?

- When `multiple components need to work together as a unit` (e.g., Toggles, Modals, Accordions).
- When you want a `more declarative API instead of passing multiple props`.
- When components `should be self-contained and not rely on external state`.

> The Compound Pattern makes UI components more flexible, reusable, and modular by defining a parent-child relationship where components naturally interact without unnecessary props.
