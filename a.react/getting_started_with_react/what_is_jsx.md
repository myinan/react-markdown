# What Is JSX?
JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside a JavaScript file. It’s not required to use JSX when writing React components, but it does make writing them more concise.

Essentially, JSX is syntactic sugar for the React `createElement` function. This function creates a React element, which is a plain object, so JSX compiles down to plain JavaScript objects.

JSX stands for JavaScript XML. It's a syntax extension for JavaScript that allows you to write HTML-like code within your JavaScript files. JSX provides a more readable and concise way to describe the structure of your UI components in React applications.

Here's an example of JSX:

```jsx
import React from 'react';

const element = <h1>Hello, world!</h1>;

function MyComponent() {
  return (
    <div>
      <h1>This is a JSX component</h1>
      <p>And this is a paragraph inside JSX</p>
    </div>
  );
}

export default MyComponent;
```

In JSX:

- You can write HTML-like tags directly in your JavaScript code, which are actually transformed into calls to `React.createElement()`.
- You can use curly braces `{}` to include JavaScript expressions within JSX.
- JSX is not mandatory for writing React applications, but it's a popular and recommended way because it makes the code more readable and easier to understand, especially for those familiar with HTML syntax.

During the build process, JSX code is transformed into regular JavaScript code that React can understand and work with. This transformation is typically done using tools like Babel, which converts JSX into calls to `React.createElement()`, enabling React to create and manipulate the corresponding React elements efficiently.

## Why would we use JSX?
There are several reasons why JSX is widely used in React development:

1. **Readability**: JSX provides a familiar HTML-like syntax, which makes the code more readable and easier to understand, especially for developers familiar with HTML. It enhances the clarity of the UI structure and component hierarchy.

2. **Ease of Use**: Writing JSX feels natural, especially for developers who are accustomed to writing HTML. It simplifies the process of creating and visualizing UI components, reducing the cognitive overhead of switching between different syntaxes.

3. **Expressiveness**: JSX allows you to embed JavaScript expressions directly within the markup, enabling dynamic rendering of components based on variables, functions, or props. This makes it easy to create dynamic and interactive user interfaces.

4. **Tooling Support**: JSX is well-supported by development tools and IDEs, providing features such as syntax highlighting, code completion, and error checking. This helps developers write code more efficiently and catch errors early in the development process.

5. **Component Composition**: JSX facilitates the composition of reusable and composable components, which is a core principle in React development. Components can be easily nested within each other, allowing for a modular and maintainable codebase.

6. **Performance Optimization**: JSX is optimized for performance by tools like Babel, which transform JSX code into efficient JavaScript code that React can work with. This helps minimize the overhead of rendering and updating components, resulting in faster and more responsive applications.

Overall, JSX enhances the developer experience in React development by providing a concise, expressive, and intuitive syntax for building user interfaces. It plays a crucial role in making React one of the most popular and powerful libraries for building modern web applications.

## Rules of JSX
If you were to take some valid HTML and copy it straight into your React component, it would not work. This is due to some of the rules JSX implements, that aren’t present in HTML.

### 1. Return a single root element
If you wish to return multiple elements in a component, you can do so by wrapping them in a parent tag. This can be a \<div>, or, if you don’t want the elements to have a container, you could use a `React fragment`, like so:

```jsx
function App() {
  return (
    <>
      <h1>Example h1</h1>
      <h2>Example h2</h2>
    </>
  );
}
```

`<Fragment> (<>...</>)`: `<Fragment>`, often used via `<>...</>` syntax, lets you group elements without a wrapper node.

```jsx
<>
  <OneChild />
  <AnotherChild />
</>
```

Wrap elements in \<Fragment> to group them together in situations where you need a single element. Grouping elements in Fragment has no effect on the resulting DOM; it is the same as if the elements were not grouped. The empty JSX tag <></> is shorthand for \<Fragment>\</Fragment> in most cases.

Use Fragment, or the equivalent <>...</> syntax, to group multiple elements together. You can use it to put multiple elements in any place where a single element can go. For example, a component can only return one element, but by using a Fragment you can group multiple elements together and then return them as a group:

```jsx
function Post() {
  return (
    <>
      <PostTitle />
      <PostBody />
    </>
  );
}
```

Fragments are useful because grouping elements with a Fragment has no effect on layout or styles, unlike if you wrapped the elements in another container like a DOM element. 

### 2. Close all tags.
In HTML, many tags are self-closing and self-wrapping. In JSX however, we must explicitly close and wrap these tags.

\<input> would become \<input />, and \<li> would become \<li>\</li>:

```jsx
function App() {
  return (
    <>
      <input />
      <li></li>
    </>
  );
}
```

### 3. camelCase **Most** things.
JSX turns into JavaScript, and attributes of elements become keys of JavaScript objects, so you can’t use dashes or reserved words such as `class`. Because of this, many HTML attributes are written in camelCase. Instead of `stroke-width`, you’d use `strokeWidth`, and instead of `class` you’d use `className`.

```jsx
function App() {
  return (
   <div className="container">
     <svg>
       <circle cx="25" cy="75" r="20" stroke="green" strokeWidth="2" />
     </svg>
   </div>
  );
}
```

### 4. JSX Expressions in Curly Braces
 JSX allows embedding JavaScript expressions within curly braces `{}`. This allows dynamic content and logic within JSX elements.

```jsx
function Greeting(props) {
  return <div>Hello, {props.name}!</div>;
}
```

### 5. Comments in JSX
You can include comments in JSX by wrapping them in curly braces and starting with `{/*` and ending with `*/}`.

```jsx
function SomeComponent() {
  return (
    <div>
      {/* This is a comment */}
      <p>Some JSX content</p>
    </div>
  );
}
```

### 6. Conditional Rendering
You can use JavaScript expressions or ternary operators for conditional rendering within JSX.

```jsx
function ConditionalComponent({ isLoggedIn }) {
  return (
    <div>
      {isLoggedIn ? <p>User is logged in</p> : <p>User is not logged in</p>}
    </div>
  );
}
```

### 7. Using JSX Spread Attributes
Spread attributes can be used to pass props as an object, which can be useful for passing a large number of props or for cleaner code.

```jsx
const props = { name: 'John', age: 30 };
function Greeting(props) {
  return <Welcome {...props} />;
}
```

### 8. Event Handling
Event handlers are written in camelCase and passed as props. They can be defined inline or as separate functions.

```jsx
function Button() {
  function handleClick() {
    console.log('Button clicked');
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

---
### Knowledge Check

**Q1.** What is JSX?

**A1.** JSX is a syntax extension for JavaScript that lets you write HTML-like markup inside a JavaScript file. It’s not required to use JSX when writing React components, but it does make writing them more concise.

Essentially, JSX is syntactic sugar for the React `createElement` function. This function creates a React element, which is a plain object, so JSX compiles down to plain JavaScript objects.

JSX stands for JavaScript XML. It's a syntax extension for JavaScript that allows you to write HTML-like code within your JavaScript files. JSX provides a more readable and concise way to describe the structure of your UI components in React applications.

Here's an example of JSX:

```jsx
import React from 'react';

const element = <h1>Hello, world!</h1>;

function MyComponent() {
  return (
    <div>
      <h1>This is a JSX component</h1>
      <p>And this is a paragraph inside JSX</p>
    </div>
  );
}

export default MyComponent;
```

In JSX:

- You can write HTML-like tags directly in your JavaScript code, which are actually transformed into calls to `React.createElement()`.
- You can use curly braces `{}` to include JavaScript expressions within JSX.
- JSX is not mandatory for writing React applications, but it's a popular and recommended way because it makes the code more readable and easier to understand, especially for those familiar with HTML syntax.

During the build process, JSX code is transformed into regular JavaScript code that React can understand and work with. This transformation is typically done using tools like Babel, which converts JSX into calls to `React.createElement()`, enabling React to create and manipulate the corresponding React elements efficiently.

**Q2.** Why do we use JSX?

**A2.** There are several reasons why JSX is widely used in React development:

1. **Readability**: JSX provides a familiar HTML-like syntax, which makes the code more readable and easier to understand, especially for developers familiar with HTML. It enhances the clarity of the UI structure and component hierarchy.

2. **Ease of Use**: Writing JSX feels natural, especially for developers who are accustomed to writing HTML. It simplifies the process of creating and visualizing UI components, reducing the cognitive overhead of switching between different syntaxes.

3. **Expressiveness**: JSX allows you to embed JavaScript expressions directly within the markup, enabling dynamic rendering of components based on variables, functions, or props. This makes it easy to create dynamic and interactive user interfaces.

4. **Tooling Support**: JSX is well-supported by development tools and IDEs, providing features such as syntax highlighting, code completion, and error checking. This helps developers write code more efficiently and catch errors early in the development process.

5. **Component Composition**: JSX facilitates the composition of reusable and composable components, which is a core principle in React development. Components can be easily nested within each other, allowing for a modular and maintainable codebase.

6. **Performance Optimization**: JSX is optimized for performance by tools like Babel, which transform JSX code into efficient JavaScript code that React can work with. This helps minimize the overhead of rendering and updating components, resulting in faster and more responsive applications.

Overall, JSX enhances the developer experience in React development by providing a concise, expressive, and intuitive syntax for building user interfaces. It plays a crucial role in making React one of the most popular and powerful libraries for building modern web applications.

**Q3.** What are the three rules of JSX?

**A3.** If you were to take some valid HTML and copy it straight into your React component, it would not work. This is due to some of the rules JSX implements, that aren’t present in HTML.

#### 1. Return a single root element
If you wish to return multiple elements in a component, you can do so by wrapping them in a parent tag. This can be a \<div>, or, if you don’t want the elements to have a container, you could use a `React fragment`, like so:

```jsx
function App() {
  return (
    <>
      <h1>Example h1</h1>
      <h2>Example h2</h2>
    </>
  );
}
```

`<Fragment> (<>...</>)`: `<Fragment>`, often used via `<>...</>` syntax, lets you group elements without a wrapper node.

```jsx
<>
  <OneChild />
  <AnotherChild />
</>
```

Wrap elements in \<Fragment> to group them together in situations where you need a single element. Grouping elements in Fragment has no effect on the resulting DOM; it is the same as if the elements were not grouped. The empty JSX tag <></> is shorthand for \<Fragment>\</Fragment> in most cases.

Use Fragment, or the equivalent <>...</> syntax, to group multiple elements together. You can use it to put multiple elements in any place where a single element can go. For example, a component can only return one element, but by using a Fragment you can group multiple elements together and then return them as a group:

```jsx
function Post() {
  return (
    <>
      <PostTitle />
      <PostBody />
    </>
  );
}
```

Fragments are useful because grouping elements with a Fragment has no effect on layout or styles, unlike if you wrapped the elements in another container like a DOM element. 

#### 2. Close all tags.
In HTML, many tags are self-closing and self-wrapping. In JSX however, we must explicitly close and wrap these tags.

\<input> would become \<input />, and \<li> would become \<li>\</li>:

```jsx
function App() {
  return (
    <>
      <input />
      <li></li>
    </>
  );
}
```

#### 3. camelCase **Most** things.
JSX turns into JavaScript, and attributes of elements become keys of JavaScript objects, so you can’t use dashes or reserved words such as `class`. Because of this, many HTML attributes are written in camelCase. Instead of `stroke-width`, you’d use `strokeWidth`, and instead of `class` you’d use `className`.

```jsx
function App() {
  return (
   <div className="container">
     <svg>
       <circle cx="25" cy="75" r="20" stroke="green" strokeWidth="2" />
     </svg>
   </div>
  );
}
```

**Q4.** How do you reference a dynamic value inside of your JSX?

**A4.** JSX lets you write HTML-like markup inside a JavaScript file, keeping rendering logic and content in the same place. Sometimes you will want to add a little JavaScript logic or reference a dynamic property inside that markup. In this situation, you can use curly braces in your JSX to incorporate JavaScript into your JSX.

Example:

```jsx
// utils.jsx
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    person.imageSize +
    '.jpg'
  );
}
```

```jsx
// App.jsx
import { getImageUrl } from './utils.js'

const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src={getImageUrl(person)}
        alt={person.name}
      />
      <ul>
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    </div>
  );
}
```
