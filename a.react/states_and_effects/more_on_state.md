# More On State
## State as a Snapshot
### Setting state triggers renders
Setting state requests a re-render from React. This means that for an interface to react to the event, you need to *update the state*.

In the following example, when you press “send”, `setIsSent(true)` tells React to re-render the UI:

```jsx
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

Here’s what happens when you click the button:

1. The onSubmit event handler executes.
2. setIsSent(true) sets isSent to true and queues a new render.
3. React re-renders the component according to the new isSent value.

When you call `useState`, React gives you a snapshot of the state for that render. Every render (and functions inside it) will always “see” the snapshot of the state that React gave to *that* render.

Example:

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>Add</button>
    </>
  )
}
```

Notice that number only increments once per click!

Setting state only changes it for the next render. During the first render, number was 0. This is why, in that render’s onClick handler, the value of number is still 0 even after setNumber(number + 1) was called:

Here is what this button’s click handler tells React to do:

1. setNumber(number + 1): number is 0 so setNumber(0 + 1).
    + React prepares to change number to 1 on the next render.
2. setNumber(number + 1): number is 0 so setNumber(0 + 1).
    + React prepares to change number to 1 on the next render.
3. setNumber(number + 1): number is 0 so setNumber(0 + 1).
    + React prepares to change number to 1 on the next render.

Even though you called setNumber(number + 1) three times, in this render’s event handler number is always 0, so you set the state to 1 three times. This is why, after your event handler finishes, React re-renders the component with number equal to 1 rather than 3.

You can also visualize this by mentally substituting state variables with their values in your code. Since the number state variable is 0 for this render, its event handler looks like this:

```jsx
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>Add</button>
```

## Queueing a Series of State Updates
Setting a state variable will queue another render. But sometimes you might want to perform multiple operations on the value before queueing the next render. To do this, it helps to understand how React batches state updates.

### React batches state updates
React batches state updates for performance optimization. Batching state updates means that multiple state updates within a single event handler or lifecycle method are grouped together and applied all at once, rather than individually. This optimization helps reduce the number of re-renders and DOM manipulations, leading to better performance.

React achieves batching through a mechanism called "transaction." When you call `setState()` multiple times within the same event handler or lifecycle method, React does not immediately update the state and re-render the component for each call. Instead, it queues up the state updates and performs them in batches before rendering. This way, React ensures that the component is rendered only once with all the accumulated state changes.

React batching mechanism is typically implemented at the level of the event loop or through its own internal mechanisms, depending on the environment (e.g., browser, server-side rendering). This ensures that state updates within the same execution context are batched together.

By batching state updates, React reduces the number of DOM manipulations and optimizations needed, resulting in improved performance and smoother user experience.

### Updating the same state variable multiple times before the next render
It is an uncommon use case, but if you would like to update the same state variable multiple times before the next render, instead of passing the next state value like `setNumber(number + 1)`, you can pass a function that calculates the next state based on the previous one in the queue, like `setNumber(n => n + 1)`. It is a way to tell React to “do something with the state value” instead of just replacing it.

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

Here, `n => n + 1` is called an **updater function**. In this example, clicking “+3” correctly increments the value by 3.

In React, an updater function is a way to update state based on the previous state. It's commonly used when you need to update state based on its current value, especially in cases where the next state depends on the previous state.

The setState() function in React can accept either an object or a function as an argument. When you pass a function as an argument to setState(), React calls that function with the previous state and props as arguments, and expects it to return an object representing the new state, based on the previous state and props.

Using updater functions is beneficial when you need to ensure that state updates are based on the most up-to-date state of the component, especially in scenarios where multiple setState() calls are being batched together or when the next state depends on the previous state.

### What happens if you update state after replacing it

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}

```

Here’s what this event handler tells React to do:

1. `setNumber(number + 5)`: number is `0`, so `setNumber(0 + 5)`. React adds *“replace with 5”* to its queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds that *function* to its queue.

During the next render, React goes through the state queue.

React stores `6` as the final result and returns it from `useState`.

### What happens if you replace state after updating it 

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```

Here’s how React works through these lines of code while executing this event handler:

1. `setNumber(number + 5)`: number is `0`, so `setNumber(0 + 5)`. React adds *“replace with 5”* to its queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds that *function* to its queue.
3. `setNumber(42)`: React adds *“replace with 42”* to its queue.

During the next render(remember that "render" in context of React is React calling the component function again), React goes through the state queue.

Then React stores `42` as the final result and returns it from `useState`.

## Updating Objects in State
State can hold any kind of JavaScript value, including objects. But you shouldn’t change objects that you hold in the React state directly. Instead, when you want to update an object, you need to create a new one (or make a copy of an existing one), and then set the state to use that copy.

### What’s a mutation? 
You can store any kind of JavaScript value in state.

```jsx
const [x, setX] = useState(0);
```

So far you’ve been working with numbers, strings, and booleans. These kinds of JavaScript values are “immutable”, meaning unchangeable or “read-only”. You can trigger a re-render to *replace* a value:

```jsx
setX(5);
```

The x state changed from 0 to 5, but the number 0 itself did not change. It’s not possible to make any changes to the built-in primitive values like numbers, strings, and booleans in JavaScript.

Now consider an object in state:

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Technically, it is possible to change the contents of the *object itself*. **This is called a mutation**:

```jsx
position.x = 5;
```

However, although objects in React state are technically mutable, you should treat them as if they were immutable—like numbers, booleans, and strings. Instead of mutating them, you should always replace them.

### Treat state as read-only 
In other words, you should **treat any JavaScript object that you put into state as read-only**.

In React, when you directly mutate a state variable without using the `setState()` function or equivalent state management mechanisms provided by React (such as `useState` hook), React won't be aware of the change. As a result, React's reconciliation process, which is responsible for updating the UI in response to state changes, won't be triggered.

This can lead to the program's state and the UI being out of sync because React relies on the virtual DOM diffing algorithm to efficiently update the actual DOM based on changes in state. When you mutate state directly, React isn't able to detect those changes and reconcile them with the virtual DOM, resulting in inconsistencies between the application's state and the rendered UI.

To ensure that the UI reflects the current state of your application accurately, it's crucial to always update state using React's state management mechanisms, such as `setState()` or `useState()`. This way, React can properly track state changes and trigger the necessary re-renders to keep the UI synchronized with the application's state.

### Copying objects with the spread syntax 
Often, you will want to include existing data as a part of the new object that you are creating and passing to the setState() function.

These input fields don’t work because the onChange handlers mutate the state:

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }

  function handleEmailChange(e) {
    person.email = e.target.value;
  }

  return (
    <>
      <label>
        First name:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

The reliable way to get the behavior you’re looking for is to create a new object and pass it to setPerson. But here, you want to also copy the existing data into it because only one of the fields has changed:

```jsx
  function handleFirstNameChange(e) {
    setPerson({
      firstName: e.target.value, // New first name from the input
      lastName: person.lastName,
      email: person.email
    });
  }
```

You can use the `...` `object spread syntax` so that you don’t need to copy every property separately.

```jsx
function handleFirstNameChange(e) {
  setPerson({
    ...person, // Copy the old fields
    firstName: e.target.value // But override this one
  });
}
```

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value
    });
  }

  return (
    <>
      <label>
        First name:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

Now the form works!

Notice how you didn’t declare a separate state variable for each input field. For large forms, keeping all data grouped in an object is very convenient—as long as you update it correctly.

**Important: The ... spread syntax is “shallow”—it only copies things one level deep.**

### Updating a "nested" object 
Consider a nested object structure like this:

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```

If you wanted to update person.artwork.city, it’s clear how to do it with mutation:

```jsx
person.artwork.city = 'New Delhi';
```

But in React, you treat state as immutable! In order to change `city`, you would first need to produce the new `artwork` object (pre-populated with data from the previous one), and then produce the new `person` object which points at the new `artwork`:

```jsx
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

Or, written as a single function call:

```jsx
setPerson({
  ...person, // Copy other fields
  artwork: { // but replace the artwork
    ...person.artwork, // with the same one
    city: 'New Delhi' // but in New Delhi!
  }
});
```

Of course the example `person` object  here only has a single object it is pointing with it's `artwork` property. Deep copying more deeply nested object can get a little bit verbose. **As an alternative you can use the `_.cloneDeep()` method from the `Lodash` library.**

More importantly, Lodash's `_.cloneDeep()` method works for both objects and arrays, as well as for cases where arrays contain objects, objects contain arrays, or any combination thereof. It recursively traverses the entire structure, creating a deep clone of every nested object and array encountered. This ensures that the cloned structure is completely independent of the original, with no shared references at any level of nesting. So, no matter how complex the data structure is, _.cloneDeep() will ensure a thorough and accurate deep clone.

### Write concise update logic with Immer 
If your state is deeply nested, you might want to consider flattening it. But, if you don’t want to change your state structure, you might prefer a shortcut to nested spreads. `Immer` is a popular library that lets you write using the convenient but mutating syntax and takes care of producing the copies for you. With Immer, the code you write looks like you are “breaking the rules” and mutating an object:

```jsx
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

But unlike a regular mutation, it doesn’t overwrite the past state!

#### How does Immer work?
The `draft` provided by Immer is a special type of object, called a Proxy, that “records” what you do with it. This is why you can mutate it freely as much as you like! Under the hood, Immer figures out which parts of the `draft` have been changed, and produces a completely new object that contains your edits.

To try Immer:

1. Run `npm install use-immer` to add Immer as a dependency.
2. Then add `import { useImmer } from 'use-immer'` to the file.

Here is the above example converted to Immer:

```jsx
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson(draft => {
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img 
        src={person.artwork.image} 
        alt={person.artwork.title}
      />
    </>
  );
}
```

Notice how much more concise the event handlers have become. You can mix and match `useState` and `useImmer` in a single component as much as you like. `Immer` is a great way to keep the update handlers concise, especially if there’s nesting in your state, and copying objects leads to repetitive code.

## Updating Arrays in State
Arrays are mutable in JavaScript, but you should treat them as immutable when you store them in state. Just like with objects, when you want to update an array stored in state, you need to create a new one (or make a copy of an existing one), and then set state to use the new array.

### Updating arrays without mutation
In JavaScript, arrays are just another kind of object. Like with objects, you should **treat arrays in React state as read-only.** This means that you shouldn’t reassign items inside an array like `arr[0] = 'bird'`, and you also shouldn’t use methods that mutate the array, such as `push()` and `pop()`.

Instead, every time you want to update an array, you’ll want to pass a new array to your state setting function. To do that, you can create a new array from the original array in your state by calling its non-mutating methods like `filter()` and `map()`. Then you can set your state to the resulting new array.

![Array methods and React](../../../img/javascript/state-array.jpg "Array methods and React")

### Adding to an array
`push()` will mutate an array, which you don’t want:

```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>Add</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

Instead, create a new array which contains the existing items and a new item at the end. There are multiple ways to do this, but the easiest one is to use the `...` array spread syntax:

```jsx
setArtists( // Replace the state
  [ // with a new array
    ...artists, // that contains all the old items
    { id: nextId++, name: name } // and one new item at the end
  ]
);
```

Now it works correctly:

```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setArtists([
          ...artists,
          { id: nextId++, name: name }
        ]);
      }}>Add</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

The array spread syntax also lets you prepend an item by placing it before the original `...artists`:

```jsx
setArtists([
  { id: nextId++, name: name },
  ...artists // Put old items at the end
]);
```

In this way, spread can do the job of both push() by adding to the end of an array and unshift() by adding to the beginning of an array.

**Remember: The spread syntax (`...`) only performs a shallow copy, which means it creates a new array but does not create new copies of nested arrays or objects. If your state variable contains nested structures and you need to ensure that all levels of nesting are copied without referencing the original objects, Lodash's `_.cloneDeep()` ensures that each level of the nested structure is duplicated recursively, resulting in a completely independent copy of the original data.**

**So, for shallow copies, the spread syntax is efficient and sufficient, but for deep copies, especially when dealing with nested arrays or objects, Lodash's `_.cloneDeep()` provides a more robust solution.**

### Removing from an array 
The easiest way to remove an item from an array is to filter it out. In other words, you will produce a new array that will not contain that item. To do this, use the `filter` method. For example:

```jsx
import { useState } from 'react';

let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [artists, setArtists] = useState(
    initialArtists
  );

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a =>
                  a.id !== artist.id
                )
              );
            }}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

### Transforming an array
If you want to change some or all items of the array, you can use `map()` to create a new array. The map() method of Array instances creates a **new** array populated with the results of calling a provided function on every element in the calling array.

In this example, an array holds coordinates of two circles and a square. When you press the button, it moves only the circles down by 50 pixels. It does this by producing a new array of data using map():

```jsx
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(
    initialShapes
  );

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // No change
        return shape;
      } else {
        // Return a new circle 50px below
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // Re-render with the new array
    setShapes(nextShapes);
  }

  return (
    <>
      <button onClick={handleClick}>
        Move circles down!
      </button>
      {shapes.map(shape => (
        <div
          key={shape.id}
          style={{
          background: 'purple',
          position: 'absolute',
          left: shape.x,
          top: shape.y,
          borderRadius:
            shape.type === 'circle'
              ? '50%' : '',
          width: 20,
          height: 20,
        }} />
      ))}
    </>
  );
}
```

### Replacing items in an array 
It is particularly common to want to replace one or more items in an array. Assignments like `arr[0] = 'bird'` are mutating the original array, so instead you’ll want to use `map` for this as well.

To replace an item, create a new array with map. Inside your map call, you will receive the item index as the second argument. Use it to decide whether to return the original item (the first argument) or something else:

```jsx
import { useState } from 'react';

let initialCounters = [
  0, 0, 0
];

export default function CounterList() {
  const [counters, setCounters] = useState(
    initialCounters
  );

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // Increment the clicked counter
        return c + 1;
      } else {
        // The rest haven't changed
        return c;
      }
    });
    setCounters(nextCounters);
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}
          <button onClick={() => {
            handleIncrementClick(i);
          }}>+1</button>
        </li>
      ))}
    </ul>
  );
}
```

### Inserting into an array 
Sometimes, you may want to insert an item at a particular position that’s neither at the beginning nor at the end. To do this, you can use the ... array spread syntax together with the slice() method. The slice() method lets you cut a “slice” of the array. To insert an item, you will create an array that spreads the slice before the insertion point, then the new item, and then the rest of the original array.

In this example, the Insert button always inserts at the index 1:

```jsx
import { useState } from 'react';

let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // Could be any index
    const nextArtists = [
      // Items before the insertion point:
      ...artists.slice(0, insertAt),
      // New item:
      { id: nextId++, name: name },
      // Items after the insertion point:
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={handleClick}>
        Insert
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

### Making other changes to an array
There are some things you can’t do with the spread syntax and non-mutating methods like map() and filter() alone. For example, you may want to reverse or sort an array. The JavaScript reverse() and sort() methods are mutating the original array, so you can’t use them directly.

However, you can copy the array first, and then make changes to it.

For example:

```jsx
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list];
    nextList.reverse();
    setList(nextList);
  }

  return (
    <>
      <button onClick={handleClick}>
        Reverse
      </button>
      <ul>
        {list.map(artwork => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

Remember that the ... spread syntax creates a shallow copy.

### Updating objects inside arrays 
Objects are not really located “inside” arrays. They might appear to be “inside” in code, but each object in an array is a separate value, to which the array “points”.

When updating nested state, you need to create copies from the point where you want to update, and all the way up to the top level. **In order to achieve that, you can use the `_.cloneDeep()` method from the `Lodash` library.**

Lodash's `_.cloneDeep()` method works for both objects and arrays, as well as for cases where arrays contain objects, objects contain arrays, or any combination thereof. It recursively traverses the entire structure, creating a deep clone of every nested object and array encountered. This ensures that the cloned structure is completely independent of the original, with no shared references at any level of nesting. So, no matter how complex the data structure is, _.cloneDeep() will ensure a thorough and accurate deep clone.

### Write concise update logic with Immer 
+ Generally, you shouldn’t need to update state more than a couple of levels deep. If your state objects are very deep, you might want to restructure them differently so that they are flat.
+ If you don’t want to change your state structure, you might prefer to use `Immer`, which lets you write using the convenient but mutating syntax and takes care of producing the copies for you.

```jsx
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer(
    initialList
  );
  const [yourList, updateYourList] = useImmer(
    initialList
  );

  function handleToggleMyList(id, nextSeen) {
    updateMyList(draft => {
      const artwork = draft.find(a =>
        a.id === id
      );
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

Note how with Immer, mutating syntax like `artwork.seen = nextSeen` is now okay.

This is because you’re not mutating the original state, but you’re mutating a special draft object provided by Immer. Similarly, you can apply mutating methods like push() and pop() to the content of the draft.

Behind the scenes, Immer always constructs the next state from scratch according to the changes that you’ve done to the draft. This keeps your event handlers very concise without ever mutating state.

## Reacting to Input with State
React provides a declarative way to manipulate the UI. Instead of manipulating individual pieces of the UI directly, you describe the different states that your component can be in, and switch between them in response to the user input. This is similar to how designers think about the UI.

Declarative programming means describing the UI for each visual state rather than micromanaging the UI (imperative).

When developing a component:
1. Identify all its visual states.
2. Determine the human and computer triggers for state changes.
3. Model the state with useState.
4. Remove non-essential state to avoid bugs and paradoxes.
5. Connect the event handlers to set state.

## Choosing the State Structure
Structuring state well can make a difference between a component that is pleasant to modify and debug, and one that is a constant source of bugs. Here are some tips you should consider when structuring state.

### Principles for structuring state
When you write a component that holds some state, you’ll have to make choices about how many state variables to use and what the shape of their data should be. While it’s possible to write correct programs even with a suboptimal state structure, there are a few principles that can guide you to make better choices:

1. **Group related state.** If you always update two or more state variables at the same time, consider merging them into a single state variable.

2. **Avoid contradictions in state.** When the state is structured in a way that several pieces of state may contradict and “disagree” with each other, you leave room for mistakes. Try to avoid this.

3. **Avoid redundant state.** If you can calculate some information from the component’s props or its existing state variables during rendering, you should not put that information into that component’s state.

4. **Avoid duplication in state.** When the same data is duplicated between multiple state variables, or within nested objects, it is difficult to keep them in sync. Reduce duplication when you can.

5. **Avoid deeply nested state.** Deeply hierarchical state is not very convenient to update. When possible, prefer to structure state in a flat way.

The goal behind these principles is to make state easy to update without introducing mistakes. Removing redundant and duplicate data from state helps ensure that all its pieces stay in sync. 

### Group related state
You might sometimes be unsure between using a single or multiple state variables.

Should you do this?

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

Or this?

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Technically, you can use either of these approaches. But **if some two state variables always change together, it might be a good idea to unify them into a single state variable**. Then you won’t forget to always keep them in sync.

### Avoid contradictions in state
Here is a hotel feedback form with `isSending` and `isSent` state variables:

```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

While this code works, it leaves the door open for “impossible” states. For example, if you forget to call `setIsSent` and `setIsSending` together, you may end up in a situation where both `isSending` and `isSent` are true at the same time. The more complex your component is, the harder it is to understand what happened.

Since `isSending` and `isSent` should never be true at the same time, **it is better to replace them with one status state variable that may take one of three valid states: `'typing'` (initial), `'sending'`, and `'sent'`:**

```jsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

### Avoid redundant state
If you can calculate some information from the component’s props or its existing state variables during rendering, you should not put that information into that component’s state.

```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

This form has three state variables: `firstName`, `lastName`, and `fullName`. However, `fullName` is redundant. You can always calculate `fullName` from `firstName` and `lastName` during render, so remove it from state.

### Avoid duplication in state 
Consider a simple example where we have a list of tasks that can be marked as completed or not. We want to display this list in two separate components: one for displaying all tasks and another for displaying only the completed tasks.

Instead of duplicating the state of the tasks in both components, we'll follow the "Avoid duplication in state" principle by managing the task list in a single parent component and passing down the necessary data to child components via props.

```jsx
import React, { useState } from 'react';

function TaskList() {
  const [tasks, setTasks] = useState([
    { id: 1, title: 'Complete React project', completed: false },
    { id: 2, title: 'Write documentation', completed: true },
    { id: 3, title: 'Test application', completed: false },
  ]);

  const toggleTaskCompletion = (taskId) => {
    setTasks(tasks.map(task => {
      if (task.id === taskId) {
        return { ...task, completed: !task.completed };
      }
      return task;
    }));
  };

  return (
    <div>
      <h2>All Tasks</h2>
      <TaskDisplay tasks={tasks} toggleTaskCompletion={toggleTaskCompletion} />
      <h2>Completed Tasks</h2>
      <CompletedTaskDisplay tasks={tasks.filter(task => task.completed)} toggleTaskCompletion={toggleTaskCompletion} />
    </div>
  );
}

function TaskDisplay({ tasks, toggleTaskCompletion }) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <input
            type="checkbox"
            checked={task.completed}
            onChange={() => toggleTaskCompletion(task.id)}
          />
          <span>{task.title}</span>
        </li>
      ))}
    </ul>
  );
}

function CompletedTaskDisplay({ tasks, toggleTaskCompletion }) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <input
            type="checkbox"
            checked={true}
            onChange={() => toggleTaskCompletion(task.id)}
            disabled
          />
          <span>{task.title}</span>
        </li>
      ))}
    </ul>
  );
}

export default TaskList;
```

In this example, the `TaskList` component manages the state of tasks and passes down the `tasks` array to both `TaskDisplay` and `CompletedTaskDisplay` components as props. The `toggleTaskCompletion` function is also passed down to allow child components to update the state when a task's completion status is toggled. This way, we avoid duplicating the state and logic for managing tasks in multiple places.

### Avoid deeply nested state 
You can nest state as much as you like, but making it “flat” can solve numerous problems. It makes state easier to update, and it helps ensure you don’t have duplication in different parts of a nested object.

## Sharing State Between Components
Sometimes, you want the state of two components to always change together. To do it, remove state from both of them, move it to their closest common parent, and then pass it down to them via props. This is known as lifting state up, and it’s one of the most common things you will do writing React code.

### Lifting state up
Identify the shared state: Determine which pieces of data need to be shared among multiple components.

Find the common parent: Identify the closest common ancestor of the components that need to share the state. This component will become the owner of the shared state.

Lift the state up: Move the shared state from the individual components into the common parent component. This means that the common parent component will now manage the state, and it will pass down the necessary data to its child components via props.

Update state via callbacks: If child components need to modify the shared state, you can pass down callback functions from the parent component to the children as props. These callback functions can then be called by the child components to update the state in the parent component.

By lifting state up, you can avoid prop drilling (passing props through multiple layers of components) and create a more centralized and maintainable state management system in your React application. It also helps in keeping your application's state consistent across different components.

### Passing an event handler down as a prop
Passing an event handler down as a prop in React is a common pattern used for communication between components. Here's how it works:

1. **Define the event handler**: First, you define a function in the parent component that will handle the event. This function can perform any necessary logic, such as updating state or triggering other actions.

2. **Pass the event handler as a prop**: Once you have the event handler function defined in the parent component, you pass it down to the child component as a prop. This is done in the parent component's JSX when rendering the child component.

3. **Use the event handler in the child component**: In the child component, you receive the event handler function as a prop. You can then use it as an event handler for any relevant events, such as onClick, onChange, etc. When the event occurs in the child component, the event handler function passed from the parent component will be called.

Here's a simple example to illustrate this process:

ParentComponent.js:
```jsx
import React from 'react';
import ChildComponent from './ChildComponent';

function ParentComponent() {
  // Define event handler function
  const handleButtonClick = () => {
    console.log('Button clicked!');
  };

  return (
    <div>
      {/* Pass event handler down as a prop */}
      <ChildComponent onClick={handleButtonClick} />
    </div>
  );
}

export default ParentComponent;
```

ChildComponent.js:
```jsx
import React from 'react';

function ChildComponent(props) {
  // Receive event handler function as a prop
  const { onClick } = props;

  return (
    <div>
      {/* Use event handler in the child component */}
      <button onClick={onClick}>Click me</button>
    </div>
  );
}

export default ChildComponent;
```

In this example, `ParentComponent` defines the `handleButtonClick` event handler function and passes it down to `ChildComponent` as the `onClick` prop. In `ChildComponent`, this prop is received and assigned to a local variable, which is then used as the onClick event handler for a button. When the button is clicked in `ChildComponent`, the `handleButtonClick` function defined in `ParentComponent` will be executed.

Passing event handlers down as props is a very useful pattern for updating state that is defined in a parent component, and it effectively reverses the traditional flow of data in React, which is typically one-way from parent to child.

By passing event handlers down as props, you enable child components to communicate with their parent components and trigger changes in the parent's state. This allows for a more flexible and dynamic architecture where child components can influence the behavior and state of their parent components.

This pattern is particularly useful in scenarios where multiple child components need to interact with the same piece of state that is defined in a common parent component. Instead of each child component managing its own separate state and potentially becoming out of sync, you can lift the shared state up to a common parent component and pass down event handlers as props to allow child components to update that state when necessary.

Overall, passing event handlers down as props is a powerful technique in React for managing state in a more centralized and controlled manner.

### Controlled and uncontrolled components
In React, components can be categorized as controlled or uncontrolled based on how they manage and interact with form elements like input fields, checkboxes, and select dropdowns.

1. **Controlled Components**:
   - In controlled components, form data is managed by React itself.
   - The value of the form element is controlled by React state.
   - When the user interacts with the form element (e.g., typing into an input field), React state is updated with the new value, and the component is re-rendered with the updated value.
   - Changes to the form element are handled through event handlers (such as onChange for input fields) that update the state.

   Example of a controlled component:
   ```jsx
   import React, { useState } from 'react';

   function ControlledComponent() {
     const [value, setValue] = useState('');

     const handleChange = (event) => {
       setValue(event.target.value);
     };

     return (
       <input
         type="text"
         value={value}
         onChange={handleChange}
       />
     );
   }
   ```

2. **Uncontrolled Components**:
   - In uncontrolled components, form data is managed by the DOM itself.
   - The value of the form element is not controlled by React state; instead, it is directly managed by the DOM.
   - Typically, refs are used to access the DOM element to get its value when needed (e.g., onSubmit of a form).
   - Uncontrolled components are useful when you want to integrate React with non-React code or when you want to handle form data directly in DOM events.

   Example of an uncontrolled component:
   ```jsx
   import React, { useRef } from 'react';

   function UncontrolledComponent() {
     const inputRef = useRef(null);

     const handleSubmit = (event) => {
       event.preventDefault();
       console.log('Input value:', inputRef.current.value);
     };

     return (
       <form onSubmit={handleSubmit}>
         <input type="text" ref={inputRef} />
         <button type="submit">Submit</button>
       </form>
     );
   }
   ```

Both controlled and uncontrolled components have their use cases, and the choice between them depends on the specific requirements of your application. Controlled components offer more control and synchronization with React state, while uncontrolled components can be simpler and more suitable for integrating with non-React code or for handling form data directly with DOM events.

### A single source of truth for each state
Having a single source of truth for each state in React means that a particular piece of state is managed in only one place within your application. In other words, there is a centralized location, typically within a component or a state management solution like Redux, where the state is defined and updated.

This concept is closely related to the principle of data immutability and helps to maintain consistency and predictability in your application. Here's what it entails:

1. **Consistency**: When a piece of state has a single source of truth, all components that need access to that state retrieve it from the same location. This ensures that the data is consistent across the application, eliminating the possibility of conflicting or out-of-sync data.

2. **Predictability**: With a single source of truth, the behavior of your application becomes more predictable. Since there's only one place where the state is modified, it's easier to understand and reason about how changes to that state will affect the rest of the application.

3. **Easier debugging and maintenance**: When state is centralized in one location, debugging and maintaining your application become simpler. If there's an issue with the state, you know exactly where to look and can trace how it's being modified more easily.

4. **Avoids duplication and redundancy**: By having a single source of truth, you avoid duplicating state across different components, which can lead to redundancy and inconsistencies. Instead, you can ensure that each piece of state is managed in one place, reducing the risk of bugs and making your codebase more maintainable.

#### To recap:
+ When you want to coordinate two components, instead of duplicating the state, move their state to their common parent.
+ Then pass the information down through props from their common parent.
+ Finally, pass the event handlers down so that the children can change the parent’s state.
+ It’s useful to consider components as “controlled” (driven by props) or “uncontrolled” (driven by state).

## Preserving and Resetting State
### State is tied to a position in the render tree 
React builds render trees for the component structure in your UI.

When you give a component state, you might think the state “lives” inside the component. But the state is actually held inside React. React associates each piece of state it’s holding with the correct component by where that component sits in the render tree.

**React preserves a component’s state for as long as it’s being rendered at its position in the UI tree.** If it gets removed, or a different component gets rendered at the same position, React discards its state.

### Same component at the same position preserves state 
In this example, there are two different \<Counter /> tags:

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

When you tick or clear the checkbox, the counter state does not get reset. Whether `isFancy` is true or false, you always have a `<Counter />` as the first child of the `div` returned from the root `App` component.

It’s the same component at the same position, so from React’s perspective, it’s the same counter.

**Remember that it’s the position in the UI tree—not in the JSX markup—that matters to React!** The following component has two return clauses with different `<Counter />` JSX tags inside and outside the if:

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          Use fancy styling
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

You might expect the state to reset when you tick checkbox, but it doesn’t! This is because both of these `<Counter />` tags are rendered at the same position. React doesn’t know where you place the conditions in your function. All it “sees” is the tree you return.

In both cases, the App component returns a `<div>` with `<Counter />` as a first child. To React, these two counters have the same “address”: the first child of the first child of the root. This is how React matches them up between the previous and next renders, regardless of how you structure your logic.

### Different components at the same position reset state
In this example, ticking the checkbox will replace `<Counter>` with a `<p>`:

```jsx
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

Here, you switch between different component types at the same position. Initially, the first child of the `<div>` contained a `Counter`. But when you swapped in a `p`, React removed the `Counter` from the UI tree and destroyed its state.

Also, **when you render a different component in the same position, it resets the state of its entire subtree.**

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

The counter state gets reset when you click the checkbox. Although you render a `Counter`, the first child of the `div` changes from a `div` to a `section`. When the child `div` was removed from the DOM, the whole tree below it (including the `Counter` and its state) was destroyed as well.

As a rule of thumb, if you want to preserve the state between re-renders, the structure of your tree needs to “match up” from one render to another. If the structure is different, the state gets destroyed because React destroys state when it removes a component from the tree.

### Resetting state at the same position 
By default, React preserves state of a component while it stays at the same position. Usually, this is exactly what you want, so it makes sense as the default behavior. But sometimes, you may want to reset a component’s state. Consider this app that lets two players keep track of their scores during each turn:

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

Currently, when you change the player, the score is preserved. The two `Counter`s appear in the same position, so React sees them as the same `Counter` whose `person` prop has changed.

But conceptually, in this app they should be two separate counters. They might appear in the same place in the UI, but one is a counter for Taylor, and another is a counter for Sarah.

There are two ways to reset state when switching between them:

1. Render components in different positions
1. Give each component an explicit identity with `key`

#### Option 1: Rendering a component in different positions
If you want these two Counters to be independent, you can render them in two different positions:

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

Initially, isPlayerA is true. So the first position contains Counter state, and the second one is empty.
When you click the “Next player” button the first position clears but the second one now contains a Counter.

Each Counter’s state gets destroyed each time it’s removed from the DOM. This is why they reset every time you click the button.

This solution is convenient when you only have a few independent components rendered in the same place. In this example, you only have two, so it’s not a hassle to render both separately in the JSX.

#### Option 2: Resetting state with a key
There is also another, more generic, way to reset a component’s state.

You might have seen keys when rendering lists. Keys aren’t just for lists! You can use keys to make React distinguish between any components. By default, React uses order within the parent (“first counter”, “second counter”) to discern between components. But keys let you tell React that this is not just a first counter, or a second counter, but a specific counter—for example, Taylor’s counter. This way, React will know Taylor’s counter wherever it appears in the tree!

In this example, the two `<Counter />`s don’t share state even though they appear in the same place in JSX:

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

Switching between Taylor and Sarah does not preserve the state. This is because you gave them different keys.

Specifying a `key` tells React to use the `key` itself as part of the position, instead of their order within the parent. This is why, even though you render them in the same place in JSX, React sees them as two different counters, and so they will never share state. Every time a counter appears on the screen, its state is created. Every time it is removed, its state is destroyed. Toggling between them resets their state over and over.

### Resetting a form with a key
Resetting state with a key is particularly useful when dealing with forms.

In below example when you select a different recipient, the `Chat` component will be recreated from scratch, including any state in the tree below it. React will also re-create the DOM elements instead of reusing them.

In the example, switching the recipient always clears the text field:

```jsx
// App.jsx

import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```jsx
// ContactList.jsx

export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```jsx
// Chat.jsx

import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Chat to ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Send to {contact.email}</button>
    </section>
  );
}
```

### Preserving state for removed components
In a real chat app, you’d probably want to recover the input state when the user selects the previous recipient again. There are a few ways to keep the state “alive” for a component that’s no longer visible:

+ You could render all chats instead of just the current one, but hide all the others with CSS. The chats would not get removed from the tree, so their local state would be preserved. This solution works great for simple UIs. But it can get very slow if the hidden trees are large and contain a lot of DOM nodes.

+ You could lift the state up and hold the pending message for each recipient in the parent component. This way, when the child components get removed, it doesn’t matter, because it’s the parent that keeps the important information. This is the most common solution.

+ You might also use a different source in addition to React state. For example, you probably want a message draft to persist even if the user accidentally closes the page. To implement this, you could have the `Chat` component initialize its state by reading from the `localStorage`, and save the drafts there too.

No matter which strategy you pick, a chat with Alice is conceptually distinct from a chat with Bob, so it makes sense to give a `key` to the `<Chat>` tree based on the current recipient.

## Extracting State Logic into a Reducer
Components with many state updates spread across many event handlers can get overwhelming. For these cases, you can consolidate all the state update logic outside your component in a single function, called a **reducer**.

### Consolidate state logic with a reducer
As your components grow in complexity, it can get harder to see at a glance all the different ways in which a component’s state gets updated. . For example, the `TaskApp` component below holds an array of `tasks` in state and uses three different event handlers to add, remove, and edit tasks:

```jsx
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];
```

Each of its event handlers calls `setTasks` in order to update the state. As this component grows, so does the amount of state logic sprinkled throughout it. To reduce this complexity and keep all your logic in one easy-to-access place, you can move that state logic into a single function outside your component, called a **“reducer”**.

Reducers are a different way to handle state. You can migrate from `useState` to `useReducer` in three steps:

1. Move from setting state to dispatching actions.
2. Write a reducer function.
3. Use the reducer from your component.

### Step 1: Move from setting state to dispatching actions
Your event handlers currently specify what to do by setting state:

```jsx
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

Remove all the state setting logic. What you are left with are three event handlers:

+ `handleAddTask(text)` is called when the user presses “Add”.
+ `handleChangeTask(task)` is called when the user toggles a task or presses “Save”.
+ `handleDeleteTask(taskId)` is called when the user presses “Delete”.

Managing state with reducers is slightly different from directly setting state. Instead of telling React “what to do” by setting state, you specify “what the user just did” by dispatching “actions” from your event handlers. (The state update logic will live elsewhere!) So instead of “setting `tasks`” via an event handler, you’re dispatching an “added/changed/deleted a task” action. This is more descriptive of the user’s intent.

```jsx
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

The object you pass to `dispatch` is called an “action”:

```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // "action" object:
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

It is a regular JavaScript object. You decide what to put in it, but generally it should contain the minimal information about what happened.

### Step 2: Write a reducer function 
A reducer function is where you will put your state logic. It takes two arguments, the current state and the action object, and it returns the next state:

```jsx
function yourReducer(state, action) {
  // return next state for React to set
}
```

React will set the state to what you return from the reducer.

To move your state setting logic from your event handlers to a reducer function in this example, you will:

1. Declare the current state (tasks) as the first argument.
2. Declare the action object as the second argument.
3. Return the next state from the reducer (which React will set the state to).

Here is all the state setting logic migrated to a reducer function:

```jsx
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

Because the reducer function takes state (tasks) as an argument, you can declare it outside of your component. This decreases the indentation level and can make your code easier to read.

Note: The code above uses if/else statements, but it’s a convention to use switch statements inside reducers. The result is the same, but it can be easier to read switch statements at a glance.

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

We recommend wrapping each case block into the { and } curly braces so that variables declared inside of different cases don’t clash with each other. Also, a case should usually end with a return. If you forget to return, the code will “fall through” to the next case, which can lead to mistakes!

### Step 3: Use the reducer from your component 
Finally, you need to hook up the `tasksReducer` to your component. Import the `useReducer` Hook from React:

```jsx
import { useReducer } from 'react';
```

Then you can replace `useState`:

```jsx
const [tasks, setTasks] = useState(initialTasks);
```

with `useReducer` like so:

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

The `useReducer` Hook is similar to `useState`—you must pass it an initial state and it returns a stateful value and a way to set state (in this case, the dispatch function). But it’s a little different.

The `useReducer` Hook takes two arguments:

+ A reducer function
+ An initial state

And it returns:

+ A stateful value
+ A dispatch function (to “dispatch” user actions to the reducer)

Now it’s fully wired up! Here, the reducer is declared at the bottom of the component file:

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];
```

Component logic can be easier to read when you separate concerns like this. Now the event handlers only specify what happened by dispatching actions, and the reducer function determines how the state updates in response to them.

### Comparing `useState` and `useReducer`
Reducers are not without downsides! Here’s a few ways you can compare them:

+ **Code size:** Generally, with useState you have to write less code upfront. With useReducer, you have to write both a reducer function and dispatch actions. However, useReducer can help cut down on the code if many event handlers modify state in a similar way.

+ **Readability:** useState is very easy to read when the state updates are simple. When they get more complex, they can bloat your component’s code and make it difficult to scan. In this case, useReducer lets you cleanly separate the how of update logic from the what happened of event handlers.

+ **Debugging:** When you have a bug with useState, it can be difficult to tell where the state was set incorrectly, and why. With useReducer, you can add a console log into your reducer to see every state update, and why it happened (due to which action). If each action is correct, you’ll know that the mistake is in the reducer logic itself. However, you have to step through more code than with useState.

+ **Testing:** A reducer is a pure function that doesn’t depend on your component. This means that you can export and test it separately in isolation. While generally it’s best to test components in a more realistic environment, for complex state update logic it can be useful to assert that your reducer returns a particular state for a particular initial state and action.

Personal preference: Some people like reducers, others don’t. That’s okay. It’s a matter of preference. You can always convert between useState and useReducer back and forth: they are equivalent!

We recommend using a reducer if you often encounter bugs due to incorrect state updates in some component, and want to introduce more structure to its code. You don’t have to use reducers for everything: feel free to mix and match! You can even useState and useReducer in the same component.

### Writing reducers well
Keep these two tips in mind when writing reducers:

**Reducers must be pure.** Similar to state updater functions, reducers run during rendering! (Actions are queued until the next render.) This means that reducers must be pure—same inputs always result in the same output. They should not send requests, schedule timeouts, or perform any side effects (operations that impact things outside the component). They should update objects and arrays without mutations.

**Each action describes a single user interaction, even if that leads to multiple changes in the data.** For example, if a user presses “Reset” on a form with five fields managed by a reducer, it makes more sense to dispatch one reset_form action rather than five separate set_field actions. If you log every action in a reducer, that log should be clear enough for you to reconstruct what interactions or responses happened in what order. This helps with debugging!

### Writing concise reducers with Immer 
Just like with updating objects and arrays in regular state, you can use the `Immer` library to make reducers more concise. Here, `useImmerReducer` lets you mutate the state with `push` or `arr[i] =` assignment:

```jsx
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];
```

Reducers must be pure, so they shouldn’t mutate state. But Immer provides you with a special draft object which is safe to mutate. Under the hood, Immer will create a copy of your state with the changes you made to the draft. This is why reducers managed by useImmerReducer can mutate their first argument and don’t need to return state.

### Recap
+ To convert from useState to useReducer:
    1. Dispatch actions from event handlers.
    2. Write a reducer function that returns the next state for a given state and action.
    3. Replace useState with useReducer.
+ Reducers require you to write a bit more code, but they help with debugging and testing.
+ Reducers must be pure.
+ Each action describes a single user interaction.
+ Use Immer if you want to write reducers in a mutating style.

## Passing Data Deeply with Context
Usually, you will pass information from a parent component to a child component via props. But passing props can become verbose and inconvenient if you have to pass them through many components in the middle, or if many components in your app need the same information. Context lets the parent component make some information available to any component in the tree below it—no matter how deep—without passing it explicitly through props.

### The problem with passing props
Passing props is a great way to explicitly pipe data through your UI tree to the components that use it.

But passing props can become verbose and inconvenient when you need to pass some prop deeply through the tree, or if many components need the same prop. The nearest common ancestor could be far removed from the components that need data, and lifting state up that high can lead to a situation called “prop drilling”.

Wouldn’t it be great if there were a way to “teleport” data to the components in the tree that need it without passing props? With React’s context feature, there is!

### Context: an alternative to passing props
Context lets a parent component provide data to the entire tree below it. There are many uses for context. Here is one example. Consider this `Heading` component that accepts a `level` for its size:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Heading level={2}>Heading</Heading>
      <Heading level={3}>Sub-heading</Heading>
      <Heading level={4}>Sub-sub-heading</Heading>
      <Heading level={5}>Sub-sub-sub-heading</Heading>
      <Heading level={6}>Sub-sub-sub-sub-heading</Heading>
    </Section>
  );
}
```

Let’s say you want multiple headings within the same `Section` to always have the same size:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Currently, you pass the `level` prop to each `<Heading>` separately:

```jsx
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

It would be nice if you could pass the `level` prop to the `<Section>` component instead and remove it from the `<Heading>`. This way you could enforce that all headings in the same section have the same size:

```jsx
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

But how can the `<Heading>` component know the `level` of its closest `<Section>`? **That would require some way for a child to “ask” for data from somewhere above in the tree.**

You can’t do it with props alone. This is where context comes into play. You will do it in three steps:

1. Create a context. (You can call it `LevelContext`, since it’s for the heading level.)
2. Use that context from the component that needs the data. (`Heading` will use `LevelContext`.)
3. Provide that context from the component that specifies the data. (`Section` will provide `LevelContext`.)

Context lets a parent—even a distant one!—provide some data to the entire tree inside of it.

### Step 1: Create the context 
First, you need to create the context. You’ll need to export it from a file so that your components can use it:

```jsx
// LevelContext.jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

The only argument to `createContext` is the *default value*. Here, `1` refers to the biggest heading level, but you could pass any kind of value (even an object). You will see the significance of the default value in the next step.

### Step 2: Use the context
Import the `useContext` Hook from React and your context:

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

Currently, the `Heading` component reads `level` from props:

```jsx
// Heading.jsx

export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

Instead, remove the `level` prop and read the value from the context you just imported, `LevelContext`:

```jsx
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext` is a Hook. Just like `useState` and `useReducer`, you can only call a Hook immediately inside a React component (not inside loops or conditions). **`useContext` tells React that the `Heading` component wants to read the `LevelContext`.**

Now that the `Heading` component doesn’t have a `level` prop, you don’t need to pass the `level` prop to `Heading` in your JSX like this anymore:

```jsx
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

Update the JSX so that it’s the `Section` that receives it instead:

```jsx
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

As a reminder, this is the markup that you were trying to get working:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Notice this example doesn’t quite work, yet! All the headings have the same size(1) because even though you’re using the context, you have not provided it yet. React doesn’t know where to get it!

If you don’t provide the context, React will use the default value you’ve specified in the previous step. In this example, you specified `1` as the argument to `createContext`, so `useContext(LevelContext)` returns `1`, setting all those headings to `<h1>`. Let’s fix this problem by having each `Section` provide its own context.

### Step 3: Provide the context
The `Section` component currently renders its children:

```jsx
// Section.js

export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Wrap them with a context provider** to provide the `LevelContext` to them:

```jsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

This tells React: “if any component inside this `<Section>` asks for `LevelContext`, give them this `level`.” The component will use the value of the nearest `<LevelContext.Provider>` in the UI tree above it.

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

It’s the same result as the original code, but you did not need to pass the `level` prop to each `Heading` component! Instead, it “figures out” its heading level by asking the closest `Section` above:

1. You pass a level prop to the `<Section>`.
1. Section wraps its children into `<LevelContext.Provider value={level}>`.
1. Heading asks the closest value of LevelContext above with `useContext(LevelContext)`.

### Using and providing context from the same component 
Currently, you still have to specify each section’s `level` manually:

```jsx
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

Since context lets you read information from a component above, each `Section` could read the `level` from the `Section` above, and pass `level + 1` down automatically. Here is how you could do it:

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

With this change, you don’t need to pass the `level` prop either to the `<Section>` or to the `<Heading>`:

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

Now both `Heading` and `Section` read the `LevelContext` to figure out how “deep” they are. And the `Section` wraps its children into the `LevelContext` to specify that anything inside of it is at a “deeper” level.

Note: This example uses heading levels because they show visually how nested components can override context. But context is useful for many other use cases too. You can pass down any information needed by the entire subtree: the current color theme, the currently logged in user, and so on.

### Context passes through intermediate components
You can insert as many components as you like between the component that provides context and the one that uses it. This includes both built-in components like `<div>` and components you might build yourself.

**Context lets you write components that “adapt to their surroundings” and display themselves differently depending on where (or, in other words, in which context) they are being rendered.**

How context works might remind you of CSS property inheritance. In CSS, you can specify `color: blue` for a `<div>`, and any DOM node inside of it, no matter how deep, will inherit that color unless some other DOM node in the middle overrides it with `color: green`. Similarly, in React, the only way to override some context coming from above is to wrap children into a context provider with a different value.

In CSS, different properties like `color` and `background-color` don’t override each other. You can set all  `<div>`’s `color` to red without impacting `background-color`. Similarly, different React contexts don’t override each other. Each context that you make with `createContext()` is completely separate from other ones, and ties together components using and providing that particular context. One component may use or provide many different contexts without a problem.

### Use cases for context 
+ **Theming:** If your app lets the user change its appearance (e.g. dark mode), you can put a context provider at the top of your app, and use that context in components that need to adjust their visual look.

+ **Current account:** Many components might need to know the currently logged in user. Putting it in context makes it convenient to read it anywhere in the tree. Some apps also let you operate multiple accounts at the same time (e.g. to leave a comment as a different user). In those cases, it can be convenient to wrap a part of the UI into a nested provider with a different current account value.

+ **Routing:** Most routing solutions use context internally to hold the current route. This is how every link “knows” whether it’s active or not. If you build your own router, you might want to do it too.

+ **Managing state:** As your app grows, you might end up with a lot of state closer to the top of your app. Many distant components below may want to change it. It is common to use a reducer together with context to manage complex state and pass it down to distant components without too much hassle.

Context is not limited to static values. If you pass a different value on the next render, React will update all the components reading it below! This is why context is often used in combination with state.

In general, if some information is needed by distant components in different parts of the tree, it’s a good indication that context will help you.

### Recap
+ Context lets a component provide some information to the entire tree below it.
+ To pass context:
    + Create and export it with `export const MyContext = createContext(defaultValue)`.
    + Pass it to the `useContext(MyContext)` Hook to read it in any child component, no matter how deep.
    + Wrap children into `<MyContext.Provider value={...}>` to provide it from a parent.
+ Context passes through any components in the middle.

## Scaling Up with Reducer and Context
Reducers let you consolidate a component’s state update logic. Context lets you pass information deep down to other components. You can combine reducers and context together to manage state of a complex screen.

### Combining a reducer with context 
In this example from the introduction to reducers, the state is managed by a reducer. The reducer function contains all of the state update logic and is declared at the bottom of this file:

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Day off in Kyoto</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

A reducer helps keep the event handlers short and concise. However, as your app grows, you might run into another difficulty. Currently, the `tasks` state and the `dispatch` function are only available in the top-level `TaskApp` component. To let other components read the list of tasks or change it, you have to explicitly pass down the current state and the event handlers that change it as props.

As an alternative to passing them through props, you might want to put both the `tasks` state and the `dispatch` function into context. This way, any component below `TaskApp` in the tree can read the tasks and dispatch actions without the repetitive “prop drilling”.

Here is how you can combine a reducer with context:

1. Create the context.
2. Put state and dispatch into context.
3. Use context anywhere in the tree.

### Step 1: Create the context 
The `useReducer` Hook returns the current `tasks` and the `dispatch` function that lets you update them:

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

To pass them down the tree, you will create two separate contexts:

+ `TasksContext` provides the current list of tasks.
+ `TasksDispatchContext` provides the function that lets components dispatch actions.

Export them from a separate file so that you can later import them from other files:

```jsx
// TasksContext.jsx

import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

Here, you’re passing `null` as the default value to both contexts. The actual values will be provided by the `TaskApp` component.

### Step 2: Put state and dispatch into context 
Now you can import both contexts in your `TaskApp` component. Take the `tasks` and `dispatch` returned by `useReducer()` and provide them to the entire tree below:

```jsx
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

For now, you pass the information both via props and in context:

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Day off in Kyoto</h1>
        <AddTask
          onAddTask={handleAddTask}
        />
        <TaskList
          tasks={tasks}
          onChangeTask={handleChangeTask}
          onDeleteTask={handleDeleteTask}
        />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

In the next step, you will remove prop passing.

### Step 3: Use context anywhere in the tree 
Now you don’t need to pass the list of tasks or the event handlers down the tree:

```jsx
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

Instead, any component that needs the task list can read it from the `TaskContext`:

```jsx
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

To update the task list, any component can read the `dispatch` function from context and call it:

```jsx
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```

The `TaskApp` component does not pass any event handlers down, and the `TaskList` does not pass any event handlers to the `Task` component either. Each component reads the context that it needs.

The state still “lives” in the top-level `TaskApp` component, managed with `useReducer`. But its tasks and dispatch are now available to every component below in the tree by importing and using these contexts.

### Moving all wiring into a single file 
You don’t have to do this, but you could further declutter the components by moving both reducer and context into a single file. Currently, TasksContext.jsx contains only two context declarations:

```jsx
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

This file is about to get crowded! You’ll move the reducer into that same file. Then you’ll declare a new `TasksProvider` component in the same file. This component will tie all the pieces together:

1. It will manage the state with a reducer.
2. It will provide both contexts to components below.
3. It will take `children` as a prop so you can pass JSX to it.

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

This removes all the complexity and wiring from your `TaskApp` component:

```jsx
// App.jsx

import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```jsx
// TasksContext.jsx

import { createContext, useReducer } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

You can also export functions that use the context from `TasksContext.js`:

```jsx
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

When a component needs to read context, it can do it through these functions:

```jsx
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

This doesn’t change the behavior in any way, but it lets you later split these contexts further or add some logic to these functions. Now all of the context and reducer wiring is in `TasksContext.js`. This keeps the components clean and uncluttered, focused on what they display rather than where they get the data:


```jsx
// App.jsx

import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```jsx
// TasksContext.jsx

import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);

const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

```jsx
// AddTask.jsx

import { useState } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        }); 
      }}>Add</button>
    </>
  );
}

let nextId = 3;
```

```jsx
// TaskList.jsx

import { useState } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Delete
      </button>
    </label>
  );
}
```

You can think of `TasksProvider` as a part of the screen that knows how to deal with tasks, `useTasks` as a way to read them, and `useTasksDispatch` as a way to update them from any component below in the tree.

**Note: Functions like `useTasks` and `useTasksDispatch` are called `Custom Hooks`. Your function is considered a custom Hook if its name starts with `use`. This lets you use other Hooks, like `useContext`, inside it.**

As your app grows, you may have many context-reducer pairs like this. This is a powerful way to scale your app and lift state up without too much work whenever you want to access the data deep in the tree.

---
### Knowledge Check

**Q1.** What should you keep in mind while declaring state?

**A1.** Structuring state well can make a difference between a component that is pleasant to modify and debug, and one that is a constant source of bugs. Here are some tips you should consider when structuring state.

#### Principles for structuring state
When you write a component that holds some state, you’ll have to make choices about how many state variables to use and what the shape of their data should be. While it’s possible to write correct programs even with a suboptimal state structure, there are a few principles that can guide you to make better choices:

1. **Group related state.** If you always update two or more state variables at the same time, consider merging them into a single state variable.

2. **Avoid contradictions in state.** When the state is structured in a way that several pieces of state may contradict and “disagree” with each other, you leave room for mistakes. Try to avoid this.

3. **Avoid redundant state.** If you can calculate some information from the component’s props or its existing state variables during rendering, you should not put that information into that component’s state.

4. **Avoid duplication in state.** When the same data is duplicated between multiple state variables, or within nested objects, it is difficult to keep them in sync. Reduce duplication when you can.

5. **Avoid deeply nested state.** Deeply hierarchical state is not very convenient to update. When possible, prefer to structure state in a flat way.

The goal behind these principles is to make state easy to update without introducing mistakes. Removing redundant and duplicate data from state helps ensure that all its pieces stay in sync.

**Q2.** Why should we always use setState to update our state?

**A2.** **State should not be mutated.** Mutating state is a no-go area in React as it leads to unpredictable results. Primitives are already immutable, but if you are using reference type values i.e. arrays and objects, never mutate them. According to React documentation, we should treat state as if it was immutable. In order for us to change state, we should always use the setState function.

State can hold any kind of JavaScript value, including objects. But you shouldn’t change objects that you hold in the React state directly. Instead, when you want to update an object(or an array which is an another kind of object in Javascript), **you need to create a new one (or make a copy of an existing one), and then set the state to use that copy.**

#### What’s a mutation? 
You can store any kind of JavaScript value in state.

```jsx
const [x, setX] = useState(0);
```

So far you’ve been working with numbers, strings, and booleans. These kinds of JavaScript values are “immutable”, meaning unchangeable or “read-only”. You can trigger a re-render to *replace* a value:

```jsx
setX(5);
```

The x state changed from 0 to 5, but the number 0 itself did not change. It’s not possible to make any changes to the built-in primitive values like numbers, strings, and booleans in JavaScript.

Now consider an object in state:

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Technically, it is possible to change the contents of the *object itself*. **This is called a mutation**:

```jsx
position.x = 5;
```

However, although objects in React state are technically mutable, you should treat them as if they were immutable—like numbers, booleans, and strings. Instead of mutating them, you should always replace them.

#### Treat state as read-only 
In other words, you should **treat any JavaScript object that you put into state as read-only**.

In React, when you directly mutate a state variable without using the `setState()` function or equivalent state management mechanisms provided by React (such as `useState` hook), React won't be aware of the change. As a result, React's reconciliation process, which is responsible for updating the UI in response to state changes, won't be triggered.

This can lead to the program's state and the UI being out of sync because React relies on the virtual DOM diffing algorithm to efficiently update the actual DOM based on changes in state. When you mutate state directly, React isn't able to detect those changes and reconcile them with the virtual DOM, resulting in inconsistencies between the application's state and the rendered UI.

To ensure that the UI reflects the current state of your application accurately, it's crucial to always update state using React's state management mechanisms, such as `setState()` or `useState()`. This way, React can properly track state changes and trigger the necessary re-renders to keep the UI synchronized with the application's state. Also, as we said before, when working with reference types(objects and arrays), you should pass a new object or array to the setState() function in order to avoid mutation.

**Q3.** What does “state as a snapshot” mean?

**A3.** <br>
#### Setting state triggers renders
Setting state requests a re-render from React. This means that for an interface to react to the event, you need to *update the state*.

In the following example, when you press “send”, `setIsSent(true)` tells React to re-render the UI:

```jsx
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

Here’s what happens when you click the button:

1. The onSubmit event handler executes.
2. setIsSent(true) sets isSent to true and queues a new render.
3. React re-renders(calls again) the component according to the new isSent value.

When you call `useState`, React gives you a snapshot of the state for that render. Every render (and functions inside it) will always “see” the snapshot of the state that React gave to *that* render.

Example:

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>Add</button>
    </>
  )
}
```

Notice that number only increments once per click!

Setting state only changes it for the next render. During the first render, number was 0. This is why, in that render’s onClick handler, the value of number is still 0 even after setNumber(number + 1) was called:

Here is what this button’s click handler tells React to do:

1. setNumber(number + 1): number is 0 so setNumber(0 + 1).
    + React prepares to change number to 1 on the next render.
2. setNumber(number + 1): number is 0 so setNumber(0 + 1).
    + React prepares to change number to 1 on the next render.
3. setNumber(number + 1): number is 0 so setNumber(0 + 1).
    + React prepares to change number to 1 on the next render.

Even though you called setNumber(number + 1) three times, in this render’s event handler number is always 0, so you set the state to 1 three times. This is why, after your event handler finishes, React re-renders the component with number equal to 1 rather than 3.

You can also visualize this by mentally substituting state variables with their values in your code. Since the number state variable is 0 for this render, its event handler looks like this:

```jsx
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>Add</button>
```

**Q4.** What’s the difference between passing a value vs a callback to the setState function?

**A4.** <br>
#### Updating the same state variable multiple times before the next render
It is an uncommon use case, but if you would like to update the same state variable multiple times before the next render, instead of passing the next state value like `setNumber(number + 1)`, you can pass a function that calculates the next state based on the previous one in the queue, like `setNumber(n => n + 1)`. It is a way to tell React to “do something with the state value” instead of just replacing it.

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

Here, `n => n + 1` is called an **updater function**. In this example, clicking “+3” correctly increments the value by 3.

In React, an updater function is a way to update state based on the previous state. It's commonly used when you need to update state based on its current value, especially in cases where the next state depends on the previous state.

The setState() function in React can accept either an object or a function as an argument. When you pass a function as an argument to setState(), React calls that function with the previous state and props as arguments, and expects it to return an object representing the new state, based on the previous state and props.

Using updater functions is beneficial when you need to ensure that state updates are based on the most up-to-date state of the component, especially in scenarios where multiple setState() calls are being batched together or when the next state depends on the previous state.

#### What happens if you update state after replacing it

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}

```

Here’s what this event handler tells React to do:

1. `setNumber(number + 5)`: number is `0`, so `setNumber(0 + 5)`. React adds *“replace with 5”* to its queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds that *function* to its queue.

During the next render, React goes through the state queue.

React stores `6` as the final result and returns it from `useState`.

#### What happens if you replace state after updating it 

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```

Here’s how React works through these lines of code while executing this event handler:

1. `setNumber(number + 5)`: number is `0`, so `setNumber(0 + 5)`. React adds *“replace with 5”* to its queue.
2. `setNumber(n => n + 1)`: `n => n + 1` is an updater function. React adds that *function* to its queue.
3. `setNumber(42)`: React adds *“replace with 42”* to its queue.

During the next render(remember that "render" in context of React is React calling the component function again), React goes through the state queue.

Then React stores `42` as the final result and returns it from `useState`.

**Q5.** Why should we always provide a new Object to setState?

**A5.** **State should not be mutated.** Mutating state is a no-go area in React as it leads to unpredictable results. Primitives are already immutable, but if you are using reference type values i.e. arrays and objects, never mutate them. According to React documentation, we should treat state as if it was immutable. In order for us to change state, we should always use the setState function.

State can hold any kind of JavaScript value, including objects. But you shouldn’t change objects that you hold in the React state directly. Instead, when you want to update an object(or an array which is an another kind of object in Javascript), **you need to create a new one (or make a copy of an existing one), and then set the state to use that copy.**

#### What’s a mutation? 
You can store any kind of JavaScript value in state.

```jsx
const [x, setX] = useState(0);
```

So far you’ve been working with numbers, strings, and booleans. These kinds of JavaScript values are “immutable”, meaning unchangeable or “read-only”. You can trigger a re-render to *replace* a value:

```jsx
setX(5);
```

The x state changed from 0 to 5, but the number 0 itself did not change. It’s not possible to make any changes to the built-in primitive values like numbers, strings, and booleans in JavaScript.

Now consider an object in state:

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Technically, it is possible to change the contents of the *object itself*. **This is called a mutation**:

```jsx
position.x = 5;
```

However, although objects in React state are technically mutable, you should treat them as if they were immutable—like numbers, booleans, and strings. Instead of mutating them, you should always replace them.

#### Treat state as read-only 
In other words, you should **treat any JavaScript object that you put into state as read-only**.

In React, when you directly mutate a state variable without using the `setState()` function or equivalent state management mechanisms provided by React (such as `useState` hook), React won't be aware of the change. As a result, React's reconciliation process, which is responsible for updating the UI in response to state changes, won't be triggered.

This can lead to the program's state and the UI being out of sync because React relies on the virtual DOM diffing algorithm to efficiently update the actual DOM based on changes in state. When you mutate state directly, React isn't able to detect those changes and reconcile them with the virtual DOM, resulting in inconsistencies between the application's state and the rendered UI.

To ensure that the UI reflects the current state of your application accurately, it's crucial to always update state using React's state management mechanisms, such as `setState()` or `useState()`. This way, React can properly track state changes and trigger the necessary re-renders to keep the UI synchronized with the application's state. Also, as we said before, when working with reference types(objects and arrays), you should pass a new object or array to the setState() function in order to avoid mutation.

**Q6.** Why would you want to control a component?

**A6.** There are native HTML elements that maintain their own internal state. The `input` element is a great example. You type into an `input` and it updates its own value on every keystroke. For many use-cases, you would like to control the value of the `input` element i.e. set its value yourself. This is where controlled components come in.

```jsx
function CustomInput() {
  const [value, setValue] = useState('');

  return (
    <input
        type="text"
        value={value}
        onChange={(event) => setValue(event.target.value)}
    />
  );
}
```

Instead of letting the `input` maintain its own state, we define our own state using the `useState` hook. We then set the `value` prop of the `input` to the state variable and update the state variable on every `onChange` event. Now, every time the user types something in the `input`, React will ensure you have the latest comment/review/post (whatever the user was typing) in `value`.

This pattern is extremely useful wherever you need user input i.e. typing in a textbox, toggling a checkbox etc.