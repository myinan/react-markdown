# Passing Data Between Components
## `props`
In React, data is transferred from parent components to child components via `props`. This data transfer is unidirectional, meaning it flows in only one direction. Any changes made to this data will only affect child components using the data, and not parent or sibling components. This restriction on the flow of data gives us more explicit control over it, resulting in fewer errors in our application.

In React, the `props` (short for properties) argument is a fundamental concept used to pass data from a parent component to a child component. It's essentially an object that contains various pieces of information or parameters that a component might need to render or behave in a certain way. When you define a component in React, you can pass data to it by setting attributes on the component tag. These attributes then become properties (props) of the component.

Here's a breakdown of the `props` argument and why it's important:

1. **Passing Data**: One of the primary purposes of `props` is to pass data from a parent component down to its child components. This allows for a hierarchical structure of components where data flows top-down from parent to child.

2. **Customization and Reusability**: Props enable you to customize the behavior or appearance of a component based on the data passed to it. This makes components highly reusable because they can be configured differently depending on the context in which they are used.

3. **Immutable**: Props are immutable, meaning that they cannot be modified by the component receiving them. This ensures that data passed from a parent remains unchanged by child components, which helps in maintaining a predictable flow of data and preventing unexpected side effects.

4. **Functional Components**: In functional components (also known as stateless functional components), props are received as a single argument, typically named `props`. This argument is an object containing all the properties passed to the component. Functional components rely solely on props and do not have their own internal state.

5. **Accessing Props**: Within a functional component, you access props directly as properties of the `props` object. For example, if you pass a prop named `name` to a component, you can access it within the component using `props.name`.

6. **Dynamic Rendering**: Props allow you to dynamically render components and data based on changing conditions or user interactions. By passing different props, you can control what content is displayed or how a component behaves under different circumstances.

7. **Event Handling**: Props can also be used to pass down event handlers from parent components to child components. This enables child components to interact with the parent or trigger actions based on user input.

Overall, the `props` argument is a fundamental mechanism in React for passing data between components and configuring their behavior. It promotes a modular and reusable approach to building user interfaces, where components can be composed and combined to create complex UIs while maintaining a clear and predictable data flow.

```jsx
function Button(props) {
  const buttonStyle = {
    color: props.color,
    fontSize: props.fontSize + 'px'
  };

  return (
    <button style={buttonStyle}>{props.text}</button>
  );
}

export default function App() {
  return (
    <div>
      <Button text="Click Me!" color="blue" fontSize={12} />
      <Button text="Don't Click Me!" color="red" fontSize={12} />
      <Button text="Click Me!" color="blue" fontSize={20} />
    </div>
  );
}
```

## `prop` Destructuring
A very common pattern you will come across in React is prop destructuring. Unpacking your props in the component arguments allows for more concise and readable code.

```jsx
function Button({ text, color, fontSize }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return <button style={buttonStyle}>{text}</button>;
}

export default function App() {
  return (
    <div>
      <Button text="Click Me!" color="blue" fontSize={12} />
      <Button text="Don't Click Me!" color="red" fontSize={12} />
      <Button text="Click Me!" color="blue" fontSize={20} />
    </div>
  );
}
```

## Default props
You may have noticed in the above examples that there is some repetition when defining props on the Button components within App. In order to stop repeating ourselves re-defining these common values, and to protect our application from undefined values, we can define default props that will be used by the component in the absence of supplied values.

```jsx
function Button({ text, color, fontSize }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return <button style={buttonStyle}>{text}</button>;
}

Button.defaultProps = {
  text: "Click Me!",
  color: "blue",
  fontSize: 12
};

export default function App() {
  return (
    <div>
      <Button />
      <Button text="Don't Click Me!" color="red" />
      <Button fontSize={20} />
    </div>
  );
}
```

As you can see, we now only need to supply prop values to Button when rendering within App if they differ from the default values defined on Button.defaultProps.

You can also combine default props and prop destructuring. Here’s how it looks in action.

```jsx
function Button({ text = "Click Me!", color = "blue", fontSize = 12 }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return <button style={buttonStyle}>{text}</button>;
}
```

## Functions as props
In addition to passing variables through to child components as props, you can also pass through functions.

```jsx
function Button({ text = "Click Me!", color = "blue", fontSize = 12, handleClick }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return (
    <button onClick={handleClick} style={buttonStyle}>
      {text}
    </button>
  );
}

export default function App() {
  const handleButtonClick = () => {
    window.location.href = "http://www.google.com";
  };

  return (
    <div>
      <Button handleClick={handleButtonClick} />
    </div>
  );
}
```

---
### Knowledge Check

**Q1.** How does data flow between React components? From child to parent? From parent to child? Both?

**A1.** In React, data can flow between components in multiple directions: from parent to child, from child to parent, and even among siblings or unrelated components using various techniques. React's component architecture follows a unidirectional data flow, which means data typically flows from parent components to child components.

1. **From Parent to Child (Prop Passing)**:
   - The most common way to pass data from a parent component to a child component is through props (short for properties).
   - The parent component can pass data down to a child component by specifying attributes on the child component's JSX tag. These attributes are then accessible as props within the child component.
   - Example:
     ```jsx
     // ParentComponent.js
     import ChildComponent from './ChildComponent';

     function ParentComponent() {
       const data = "Hello from parent";
       return <ChildComponent data={data} />;
     }

     // ChildComponent.js
     function ChildComponent(props) {
       return <div>{props.data}</div>;
     }
     ```

2. **From Child to Parent (Callback Functions)**:
   - Data can be passed from a child component to a parent component using callback functions.
   - The parent component passes a function down to the child component via props. The child component then calls this function, passing data back to the parent.
   - Example:
     ```jsx
     // ParentComponent.js
     import ChildComponent from './ChildComponent';

     function ParentComponent() {
       function handleData(data) {
         console.log(data); // Data received from child
       }

       return <ChildComponent sendDataToParent={handleData} />;
     }

     // ChildComponent.js
     function ChildComponent(props) {
       function handleClick() {
         const data = "Hello from child";
         props.sendDataToParent(data);
       }

       return <button onClick={handleClick}>Send Data</button>;
     }
     ```

3. **Sibling Components or Unrelated Components**:
   - To pass data between sibling components or unrelated components that do not have a direct parent-child relationship, you can lift the state up to a common ancestor and then pass down the data through props or utilize context API or state management libraries like Redux or MobX.

4. **Using Context API**:
   - React's Context API allows you to create a global state that can be accessed by any component in the component tree, regardless of their nesting level. Components can both read from and write to this global state.
   - This can be particularly useful when passing data between components that are deeply nested or have no direct parent-child relationship.

In summary, while the primary direction of data flow in React is from parent to child via props, React provides mechanisms for bidirectional communication between components and for managing shared state in complex applications.

**Q2.** Why do we use props in React?

**A2.** Props, short for properties, are a fundamental concept in React and are used for several reasons:

1. **Passing Data from Parent to Child Components**: Props allow data to be passed from a parent component to a child component. This enables the parent component to communicate information to its children, allowing child components to render dynamically based on the data they receive.

2. **Reusability**: By using props, components can be made reusable. A component can accept different sets of props, allowing it to render differently based on the data it receives. This reusability promotes a modular and efficient development workflow, as components can be composed and reused across different parts of an application.

3. **Dynamic Rendering**: Props enable components to render dynamically based on changing data. As props can be updated by the parent component, child components can react to these changes and update their rendering accordingly. This allows for dynamic and interactive user interfaces.

4. **Component Configuration**: Props can be used to configure the behavior and appearance of components. By passing different props to a component, you can customize its rendering, styling, or functionality without having to modify the component itself. This promotes a flexible and configurable component architecture.

5. **Data Flow**: Props facilitate the unidirectional flow of data in React applications. Data typically flows from parent components to child components via props, following React's principle of downward data flow. This promotes a clear and predictable data flow, making it easier to reason about the state and behavior of components.

Overall, props are a fundamental mechanism in React for passing data, configuring components, promoting reusability, and facilitating the dynamic rendering of user interfaces. They play a crucial role in building modular, maintainable, and scalable React applications.

**Q3.** How do we define default properties on a React component? What are some benefits in doing so?

**A3.** We can define default props that will be used by the component in the absence of supplied values.

```jsx
function Button({ text, color, fontSize }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return <button style={buttonStyle}>{text}</button>;
}

Button.defaultProps = {
  text: "Click Me!",
  color: "blue",
  fontSize: 12
};

export default function App() {
  return (
    <div>
      <Button />
      <Button text="Don't Click Me!" color="red" />
      <Button fontSize={20} />
    </div>
  );
}
```

As you can see, we now only need to supply prop values to Button when rendering within App if they differ from the default values defined on Button.defaultProps.

You can also combine default props and prop destructuring. Here’s how it looks in action.

```jsx
function Button({ text = "Click Me!", color = "blue", fontSize = 12 }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return <button style={buttonStyle}>{text}</button>;
}
```

Benefits of defining default props in React components include:

1. **Robustness**: Default props provide a fallback mechanism, ensuring that components have sensible default values for props even if the parent component does not provide them explicitly. This makes components more robust and less prone to errors due to missing props.

2. **Flexibility**: Default props allow components to be used with minimal configuration. Users of the component can choose to override the default props with their own values when necessary, but they are not required to provide every prop, reducing the boilerplate required to use the component.

3. **Documentation**: Default props serve as documentation for the component, indicating what props the component expects and providing default values for each prop. This makes it easier for other developers to understand how to use the component and what props are available.

4. **Testing**: Default props simplify testing by providing a predictable baseline for component behavior. When writing unit tests for components, you can rely on default props to ensure consistent behavior in the absence of explicit prop values.

**Q4.** How can we pass functions as props?

**A4.** We can pass any JavaScript value through props, including objects, arrays, and functions.

```jsx
function Button({ text = "Click Me!", color = "blue", fontSize = 12, handleClick }) {
  const buttonStyle = {
    color: color,
    fontSize: fontSize + "px"
  };

  return (
    <button onClick={handleClick} style={buttonStyle}>
      {text}
    </button>
  );
}

export default function App() {
  const handleButtonClick = () => {
    window.location.href = "http://www.google.com";
  };

  return (
    <div>
      <Button handleClick={handleButtonClick} />
    </div>
  );
}
```