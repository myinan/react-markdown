# Styling React Applications
## Vanilla CSS
Vanilla CSS is the simplest way to style. In the previous courses, you’ve learned a lot of CSS and all of those skills are applicable to React. There are a couple of things we’d like to highlight.

1. [`CSS Modules`](https://github.com/css-modules/css-modules "github.com/css-modules/css-modules") let you write CSS style declarations that are scoped locally. **[(How to Style React Components Using CSS Modules)](https://www.makeuseof.com/react-components-css-modules-style/ "www.makeuseof.com/react-components-css-modules-style/")**
2. `CSS Utility Frameworks` are a popular choice for styling React applications. They provide a set of pre-defined classes that you can directly use in your HTML, or JSX in our case. [`Tailwind CSS`](https://tailwindcss.com "tailwindcss.com") is by far the most popular choice.

## CSS-in-JS
CSS-in-JS is a paradigm for styling front-end projects. It allows you to entirely take control of CSS with JavaScript and extends it with various features. Additionally, it also helps to apply styling in a logical fashion i.e. based on state. There are various CSS-in-JS solutions. One of the most popular ones in the React ecosystem is [`styled-components`](https://styled-components.com "styled-components.com").

**[CSS vs CSS-in-JS](https://blog.logrocket.com/css-vs-css-in-js/ "blog.logrocket.com/css-vs-css-in-js/")**

**[A Thorough Analysis of CSS-in-JS](https://css-tricks.com/a-thorough-analysis-of-css-in-js/ "css-tricks.com/a-thorough-analysis-of-css-in-js/")**

## Component libraries
What if everything’s already done for you? Styling, behavior, and accessibility are taken care of for you in component libraries. As the name suggests, these libraries provide adaptable and reusable components that you can use directly in your project. These components include, but are not limited to, dropdowns, drawers, calendars, toggles, tabs, and all other components you can think of.

[`Material UI`](https://mui.com "mui.com"), [`Radix`](https://www.radix-ui.com "radix-ui.com"), and [`Chakra UI`](https://chakra-ui.com "chakra-ui.com") are worth a mention when talking about component libraries.

---
### Knowledge Check

**Q1.** How can you use CSS Modules in your React app?

**A1.** The CSS modules docs describe a CSS module as a CSS file whose class names are locally scoped by default. This means you can address CSS variables with the same name in different CSS files.

You write CSS module classes just like normal classes. Then the compiler generates unique class names before sending the CSS to the browser.

For example, consider the following .btn class in a file called Button.module.css:

```css
.btn {
    width: 90px;
    height: 40px;
    padding: 10px 20px;
    border-radius: 4px;
    border: none;
}
```

To use this class in a React component, import it as styles and reference it in the class name of the React element like this:

```jsx
import styles from "./Button.module.css"
 
export default function Button() {
    return (
        <button className={styles.btn}>Submit</button>
    )
}
```

The build process will replace the CSS class with a unique name of the format like **_Button__btn_118346908**.

Note that the name of the css file should be in the format as "Button.module.css" so that Vite recognizes that the CSS file should be treated as a CSS Module.

The uniqueness of the class names means that, even if you use the same class name for different components, they will not collide. You can guarantee greater code independence since you can store a component’s CSS styles in a single file, specific to that component.

This simplifies debugging, especially if you are working with multiple stylesheets. You’ll only need to track down the CSS module for a particular component.

CSS modules also allow you to combine multiple classes through the **composes** keyword. For example, consider the following .btn class above. You could “extend” that class in other classes using composes.

For a submit button, you could have:

```css
.btn {
    /* styles */
}
 
.submit {
    composes: btn;
    background-color: green;
    color:#FFFFFF
}
```

This combines the .btn and the .submit classes. You can also compose styles from another CSS module like this:

```cs
.submit {
    composes:primary from "./colors.css"
    background-color: green;
}
```

Note that you must write the composes rule before other rules.

**Q2.** What does CSS-in-JS mean?

**A2.** CSS-in-JS, in a nutshell, is an external layer of functionality that allows you to write CSS properties for components through JavaScript.

When JavaScript took over rendering and managing the frontend with libraries like React, a CSS-in-JS solution called **styled-components** emerged. Another increasingly popular way to do the same thing is by using the **Emotion** library.

#### Example using CSS-in-JS with styled-components:

In your React app, install the styled-components library using the below Yarn command. If you are using a different package manager, see the styled-components installation docs to find the appropriate installation command:

```
yarn add styled-components
```

After installing the styled-components library, import the `styled` function and use it as shown in the code below:

```jsx
import styled from "styled-components";

const StyledButton = styled.a`
  padding: 0.75em 1em;
  background-color: ${ ({ primary }) => ( primary ? "#07c" : "#333" ) };
  color: white;

  &:hover {
    background-color: #111;
  }
`;

export default StyledButton;
```

The code above demonstrates how to style a button-link component in React. This styled component can now be imported anywhere and used directly to build a functional component without having to worry about the styles:

```jsx
import StyledButton from './components/styles/StyledButton';

function App() {
  return (
    <div className="App">
      ...
      <StyledButton href="#">Default Call-to-action</StyledButton>
      <StyledButton primary href="#">Primary Call-to-action</StyledButton>
    </div>
  );
}

export default App;
```

Note that the styles applied to the styled components are **locally scoped**, which eliminates the cumbersome need to be mindful of CSS class naming and the global scope. In addition, we can add or remove CSS **dynamically**, based on the props supplied to our component or any other logic demanded by an app feature.

#### Pros of CSS-in-JS:
+ **No scoping and specificity problems:** Since styles are available in a local scope, they are not prone to clashing with the styles of other components. You don’t even have to worry about naming things strictly to avoid style clashes.

Styles are written exclusively for one component without prepending child selectors, so specificity issues are rare.

+ **Dynamic styling:** Conditional CSS is another highlight of CSS-in-JS. As the button example above demonstrates, checking for prop values and adding suitable styles is way cooler than writing separate CSS styles for each variation.

**Q3.** What are component libraries?

**A3.** React component libraries like Material UI and Chakra UI provide a set of pre-built, customizable components that developers can use to build user interfaces more quickly and consistently. Here's a general overview of how they work:

### Material UI

#### Core Concepts:
1. **Components**: Material UI offers a wide range of React components, such as buttons, cards, modals, and more, that are styled according to the Material Design guidelines.

2. **ThemeProvider**: Material UI uses a `ThemeProvider` component to provide a theme to all the components within its hierarchy. Themes include settings like colors, typography, and other design tokens.

3. **Customization**: Material UI allows for easy customization of the components using the theme object. You can override the default styles to match your application's design.

#### Basic Usage:
```javascript
import React from 'react';
import Button from '@mui/material/Button';
import { ThemeProvider, createTheme } from '@mui/material/styles';

const theme = createTheme({
  palette: {
    primary: {
      main: '#ff5722',
    },
  },
});

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Button variant="contained" color="primary">
        Click me
      </Button>
    </ThemeProvider>
  );
}

export default App;
```

### Chakra UI

#### Core Concepts:
1. **Components**: Chakra UI provides a set of accessible and customizable UI components that follow a design system, similar to Material UI.

2. **ChakraProvider**: Chakra UI uses a `ChakraProvider` component to provide a theme and other context to its components.

3. **Style Props**: Chakra UI allows you to style components using style props, which makes it easy to customize the appearance without writing CSS.

#### Basic Usage:
```javascript
import React from 'react';
import { ChakraProvider, Button } from '@chakra-ui/react';

function App() {
  return (
    <ChakraProvider>
      <Button colorScheme="blue">Click me</Button>
    </ChakraProvider>
  );
}

export default App;
```

### How They Work Under the Hood:

1. **Component Composition**: Both libraries utilize React's component composition to create complex UI components from simpler ones. This makes it easier to build and maintain UIs.

2. **Context & Providers**: They use React context and provider patterns to pass down themes, styles, and other configurations to the components.

3. **Hooks**: Both libraries use React hooks to manage state, side effects, and other functionalities within the components.

4. **Accessibility**: Both Material UI and Chakra UI prioritize accessibility by providing accessible components out of the box and following best practices for ARIA labels and roles.

5. **Responsive Design**: They both offer responsive design utilities and breakpoints to build mobile-responsive applications.

### Summary:

- **Material UI** is based on Google's Material Design guidelines and uses a `ThemeProvider` for theming and customization.
  
- **Chakra UI** provides a set of customizable components with a focus on accessibility and uses a `ChakraProvider` for theming and styling.

Both libraries aim to simplify and speed up the UI development process by providing a set of high-quality, customizable components that developers can easily integrate into their React applications.