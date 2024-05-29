# React Components
React components are the building blocks of React applications, which are JavaScript-based user interface elements responsible for rendering UI elements to the browser. They encapsulate the logic and presentation of a part of the UI, making it easier to manage and reuse code.

Here's a detailed breakdown of React components:

1. **Component Types**:
   - **Functional Components**: These are simple JavaScript functions that take props (short for properties) as arguments and return React elements. They are also known as stateless components because they don't have their own state.
   - **Class Components**: These are ES6 classes that extend from React.Component. They can hold their own state and have access to React lifecycle methods.

2. **Props**:
   - Props are inputs to a React component. They are passed from parent to child components and are immutable within the child component.
   - Props allow components to be customizable and reusable, as they can be dynamically changed based on the parent's state or other external factors.

3. **State**:
   - State represents the internal data of a component. It's mutable and can be changed over time in response to user actions, network responses, or other events.
   - Class components have state, while functional components can use the `useState` hook to introduce state management.

4. **Lifecycle Methods** (Class Components Only):
   - Lifecycle methods are special methods that are invoked at specific points in a component's life cycle, such as when it is mounted, updated, or unmounted.
   - Common lifecycle methods include `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`. They allow developers to execute code at different stages of a component's life cycle.

5. **Rendering**:
   - Components are responsible for rendering UI elements. They return JSX (JavaScript XML), which is a syntax extension for JavaScript that resembles HTML.
   - JSX allows developers to write HTML-like code within JavaScript, making it easier to visualize and manipulate the UI structure.

6. **Composition**:
   - React components can be composed together to create complex UIs. Parent components can include child components, which can further include other child components, forming a tree-like structure.
   - This compositional approach promotes code reusability and maintainability, as components can be easily reused across different parts of the application.

7. **Component Communication**:
   - Components can communicate with each other through props and callbacks. Parent components can pass data and functions as props to child components, enabling them to interact with each other.
   - Additionally, React provides a mechanism called "context" for sharing data across multiple components without explicitly passing props through every level of the component tree.

8. **Virtual DOM**:
   - React uses a virtual DOM to optimize the rendering process. Instead of directly updating the actual DOM for every state change, React updates a lightweight copy of the DOM (virtual DOM) and then efficiently reconciles the changes with the actual DOM.
   - This approach minimizes DOM manipulation, resulting in improved performance and a smoother user experience.

In summary, React components are modular, reusable, and composable building blocks that encapsulate the logic and presentation of UI elements in React applications. They facilitate the development of scalable and maintainable user interfaces by promoting code organization, reusability, and efficient rendering.

## React Functional Component Syntax

```jsx
// FavoriteFoods.jsx

export default function Greeting() {
  return <h1>Favorite Food:</h1>;
}

function FirstFood() {
  return <h2>Apple</h2>;
}

function SecondFood() {
  return <h2>Banana</h2>;
}

export { FirstFood, SecondFood };
```

```jsx
// main.jsx

import React from "react";
import ReactDOM from "react-dom/client";
import Greeting, { FirstFood, SecondFood } from "./Greeting.jsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <Greeting />
    <FirstFood />
    <SecondFood />
  </React.StrictMode>
);
```

**Note: Pay attention that exported/imported functions are capitalized. When the JSX is parsed, React uses the capitalization to tell the difference between an HTML tag and an instance of a React component. For example, \<greeting /> would be interpreted as a normal HTML element with no special meaning, instead of a React component.**

---
### Knowledge Check

**Q1.** What does a React element look like?

**A1.** In React, an element is the smallest building block while the component is a reusable piece of the code. The element contains the information to be rendered on the UI and the Components are composed of the elements.

A React element is the basic building block in a react application, it is an object representation of a virtual DOM node. React Element contains both type and property. It may contain other Elements in its props. React Element does not have any methods, making it light and faster to render than components.

Syntax:

```jsx
const ele1 =<h1> Hello, World. </h1>;
// OR
const ele1 = React.createElement('h1', {props}, "Hello, World.")
```

Example1: The following piece of code creates an element ele1 in the JSX format and render it on the UI.

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
 
const ele1 = <h1>Welcome</h1>;
 
// Simple javaScript to get the div with id ="root"
ReactDOM.render(ele1, document.getElementById("root"));
```

Example2: The following piece of code uses the react method to create element in the javascript format and render it on the UI.

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
// Without JSX
const ele1 = React.createElement(
    'h1',
    { id: 'header' },
    'Hey Geek'
);
ReactDOM.render(ele1, document.getElementById('root'));
```

**Q2.** How would you create a functional component?

**A2.** To create a functional component in React, you simply define a JavaScript function that returns JSX (JavaScript XML), which describes what should be rendered on the screen. Here's an example of a basic functional component:

```javascript
import React from 'react';

// Functional component named MyComponent
function MyComponent() {
  return (
    <div>
      <h1>Hello, I'm a functional component!</h1>
      <p>I don't have any state or lifecycle methods.</p>
    </div>
  );
}

export default MyComponent; // Export the component to be used elsewhere
```

In this example:

- We import React at the beginning because JSX syntax requires it.
- We define a function called `MyComponent` which returns JSX. This JSX describes the structure of the component we want to render.
- We export the component using `export default` so that it can be imported and used in other parts of our application.

Functional components are stateless by default, but with React Hooks introduced in React 16.8, functional components can now have state and other features previously available only in class components. However, the example above demonstrates a basic functional component without state or lifecycle methods.

**Q3.** How do you export and then import a component?

**A3.** Exporting and importing components in React is straightforward. Here's how you do it:

1. **Exporting a Component:**
   To export a component, you use the `export` keyword. You can either export the component directly after its declaration or export it later.

   ```javascript
   // MyComponent.js
   import React from 'react';

   function MyComponent() {
     return <h1>Hello, I'm a component!</h1>;
   }

   export default MyComponent; // Exporting MyComponent as the default export
   ```

   In this example, `MyComponent` is exported as the default export, meaning that when you import it, you can choose any name for it.

2. **Importing a Component:**
   To import a component into another file, you use the `import` statement followed by the name you want to give the imported component and the path to the file where it's exported.

   ```javascript
   // App.js
   import React from 'react';
   import MyComponent from './MyComponent'; // Importing MyComponent

   function App() {
     return (
       <div>
         <h1>This is my app</h1>
         <MyComponent /> {/* Using MyComponent */}
       </div>
     );
   }

   export default App;
   ```

   In this example, `MyComponent` is imported from the file `'./MyComponent'`. Since it's the default export, we can give it any name when importing. Here, we've chosen to call it `MyComponent`.

That's it! You've exported a component from one file and imported it into another, making it available for use within your application.