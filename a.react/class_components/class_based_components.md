# Class Based Components
All the components we have built so far have been functional in style and syntax. This is common now, but you will see a different `class` based syntax too. In this lesson, we explore how a class-based component is written and how concepts like props and state are used in one.

## Historical React component patterns
In your previous lessons, you have already been introduced to functional components, and the basic patterns in which components get written nowadays. However, React components did not look this way when React was introduced.

If you look into any older React codebase, you’ll notice a lot of classes. These are known as class-based components. Prior to February 2019, functional components were also called state-less, as there was no way to manage state in them. This was changed when hooks were introduced, leading to less verbose and ‘neater’ components.

In your career, chances are, you will be dealing with legacy code, so there will be days where you would be dealing with class components. Let’s peek into the intricacies of a class-based component, and how they work.

## Building a class component
As we already know about functional components, let us build a class-based component from a functional one. Usually, you will want to divide the contents of a component, like the one we use, into smaller, reusable components, but for the purposes of this exercise, we stick to one component. Below, we have a sample functional component:

```jsx
import React, { useState } from "react";

const FunctionalInput = ({ name }) => {
  const [todos, setTodos] = useState(["Just some demo tasks", "As an example"]);
  const [inputVal, setInputVal] = useState("");

  const handleInputChange = (e) => {
    setInputVal(e.target.value);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    setTodos((todo) => [...todo, inputVal]);
    setInputVal("");
  };

  return (
    <section>
      <h3>{name}</h3>
      <form onSubmit={handleSubmit}>
        <label htmlFor="task-entry">Enter a task: </label>
        <input
          type="text"
          name="task-entry"
          value={inputVal}
          onChange={handleInputChange}
        />
        <button type="submit">Submit</button>
      </form>
      <h4>All the tasks!</h4>
      <ul>
        {todos.map((todo) => (
          <li key={todo}>{todo}</li>
        ))}
      </ul>
    </section>
  );
};

export default FunctionalInput;
```

### The start of a class-based component
Now, let’s try to recreate it as a class-based component. The first thing it should have is, not surprisingly, a class! But it cannot be just another class, it will need to have certain properties that qualifies it as a React component. React provides us with all those properties on a class called `Component`, and we can write our components by extending the given class, as shown below:

```jsx
import React, { Component } from "react";

class ClassInput extends Component {
  // Some code goes here
}

/*
  This can also be written as:

  import React from 'react';
  class ClassInput extends React.Component {}
  export default ClassInput;

  instead of destructuring the `Component` during import
*/

export default ClassInput;
```

### The use of a constructor and props
A class is generally incomplete without a constructor, so let’s add one.

The props, that get passed into this component, gets passed into the class’s `constructor`. This, along with the `super` method, allows you to use the props in context to `this`, which, in *this* case, refers to the component.

If your component doesn’t have any props, it is fine to leave the `constructor` and the `super` with no arguments.

```jsx
import React, { Component } from "react";

class ClassInput extends Component {
  constructor(props) {
    super(props);
  }
  // Some more code goes here
}

export default ClassInput;
```

### How you can render JSX
Now that the props can be accessed inside of the class component, the next issue is to find a way to render the JSX.

Well, you can do that by returning your JSX from a `render` method! You can use the props that you declared in the constructor too!

```jsx
import React, { Component } from "react";

class ClassInput extends Component {
  constructor(props) {
    super(props);
  }
  // Some more code goes here

  render() {
    return (
      <section>
        <h3>{this.props.name}</h3>
        {/* The input field to enter To-Do's */}
        <form>
          <label htmlFor="task-entry">Enter a task: </label>
          <input type="text" name="task-entry" />
          <button type="submit">Submit</button>
        </form>
        <h4>All the tasks!</h4>
        {/* The list of all the To-Do's, displayed */}
        <ul></ul>
      </section>
    );
  }
}

export default ClassInput;
```

### How to use state and manage context
Next comes the state. In a class-based component, the state gets initialized as a part of the constructor.

```jsx
import React, { Component } from "react";

class ClassInput extends Component {
  constructor(props) {
    super(props);

    this.state = {
      todos: [],
      inputVal: "",
    };
  }
  // Some more code goes here

  render() {
    return (
      <section>
        <h3>{this.props.name}</h3>
        <form>
          <label htmlFor="task-entry">Enter a task: </label>
          <input type="text" name="task-entry" />
          <button type="submit">Submit</button>
        </form>
        <h4>All the tasks!</h4>
        <ul></ul>
      </section>
    );
  }
}

export default ClassInput;
```

The pre-defined `setState` method can be used to set it again! Remember, state must not be mutated, so a new state must be set, every time.

Now, it is time to finish it off by adding all the functionality! It is nearly the same, except for a single difference. Whenever a method is declared, you must `bind` the `this` of the method to that of the class in order to work with it, as by default, the methods in a class are not bound to it. Usually, you do this inside the constructor and not at runtime [in the render method].

```jsx
import React, { Component } from "react";

class ClassInput extends Component {
  constructor(props) {
    super(props);

    this.state = {
      todos: [],
      inputVal: "",
    };

    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleInputChange(e) {
    this.setState((state) => ({
      ...state,
      inputVal: e.target.value,
    }));
  }

  handleSubmit(e) {
    e.preventDefault();
    this.setState((state) => ({
      todos: state.todos.concat(state.inputVal),
      inputVal: "",
    }));
  }

  render() {
    return (
      <section>
        <h3>{this.props.name}</h3>
        <form onSubmit={this.handleSubmit}>
          <label htmlFor="task-entry">Enter a task: </label>
          <input
            type="text"
            name="task-entry"
            value={this.state.inputVal}
            onChange={this.handleInputChange}
          />
          <button type="submit">Submit</button>
        </form>
        <h4>All the tasks!</h4>
        <ul>
          {this.state.todos.map((todo) => (
            <li key={todo}>{todo}</li>
          ))}
        </ul>
      </section>
    );
  }
}

export default ClassInput;
```

And there we go, we have successfully made our first class-based component, as easy as that!

---
### Knowledge Check

**Q1.** How do props get used in a class-based component?

**A1.** 

```javascript
import React, { Component } from "react";

class MyComponent extends Component {
  constructor(props) {
    super(props);
    // Now, props are accessible within the constructor
    // You can use props to initialize state or perform other setup operations
    // For example, let's initialize state with a prop value
    this.state = {
      inputValue: props.defaultValue || '', // Initialize inputValue with defaultValue prop or an empty string if defaultValue is not provided
    };
  }

  // Some more code goes here

  render() {
    // Render method code
  }
}

export default ClassInput;
```

In the constructor:

- The `props` parameter is passed to the constructor function.
- `super(props)` calls the constructor of the parent class (`Component`) and passes the props to it. This is necessary to ensure that the component is properly initialized and can access its props correctly.
- You can then use `this.props` within the constructor to access the props passed to the component. In this example, we're initializing the component's state (`inputValue`) with a value from the props (`defaultValue`), or an empty string if `defaultValue` is not provided.

In a class-based component in React, props are accessed through the `this.props` object within the component's methods. Here's how you typically use props in a class-based component:

```javascript
import React, { Component } from 'react';

class MyComponent extends Component {
  render() {
    // Accessing props in render method
    const { prop1, prop2 } = this.props;

    return (
      <div>
        <p>Prop 1: {prop1}</p>
        <p>Prop 2: {prop2}</p>
      </div>
    );
  }
}

export default MyComponent;
```

In the example above:

- `this.props` refers to an object containing all the props passed to the component.
- You can destructure specific props if needed, as shown with `const { prop1, prop2 } = this.props;`.
- Props can be accessed anywhere within the class-based component's methods, not just in the `render` method.

When you use this component elsewhere in your application, you would pass props to it like this:

```javascript
<MyComponent prop1="Value 1" prop2="Value 2" />
```

And those props will be accessible within `MyComponent` as `this.props.prop1` and `this.props.prop2`.

**Q2.** How does JSX get displayed?

**A2.** Now that the props can be accessed inside of the class component, the next issue is to find a way to render the JSX. In a class component in React, you typically define a render() method to return JSX, which will be rendered by React. Inside the render method, you can use the props that you declared in the constructor too.

```javascript
import React, { Component } from 'react';

class MyComponent extends Component {
  render() {
    return (
      <div>
        <h1>Hello, World!</h1>
        <p>This is a class component.</p>
      </div>
    );
  }
}

export default MyComponent;
```

In this example:

- The `MyComponent` class extends `Component` from React.
- Inside the class, a `render()` method is defined, which returns JSX.
- This JSX will be rendered by React when the component is used in your application.

You can then use this component in your application like this:

```javascript
import React from 'react';
import MyComponent from './MyComponent';

function App() {
  return (
    <div>
      <MyComponent />
    </div>
  );
}

export default App;
```

When React encounters `<MyComponent />` in the `App` component, it will render the JSX returned by the `render()` method of the `MyComponent` class.

**Q3.** How do we deal with state in a class-based component?

**A3.** In React, state is declared and used in class-based components using the `state` property. In a class-based component, the state gets initialized as a part of the constructor. Here's a basic example of how you can declare and use state in a class-based component:

```javascript
import React, { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    // Initialize state
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
        <button onClick={this.decrement}>Decrement</button>
      </div>
    );
  }

  // Methods to update state
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }

  decrement = () => {
    this.setState({ count: this.state.count - 1 });
  }
}

export default Counter;
```

In this example:

- We have a class component `Counter` that extends `Component`.
- In the constructor, we initialize the component's state using `this.state`.
- In the `render` method, we access the state using `this.state.count` to display the current count.
- We have two methods, `increment` and `decrement`, which update the state using `this.setState`. The setState() method is defined in the Component class. It's crucial to use `setState` to update the state, as it triggers React's re-rendering mechanism.
- We attach event handlers to buttons to call these methods when clicked.

In class-based components, the `state` property is a single JavaScript object that contains all the state variables you want to manage within that component.

**Q4.** How do you restore the context of `this` in a method of a class based React component? 

**A4.** Whenever a method is declared, you must `bind` the `this` of the method to that of the class in order to work with it, as by default, the methods in a class are not bound to it. Usually, you do this inside the constructor and not at runtime [in the render method].

```jsx
import React, { Component } from "react";

class ClassInput extends Component {
  constructor(props) {
    super(props);

    this.state = {
      todos: [],
      inputVal: "",
    };

    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleInputChange(e) {
    this.setState((state) => ({
      ...state,
      inputVal: e.target.value,
    }));
  }

  handleSubmit(e) {
    e.preventDefault();
    this.setState((state) => ({
      todos: state.todos.concat(state.inputVal),
      inputVal: "",
    }));
  }

  render() {
    return (
      <section>
        <h3>{this.props.name}</h3>
        <form onSubmit={this.handleSubmit}>
          <label htmlFor="task-entry">Enter a task: </label>
          <input
            type="text"
            name="task-entry"
            value={this.state.inputVal}
            onChange={this.handleInputChange}
          />
          <button type="submit">Submit</button>
        </form>
        <h4>All the tasks!</h4>
        <ul>
          {this.state.todos.map((todo) => (
            <li key={todo}>{todo}</li>
          ))}
        </ul>
      </section>
    );
  }
}

export default ClassInput;
```

In JavaScript, the value of `this` is determined by how a function is called. When a method is called directly, such as `this.handleInputChange()`, `this` refers to the object on which the method was called. However, when you pass a method as a callback, such as `onChange={this.handleInputChange}`, the context of `this` can become ambiguous, leading to errors or unexpected behavior.

In React class components, if you define a method like `handleInputChange` and pass it as a callback without binding `this` to the method within the constructor, the `this` inside `handleInputChange` would refer to the context of the event caller, not the component instance. This would lead to errors because `setState` would not be defined in the context of the component.

Therefore, to ensure that `this` refers to the component instance within all methods, it's a common practice to bind the methods to the component instance in the constructor. This ensures that `this` within the methods always refers to the component instance, allowing you to access properties like `state` and `props`, as well as call other methods on the component.

In the provided code snippet, `this.handleInputChange = this.handleInputChange.bind(this);` and `this.handleSubmit = this.handleSubmit.bind(this);` are binding the `this` context of `handleInputChange` and `handleSubmit` methods to the `ClassInput` component instance. This ensures that when these methods are called, they have access to the correct `this` context, including the component's state and props.