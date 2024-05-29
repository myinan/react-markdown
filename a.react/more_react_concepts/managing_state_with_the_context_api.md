# Managing State With The Context API
## Passing Data Deeply with Context
Usually, you will pass information from a parent component to a child component via props. But passing props can become verbose and inconvenient if you have to pass them through many components in the middle, or if many components in your app need the same information. Context lets the parent component make some information available to any component in the tree below it—no matter how deep—without passing it explicitly through props.

### The problem with passing props
Passing props is a great way to explicitly pipe data through your UI tree to the components that use it.

But passing props can become verbose and inconvenient when you need to pass some prop deeply through the tree, or if many components need the same prop. The nearest common ancestor could be far removed from the components that need data, and lifting state up that high can lead to a situation called “prop drilling”.

Wouldn’t it be great if there were a way to “teleport” data to the components in the tree that need it without passing props? With React’s context feature, there is!

### Context: an alternative to passing props
Context lets a parent component provide data to the entire tree below it. There are many uses for context. Here is one example. Consider this `Heading` component that accepts a `level` for its size:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Heading level={2}>Heading</Heading>
      <Heading level={3}>Sub-heading</Heading>
      <Heading level={4}>Sub-sub-heading</Heading>
      <Heading level={5}>Sub-sub-sub-heading</Heading>
      <Heading level={6}>Sub-sub-sub-sub-heading</Heading>
    </Section>
  );
}
```

Let’s say you want multiple headings within the same `Section` to always have the same size:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Currently, you pass the `level` prop to each `<Heading>` separately:

```jsx
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

It would be nice if you could pass the `level` prop to the `<Section>` component instead and remove it from the `<Heading>`. This way you could enforce that all headings in the same section have the same size:

```jsx
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

But how can the `<Heading>` component know the `level` of its closest `<Section>`? **That would require some way for a child to “ask” for data from somewhere above in the tree.**

You can’t do it with props alone. This is where context comes into play. You will do it in three steps:

1. Create a context. (You can call it `LevelContext`, since it’s for the heading level.)
2. Use that context from the component that needs the data. (`Heading` will use `LevelContext`.)
3. Provide that context from the component that specifies the data. (`Section` will provide `LevelContext`.)

Context lets a parent—even a distant one!—provide some data to the entire tree inside of it.

### Step 1: Create the context 
First, you need to create the context. You’ll need to export it from a file so that your components can use it:

```jsx
// LevelContext.jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

The only argument to `createContext` is the *default value*. Here, `1` refers to the biggest heading level, but you could pass any kind of value (even an object). You will see the significance of the default value in the next step.

### Step 2: Use the context
Import the `useContext` Hook from React and your context:

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

Currently, the `Heading` component reads `level` from props:

```jsx
// Heading.jsx

export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Instead, remove the `level` prop and read the value from the context you just imported, `LevelContext`:

```jsx
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext` is a Hook. Just like `useState` and `useReducer`, you can only call a Hook immediately inside a React component (not inside loops or conditions). **`useContext` tells React that the `Heading` component wants to read the `LevelContext`.**

Now that the `Heading` component doesn’t have a `level` prop, you don’t need to pass the `level` prop to `Heading` in your JSX like this anymore:

```jsx
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

Update the JSX so that it’s the `Section` that receives it instead:

```jsx
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

As a reminder, this is the markup that you were trying to get working:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Notice this example doesn’t quite work, yet! All the headings have the same size(1) because even though you’re using the context, you have not provided it yet. React doesn’t know where to get it!

If you don’t provide the context, React will use the default value you’ve specified in the previous step. In this example, you specified `1` as the argument to `createContext`, so `useContext(LevelContext)` returns `1`, setting all those headings to `<h1>`. Let’s fix this problem by having each `Section` provide its own context.

### Step 3: Provide the context
The `Section` component currently renders its children:

```jsx
// Section.js

export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Wrap them with a context provider** to provide the `LevelContext` to them:

```jsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

This tells React: “if any component inside this `<Section>` asks for `LevelContext`, give them this `level`.” The component will use the value of the nearest `<LevelContext.Provider>` in the UI tree above it.

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

It’s the same result as the original code, but you did not need to pass the `level` prop to each `Heading` component! Instead, it “figures out” its heading level by asking the closest `Section` above:

1. You pass a level prop to the `<Section>`.
1. Section wraps its children into `<LevelContext.Provider value={level}>`.
1. Heading asks the closest value of LevelContext above with `useContext(LevelContext)`.

### Using and providing context from the same component 
Currently, you still have to specify each section’s `level` manually:

```jsx
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

Since context lets you read information from a component above, each `Section` could read the `level` from the `Section` above, and pass `level + 1` down automatically. Here is how you could do it:

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

With this change, you don’t need to pass the `level` prop either to the `<Section>` or to the `<Heading>`:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Now both `Heading` and `Section` read the `LevelContext` to figure out how “deep” they are. And the `Section` wraps its children into the `LevelContext` to specify that anything inside of it is at a “deeper” level.

Note: This example uses heading levels because they show visually how nested components can override context. But context is useful for many other use cases too. You can pass down any information needed by the entire subtree: the current color theme, the currently logged in user, and so on.

### Context passes through intermediate components
You can insert as many components as you like between the component that provides context and the one that uses it. This includes both built-in components like `<div>` and components you might build yourself.

**Context lets you write components that “adapt to their surroundings” and display themselves differently depending on where (or, in other words, in which context) they are being rendered.**

How context works might remind you of CSS property inheritance. In CSS, you can specify `color: blue` for a `<div>`, and any DOM node inside of it, no matter how deep, will inherit that color unless some other DOM node in the middle overrides it with `color: green`. Similarly, in React, the only way to override some context coming from above is to wrap children into a context provider with a different value.

In CSS, different properties like `color` and `background-color` don’t override each other. You can set all  `<div>`’s `color` to red without impacting `background-color`. Similarly, different React contexts don’t override each other. Each context that you make with `createContext()` is completely separate from other ones, and ties together components using and providing that particular context. One component may use or provide many different contexts without a problem.

### Use cases for context 
+ **Theming:** If your app lets the user change its appearance (e.g. dark mode), you can put a context provider at the top of your app, and use that context in components that need to adjust their visual look.

+ **Current account:** Many components might need to know the currently logged in user. Putting it in context makes it convenient to read it anywhere in the tree. Some apps also let you operate multiple accounts at the same time (e.g. to leave a comment as a different user). In those cases, it can be convenient to wrap a part of the UI into a nested provider with a different current account value.

+ **Routing:** Most routing solutions use context internally to hold the current route. This is how every link “knows” whether it’s active or not. If you build your own router, you might want to do it too.

+ **Managing state:** As your app grows, you might end up with a lot of state closer to the top of your app. Many distant components below may want to change it. It is common to use a reducer together with context to manage complex state and pass it down to distant components without too much hassle.

Context is not limited to static values. If you pass a different value on the next render, React will update all the components reading it below! This is why context is often used in combination with state.

In general, if some information is needed by distant components in different parts of the tree, it’s a good indication that context will help you.

### Recap
+ Context lets a component provide some information to the entire tree below it.
+ To pass context:
    + Create and export it with `export const MyContext = createContext(defaultValue)`.
    + Pass it to the `useContext(MyContext)` Hook to read it in any child component, no matter how deep.
    + Wrap children into `<MyContext.Provider value={...}>` to provide it from a parent.
+ Context passes through any components in the middle.

### Optimizing re-renders when passing objects and functions 
You can pass any values via context, including objects and functions.

```jsx
unction MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

Here, the context value is a JavaScript object with two properties, one of which is a function. Whenever `MyApp` re-renders (for example, on a route update), this will be a different object pointing at a different function, so React will also have to re-render all components deep in the tree that call `useContext(AuthContext)`.

In smaller apps, this is not a problem. However, there is no need to re-render them if the underlying data, like currentUser, has not changed. To help React take advantage of that fact, you may wrap the login function with `useCallback` and wrap the object creation into `useMemo`. This is a performance optimization:

```jsx
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

As a result of this change, even if `MyApp` needs to re-render, the components calling `useContext(AuthContext)` won’t need to re-render unless `currentUser` has changed.

---
### Knowledge Check

**Q1.** What are the benefits of using the Context API over passing props down through multiple levels of components?

**A1.** Using the Context API in React offers several benefits over passing props down through multiple levels of components:

#### 1. Avoids Prop Drilling

Prop drilling refers to the process of passing props from a parent component through intermediate child components down to a deeply nested child component that needs access to the props. Context API eliminates the need for prop drilling, making the code cleaner and more maintainable.

#### 2. Cleaner and More Readable Code

By using the Context API, you can make your code more readable and maintainable by reducing the number of props that need to be passed through the component tree. This leads to a more concise and easier-to-understand codebase.

#### 3. Centralized State Management

Context API allows you to manage state in a centralized way, making it easier to share data and functionality across multiple components without passing props manually at each level. This can be particularly useful for global state management, such as user authentication, theme settings, or language preferences.

#### 4. Improved Component Reusability

With the Context API, you can create reusable components that consume the context, making it easier to share functionality and state across different parts of your application. This promotes a more modular and scalable codebase.

#### 5. Performance Optimization

In some cases, using the Context API can lead to performance optimizations by reducing the number of unnecessary re-renders caused by prop changes. Context consumers can selectively re-render only when the context value they are subscribed to changes.

#### Example:

Here's a simple example to illustrate the difference between prop drilling and using the Context API:

#### Prop Drilling

```jsx
const GrandChild = ({ text }) => {
  return <div>{text}</div>;
};

const Child = ({ text }) => {
  return <GrandChild text={text} />;
};

const Parent = ({ text }) => {
  return <Child text={text} />;
};

const App = () => {
  return <Parent text="Hello, Context API!" />;
};
```

#### Using Context API

```jsx
import React, { createContext, useContext } from 'react';

const TextContext = createContext();

const GrandChild = () => {
  const text = useContext(TextContext);
  return <div>{text}</div>;
};

const Child = () => {
  return <GrandChild />;
};

const Parent = () => {
  return <Child />;
};

const App = () => {
  return (
    <TextContext.Provider value="Hello, Context API!">
      <Parent />
    </TextContext.Provider>
  );
};
```

In the example above, using the Context API eliminates the need to pass the `text` prop down through the component tree, making the code cleaner and easier to manage.

Overall, the Context API provides a more elegant and efficient way to manage and share state and functionality across components compared to prop drilling.

**Q2.** What are the drawbacks in using the Context API?

**A2.** While the Context API in React provides several benefits, there are also some drawbacks to consider:

#### 1. Global State

One of the main concerns with the Context API is the potential for creating a global state, which can lead to a complex and hard-to-manage state tree. It can become difficult to track the flow of data and state changes, making the codebase harder to maintain and debug.

#### 2. Over-Reliance on Context

Overusing the Context API can lead to a situation where many components are tightly coupled to the context, making it harder to reuse these components in different parts of the application. This can reduce the modularity and scalability of the codebase.

#### 3. Performance Concerns

In some cases, using the Context API can lead to performance issues due to unnecessary re-renders. If the context value changes frequently, it can trigger re-renders in all components that consume the context, even if the component doesn't actually need to re-render.

#### 4. Less Explicit Data Flow

Prop drilling, while more verbose, provides a clear and explicit data flow through the component tree. On the other hand, using the Context API can make the data flow less clear and harder to trace, especially in larger and more complex applications.

#### 5. Limited Built-in Features

The Context API is relatively low-level and lacks some advanced features that are available in other state management libraries, such as built-in support for asynchronous actions, middleware, and time-travel debugging.

#### Example:

Here's a simple example to illustrate the potential drawbacks of using the Context API:

```jsx
import React, { createContext, useContext, useState } from 'react';

const CountContext = createContext();

const GrandChild = () => {
  const { count, setCount } = useContext(CountContext);

  return (
    <div>
      <button onClick={() => setCount(count - 1)}>-</button>
      <span>{count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
};

const Child = () => {
  return <GrandChild />;
};

const Parent = () => {
  return <Child />;
};

const App = () => {
  const [count, setCount] = useState(0);

  return (
    <CountContext.Provider value={{ count, setCount }}>
      <Parent />
    </CountContext.Provider>
  );
};
```

In the example above, the state is managed globally using the Context API. This can lead to a complex and hard-to-manage state tree, especially as the application grows in size and complexity.

#### Conclusion:

While the Context API is a powerful tool for managing global state in React applications, it's important to use it judiciously and consider its limitations and potential drawbacks. In some cases, using a dedicated state management library or a combination of local state and prop drilling may be a more suitable and scalable solution for managing state in large and complex applications.

**Q3.** What are the ways you can avoid prop drilling?

**A3.** One of the things that really aggravates problems with prop drilling is breaking out your render method into multiple components unnecessarily. You'll be surprised how simple a big render method can be when you just inline as much as you can. There's no reason to break things out prematurely. Wait until you really need to reuse a block before breaking it out. Then you won't need to pass props anyway!

Another thing you can do to mitigate the effects of prop-drilling is avoid using defaultProps for anything that's a required prop. Using a defaultProp for something that's actually required for the component to function properly is just hiding important errors and making things fail silently. So only use defaultProps for things that are not required.

Keep state as close to where it's relevant as possible. If only one section of your app needs some state, then manage that in the least common parent of those components rather than putting it at the highest level of the app.

Use React's Context API for things that are truly necessary deep in the react tree. They don't have to be things you need everywhere in the application (you can render a provider anywhere in the app). This can really help avoid some issues with prop drilling. It's been noted that context is kinda taking us back to the days of global variables. The difference is that because of the way the API was designed, you can still statically find the source of the context as well as any consumers with relative ease.
