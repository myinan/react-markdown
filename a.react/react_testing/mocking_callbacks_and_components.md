# Mocking Callbacks And Components

### Mocking callback functions
Callbacks are ubiquitous. Every avenue of user interaction involves callbacks. Sometimes they’re passed in as props to alter state of the parent component. Consider this button component:

```jsx
// CustomButton.jsx

const CustomButton = ({ onClick }) => {
  return (
    <button onClick={onClick}>Click me</button> 
  );
};

export default CustomButton;
```

Nothing fancy. CustomButton is a component with a prop passed in. We’re interested in the onClick prop. We have no idea what the function does. We have no idea how the function will affect the application. All we know is it must be called when user clicks the button. Let’s test it.

Notice how we mock and test the onClick function:

```jsx
// CustomButton.test.jsx

import { vi } from 'vitest'
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import CustomButton from "./CustomButton";

describe("CustomButton", () => {
    it("should render a button with the text 'Click me'", () => {
        render(<CustomButton onClick={() => {}} />);

        const button = screen.getByRole("button", { name: "Click me" });

        expect(button).toBeInTheDocument();
    });
  
    it("should call the onClick function when clicked", async () => {
        const onClick = vi.fn();
        const user = userEvent.setup()
        render(<CustomButton onClick={onClick} />);

        const button = screen.getByRole("button", { name: "Click me" });

        await user.click(button);

        expect(onClick).toHaveBeenCalled();
    });

    it("should not call the onClick function when it isn't clicked", async () => {
        const onClick = vi.fn();
        render(<CustomButton onClick={onClick} />);

        expect(onClick).not.toHaveBeenCalled();
    });
});
```

Three tests and we are done with this component. You should be already familiar with how the first test works. Take some time to figure out what functions come from which package.

For the second and third tests, we mock the onClick handler using one of Vitest’s functions, vi.fn(). Then we assert that it is called/not called when the button is clicked or not.

You could also set up your mocks in a beforeEach block instead of in every test block. This may be suitable for some situations. However, for better readability, it is recommended that all setups be done in the same test block. Doing so eliminates the need to search through the entire file for context, making it easier to review future changes. This also decreases the chance of having leakage create problems throughout the test suite. Unless your test file is particularly long and the test preparation takes up dozens of lines, it is recommended to set up your mocks in each test block; otherwise, you may use beforeEach.

It is recommended to invoke userEvent.setup() before rendering the component. It is discouraged to call renders and userEvent functions outside of the test itself, (for example, in a beforeEach block). If you find yourself repeating the same code in multiple tests, the recommended approach to shorten each test is to write a setup function.

### Mocking child components
Mocking child components is a common practice in testing React applications, and it serves several purposes:

1. **Isolation of Concerns**: When testing a parent component, you typically want to focus on its behavior in isolation from its children. Mocking allows you to isolate the parent component and test its functionality without being affected by the implementation details or behavior of its child components.

2. **Controlled Testing Environment**: Mocking child components gives you control over the environment in which you test the parent component. By replacing the actual child components with mocks, you can simulate different scenarios and edge cases to ensure that the parent component behaves correctly under various conditions.

3. **Avoiding Dependencies**: Sometimes, child components might have dependencies on external services, APIs, or complex logic that you don't want to deal with in your tests. By mocking these child components, you can avoid these dependencies and focus solely on testing the parent component's logic.

Here's how mocking child components typically works:

1. **Replace Child Components with Mocks**: In your test setup, you use mocking libraries (like Jest's `jest.mock()` function) to replace the imports of child components with mock implementations. These mocks simulate the behavior of the actual child components but allow you to control their output and behavior during testing.

2. **Define Mock Behavior**: You define the behavior of the mock components based on what the parent component expects from its children. This can include returning specific values, triggering callbacks, or simulating different states.

3. **Test the Parent Component**: With the child components mocked, you can now focus on testing the parent component in isolation. You render the parent component in your tests and verify its behavior based on the mocked child components. This might involve making assertions about the rendered output, handling user interactions, or testing component lifecycle methods.

By mocking child components, you create a controlled testing environment that allows you to thoroughly test the parent component's functionality without being dependent on the implementation details of its children. This helps ensure that your tests are focused, reliable, and maintainable, ultimately leading to more robust React applications.

## A React Testing in the Real World Example
The following is the `submissions-list.jsx` component and its test file `submissions-list.test.jsx` from "The Odin Project" website.

```jsx
// submissions-list.jsx

import React, { useContext } from 'react';
import {
  object, func, arrayOf, bool,
} from 'prop-types';
import FlipMove from 'react-flip-move';

import Submission from './submission';
import ProjectSubmissionContext from '../ProjectSubmissionContext';

const noop = () => { };

const SubmissionsList = ({
  submissions,
  handleDelete,
  onFlag,
  handleUpdate,
  isDashboardView,
  handleLikeToggle,
  userSubmission,
}) => {
  const { allSubmissionsPath } = useContext(ProjectSubmissionContext);
  const hasSubmissions = submissions.length > 0 || Boolean(userSubmission);

  return (
    <div data-test-id="submissions-list">
      { userSubmission
        ? (
          <Submission
            key={userSubmission.id}
            submission={userSubmission}
            handleUpdate={handleUpdate}
            onFlag={onFlag}
            handleDelete={handleDelete}
            isDashboardView={isDashboardView}
            handleLikeToggle={handleLikeToggle}
          />
        )
        : ''}
      { hasSubmissions ? (
        <FlipMove>
          {submissions.sort((a, b) => b.likes - a.likes).map((submission) => (
            <Submission
              key={submission.id}
              submission={submission}
              handleUpdate={handleUpdate}
              onFlag={onFlag}
              handleDelete={handleDelete}
              isDashboardView={isDashboardView}
              handleLikeToggle={handleLikeToggle}
            />
          ))}
        </FlipMove>
      ) : (
        <h2 className="text-center text-xl text-gray-600 dark:text-gray-300 font-medium pr-0 pb-10 mb-0">
          No Submissions yet, be the first!
        </h2>
      )}

      { (allSubmissionsPath && hasSubmissions)
        && (
          <p className="text-center py-6 px-0">
            <span>
              Showing
              {' '}
              {submissions.length}
              {' '}
              most liked submissions -
              {' '}
            </span>
            <a
              className="underline hover:no-underline text-gray-600 hover:text-gray-800 dark:text-gray-400
               dark:hover:text-gray-300"
              data-test-id="view-all-projects-link"
              href={allSubmissionsPath}
            >
              View full list of solutions
            </a>
          </p>
        )}
    </div>
  );
};

SubmissionsList.defaultProps = {
  userSubmission: null,
  onFlag: noop,
  isDashboardView: false,
};

SubmissionsList.propTypes = {
  submissions: arrayOf(object).isRequired,
  userSubmission: object,
  handleDelete: func.isRequired,
  onFlag: func,
  handleUpdate: func.isRequired,
  handleLikeToggle: func.isRequired,
  isDashboardView: bool,
};

export default SubmissionsList;
```

```jsx
// submissions-list.test.jsx

/* eslint-disable react/prop-types */
import React from 'react';
import { render, screen } from '@testing-library/react';

import ProjectSubmissionContext from '../../ProjectSubmissionContext';
import SubmissionsList from '../submissions-list';

jest.mock('react-flip-move', () => ({ children }) => <div>{children}</div>);

jest.mock('../submission', () => ({ submission, isDashboardView }) => (
  <>
    <div data-test-id="submission">{submission.id}</div>
    <div data-test-id="dashboard">{isDashboardView.toString()}</div>
  </>
));

// setup props
const submissions = [
  { id: 'foo', likes: 3 },
  { id: 'bar', likes: 2 },
  { id: 'baz', likes: 1 },
];
const userSubmission = { id: 'foobar' };
const handleDelete = jest.fn();
const onFlag = jest.fn();
const handleUpdate = jest.fn();
const handleLikeToggle = jest.fn();

describe('submissions list', () => {
  describe('submissions', () => {
    it('renders the submissions array in order of likes', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '#' }}>
          <SubmissionsList
            submissions={submissions}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(screen.queryAllByTestId('submission').length).toBe(3);
      expect(screen.queryAllByTestId('submission')[0].textContent).toBe('foo');
      expect(screen.queryAllByTestId('submission')[1].textContent).toBe('bar');
      expect(screen.queryAllByTestId('submission')[2].textContent).toBe('baz');
    });

    it('does not render any submissions when array is empty and user submission not provided, and instead renders a no submissions message', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '#' }}>
          <SubmissionsList
            submissions={[]}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(screen.queryAllByTestId('submission').length).toBe(0);
      expect(
        screen.getByText('No Submissions yet, be the first!'),
      ).toBeInTheDocument();
    });
  });

  describe('user submission', () => {
    it('renders a user submission when provided', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '#' }}>
          <SubmissionsList
            submissions={submissions}
            userSubmission={userSubmission}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(screen.queryAllByTestId('submission').length).toBe(4);
      expect(screen.getByText('foobar')).toBeInTheDocument();
    });

    it('does not render a user submission when not provided', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '#' }}>
          <SubmissionsList
            submissions={submissions}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(screen.queryAllByTestId('submission').length).toBe(3);
      expect(screen.queryByText('foobar')).not.toBeInTheDocument();
    });

    it('does not render \'no submissions\' message when array is empty but user submission is provided', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '#' }}>
          <SubmissionsList
            submissions={[]}
            userSubmission={userSubmission}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(screen.getByText('foobar')).toBeInTheDocument();
      expect(
        screen.queryByText('No Submissions yet, be the first!'),
      ).not.toBeInTheDocument();
    })
  });

  describe('context', () => {
    it('renders view more section if allSubmissionsPath is provided in context', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '#' }}>
          <SubmissionsList
            submissions={submissions}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
            onFlag={onFlag}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(screen.getByTestId('view-all-projects-link')).toBeInTheDocument();
    });

    it('does not render view more section if allSubmissionsPath is not provided in context', () => {
      render(
        <ProjectSubmissionContext.Provider value={{ allSubmissionsPath: '' }}>
          <SubmissionsList
            submissions={submissions}
            handleDelete={handleDelete}
            handleUpdate={handleUpdate}
            handleLikeToggle={handleLikeToggle}
            onFlag={onFlag}
          />
        </ProjectSubmissionContext.Provider>,
      );

      expect(
        screen.queryByTestId('view-all-projects-link'),
      ).not.toBeInTheDocument();
    });
  });
});
```

One important thing to note about this test that it follows a certain pattern in terms of the way it's written which is the "Arrange-Act-Assert" pattern. 

The "Arrange-Act-Assert" (AAA) pattern is a fundamental principle in unit testing, regardless of the technology or framework being used. It provides a clear structure for organizing tests, making them more readable, maintainable, and understandable.

In the context of unit testing with React and Vitest, the AAA pattern can be applied as follows:

1. **Arrange**: In this phase, you set up the necessary preconditions and initialize any objects or variables needed for the test. This might involve creating mock objects, setting the initial state of your component, providing mock data, or any other setup required before executing the test.

2. **Act**: This is where you perform the action or invoke the method that you want to test. In the case of React components, it typically involves triggering an event, calling a function, or simulating user interaction that causes the component to change its state or behavior.

3. **Assert**: Finally, in this phase, you verify that the action performed in the "Act" phase produced the expected results. You make assertions about the state of the component, the output it produces, or any other behavior that you expect based on the actions taken. If the assertions pass, the test is considered successful; otherwise, it indicates a failure, and you need to investigate and fix the issue.

---
### Knowledge Check

**Q1.** How can you mock a callback handler?

**A1.** 

```jsx
// CustomButton.jsx

const CustomButton = ({ onClick }) => {
  return (
    <button onClick={onClick}>Click me</button> 
  );
};

export default CustomButton;
```

```jsx
// CustomButton.test.jsx

import { vi } from 'vitest'
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import CustomButton from "./CustomButton";

describe("CustomButton", () => {
    it("should render a button with the text 'Click me'", () => {
        render(<CustomButton onClick={() => {}} />);

        const button = screen.getByRole("button", { name: "Click me" });

        expect(button).toBeInTheDocument();
    });
  
    it("should call the onClick function when clicked", async () => {
        const onClick = vi.fn();
        const user = userEvent.setup()
        render(<CustomButton onClick={onClick} />);

        const button = screen.getByRole("button", { name: "Click me" });

        await user.click(button);

        expect(onClick).toHaveBeenCalled();
    });

    it("should not call the onClick function when it isn't clicked", async () => {
        const onClick = vi.fn();
        render(<CustomButton onClick={onClick} />);

        expect(onClick).not.toHaveBeenCalled();
    });
});
```

**Q2.** How can you mock a child component?

**A2.**

```jsx
jest.mock('../submission', () => ({ submission, isDashboardView }) => (
  <>
    <div data-test-id="submission">{submission.id}</div>
    <div data-test-id="dashboard">{isDashboardView.toString()}</div>
  </>
));
```

When testing a parent component in React that renders child components, you often want to isolate the testing focus to just the parent component itself. This can be achieved by mocking the child components using testing libraries like Jest and Enzyme. Below is an example of how you can mock child components when testing a parent component in React using Jest and Enzyme:

Let's say you have a parent component called `ParentComponent` that renders a child component called `ChildComponent`:

```jsx
// ParentComponent.js
import React from 'react';
import ChildComponent from './ChildComponent';

const ParentComponent = () => {
  return (
    <div>
      <h1>Parent Component</h1>
      <ChildComponent />
    </div>
  );
}

export default ParentComponent;
```

And your `ChildComponent` might look like this:

```jsx
// ChildComponent.js
import React from 'react';

const ChildComponent = () => {
  return (
    <div>
      <h2>Child Component</h2>
      <p>This is a child component.</p>
    </div>
  );
}

export default ChildComponent;
```

Now, let's write a test for `ParentComponent` where we mock `ChildComponent`:

```jsx
// ParentComponent.test.js
import React from 'react';
import { shallow } from 'enzyme';
import ParentComponent from './ParentComponent';

// Mocking ChildComponent
jest.mock('./ChildComponent', () => () => <div className="mocked-child-component">Mocked Child Component</div>);

describe('ParentComponent', () => {
  it('renders ParentComponent with mocked ChildComponent', () => {
    const wrapper = shallow(<ParentComponent />);
    expect(wrapper.find('.mocked-child-component')).toHaveLength(1);
    expect(wrapper.text()).toContain('Parent Component');
    expect(wrapper.text()).toContain('Mocked Child Component');
  });
});
```

In the test file above:

- We import `shallow` from Enzyme to render the `ParentComponent` shallowly, which means it will only render the component itself without rendering its child components.
- We use `jest.mock` to mock the `ChildComponent`. We provide a function that returns a dummy component in place of `ChildComponent`.
- Then, we write a test to ensure that `ParentComponent` renders the mocked `ChildComponent` correctly along with its own content.

This way, you can isolate the testing of `ParentComponent` and ensure it works as expected without being affected by the implementation details of its child components.