# Class Component Lifecycle Methods
There are three stages to a component’s life: **mounting, updating, and unmounting**. Within class components, each of these have a method assigned to them , which is what we are going to cover in this lesson.

### render()
The render method is the only required lifecycle method in a class component. It runs on mount and update of a component. Render should be pure, meaning it doesn’t modify component state, returns the same thing each time it’s called (given the same inputs), and doesn’t cause any side effects.

### componentDidMount()
This method is run after the component is mounted (inserted in the DOM tree). You should make any calls to fetch data that is needed for the component here. It is also a good place to do anything that is reliant on the component, such as drawing on a canvas element that you just rendered.

### componentDidUpdate()
This method is run after a component re-renders. Because of this, you have to be careful about what you update in this method, as if you’re updating state indiscriminately, a re-render is caused, and you’ll end up in an endless loop. You can avoid this issue by using conditional statements about the equality of previous and current props when updating state.

In this method you should be updating anything that needs to be changed in response to either the DOM changing, or any states that you might want to act on upon change. For example, you’d want to refetch user data if the user changes.

### componentWillUnmount()
This is the last lifecycle method, which is called before a component is unmounted and destroyed. In this method you should be performing cleanup actions, so that would be cancelling network requests, clearing timers, etc.

## How the `useEffect()` Hook of functional components combines the lifecycle methods of class components
Now that we’ve learnt about class lifecycle methods, it’s useful to understand that the `useEffect` hook used in functional components is essentially a combination of `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`. Which method/methods it relates to varies based on it’s dependency array, and if it returns anything.

+ An empty dependency array would be equivalent to `componentDidMount`.
+ A dependency array with a value/values in it would be a combination of `componentDidMount` and `componentDidUpdate`, but only updating when dependencies change.
+ No dependency array would be equivalent to `componentDidMount` and `componentDidUpdate` combined.
+ A return function inside of a useEffect() hook would be equivalent to `componentWillUnmount`.

For example:

```jsx
  useEffect(() => {
    placeholderFunction();
    return () => cleanupFunction();
  }, [])
```

In this snippet, the useEffect contains the functionality of `componentDidMount`, and `componentWillUnmount` via the return function. This example doesn’t have the `componentDidUpdate` functionality because of an empty dependency array.

---
### Knowledge Check

**Q1.** What is the only required lifecycle method?

**A1.** The `render()` function is the most used lifecycle method and also it is the only required lifecycle method in a class component. It runs on mount and update of a component. Render should be pure, meaning it doesn’t modify component state, returns the same thing each time it’s called (given the same inputs), and doesn’t directly interact with the browser.

**Q2.** What lifecycle method should you use for initial data fetching?

**A2.** The `componentDidMount()` method is run after the component is mounted (inserted in the DOM tree). You should make any calls to fetch data that is needed for the component here. It is also a good place to do anything that is reliant on the component, such as drawing on a canvas element that you just rendered.

**Q3.** When you want to act upon change of the DOM, or of state, what lifecycle method would you use?

**A3.** The `componentDidUpdate()` method is run after a component re-renders. Because of this, you have to be careful about what you update in this method, as if you’re updating state indiscriminately, a re-render is caused, and you’ll end up in an endless loop. You can avoid this issue by using conditional statements about the equality of previous and current props when updating state.

In this method you should be updating anything that needs to be changed in response to either the DOM changing, or any states that you might want to act on upon change. For example, you’d want to refetch user data if the user changes.

**Q4.** When performing cleanup actions, what lifecycle method should be used?

**A4.** The `componentWillUnmount()` method is the last lifecycle method, which is called before a component is unmounted and destroyed. In this method you should be performing cleanup actions, so that would be cancelling network requests, clearing timers, etc.

**Q5.** How does the useEffect hook combine some of the lifecycle methods?

**A5.** Now that we’ve learnt about class lifecycle methods, it’s useful to understand that the `useEffect` hook used in functional components is essentially a combination of `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`. Which method/methods it relates to varies based on it’s dependency array, and if it returns anything.

+ An empty dependency array would be equivalent to `componentDidMount`.
+ A dependency array with a value/values in it would be a combination of `componentDidMount` and `componentDidUpdate`, but only updating when dependencies change.
+ No dependency array would be equivalent to `componentDidMount` and `componentDidUpdate` combined.
+ A return function inside of a useEffect() hook would be equivalent to `componentWillUnmount`.

For example:

```jsx
  useEffect(() => {
    placeholderFunction();
    return () => cleanupFunction();
  }, [])
```

In this snippet, the useEffect contains the functionality of `componentDidMount`, and `componentWillUnmount` via the return function. This example doesn’t have the `componentDidUpdate` functionality because of an empty dependency array.