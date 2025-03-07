# 2 - React Patterns

## 2.1 - Container/Presentation Pattern

The module pattern is a great way to `split a larger file into multiple smaller`, reusable pieces.
It also `promotes code encapsulation`, since the `values within modules are kept private` inside the module by default, and cannot be modified. This protects values from leaking into the global scope or ending up in a naming collision.

> Only the values that are explicitly exported with the export keyword are accessible to other files.

In Node, you can use ES2015 modules either by:

- Using the .mjs extension
- Adding "type": "module" to your package.json

Example:

```jsonc
// package.json
{
  "name": "node-test",
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  }
}
```

```js
// math.js
const secret = "abc"; // not accesible variable outside the module as it hasn't been exported

export function sum(x, y) {
  return x + y;
}

export function multiply(x, y) {
  return x * y;
}

export function subtract(x, y) {
  return x - y;
}

export function divide(x, y) {
  return x / y;
}
```

```js
// index.js
import { sum, subtract, divide, multiply } from "./math.js";

console.log("Sum", sum(1, 2));
console.log("Subtract", subtract(1, 2));
console.log("Divide", divide(1, 2));
console.log("Multiply", multiply(1, 2));
```

## 1.2 - Singleton Pattern (Unnecessary)

> Unnecessary: ES2015 Modules are singletons by default. We no longer need to explicitly create singletons to achieve this global, non-modifiable behavior.

Shares a `single global instance throughout our application`.

Cons:

- Testing: It's really difficult to test singletons because we change it globally.

- Depedency Hiding: When importing another module, it may not always be obvious that that module is importing a Singleton. This could lead to unexpected value modification within the Singleton, which would be reflected throughout the application.

- Global Scope Pollution: The global behavior of Singletons is essentially the same as a global variable. Global Scope Pollution can end up in accidentally overwriting the value of a global variable, which can lead to a lot of unexpected behavior

Example:

```js
let instance;

class DBConnection {
  constructor(uri) {
    if (instance) {
      throw new Error("Only one connection can exist!");
    }
    this.uri = uri;
    instance = this;
  }

  connect() {
    this.isConnected = true;
    console.log(`DB ${this.uri} has been connected!`);
  }

  disconnect() {
    this.isConnected = false;
    console.log("DB disconnected");
  }
}

const databaseConnector = Object.freeze(new DBConnection());
const connection = databaseConnector;
```

We can also directly create objects without having to use a class, which can lead to much simpler and cleaner code.

To create a singleton using a regular object, we have to:

```js
let counter = 0;

// 1. Create an object containing the `getCount`, `increment`, and `decrement` method.
const counterObject = {
  getCount: () => counter,
  increment: () => ++counter,
  decrement: () => --counter,
};

// 2. Freeze the object using the `Object.freeze` method, it ensures immutability, which aligns with the singleton's non-modifiable behavior.
const singletonCounter = Object.freeze(counterObject);

// 3. Export the object as the `default` value to make it globally accessible.
export default singletonCounter;
```

We could even export the frozen object directly, without having to declare multiple variables.

```js
let counter = 0;

export default Object.freeze({
  getCount: () => counter,
  increment: () => ++counter,
  decrement: () => --counter,
});
```

## 1.3 - Difference Between Modules and Singletons

| Aspect            | **Module**                                                                                        | **Singleton**                                                     |
| ----------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Definition**    | A file or function that exports values, functions, or objects for reuse.                          | A pattern ensuring only one instance of a class or object exists. |
| **Scope**         | Depends on what is exported: functions or objects can be reused with new instances or references. | Always maintains a single global instance.                        |
| **Encapsulation** | Strong encapsulation of variables and functions. Non-exported values remain private.              | Encapsulates the instance but usually exposes methods publicly.   |
| **State**         | Has no intrinsic state (unless explicitly created).                                               | Retains state because it is always the same object or instance.   |
| **Applications**  | Organizing code, utility libraries.                                                               | Managing global resources (e.g., DB connections, configuration).  |

## 1.4 - Proxy Pattern

It uses a `Proxy intercept and control interactions to target objects`.

The Proxy object `receives two arguments`:

1. The `target` object
2. A `handler object`, which we can use to add functionality to the proxy. This object comes with some built-in functions that we can use, such as get and set.

```js
const person = {
  name: "John Doe",
  age: 42,
  email: "john@doe.com",
  country: "Canada",
};

const personProxy = new Proxy(person, {
  get: (target, prop) => {
    console.log(`The value of ${prop} is ${target[prop]}`);
    return target[prop];
  },
  set: (target, prop, value) => {
    console.log(`Changed ${prop} from ${target[prop]} to ${value}`);
    target[prop] = value;
    return true;
  },
});

// The get method on the handler object gets invoked when we want to access a property,
// and the set method gets invoked when we want to modify a property.
```

### 1.4.1 - Reflect

The built-in Reflect object makes it easier to manipulate the target object.

Instead of accessing properties through `obj[prop]` or setting properties through `obj[prop]` = value, we can access or modify properties on the target object through Reflect.get() and Reflect.set(). The methods receive the same arguments as the methods on the handler object.

```js
new Proxy(person, {
  get: (target, prop) => {
    return Reflect.get(target, prop);
  },
  set: (target, prop, value) => {
    return Reflect.get(target, prop, value);
  },
});
```

Pros:

- Control - Proxies make it easy to add functionality when interacting with a certain object, such as validation, logging, formatting, notifications, debugging.

Cons:

- Long handler execution - Executing handlers on every object interaction could lead to performance issues.

## 1.5 - Observer Pattern

The Observer Pattern is a design pattern that `defines a one-to-many relationship between an observable object and its observers`. When the `observable's state changes or an event occurs, it notifies all its observers`, which then react accordingly.

> This pattern is not exclusive to JavaScript but is widely used in the language due to its event-driven nature.

---

### 1.5.1 Key Components

1. Observable:

   - The `object that emits events or state changes`.
   - Maintains a list of observers that will be notified.

2. Observers:
   - The objects (or functions) that `subscribe to the observable to be notified when something happens`.

---

### 1.5.2 Common Implementations in JavaScript

**DOM Events (addEventListener)**:

The DOM's event-handling mechanism is a built-in example of the Observer Pattern. Here, the `addEventListener method allows observers to subscribe to events emitted by a DOM element` (the observable).

```javascript
const button = document.querySelector("#myButton");

const handleClick = () => console.log("Button clicked!");

button.addEventListener("click", handleClick);
```

In this example:

- The `button is the observable`.
- `handleClick is the observer`.
- The event `click triggers the notification`.

---

**Manual Implementation with notify, subscribe, and unsubscribe**:

This is a classic way to implement the Observer Pattern manually. `Observers subscribe to an observable, which then notifies` them when an event occurs.

```javascript
const observers = [];

export default Object.freeze({
  notify: (data) => observers.forEach((observer) => observer(data)),
  subscribe: (func) => observers.push(func),
  unsubscribe: (func) => {
    observers.forEach((observer, index) => {
      if (observer === func) {
        observers.splice(index, 1);
      }
    });
  },
});
```

---

**Classes**:

Using classes `allows for multiple independent observables, each managing its own observers`.

```javascript
class Observable {
  constructor() {
    this.observers = [];
  }

  subscribe(func) {
    this.observers.push(func);
  }

  unsubscribe(func) {
    this.observers = this.observers.filter((observer) => observer !== func);
  }

  notify(data) {
    this.observers.forEach((observer) => observer(data));
  }
}

const observable = new Observable();
const observer = (data) => console.log(`Received: ${data}`);

observable.subscribe(observer);
observable.notify("Hello, Observer Pattern!");
```

---

**Using Object.freeze (Immutability and Singleton)**:

This approach `creates an immutable singleton observable object`. It's `useful when you have a single global event` source.

```javascript
const observers = [];

export default Object.freeze({
  notify: (data) => observers.forEach((observer) => observer(data)),
  subscribe: (func) => observers.push(func),
  unsubscribe: (func) => {
    observers.forEach((observer, index) => {
      if (observer === func) {
        observers.splice(index, 1);
      }
    });
  },
});
```

---

**Promises and then (Observing Async Flows)**:

You "subscribe" to a result using then or catch.

> While not a direct implementation, the Promise API behaves similarly to the Observer Pattern.

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve("Data received!"), 1000);
});

promise.then((data) => console.log(data));
```

---

**RxJS and Observables (Reactive Streams)**:

RxJS `provides a robust implementation of the Observer Pattern`, enabling asynchronous streams of data.

```javascript
import { Observable } from "rxjs";

const observable = new Observable((subscriber) => {
  subscriber.next("First event");
  setTimeout(() => subscriber.next("Second event"), 1000);
});

observable.subscribe((data) => console.log(data));
```

---

**EventEmitter in Node.js**:

Node.js provides a built-in implementation of the Observer Pattern through the `EventEmitter class`.

```javascript
const EventEmitter = require("events");
const emitter = new EventEmitter();

emitter.on("event", (message) => {
  console.log(message);
});

emitter.emit("event", "Hello, EventEmitter!");
```

---

## 1.6 - Factory Pattern

The factory pattern is `useful when creating multiple objects with shared properties`, avoiding repetitive code. A factory function `returns a custom object based on specific configurations, such as environment or user preferences`.

```js
const createUser = (firstName, lastName) => ({
  id: crypto.randomUUID(),
  createdAt: Date.now(),
  firstName,
  lastName,
  fullName: `${firstName} ${lastName}`,
});

createUser("John", "Doe");
createUser("Sarah", "Doe");
createUser("Lydia", "Hallie");
```

> Note: In JavaScript, the `factory pattern is essentially a function that returns an object without using the new keyword`. ES6 arrow functions make it easier to create concise factory functions that implicitly return objects.

However, for `cases where methods need to be shared across all instances, using class may be more memory-efficient`. This ensures methods are stored on the prototype rather than recreated for every object.

```js
class User {
  constructor({ firstName, lastName, email }) {
    this.id = crypto.randomUUID();
    this.createdAt = Date.now();
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
  }

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  async getPosts() {
    const posts = await fetch(`https://my.cms.com/posts/user/${this.id}`);
    return posts;
  }
}

const user1 = new User({
  firstName: "John",
  lastName: "Doe",
  email: "john@doe.com",
});

const user2 = new User({
  firstName: "Jane",
  lastName: "Doe",
  email: "jane@doe.com",
});

// Methods like getPosts are shared across instances via the prototype, saving memory.

// The fullName method is the same for all the objects that were created.
// By creating new instances, the fullName method is available on the prototype instead of on the objec, which saves memory.
```

## 1.7 - Prototye Pattern

The Prototype Pattern is `used to share properties and methods among many objects of the same type, reducing memory overhead`.

For example, if we wanted to create many dogs with a factory function:

```js
// This way, we can easily create many dog objects with the same properties.
const createDog = (name, age) => ({
  name,
  age,
  bark() {
    console.log(`${name} is barking!`);
  },
  wagTail() {
    console.log(`${name} is wagging their tail!`);
  },
});

const dog1 = createDog("Max", 4);
const dog2 = createDog("Sam", 2);
const dog3 = createDog("Joy", 6);
const dog4 = createDog("Spot", 8);
```

![](https://javascriptpatterns.vercel.app/design-patterns/prototype-pattern/3.png)

> This way creates unique bark and wagTail functions for every dog object, leading to unnecessary memory usage.

We can use the `Prototype Pattern to share methods across all instances`:

```js
class Dog {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  bark() {
    console.log(`${this.name} is barking!`);
  }
  wagTail() {
    console.log(`${this.name} is wagging their tail!`);
  }
}

const dog1 = new Dog("Max", 4);
const dog2 = new Dog("Sam", 2);
const dog3 = new Dog("Joy", 6);
const dog4 = new Dog("Spot", 8);
// In this example, bark and wagTail are defined on Dog.prototype and shared across all instances.
```

![](https://javascriptpatterns.vercel.app/design-patterns/prototype-pattern/4.png)

Pros: 
- Memory efficient: Shared methods on the prototype chain reduce duplication.
- Centralized Maintenance: Changes to prototype methods apply to all instances.

Cons: 
- Readaibility: Deep inheritance chains can make it difficult to trace property origins. For example, a BorderCollie class inheriting from Animal might obscure where specific properties are defined.

![](https://javascriptpatterns.vercel.app/design-patterns/prototype-pattern/2.png)
