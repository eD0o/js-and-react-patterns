# 4 - Rendering Patterns

Rendering content on the web can be done in many ways. `The decision on how and where to fetch and render content is key to the performance` of an application.

The available frameworks and libraries can be used to implement different rendering patterns like Client-Side Rendering, Static Rendering, Incremental Static Regeneration (specific to Next.js), Progressive Rendering, Server-Side Rendering, and many more. `Understanding the tradeoffs and use cases for these patterns can drastically help the performance` of your application, resulting in a great user and developer experience.

## 4.1 - Core Web Vitals

To measure how well our website performs, `we can use a set of useful measurements called Google's Web Vitals. A subset of these measurements - the Core Web Vitals - is usually used to determine the performance of a page` and can affect your website's SEO.

### Core Web Vitals Metrics:

- **TTFB (Time To First Byte):** Time it takes for a client to receive the `first byte of page` content.
- **FCP (First Contentful Paint):** Time it takes the browser to render the `first piece of content after navigation`.
- **LCP (Largest Contentful Paint):** Time it takes to `load and render the page's main content`.
- **FID (First Input Delay):** Time from when the user interacts with the page to the time when the `event handlers are able to run`.
- **TTI (Time To Interactive):** Time from when the page starts loading to when it's `reliably responding to user input quickly`.
- **CLS (Cumulative Layout Shift):** Measures `visual stability` to avoid unexpected layout shifts.

### Key Terms for Rendering Techniques:

- **Compiling:** Converting JavaScript into native machine code.
- **Execution time:** The time it takes to run the JavaScript code after it has been parsed and compiled.
- **Hydration:** The process of attaching event handlers to static HTML elements (often server-rendered) to make them interactive in a client-side framework like React.
- **Idle:** The browser's state when it's not performing any action.
- **Loading time:** The time it takes to fetch the data from the server.
- **Main thread:** The thread on which the browser executes all the JavaScript, performs layout, reflows, and garbage collection.
- **Parsing:** Converting an HTML source into DOM nodes and generating an AST (Abstract Syntax Tree).
- **Processing:** Parsing, compiling, and executing the previously fetched data.
- **Processing time:** The time it takes to parse and compile the previously fetched data.

Understanding these concepts and their relationship with different rendering patterns can help optimize web applications for performance and user experience.
