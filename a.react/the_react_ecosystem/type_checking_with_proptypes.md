# Type Checking With PropTypes
Type checking is a process in programming where the types of values passed to functions or assigned to variables are verified to ensure they match the expected types. This helps catch errors early in the development process, reducing bugs and improving code reliability.

In React, PropTypes is a built-in type checking mechanism used to validate the types of props passed to components. PropTypes provide a way to define the expected types for the props a component expects to receive. By defining PropTypes for a component, you can ensure that the props passed to it conform to the expected types, helping to catch bugs and document component usage.

### Using propTypes
To start using PropTypes in our React projects, we first need to install the corresponding library. We can do that with npm. In your React project run the following command:

```jsx
npm install --save prop-types
```

Next, we want to import the PropTypes package in the component whose props we want to validate.

```jsx
import PropTypes from 'prop-types';
```

Here is a very basic example of how we would use it in a component that renders out a name prop.

```jsx
import PropTypes from 'prop-types';
import React from 'react';

const RenderName = (props) => {
  return <div>{props.name}</div>;
};

RenderName.propTypes = {
  name: PropTypes.string,
};

export default RenderName;
```

In this example, the component RenderName expects to receive a prop called name which is a string. If this prop is not a string, a warning will be displayed. If you want to make sure a prop is being passed in, use isRequired like so:

```jsx
RenderName.propTypes = {
  name: PropTypes.string.isRequired,
}
```

### Using defaultProps
Another cool thing we can do in combination with PropTypes is passing in default props:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

const RenderName = (props) => {
  return <div>{props.name}</div>;
};

RenderName.propTypes = {
  name: PropTypes.string,
};

RenderName.defaultProps = {
  name: 'Zach',
};

export default RenderName;
```

In this example, with the help of the `defaultProps` property we are defining a default value for the `name` prop. This way, if the `RenderName` component is called without passing in the `name` prop, it will default to “Zach”. When you do pass in props, they will take precedence over the default props.

### Validators of propTypes
Here is an example documenting the different validators provided by propTypes:

```jsx
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // You can declare that a prop is a specific JS type. By default, these
  // are all optional.
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // Anything that can be rendered: numbers, strings, elements or an array
  // (or fragment) containing these types.
  optionalNode: PropTypes.node,

  // A React element.
  optionalElement: PropTypes.element,

  // A React element type (ie. MyComponent).
  optionalElementType: PropTypes.elementType,

  // You can also declare that a prop is an instance of a class. This uses
  // JS's instanceof operator.
  optionalMessage: PropTypes.instanceOf(Message),

  // You can ensure that your prop is limited to specific values by treating
  // it as an enum.
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // An object that could be one of many types
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // An array of a certain type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // An object with property values of a certain type
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // An object taking on a particular shape
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),   

  // You can chain any of the above with `isRequired` to make sure a warning
  // is shown if the prop isn't provided.
  requiredFunc: PropTypes.func.isRequired,

  // A required value of any data type
  requiredAny: PropTypes.any.isRequired,

  // You can also specify a custom validator. It should return an Error
  // object if the validation fails. Don't `console.warn` or throw, as this
  // won't work inside `oneOfType`.
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // You can also supply a custom validator to `arrayOf` and `objectOf`.
  // It should return an Error object if the validation fails. The validator
  // will be called for each key in the array or object. The first two
  // arguments of the validator are the array or object itself, and the
  // current item's key.
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

### What about TypeScript?
Now is also a good time to mention TypeScript - a strongly typed language that builds on JavaScript. TypeScript is an open-source programming language developed and maintained by Microsoft. It is a statically typed superset of JavaScript that compiles down to plain JavaScript code. Here's a detailed breakdown:

1. **Static Typing**: TypeScript introduces static typing to JavaScript, which means that you can explicitly declare the types of variables, function parameters, and return values. This allows for catching type-related errors during development rather than at runtime, which can help in writing more robust and maintainable code.

2. **Superset of JavaScript**: TypeScript is designed to be a superset of JavaScript, meaning that any valid JavaScript code is also valid TypeScript code. This makes it easy for developers already familiar with JavaScript to transition to TypeScript gradually without having to completely rewrite their existing codebase.

3. **Type Annotations**: TypeScript introduces syntax for type annotations, allowing developers to specify the data types of variables, function parameters, and return values. For example:
   ```typescript
   let message: string = "Hello, TypeScript!";
   function greet(name: string): string {
       return "Hello, " + name;
   }
   ```

4. **Interfaces and Type Aliases**: TypeScript supports interfaces and type aliases, which allow developers to define custom data types and structures. Interfaces provide a way to describe the shape of objects, while type aliases allow developers to create aliases for complex types.
   ```typescript
   interface Person {
       name: string;
       age: number;
   }
   type Point = { x: number; y: number };
   ```

5. **Enums**: TypeScript supports enums, which allow developers to define a set of named constants. Enums are useful for representing a fixed set of related values.
   ```typescript
   enum Color {
       Red,
       Green,
       Blue
   }
   let myColor: Color = Color.Red;
   ```

6. **Class-Based Object-Oriented Programming**: TypeScript supports class-based object-oriented programming, including features such as inheritance, encapsulation, and access modifiers like `public`, `private`, and `protected`.
   ```typescript
   class Animal {
       constructor(public name: string) {}
       move(distanceInMeters: number = 0) {
           console.log(`${this.name} moved ${distanceInMeters}m.`);
       }
   }
   ```

7. **Advanced Features**: TypeScript offers many advanced features such as generics, decorators, intersection types, union types, and more, which enable developers to write sophisticated and expressive code.

8. **Compiler**: TypeScript comes with a compiler (`tsc`) that translates TypeScript code into JavaScript code. This allows developers to write and maintain their code in TypeScript and then compile it into JavaScript that can run in any JavaScript runtime environment.

In summary, TypeScript enhances JavaScript by adding static typing, advanced features, and improved tooling for large-scale application development while still maintaining compatibility with existing JavaScript codebases. It aims to address some of the common pain points of JavaScript development, such as type safety and code maintainability, without sacrificing the dynamic and flexible nature of the language.

Learning TypeScript can be a lot of overhead when you’re already learning React and the best way to prepare for this is to continue developing your JavaScript fundamentals. The TypeScript documentation encourages enhancing JavaScript skills before tackling TypeScript complexities, as highlighted in their discussion on the importance of JavaScript fundamentals. In the future however, if you do decide to go in the direction of learning TypeScript, our recommendation would be picking up a previous project and refactoring the components one by one to TypeScript.

### propTypes vs TypeScript
Typescript and PropTypes serve different purposes. Typescript validates types at **compile time**, whereas PropTypes are checked at **runtime**.

Typescript is useful when you are writing code: it will warn you if you pass an argument of the wrong type to your React components, give you autocomplete for function calls, etc.

PropTypes are useful when you test how the components interact with external data, for example when you load JSON from an API. PropTypes will help you debug (when in React's Development mode) why your component is failing by printing helpful messages like:

```
Warning: Failed prop type: Invalid prop `id` of type `number` supplied to `Table`, expected `string`
```

PropTypes validation does also make sense to validate data structures loaded dymanically (coming from server via AJAX). PropTypes is runtime validation, and thus can really help to debug stuff. As problems will output clear and human-friendly messages.

In summary, while both TypeScript and PropTypes help catch type-related errors, they operate at different stages of development. TypeScript catches errors at compile time, while PropTypes provide runtime validation during React component execution. 

---
### Knowledge Check

**Q1.** How would we set up a basic implementation of PropTypes?

**A1.** 

```jsx
import PropTypes from 'prop-types';
import React from 'react';

const RenderName = (props) => {
  return <div>{props.name}</div>;
};

RenderName.propTypes = {
  name: PropTypes.string,
};

export default RenderName;
```

**Q2.** If we pass in a prop to a component that has a defaultProp defined, what would happen?

**A2.** 

```jsx
import React from 'react';
import PropTypes from 'prop-types';

const RenderName = (props) => {
  return <div>{props.name}</div>;
};

RenderName.propTypes = {
  name: PropTypes.string,
};

RenderName.defaultProps = {
  name: 'Zach',
};

export default RenderName;
```

In this example, with the help of the defaultProps property we are defining a default value for the name prop. This way, if the RenderName component is called without passing in the name prop, it will default to “Zach”. **When you do pass in props, they will take precedence over the default props.**

**Q3.** What is the difference between PropTypes and TypeScript?

**A3.** Typescript and PropTypes serve different purposes. Typescript validates types at **compile time**, whereas PropTypes are checked at **runtime**.

Typescript is useful when you are writing code: it will warn you if you pass an argument of the wrong type to your React components, give you autocomplete for function calls, etc.

PropTypes are useful when you test how the components interact with external data, for example when you load JSON from an API. PropTypes will help you debug (when in React's Development mode) why your component is failing by printing helpful messages like:

```
Warning: Failed prop type: Invalid prop `id` of type `number` supplied to `Table`, expected `string`
```

PropTypes validation does also make sense to validate data structures loaded dymanically (coming from server via AJAX). PropTypes is runtime validation, and thus can really help to debug stuff. As problems will output clear and human-friendly messages.

In summary, while both TypeScript and PropTypes help catch type-related errors, they operate at different stages of development. TypeScript catches errors at compile time, while PropTypes provide runtime validation during React component execution. 