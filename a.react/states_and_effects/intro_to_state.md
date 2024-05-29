# Introduction To State
Any exciting application you build is likely to change over the time the user is exploring it. The changes could be as basic as toggling a dropdown menu or as complex as fetching data from an API. React provides primitives to manipulate the state of your apps, more specifically components, to make them dynamic. In this lesson, we will learn how to use state in React.

In React, "state" refers to the data structure that represents the internal state of a component. It is essentially an object that holds information that may change over time and influences the rendering of the component. We can think of "state" as a "component’s memory". Understanding state is crucial in React development as it allows components to manage and update their data dynamically, resulting in a reactive user interface.

Here's a detailed explanation of state in React:

1. **Component State**:
   - Each component in React can have its own state, which is independent of other components. This means that changes to the state of one component do not affect the state of another component unless explicitly passed down as props.
   - State is typically initialized in the constructor of a class component or by using the `useState` hook in a functional component. For class components, you use `this.state = { /* initial state */ };` in the constructor. In functional components, you use the `useState` hook like so: `const [state, setState] = useState(initialState);`.

2. **Immutable**:
   - State in React is immutable, meaning it cannot be modified directly. Instead, you use the `setState` method (for class components) or the function returned by `useState` (for functional components) to update the state.
   - When you call `setState`, React schedules an update to the component's state, triggering a re-render with the updated state. This ensures that changes to the state are reflected in the UI.

3. **Updating State**:
   - In class components, you use `setState` method to update the state. It takes an object as an argument, which represents the partial state you want to update. React then merges this object with the current state.
   - In functional components using hooks, you use the function returned by `useState` to update the state. This function accepts either a new state value or a function that returns the new state based on the previous state.

4. **Async State Updates**:
   - State updates in React are asynchronous, meaning React may batch multiple `setState` calls into a single update for performance reasons.
   - If you need to perform actions based on the updated state, you can pass a callback function to `setState` as a second argument. This function will be executed after the state has been updated and the component re-rendered.

5. **Local to Component**:
   - State is local to the component in which it is defined. It cannot be accessed or modified by other components directly. If you need to share state between components, you can lift the state up to a common ancestor or use React's context API for more complex scenarios.

6. **Rendering based on State**:
   - Components in React re-render automatically whenever their state changes. This means that you can define your component's render method based on the current state, and React will take care of updating the UI accordingly.

Overall, state in React provides a way for components to manage and respond to changes in data, enabling dynamic and interactive user interfaces.

## The `useState` hook
Hooks in React are functions that enable functional components to use state, lifecycle methods, and other React features without writing a class. Prior to the introduction of hooks, functional components in React were stateless and lacked lifecycle methods. Hooks allow functional components to manage state and have access to other React features previously available only in class components.

The `useState` hook is a built-in hook in React that allows you to define state in a functional component. It takes an initial value as a parameter and returns an array with two elements that we can destructure to get:

1. The current state value
2. A function to update the state value

State definition with `useState` commonly follows this pattern:

```jsx
const [stateValue, setStateValue] = useState(initialValue);
```

You can define more than one state in a single functional component in React using the `useState` hook multiple times. Each call to `useState` creates a separate piece of state, allowing you to manage multiple state variables independently within the same component.

Here's an example demonstrating how to define multiple state variables in a functional component:

```jsx
import React, { useState } from 'react';

function MyComponent() {
  // Define multiple state variables using useState hook
  const [count, setCount] = useState(0); // State for count
  const [text, setText] = useState(''); // State for text input

  // Event handler to update count state
  const incrementCount = () => {
    setCount(prevCount => prevCount + 1);
  };

  // Event handler to update text state
  const handleTextChange = (event) => {
    setText(event.target.value);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementCount}>Increment</button>

      <input 
        type="text"
        value={text}
        onChange={handleTextChange}
        placeholder="Type something..."
      />
      <p>Text: {text}</p>
    </div>
  );
}

export default MyComponent;
```

In this example:

- Two state variables, `count` and `text`, are defined using the `useState` hook.
- `count` state is initialized to `0`, and `text` state is initialized to an empty string `''`.
- Two event handlers, `incrementCount` and `handleTextChange`, are defined to update the respective state variables.
- The component renders a button to increment the count, an input field to capture text input, and displays the current count and text.

By using `useState` multiple times, you can manage multiple state variables within the same functional component.

## React's Rerendering Process and Reconciliation Algorithm
### Rerendering Process:

1. **Triggering a Render**:
    1. **State Changes**: When the state of a component changes via `setState()`, React will re-render that component and its children.
      
    2. **Props Changes**: If the props passed to a component change, React will re-render that component with the new props.

    3. **Context Changes**: If the context used by a component changes, React will re-render that component and its descendants that are consuming the context.

    4. **Force Update**: Manually calling `forceUpdate()` on a component will trigger a re-render of that component and its children.

    5. **Parent Re-renders**: If a component's parent re-renders, the component will also re-render, even if its own state or props haven't changed. This can trigger re-renders down the component tree.

2. **Component Rerenders**:
   - When a rerender is triggered, React schedules a rerender of that component.
   - React will call the component's render method or functional component body to generate a new virtual DOM representation of the component.

3. **Virtual DOM Update**:
   - React constructs a new virtual DOM representation based on the updated state and props of the component.
   - This virtual DOM representation is a lightweight, in-memory representation of the actual DOM structure of the component.

4. **Diffing Algorithm**:
   - React compares the new virtual DOM representation with the previous one to identify the differences or changes.
   - It performs a process called reconciliation, also known as the diffing algorithm, to determine the minimal set of changes needed to update the actual DOM.

5. **DOM Update**:
   - React applies the identified changes to the actual DOM, updating only the parts of the UI that have changed.
   - This process optimizes performance by avoiding unnecessary DOM manipulations and minimizing the impact on the browser's rendering engine.

### Reconciliation Algorithm:

1. **Virtual DOM Comparison**:
   - The reconciliation algorithm compares the new virtual DOM representation of a component with the previous one.
   - It performs a deep comparison of the component's virtual DOM subtree to identify additions, removals, or updates.

2. **Keyed Elements**:
   - React uses keys to optimize the reconciliation process, especially when dealing with lists or collections of elements.
   - Keys provide a way to uniquely identify elements within a list, allowing React to efficiently determine which elements have been added, removed, or updated.

3. **Minimal Set of Changes**:
   - The reconciliation algorithm aims to calculate the minimal set of changes needed to update the actual DOM.
   - It prioritizes efficiency by avoiding unnecessary DOM manipulations and only applying the changes necessary to reflect the updated state of the component.

4. **Component Lifecycle Methods**:
   - Throughout the reconciliation process, React may call various component lifecycle methods, such as `componentWillMount`, `componentDidUpdate`, or their functional component equivalents like `useEffect`.
   - These lifecycle methods provide hooks for performing side effects, such as data fetching, subscriptions, or interacting with the DOM, during different stages of the component's lifecycle.

The entire rerendering process, including updating the virtual DOM, performing the reconciliation algorithm, and applying changes to the actual DOM, is handled automatically by React upon a props,state or context change.

For example, when you update the state of a component using `setState()` in a class component or the state updater function returned by `useState()` in a functional component, React takes care of triggering the rerendering process for that component.

React's declarative programming model allows you to define how your UI should look based on the current state, and React takes care of efficiently updating the DOM to reflect those changes. This automatic handling of the rerendering process is one of the key benefits of using React, as it simplifies the development process and optimizes performance by abstracting away the complexities of manual DOM manipulation.

In summary, the rerendering process in React involves updating the virtual DOM representation of a component based on changes in state, props or context and applying the minimal set of changes to the actual DOM using the reconciliation algorithm. This process ensures efficient and optimized updates to the user interface, resulting in a responsive and dynamic user experience.

## React `hook`s
Hooks in React are functions that enable functional components to use state, lifecycle methods, and other React features without writing a class. They were introduced in React version 16.8 to address various concerns with class components, such as code reuse, complexity, and the need for class inheritance.

Here are some key points about hooks in React:

1. **Functional Components**: Prior to the introduction of hooks, functional components in React were stateless and lacked lifecycle methods. Hooks allow functional components to manage state and have access to other React features previously available only in class components.

2. **Built-in Hooks**: React provides several built-in hooks, such as `useState`, `useEffect`, `useContext`, and `useReducer`, among others. Each hook serves a specific purpose:
   - `useState`: Allows functional components to add state variables.
   - `useEffect`: Enables performing side effects in functional components, such as data fetching, subscriptions, or manually changing the DOM.
   - `useContext`: Provides access to React context within functional components.
   - `useReducer`: A more powerful alternative to `useState`, especially for managing complex state logic.
   - And many more.

3. **Custom Hooks**: Developers can create custom hooks to encapsulate reusable logic and share it across multiple components. Custom hooks are regular JavaScript functions whose names start with "use" and may call other hooks internally. They provide a way to abstract complex logic into reusable units, enhancing code readability and maintainability.

4. **Rules of Hooks**:
   - Hooks must be called at the top level of a functional component or another custom hook. They cannot be called conditionally or inside loops, as this would disrupt React's internal state management.
   - Hooks must be called from within React function components or custom hooks. They cannot be called from regular JavaScript functions or class components.

5. **Benefits**:
   - Simplified Component Logic: Hooks allow developers to separate concerns and write more readable and maintainable code by extracting stateful logic from components.
   - Code Reusability: Custom hooks promote code reusability by encapsulating logic that can be shared across multiple components.
   - Improved Performance: Hooks can help optimize performance by enabling more efficient management of component lifecycle and state updates.

In summary, hooks in React are functions that enable functional components to use state and other React features without the need for class components. They provide a more flexible and concise way to write React components, promoting code reuse, readability, and maintainability.

---
### Knowledge Check

**Q1.** What is state?

**A1.** In React, "state" refers to the data structure that represents the internal state of a component. It is essentially an object that holds information that may change over time and influences the rendering of the component. We can think of "state" as a "component’s memory". Understanding state is crucial in React development as it allows components to manage and update their data dynamically, resulting in a reactive user interface.

Here's a detailed explanation of state in React:

1. **Component State**:
   - Each component in React can have its own state, which is independent of other components. This means that changes to the state of one component do not affect the state of another component unless explicitly passed down as props.
   - State is typically initialized in the constructor of a class component or by using the `useState` hook in a functional component. For class components, you use `this.state = { /* initial state */ };` in the constructor. In functional components, you use the `useState` hook like so: `const [state, setState] = useState(initialState);`.

2. **Immutable**:
   - State in React is immutable, meaning it cannot be modified directly. Instead, you use the `setState` method (for class components) or the function returned by `useState` (for functional components) to update the state.
   - When you call `setState`, React schedules an update to the component's state, triggering a re-render with the updated state. This ensures that changes to the state are reflected in the UI.

3. **Updating State**:
   - In class components, you use `setState` method to update the state. It takes an object as an argument, which represents the partial state you want to update. React then merges this object with the current state.
   - In functional components using hooks, you use the function returned by `useState` to update the state. This function accepts either a new state value or a function that returns the new state based on the previous state.

4. **Async State Updates**:
   - State updates in React are asynchronous, meaning React may batch multiple `setState` calls into a single update for performance reasons.
   - If you need to perform actions based on the updated state, you can pass a callback function to `setState` as a second argument. This function will be executed after the state has been updated and the component re-rendered.

5. **Local to Component**:
   - State is local to the component in which it is defined. It cannot be accessed or modified by other components directly. If you need to share state between components, you can lift the state up to a common ancestor or use React's context API for more complex scenarios.

6. **Rendering based on State**:
   - Components in React re-render automatically whenever their state changes. This means that you can define your component's render method based on the current state, and React will take care of updating the UI accordingly.

Overall, state in React provides a way for components to manage and respond to changes in data, enabling dynamic and interactive user interfaces.

**Q2.** What is the useState hook and how to use it?

**A2.** Hooks in React are functions that enable functional components to use state, lifecycle methods, and other React features without writing a class. Prior to the introduction of hooks, functional components in React were stateless and lacked lifecycle methods. Hooks allow functional components to manage state and have access to other React features previously available only in class components.

The `useState` hook is a built-in hook in React that allows you to define state in a functional component. It takes an initial value as a parameter and returns an array with two elements that we can destructure to get:

1. The current state value
2. A function to update the state value

State definition with `useState` commonly follows this pattern:

```jsx
const [stateValue, setStateValue] = useState(initialValue);
```

You can define more than one state in a single functional component in React using the `useState` hook multiple times. Each call to `useState` creates a separate piece of state, allowing you to manage multiple state variables independently within the same component.

Here's an example demonstrating how to define multiple state variables in a functional component:

```jsx
import React, { useState } from 'react';

function MyComponent() {
  // Define multiple state variables using useState hook
  const [count, setCount] = useState(0); // State for count
  const [text, setText] = useState(''); // State for text input

  // Event handler to update count state
  const incrementCount = () => {
    setCount(prevCount => prevCount + 1);
  };

  // Event handler to update text state
  const handleTextChange = (event) => {
    setText(event.target.value);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementCount}>Increment</button>

      <input 
        type="text"
        value={text}
        onChange={handleTextChange}
        placeholder="Type something..."
      />
      <p>Text: {text}</p>
    </div>
  );
}

export default MyComponent;
```

In this example:

- Two state variables, `count` and `text`, are defined using the `useState` hook.
- `count` state is initialized to `0`, and `text` state is initialized to an empty string `''`.
- Two event handlers, `incrementCount` and `handleTextChange`, are defined to update the respective state variables.
- The component renders a button to increment the count, an input field to capture text input, and displays the current count and text.

By using `useState` multiple times, you can manage multiple state variables within the same functional component.

**Q3.** What happens to the component when one of its state is changed?

**A3.** When the state of a component changes via `setState()`, it triggers the rerendering process of that component and its children.

When a rerender is triggered, React schedules a rerender of that component.
React will call the component's render method or functional component body to generate a new virtual DOM representation of the component.

React constructs a new virtual DOM representation based on the updated state and props of the component.
This virtual DOM representation is a lightweight, in-memory representation of the actual DOM structure of the component.

React compares the new virtual DOM representation with the previous one to identify the differences or changes.
It performs a process called reconciliation, also known as the diffing algorithm, to determine the minimal set of changes needed to update the actual DOM.

React applies the identified changes to the actual DOM, updating only the parts of the UI that have changed.
This process optimizes performance by avoiding unnecessary DOM manipulations and minimizing the impact on the browser's rendering engine.

It should be noted that a rerender is triggered when:

  1. **State Changes**: When the state of a component changes via `setState()`, React will re-render that component and its children.
    
  2. **Props Changes**: If the props passed to a component change, React will re-render that component with the new props.

  3. **Context Changes**: If the context used by a component changes, React will re-render that component and its descendants that are consuming the context.

  4. **Force Update**: Manually calling `forceUpdate()` on a component will trigger a re-render of that component and its children.

  5. **Parent Re-renders**: If a component's parent re-renders, the component will also re-render, even if its own state or props haven't changed. This can trigger re-renders down the component tree.

**Q4.** What are some of the rules of hooks?

**A4.** Hooks in React are functions that enable functional components to use state, lifecycle methods, and other React features without writing a class. They were introduced in React version 16.8 to address various concerns with class components, such as code reuse, complexity, and the need for class inheritance.

Here are some key points about hooks in React:

1. **Functional Components**: Prior to the introduction of hooks, functional components in React were stateless and lacked lifecycle methods. Hooks allow functional components to manage state and have access to other React features previously available only in class components.

2. **Built-in Hooks**: React provides several built-in hooks, such as `useState`, `useEffect`, `useContext`, and `useReducer`, among others. Each hook serves a specific purpose:
   - `useState`: Allows functional components to add state variables.
   - `useEffect`: Enables performing side effects in functional components, such as data fetching, subscriptions, or manually changing the DOM.
   - `useContext`: Provides access to React context within functional components.
   - `useReducer`: A more powerful alternative to `useState`, especially for managing complex state logic.
   - And many more.

3. **Custom Hooks**: Developers can create custom hooks to encapsulate reusable logic and share it across multiple components. Custom hooks are regular JavaScript functions whose names start with "use" and may call other hooks internally. They provide a way to abstract complex logic into reusable units, enhancing code readability and maintainability.

4. **Rules of Hooks**:
   - Hooks must be called at the top level of a functional component or another custom hook. They cannot be called conditionally or inside loops, as this would disrupt React's internal state management.
   - Hooks must be called from within React function components or custom hooks. They cannot be called from regular JavaScript functions or class components.

5. **Benefits**:
   - Simplified Component Logic: Hooks allow developers to separate concerns and write more readable and maintainable code by extracting stateful logic from components.
   - Code Reusability: Custom hooks promote code reusability by encapsulating logic that can be shared across multiple components.
   - Improved Performance: Hooks can help optimize performance by enabling more efficient management of component lifecycle and state updates.

In summary, hooks in React are functions that enable functional components to use state and other React features without the need for class components. They provide a more flexible and concise way to write React components, promoting code reuse, readability, and maintainability.