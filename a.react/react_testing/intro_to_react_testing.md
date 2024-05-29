# Introduction To React Testing

## Setting up a React testing environment (Requirements)
+ [Vite](https://vitejs.dev)
+ [Vitest](https://vitest.dev)
+ [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) // [React Testing Library Tutorial](https://www.robinwieruch.de/react-testing-library/)
+ [Vitest with RTL Guide](https://www.robinwieruch.de/vitest-react-testing-library/)
+ Lastly, run the following command :

```bash
npm install @testing-library/user-event --save-dev
```

Now that we have everything we need, let’s briefly go over what some of those packages do. We’ll primarily focus on the `@testing-library` packages.

  - `@testing-library/react` will give us access to useful functions like `render` which we’ll demonstrate later on.
  - `@testing-library/jest-dom` includes some handy custom matchers (assertive functions) like `toBeInTheDocument` and more. Jest already has a lot of matchers so this package is not compulsory to use.
  - `@testing-library/user-event` provides the `userEvent` API that simulates user interactions with the webpage. 

## Our first query

```jsx
// App.jsx

const App = () => <h1>Our First Test</h1>;

export default App;
```

```jsx
// App.test.jsx

import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/react";
import App from "./App";

describe("App component", () => {
  it("renders correct heading", () => {
    render(<App />);
    expect(screen.getByRole("heading").textContent).toMatch(/our first test/i);
  });
});
```

Execute npm test App.test.jsx on the terminal and see the test pass. getByRole is just one of the dozen query methods that we could’ve used. Essentially, queries are classified into three types: getBy, queryBy and findBy.

As stated by the React Testing Library docs, ByRole methods are favored methods for querying, especially when paired with the name option. For example, we could improve the specificity of the above query like so: getByRole("heading", { name: "Our First Test" }). Queries that are done through ByRole ensure that our UI is accessible to everyone no matter what mode they use to navigate the webpage (i.e. mouse or assistive technologies).

## Simulating user events
There are numerous ways a user can interact with a webpage. Even though live user feedback and interaction is irreplaceable, we can still build some confidence in our components through tests. Here’s a button which changes the heading of the App:

```jsx
// App.jsx

import React, { useState } from "react";

const App = () => {
  const [heading, setHeading] = useState("Magnificent Monkeys");

  const clickHandler = () => {
    setHeading("Radical Rhinos");
  };

  return (
    <>
      <button type="button" onClick={clickHandler}>
        Click Me
      </button>
      <h1>{heading}</h1>
    </>
  );
};

export default App;
```

Let’s test if the button works as intended. React Testing Library provides the screen object which has all the methods for querying. With screen, we don’t have to worry about keeping render’s destructuring up-to-date. Hence, it’s better to use screen to access queries rather than to destructure render.


```jsx
// App.test.jsx

import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import App from "./App";

describe("App component", () => {
  it("renders magnificent monkeys", () => {
    // since screen does not have the container property, we'll destructure render to obtain a container for this test
    const { container } = render(<App />);
    expect(container).toMatchSnapshot();
  });

  it("renders radical rhinos after button click", async () => {
    const user = userEvent.setup();

    render(<App />);
    const button = screen.getByRole("button", { name: "Click Me" });

    await user.click(button);

    expect(screen.getByRole("heading").textContent).toMatch(/radical rhinos/i);
  });
});
```

The tests speak for themselves. In the first test, we utilize snapshots to check whether all the nodes render as we expect them to. In the second test, we simulate a click event. Then we check if the heading changed. toMatch is one of the various assertions we could have made. Notice that the callback function for the second test case is an async one, as we need this in order to await user.click().

It’s also important to note that after every test, React Testing Library unmounts the rendered components. That’s why we render for each test. For a lot of test cases for a component, the beforeEach Jest function could prove handy.

## What are snapshots?
Snapshot testing is a method used in software testing to capture the "snapshot" or the current state of a component, such as a user interface, at a particular point in time. This snapshot is then stored as a reference for future comparisons. When the component is modified, the new state is compared against the stored snapshot to detect any unintended changes.

The primary problem that snapshots aim to solve is the regression problem. Regression occurs when changes made to a codebase inadvertently introduce new bugs or alter the behavior of existing features. Traditional testing methods might not catch these regressions, especially in large and complex applications.

By using snapshot testing, developers can quickly and easily verify that their changes have not unintentionally modified the output of a component. This is particularly useful for user interfaces, where visual changes are common but may not always be immediately apparent during manual testing.

Snapshot testing helps in:

1. **Detecting unintended changes**: Snapshots provide a baseline representation of the component's output. When changes are made, the new output can be compared against the snapshot to identify any differences. This helps catch unintended changes early in the development process.

2. **Streamlining testing**: Writing snapshot tests is often simpler and more concise than writing traditional assertion-based tests. With just one assertion, developers can cover a wide range of potential changes to the component's output.

3. **Improving confidence in refactoring**: When refactoring code, developers can use snapshots to ensure that the behavior of the component remains consistent even after significant changes.

Snapshot testing also comes with its own set of challenges and limitations:

1. **False positives**: Snapshots may pass even if the component behavior is incorrect, leading to false positives. This happens when the snapshot captures a state that is technically valid but semantically incorrect.

2. **False negatives**: Snapshots may fail due to insignificant changes, such as formatting or markup changes, leading to false negatives. This can reduce confidence in the test suite and make developers reluctant to use snapshot testing.

3. **Limited validation**: Snapshot tests primarily validate the output of a component but may not cover all aspects of its behavior. They may miss edge cases or complex interactions that traditional tests would catch.

In summary, snapshot testing is a valuable tool for detecting unintended changes in the output of components, particularly in user interfaces. However, it is essential to use snapshot testing judiciously and supplement it with other testing techniques to ensure comprehensive test coverage and minimize false positives and false negatives.

---
### Knowledge Check

**Q1.** What packages are required for React testing?

**A1.** **Setting up a React testing environment (Requirements)**:

+ [Vite](https://vitejs.dev)
+ [Vitest](https://vitest.dev)
+ [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) // [React Testing Library Tutorial](https://www.robinwieruch.de/react-testing-library/)
+ [Vitest with RTL Guide](https://www.robinwieruch.de/vitest-react-testing-library/)
+ Lastly, run the following command :

```bash
npm install @testing-library/user-event --save-dev
```

Now that we have everything we need, let’s briefly go over what some of those packages do. We’ll primarily focus on the `@testing-library` packages.

  - `@testing-library/react` will give us access to useful functions like `render` which we’ll demonstrate later on.
  - `@testing-library/jest-dom` includes some handy custom matchers (assertive functions) like `toBeInTheDocument` and more. Jest already has a lot of matchers so this package is not compulsory to use.
  - `@testing-library/user-event` provides the `userEvent` API that simulates user interactions with the webpage. 

**Q2.** What is the significance of the user-event package?

**A2.** `@testing-library/user-event` provides the `userEvent` API that simulates user interactions with the webpage.

**Q3.** What does the render method do?

**A3.** The `render` method in the React Testing Library is used to render React components into a container, which is then appended to the `document.body`. This allows you to test how your components behave when rendered in a realistic DOM environment, similar to how they would be rendered in a browser.

Here's a breakdown of what the `render` method does:

1. **Rendering Components**: It takes a React element (`ui`) as its first argument. This element represents the component you want to render for testing.

2. **Container**: It renders the component into a container, which is then appended to the `document.body`. This ensures that the component is rendered within a realistic DOM environment.

3. **Options**: The `options` parameter is optional and allows you to provide additional configuration options for rendering.

4. **Return Value**: The `render` method returns a `RenderResult` object, which contains various utility functions and references to interact with the rendered component during testing. These utilities include functions to query elements within the rendered component, such as `getByText`, `getByTestId`, etc., and methods to access the rendered component itself, such as `asFragment`.

**Q4.** What is the most preferred method for querying?

**A4.** `getByRole`: This can be used to query every element that is exposed in the accessibility tree. With the `name` option you can filter the returned elements by their accessible name. This should be your top preference for just about everything. There's not much you can't get with this (if you can't, it's possible your UI is inaccessible). Most often, this will be used with the name option like so: `getByRole('button', {name: /submit/i})`. 

**Q5.** How would you test for a click event with userEvent?

**A5.** 

```jsx
// App.test.jsx

import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import App from "./App";

describe("App component", () => {
  it("renders magnificent monkeys", () => {
    // since screen does not have the container property, we'll destructure render to obtain a container for this test
    const { container } = render(<App />);
    expect(container).toMatchSnapshot();
  });

  it("renders radical rhinos after button click", async () => {
    const user = userEvent.setup();

    render(<App />);
    const button = screen.getByRole("button", { name: "Click Me" });

    await user.click(button);

    expect(screen.getByRole("heading").textContent).toMatch(/radical rhinos/i);
  });
});
```

In the second test case, we simulate a click event. Then we check if the heading changed. toMatch is one of the various assertions we could have made. Notice that the callback function for the second test case is an async one, as we need this in order to await user.click().

**Q6.** What is the advantage of snapshot tests?

**A6.** Snapshot testing is a method used in software testing to capture the "snapshot" or the current state of a component, such as a user interface, at a particular point in time. This snapshot is then stored as a reference for future comparisons. When the component is modified, the new state is compared against the stored snapshot to detect any unintended changes.

The primary problem that snapshots aim to solve is the regression problem. Regression occurs when changes made to a codebase inadvertently introduce new bugs or alter the behavior of existing features. Traditional testing methods might not catch these regressions, especially in large and complex applications.

By using snapshot testing, developers can quickly and easily verify that their changes have not unintentionally modified the output of a component. This is particularly useful for user interfaces, where visual changes are common but may not always be immediately apparent during manual testing.

Snapshot testing helps in:

1. **Detecting unintended changes**: Snapshots provide a baseline representation of the component's output. When changes are made, the new output can be compared against the snapshot to identify any differences. This helps catch unintended changes early in the development process.

2. **Streamlining testing**: Writing snapshot tests is often simpler and more concise than writing traditional assertion-based tests. With just one assertion, developers can cover a wide range of potential changes to the component's output.

3. **Improving confidence in refactoring**: When refactoring code, developers can use snapshots to ensure that the behavior of the component remains consistent even after significant changes.

**Q7.** What are the disadvantages of snapshot tests?

**A7.** Snapshot testing also comes with its own set of challenges and limitations:

1. **False positives**: Snapshots may pass even if the component behavior is incorrect, leading to false positives. This happens when the snapshot captures a state that is technically valid but semantically incorrect.

2. **False negatives**: Snapshots may fail due to insignificant changes, such as formatting or markup changes, leading to false negatives. This can reduce confidence in the test suite and make developers reluctant to use snapshot testing.

3. **Limited validation**: Snapshot tests primarily validate the output of a component but may not cover all aspects of its behavior. They may miss edge cases or complex interactions that traditional tests would catch.

In summary, snapshot testing is a valuable tool for detecting unintended changes in the output of components, particularly in user interfaces. However, it is essential to use snapshot testing judiciously and supplement it with other testing techniques to ensure comprehensive test coverage and minimize false positives and false negatives.