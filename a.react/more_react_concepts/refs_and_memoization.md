# `useRef` Hook
## Referencing Values with Refs
When you want a component to “remember” some information, but you don’t want that information to trigger new renders, you can use a `ref`.

### Adding a ref to your component 
You can add a ref to your component by importing the `useRef` Hook from React:

```jsx
import { useRef } from 'react';
```

Inside your component, call the useRef Hook and pass the initial value that you want to reference as the only argument. For example, here is a ref to the value 0:

```jsx
const ref = useRef(0);
```

useRef returns an object like this:

```jsx
{ 
  current: 0 // The value you passed to useRef
}
```

You can access the current value of that ref through the `ref.current` property. This value is intentionally mutable, meaning you can both read and write to it. It’s like a secret pocket of your component that React doesn’t track. (This is what makes it an “escape hatch” from React’s one-way data flow—more on that below!)

Here, a button will increment `ref.current` on every click:

```jsx
export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

The ref points to a number, but, like state, you could point to anything: a string, an object, or even a function. Unlike state, ref is a plain JavaScript object with the `current` property that you can read and modify.

Note that the component doesn’t re-render with every increment. Like state, refs are retained by React between re-renders. However, setting state re-renders a component. Changing a ref does not!

When a piece of information is used for rendering, keep it in state. When a piece of information is only needed by event handlers and changing it doesn’t require a re-render, using a ref may be more efficient.

### Differences between ref and state

`ref`:

+ useRef(initialValue) returns { current: initialValue }
+ Doesn’t trigger re-render when you change it.
+ Mutable—you can modify and update current’s value outside of the rendering process.
+ You shouldn’t read (or write) the current value during rendering.

`state`:

+ useState(initialValue) returns the current value of a state variable and a state setter function ( [value, setValue])
+ Triggers re-render when you change it.
+ ”Immutable”—you must use the state setting function to modify state variables to queue a re-render.
+ You can read state at any time. However, each render has its own snapshot of state which does not change.

### When to use refs 
Typically, you will use a ref when your component needs to “step outside” React and communicate with external APIs—often a browser API that won’t impact the appearance of the component. Here are a few of these rare situations:

+ Storing timeout IDs
+ Storing and manipulating DOM elements (which will be covered later)
+ Storing other objects that aren’t necessary to calculate the JSX.

If your component needs to store some value, but it doesn’t impact the rendering logic, choose refs.

### Best practices for refs 
Following these principles will make your components more predictable:

+ Treat refs as an escape hatch. Refs are useful when you work with external systems or browser APIs. If much of your application logic and data flow relies on refs, you might want to rethink your approach.

+ Don’t read or write ref.current during rendering. If some information is needed during rendering, use state instead. Since React doesn’t know when ref.current changes, even reading it while rendering makes your component’s behavior difficult to predict.

### Refs and the DOM 
You can point a ref to any value. However, the most common use case for a ref is to access a DOM element. For example, this is handy if you want to focus an input programmatically. When you pass a ref to a ref attribute in JSX, like `<div ref={myRef}>`, React will put the corresponding DOM element into `myRef.current`. Once the element is removed from the DOM, React will update `myRef.current` to be `null`.

## Manipulating the DOM with Refs
React automatically updates the DOM to match your render output, so your components won’t often need to manipulate it. However, sometimes you might need access to the DOM elements managed by React—for example, to focus a node, scroll to it, or measure its size and position. There is no built-in way to do those things in React, so you will need a ref to the DOM node.

### Getting a ref to the node
To access a DOM node managed by React, first, import the `useRef` Hook:

```jsx
import { useRef } from 'react';
```

Then, use it to declare a ref inside your component:

```jsx
const myRef = useRef(null);
```

Finally, pass your ref as the `ref` attribute to the JSX tag for which you want to get the DOM node:

```jsx
<div ref={myRef}>
```

The useRef Hook returns an object with a single property called current. Initially, myRef.current will be null. When React creates a DOM node for this `<div>`, React will put a reference to this node into myRef.current. You can then access this DOM node from your event handlers and use the built-in browser APIs defined on it.

```jsx
// You can use any browser APIs, for example:
myRef.current.scrollIntoView();
```

### Accessing another component’s DOM nodes 
When you put a `ref` on a built-in component that outputs a browser element like `<input />`, React will set that ref’s `current` property to the corresponding DOM node (such as the actual `<input />` in the browser).

However, if you try to put a ref on your own component, like `<MyInput />`, by default you will get `null`.

This happens because by default React does not let a component access the DOM nodes of other components. Not even for its own children! This is intentional. Refs are an escape hatch that should be used sparingly. Manually manipulating another component’s DOM nodes makes your code even more fragile.

Instead, components that want to expose their DOM nodes have to **opt in** to that behavior. A component can specify that it “forwards” its `ref` to one of its children. Here’s how `MyInput` can use the `forwardRef` API:

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

This is how it works:

1. `<MyInput ref={inputRef} />` tells React to put the corresponding DOM node into `inputRef.current`. However, it’s up to the `MyInput` component to opt into that—by default, it doesn’t.

1. The `MyInput` component is declared using `forwardRef`. This opts it into receiving the `inputRef` from above as the second `ref` argument which is declared after `props`.

1. `MyInput` itself passes the `ref` it received to the `<input>` inside of it.

Example:

```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

### When React attaches the refs
In React, every update is split in two phases:

+ During **render**, React calls your components to figure out what should be on the screen.
+ During **commit**, React applies changes to the DOM.

In general, you don’t want to access refs during rendering. That goes for refs holding DOM nodes as well. During the first render, the DOM nodes have not yet been created, so ref.current will be null. And during the rendering of updates, the DOM nodes haven’t been updated yet. So it’s too early to read them.

React sets ref.current during the commit. Before updating the DOM, React sets the affected ref.current values to null. After updating the DOM, React immediately sets them to the corresponding DOM nodes.

Usually, you will access refs from event handlers. If you want to do something with a ref, but there is no particular event to do it in, you might need an `Effect`. We will discuss effects on the next pages.

### Best practices for DOM manipulation with refs 
Refs are an escape hatch. You should only use them when you have to “step outside React”. Common examples of this include managing focus, scroll position, or calling browser APIs that React does not expose.

If you stick to non-destructive actions like focusing and scrolling, you shouldn’t encounter any problems. However, if you try to **modify** the DOM manually, you can risk conflicting with the changes React is making.

**Avoid changing DOM nodes managed by React.** Modifying, adding children to, or removing children from elements that are managed by React can lead to inconsistent visual results or crashes.

However, this doesn’t mean that you can’t do it at all. It requires caution. **You can safely modify parts of the DOM that React has no reason to update.** For example, if some `<div>` is always empty in the JSX, React won’t have a reason to touch its children list. Therefore, it is safe to manually add or remove elements there.

# `useMemo` Hook
useMemo is a React Hook that lets you cache the result of a calculation between re-renders.

```jsx
const cachedValue = useMemo(calculateValue, dependencies)
```

## Reference
### `useMemo(calculateValue, dependencies) `
Call useMemo at the top level of your component to cache a calculation between re-renders:

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

#### Parameters 
+ `calculateValue`: The function calculating the value that you want to cache. It should be pure, should take no arguments, and should return a value of any type. React will call your function during the initial render. On next renders, React will return the same value again if the dependencies have not changed since the last render. Otherwise, it will call `calculateValue`, return its result, and store it so it can be reused later.

+ `dependencies`: The list of all reactive values referenced inside of the `calculateValue` code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is configured for React, it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like [dep1, dep2, dep3]. React will compare each dependency with its previous value using the Object.is comparison.

#### Returns 
On the initial render, `useMemo` returns the result of calling `calculateValue` with no arguments.

During next renders, it will either return an already stored value from the last render (if the dependencies haven’t changed), or call calculateValue again, and return the result that calculateValue has returned.

#### Caveats 
+ useMemo is a Hook, so you can only call it at the top level of your component or your own Hooks. You can’t call it inside loops or conditions. If you need that, extract a new component and move the state into it.

+ In Strict Mode, React will call your calculation function twice in order to help you find accidental impurities. This is development-only behavior and does not affect production. If your calculation function is pure (as it should be), this should not affect your logic. The result from one of the calls will be ignored.

Note: Caching return values like this is also known as memoization, which is why this Hook is called useMemo.

## Usage 
### Skipping expensive recalculations 
To cache a calculation between re-renders, wrap it in a `useMemo` call at the top level of your component:

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

You need to pass two things to useMemo:
1. **A calculation function** that takes no arguments, like `() =>`, and returns what you wanted to calculate.
1. A list of dependencies including every value within your component that’s used inside your calculation.

On the initial render, the value you’ll get from useMemo will be the result of calling your calculation.

On every subsequent render, React will compare the dependencies with the dependencies you passed during the last render. If none of the dependencies have changed (compared with Object.is), useMemo will return the value you already calculated before. Otherwise, React will re-run your calculation and return the new value.

In other words, useMemo caches a calculation result between re-renders until its dependencies change.

#### Let’s walk through an example to see when this is useful.
By default, React will re-run the entire body of your component every time that it re-renders. For example, if this `TodoList` updates its state or receives new props from its parent, the `filterTodos` function will re-run:

```jsx
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

Usually, this isn’t a problem because most calculations are very fast. However, if you’re filtering or transforming a large array, or doing some expensive computation, you might want to skip doing it again if data hasn’t changed. If both todos and tab are the same as they were during the last render, wrapping the calculation in useMemo like earlier lets you reuse visibleTodos you’ve already calculated before.

This type of caching is called memoization.

Note: You should only rely on useMemo as a performance optimization. If your code doesn’t work without it, find the underlying problem and fix it first. Then you may add useMemo to improve performance.

#### How to tell if a calculation is expensive? 
In general, unless you’re creating or looping over thousands of objects, it’s probably not expensive. If you want to get more confidence, you can add a console log to measure the time spent in a piece of code:

```jsx
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

Perform the interaction you’re measuring (for example, typing into the input). You will then see logs like filter array: 0.15ms in your console. If the overall logged time adds up to a significant amount (say, 1ms or more), it might make sense to memoize that calculation. As an experiment, you can then wrap the calculation in useMemo to verify whether the total logged time has decreased for that interaction or not:

```jsx
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // Skipped if todos and tab haven't changed
}, [todos, tab]);
console.timeEnd('filter array');
```

useMemo won’t make the first render faster. It only helps you skip unnecessary work on updates.

Keep in mind that your machine is probably faster than your users’ so it’s a good idea to test the performance with an artificial slowdown. For example, Chrome offers a CPU Throttling option for this.

Also note that measuring performance in development will not give you the most accurate results. (For example, when Strict Mode is on, you will see each component render twice rather than once.) To get the most accurate timings, build your app for production and test it on a device like your users have.

#### Should you add useMemo everywhere? 
Optimizing with useMemo is only valuable in a few cases:
+ The calculation you’re putting in `useMemo` is noticeably slow, and its dependencies rarely change.
+ You pass it as a prop to a component wrapped in `memo`. You want to skip re-rendering if the value hasn’t changed. Memoization lets your component re-render only when dependencies aren’t the same.
+ The value you’re passing is later used as a dependency of some Hook. For example, maybe another `useMemo` calculation value depends on it. Or maybe you are depending on this value from `useEffect`.

There is no benefit to wrapping a calculation in useMemo in other cases. There is no significant harm to doing that either, so some teams choose to not think about individual cases, and memoize as much as possible. The downside of this approach is that code becomes less readable. Also, not all memoization is effective: a single value that’s “always new” is enough to break memoization for an entire component.

In practice, you can make a lot of memoization unnecessary by following a few principles:

1. When a component visually wraps other components, let it accept JSX as children. This way, when the wrapper component updates its own state, React knows that its children don’t need to re-render.
1. Prefer local state and don’t lift state up any further than necessary. For example, don’t keep transient state like forms and whether an item is hovered at the top of your tree or in a global state library.
1. Keep your rendering logic pure. If re-rendering a component causes a problem or produces some noticeable visual artifact, it’s a bug in your component! Fix the bug instead of adding memoization.
1. Avoid unnecessary Effects that update state. Most performance problems in React apps are caused by chains of updates originating from Effects that cause your components to render over and over.
1. Try to remove unnecessary dependencies from your Effects. For example, instead of memoization, it’s often simpler to move some object or a function inside an Effect or outside the component.

If a specific interaction still feels laggy, use the React Developer Tools profiler to see which components would benefit the most from memoization, and add memoization where needed. These principles make your components easier to debug and understand, so it’s good to follow them in any case.

### Skipping re-rendering of components 
In some cases, useMemo can also help you optimize performance of re-rendering child components. To illustrate this, let’s say this TodoList component passes the visibleTodos as a prop to the child List component:

```jsx
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

You’ve noticed that toggling the theme prop freezes the app for a moment, but if you remove \<List /> from your JSX, it feels fast. This tells you that it’s worth trying to optimize the List component.

**By default, when a component re-renders, React re-renders all of its children recursively.** This is why, when TodoList re-renders with a different theme, the List component also re-renders. This is fine for components that don’t require much calculation to re-render. But if you’ve verified that a re-render is slow, you can tell List to skip re-rendering when its props are the same as on last render by wrapping it in `memo`:

```jsx
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

**With this change, `List` will skip re-rendering if all of its props are the same as on the last render.** This is where caching the calculation becomes important! Imagine that you calculated visibleTodos without useMemo:

```jsx
export default function TodoList({ todos, tab, theme }) {
  // Every time the theme changes, this will be a different array...
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ... so List's props will never be the same, and it will re-render every time */}
      <List items={visibleTodos} />
    </div>
  );
}
```

**In the above example, the `filterTodos` function always creates a different array, similar to how the {} object literal always creates a new object.** Normally, this wouldn’t be a problem, but it means that List props will never be the same, and your memo optimization won’t work. This is where useMemo comes in handy:

```jsx
export default function TodoList({ todos, tab, theme }) {
  // Tell React to cache your calculation between re-renders...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...so as long as these dependencies don't change...
  );
  return (
    <div className={theme}>
      {/* ...List will receive the same props and can skip re-rendering */}
      <List items={visibleTodos} />
    </div>
  );
}
```

By wrapping the `visibleTodos` calculation in `useMemo`, you ensure that it has the same value between the re-renders (until dependencies change).  You don’t have to wrap a calculation in useMemo unless you do it for some specific reason. In this example, the reason is that you pass it to a component wrapped in memo, and this lets it skip re-rendering.

### Memoizing a dependency of another Hook 
Suppose you have a calculation that depends on an object created directly in the component body:

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 Caution: Dependency on an object created in the component body
  // ...
```

Depending on an object like this defeats the point of memoization. When a component re-renders, all of the code directly inside the component body runs again. The lines of code creating the searchOptions object will also run on every re-render. Since searchOptions is a dependency of your useMemo call, and it’s different every time, React knows the dependencies are different, and recalculate searchItems every time.

To fix this, you could memoize the searchOptions object itself before passing it as a dependency:

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ Only changes when text changes

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ Only changes when allItems or searchOptions changes
  // ...
```

In the example above, if the text did not change, the searchOptions object also won’t change. However, an even better fix is to move the searchOptions object declaration inside of the useMemo calculation function:

```jsx
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ Only changes when allItems or text changes
  // ...
```

Now your calculation depends on text directly (which is a string and can’t “accidentally” become different).

### Memoizing a function 
Suppose the `Form` component is wrapped in `memo`. You want to pass a function to it as a prop:

```jsx
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

Just as `{}` creates a different object, function declarations like `function() {}` and expressions like `() => {}` produce a different function on every re-render. By itself, creating a new function is not a problem. This is not something to avoid! However, if the Form component is memoized, presumably you want to skip re-rendering it when no props have changed. A prop that is *always* different would defeat the point of memoization.

To memoize a function with `useMemo`, your calculation function would have to return another function:

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

This looks clunky! **Memoizing functions is common enough that React has a built-in Hook specifically for that. Wrap your functions into `useCallback` instead of `useMemo`** to avoid having to write an extra nested function:

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

**The two examples above are completely equivalent. The only benefit to `useCallback` is that it lets you avoid writing an extra nested function inside. It doesn’t do anything else.**

# `useCallback` Hook
`useCallback` is a React Hook that lets you cache a function definition between re-renders.

```jsx
const cachedFn = useCallback(fn, dependencies)
```

## Reference 
### `useCallback(fn, dependencies) `
Call useCallback at the top level of your component to cache a function definition between re-renders:

```jsx
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

#### Parameters 
+ `fn`: The function value that you want to cache. It can take any arguments and return any values. React will return (not call!) your function back to you during the initial render. On next renders, React will give you the same function again if the dependencies have not changed since the last render. Otherwise, it will give you the function that you have passed during the current render, and store it in case it can be reused later. React will not call your function. The function is returned to you so you can decide when and whether to call it.

+ `dependencies`: The list of all reactive values referenced inside of the `fn` code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is configured for React, it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like [dep1, dep2, dep3]. React will compare each dependency with its previous value using the Object.is comparison algorithm.

#### Returns 
On the initial render, useCallback returns the `fn` function you have passed.

During subsequent renders, it will either return an already stored `fn` function from the last render (if the dependencies haven’t changed), or return the `fn` function you have passed during this render.

#### Caveats 
+ useCallback is a Hook, so you can only call it at the top level of your component or your own Hooks. You can’t call it inside loops or conditions. If you need that, extract a new component and move the state into it.

## Usage 
`useCallback` caches a function between re-renders until its dependencies change.

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

### How is useCallback related to useMemo? 
You will often see useMemo alongside useCallback. They are both useful when you’re trying to optimize a child component. They let you memoize (or, in other words, cache) something you’re passing down:

```jsx
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { // Calls your function and caches its result
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // Caches your function itself
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

The difference is in what they’re letting you cache:

+ **`useMemo` caches the result of calling your function.** In this example, it caches the result of calling computeRequirements(product) so that it doesn’t change unless product has changed. This lets you pass the requirements object down without unnecessarily re-rendering ShippingForm. When necessary, React will call the function you’ve passed during rendering to calculate the result.
+ **`useCallback` caches the function itself.** Unlike useMemo, it does not call the function you provide. Instead, it caches the function you provided so that handleSubmit itself doesn’t change unless productId or referrer has changed. This lets you pass the handleSubmit function down without unnecessarily re-rendering ShippingForm. Your code won’t run until the user submits the form.

# `memo` Function
`memo` lets you skip re-rendering a component when its props are unchanged.

```jsx
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```

## Reference 
### `memo(Component, arePropsEqual?) `
Wrap a component in memo to get a memoized version of that component. This memoized version of your component will usually not be re-rendered when its parent component is re-rendered as long as its props have not changed. But React may still re-render it: memoization is a performance optimization, not a guarantee.

```jsx
import { memo } from 'react';

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

#### Parameters 
+ `Component`: The component that you want to memoize. The memo does not modify this component, but returns a new, memoized component instead. Any valid React component, including functions and forwardRef components, is accepted.
+ **optional** `arePropsEqual`: A function that accepts two arguments: the component’s previous props, and its new props. It should return true if the old and new props are equal: that is, if the component will render the same output and behave in the same way with the new props as with the old. Otherwise it should return false. Usually, you will not specify this function. By default, React will compare each prop with Object.is.

#### Returns
`memo` returns a new React component. It behaves the same as the component provided to `memo` except that React will not always re-render it when its parent is being re-rendered unless its props have changed.

## Usage
### Skipping re-rendering when props are unchanged
React normally re-renders a component whenever its parent re-renders. With memo, you can create a component that React will not re-render when its parent re-renders so long as its new props are the same as the old props. Such a component is said to be memoized.

To memoize a component, wrap it in memo and use the value that it returns in place of your original component:

```jsx
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

A React component should always have pure rendering logic. This means that it must return the same output if its props, state, and context haven’t changed. By using memo, you are telling React that your component complies with this requirement, so React doesn’t need to re-render as long as its props haven’t changed. Even with memo, your component will re-render if its own state changes or if a context that it’s using changes.

Note: You should only rely on memo as a performance optimization. If your code doesn’t work without it, find the underlying problem and fix it first. Then you may add memo to improve performance.

### Updating a memoized component using state 
Even when a component is memoized, it will still re-render when its own state changes. Memoization only has to do with props that are passed to the component from its parent.

If you set a state variable to its current value, React will skip re-rendering your component even without memo. You may still see your component function being called an extra time, but the result will be discarded.

### Updating a memoized component using a context
Even when a component is memoized, it will still re-render when a context that it’s using changes. Memoization only has to do with props that are passed to the component from its parent.

To make your component re-render only when a part of some context changes, split your component in two. Read what you need from the context in the outer component, and pass it down to a memoized child as a prop.

### Minimizing props changes 
When you use `memo`, your component re-renders whenever any prop is not shallowly equal to what it was previously. This means that React compares every prop in your component with its previous value using the `Object.is` comparison. Note that **Object.is(3, 3)** is **true**, but **Object.is({}, {})** is **false**.

To get the most out of `memo`, minimize the times that the props change. For example, if the prop is an object, prevent the parent component from re-creating that object every time by using `useMemo`:

```jsx
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  const person = useMemo(
    () => ({ name, age }),
    [name, age]
  );

  return <Profile person={person} />;
}

const Profile = memo(function Profile({ person }) {
  // ...
});
```

A better way to minimize props changes is to make sure the component accepts the minimum necessary information in its props. For example, it could accept individual values instead of a whole object:

```jsx
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);
  return <Profile name={name} age={age} />;
}

const Profile = memo(function Profile({ name, age }) {
  // ...
});
```

Even individual values can sometimes be projected to ones that change less frequently. For example, here a component accepts a boolean indicating the presence of a value rather than the value itself:

```jsx
function GroupsLanding({ person }) {
  const hasGroups = person.groups !== null;
  return <CallToAction hasGroups={hasGroups} />;
}

const CallToAction = memo(function CallToAction({ hasGroups }) {
  // ...
});
```

**When you need to pass a function to memoized component, either declare it outside your component so that it never changes, or `useCallback` to cache its definition between re-renders.**

### Specifying a custom comparison function 
In rare cases it may be infeasible to minimize the props changes of a memoized component. In that case, you can provide a custom comparison function, which React will use to compare the old and new props instead of using shallow equality. This function is passed as a second argument to memo. It should return true only if the new props would result in the same output as the old props; otherwise it should return false.

```jsx
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```

If you do this, use the Performance panel in your browser developer tools to make sure that your comparison function is actually faster than re-rendering the component. You might be surprised.

## Troubleshooting 
### My component re-renders when a prop is an object, array, or function 
React compares old and new props by shallow equality: that is, it considers whether each new prop is reference-equal to the old prop. If you create a new object or array each time the parent is re-rendered, even if the individual elements are each the same, React will still consider it to be changed. Similarly, if you create a new function when rendering the parent component, React will consider it to have changed even if the function has the same definition. To avoid this, **simplify props or memoize props in the parent component.**

---
### Knowledge Check

**Q1.** Why should you prefer useRef hook over other DOM manipulation methods like querySelector?

**A1.** Using `useRef` in React has several advantages over traditional DOM manipulation methods like `querySelector`.

#### 1. Performance

`useRef` provides a direct reference to a DOM element or component instance without causing any re-renders when the referenced value changes. This is more efficient than querying the DOM repeatedly using `querySelector`, which can be a performance bottleneck, especially in large applications.

#### 2. Readability and Maintainability

Using `useRef` makes the code more readable and easier to maintain. It clearly indicates that you are working with a reference to a specific DOM element or component, making the code easier to understand for other developers.

#### 3. React Conventions

`useRef` aligns with React's declarative programming paradigm. React encourages a declarative approach where you describe what you want to achieve, and React takes care of updating the DOM efficiently. `useRef` allows you to work within this paradigm by providing a way to reference DOM elements or components without directly manipulating the DOM.

#### 4. Integration with React Lifecycle

`useRef` integrates well with React's lifecycle methods. You can use `useEffect` to perform side effects like adding event listeners or performing DOM manipulations when the component mounts or updates. This ensures that the side effects are managed by React and are cleaned up properly.

#### Example:

Here's a simple example to illustrate the difference:

#### Using `useRef`:

```jsx
import React, { useRef, useEffect } from 'react';

function MyComponent() {
  const buttonRef = useRef(null);

  useEffect(() => {
    const handleButtonClick = () => {
      console.log('Button clicked!');
    };

    if (buttonRef.current) {
      buttonRef.current.addEventListener('click', handleButtonClick);
    }

    return () => {
      if (buttonRef.current) {
        buttonRef.current.removeEventListener('click', handleButtonClick);
      }
    };
  }, []);

  return (
    <button ref={buttonRef}>
      Click me
    </button>
  );
}
```

#### Using `querySelector`:

```jsx
import React, { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    const button = document.querySelector('button');

    const handleButtonClick = () => {
      console.log('Button clicked!');
    };

    if (button) {
      button.addEventListener('click', handleButtonClick);
    }

    return () => {
      if (button) {
        button.removeEventListener('click', handleButtonClick);
      }
    };
  }, []);

  return (
    <button>
      Click me
    </button>
  );
}
```

In the example using `useRef`, the `buttonRef` directly references the button element, making the code easier to understand and maintain. In contrast, the example using `querySelector` queries the DOM to find the button element, which is less efficient and less readable.

#### Conclusion

While there are situations where direct DOM manipulation using `querySelector` might be necessary, in most cases, using `useRef` in combination with React's declarative programming paradigm is preferred for better performance, readability, and maintainability.

**Q2.** What is the difference between useMemo and useCallback?

**A2.** Both `useMemo` and `useCallback` are React hooks that are used to optimize performance by memoizing values and functions, respectively. However, they are used for different purposes and have different use cases.

#### `useMemo`

The `useMemo` hook is used to memoize the result of a computation and recompute the result only when one of the dependencies has changed.

#### Syntax:

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

#### Example:

```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculation({ a, b }) {
  const expensiveValue = useMemo(() => {
    console.log('Computing expensive value...');
    return a * b;
  }, [a, b]);

  return <div>Expensive Value: {expensiveValue}</div>;
}

function App() {
  const [valueA, setValueA] = useState(2);
  const [valueB, setValueB] = useState(3);

  return (
    <div>
      <button onClick={() => setValueA(valueA + 1)}>Increment A</button>
      <button onClick={() => setValueB(valueB + 1)}>Increment B</button>
      <ExpensiveCalculation a={valueA} b={valueB} />
    </div>
  );
}

export default App;
```

In this example, `expensiveValue` is computed using `useMemo`, and it will only recompute when `valueA` or `valueB` changes.

#### `useCallback`

The `useCallback` hook is used to memoize a function instance and re-create the function only when one of the dependencies has changed.

#### Syntax:

```javascript
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

#### Example:

```javascript
import React, { useState, useCallback } from 'react';

function MemoizedButton({ onClick }) {
  return <button onClick={onClick}>Click Me</button>;
}

function App() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log('Button clicked!');
    setCount(count + 1);
  }, [count]);

  return (
    <div>
      <MemoizedButton onClick={handleClick} />
      <div>Count: {count}</div>
    </div>
  );
}

export default App;
```

In this example, `handleClick` is a memoized function created using `useCallback`, and it will only re-create when `count` changes.

#### Key Differences:

1. **Purpose**:
    - `useMemo`: Memoizes the result of a computation.
    - `useCallback`: Memoizes a function instance.

2. **Return Value**:
    - `useMemo`: Returns the memoized value.
    - `useCallback`: Returns the memoized callback function.

3. **Use Case**:
    - Use `useMemo` when you want to memoize a computed value to avoid unnecessary re-computations.
    - Use `useCallback` when you want to memoize a callback function to avoid unnecessary re-creations of the function instance.

4. **Dependencies**:
    - Both `useMemo` and `useCallback` accept an array of dependencies. The memoized value or function will be re-computed or re-created when any of the dependencies change.

#### Conclusion

`useMemo` and `useCallback` are both useful for optimizing performance by memoizing values and functions in React components. Choose `useMemo` when you want to memoize a computed value, and choose `useCallback` when you want to memoize a callback function.

**Q3.** How do useMemo and useCallback help optimize the performance of React components?

**A3.** useMemo and useCallback are both hooks in React that help optimize the performance of components by memoizing values and functions, respectively. By memoizing expensive computations or functions, React can avoid unnecessary re-calculations and re-renders, leading to improved performance and a better user experience.

- **`useMemo`** is used to memoize the result of a computation, preventing unnecessary re-calculations and re-renders of components.
  
- **`useCallback`** is used to memoize a callback function, preventing unnecessary re-creations of the function instance and re-renders of components that use the callback.

**Q4.** When should you memoize a value?

**A4.** <br>
Optimizing with `useMemo` is only valuable in a few cases:
+ The calculation you’re putting in `useMemo` is noticeably slow, and its dependencies rarely change.
+ You pass it as a prop to a component wrapped in `memo`. You want to skip re-rendering if the value hasn’t changed. Memoization lets your component re-render only when dependencies aren’t the same.
+ The value you’re passing is later used as a dependency of some Hook. For example, maybe another `useMemo` calculation value depends on it. Or maybe you are depending on this value from `useEffect`.

There is no benefit to wrapping a calculation in useMemo in other cases. There is no significant harm to doing that either, so some teams choose to not think about individual cases, and memoize as much as possible. The downside of this approach is that code becomes less readable. Also, not all memoization is effective: a single value that’s “always new” is enough to break memoization for an entire component.

Caching a function with `useCallback`  is only valuable in a few cases:
+ You pass it as a prop to a component wrapped in `memo`. You want to skip re-rendering if the value hasn’t changed. Memoization lets your component re-render only if dependencies changed.
+ The function you’re passing is later used as a dependency of some Hook. For example, another function wrapped in `useCallback` depends on it, or you depend on this function from `useEffect`.

There is no benefit to wrapping a function in `useCallback` in other cases. There is no significant harm to doing that either, so some teams choose to not think about individual cases, and memoize as much as possible. The downside is that code becomes less readable. Also, not all memoization is effective: a single value that’s “always new” is enough to break memoization for an entire component.

Note that `useCallback` does not prevent creating the function. You’re always creating a function (and that’s fine!), but React ignores it and gives you back a cached function if nothing changed.
