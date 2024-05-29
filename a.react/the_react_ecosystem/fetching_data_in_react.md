# Fetching Data In React
## A Basic Fetch Request
Before we dive into the specifics of fetching data in React, let’s briefly revisit how we can use the fetch API to get data from a server.

```js
const image = document.querySelector("img");
fetch("https://jsonplaceholder.typicode.com/photos", {
  mode: "cors",
})
  .then((response) => response.json())
  .then((response) => {
    image.src = response[0].url;
  })
  .catch((error) => console.error(error));
```

We’re making a request to the JSONPlaceholder API to retrieve an image, and then setting that URL to the src of an `<img>` element.

## Using fetch in React components
Now, let’s take a look at how we can incorporate `fetch` into a similar React component. One common use case is to fetch data from an API when a component mounts, so that the data can be displayed on screen.

Whenever a component needs to make a request as it renders, it’s often best to wrap that `fetch` inside of an Effect.

```jsx
import { useEffect, useState } from "react";

const Image = () => {
  const [imageURL, setImageURL] = useState(null);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/photos", { mode: "cors" })
      .then((response) => response.json())
      .then((response) => setImageURL(response[0].url))
      .catch((error) => console.error(error));
  }, []);

  return (
    imageURL && (
      <>
        <h1>An image</h1>
        <img src={imageURL} alt={"placeholder text"} />
      </>
    )
  );
};

export default Image;
```

useState lets us add the imageURL state, whereas useEffect allows us to perform side effects. In this case, the side effect is fetching data from an external API. Since we need to fetch the data only once when the component mounts, we pass an empty dependency array.

## Handling errors
Working over the network is inherently unreliable. The API you’re making a request to might be down, there could be network connectivity issues, or the response you receive could contain errors. A multitude of things can go wrong, and if you don’t preemptively plan for errors, your website can break or appear unresponsive to users.

To simulate a network error, scroll up to the previous code snippet and change the fetch URL to something random. After a refresh of your browser window, the page will remain a blank white screen, without giving the user any indication that the page has finished loading or that there was an error.

To fix this, we need to check for *something* before `Image` component returns JSX. We’ll call it: `error`.

```jsx
if (error) return <p>A network error was encountered</p>

return (
  imageURL && (
    <>
      <h1>An image</h1>
      <img src={imageURL} alt={"placeholder text"} />
    </>
  )
);
```

To set this `error`, we’ll add it to the component’s state.

```jsx
const [imageURL, setImageURL] = useState(null);
const [error, setError] = useState(null);
```

And finally, we modify the `catch` call accordingly:

```jsx
useEffect(() => {
  fetch("https://jsonplaceholder.typicode.com/photos", { mode: "cors" })
    .then((response) => {
      if (response.status >= 400) {
        throw new Error("server error");
      }
      return response.json();
    })
    .then((response) => setImageURL(response[0].url))
    .catch((error) => setError(error));
}, []);
```

Note: Notice how we also handle errors in the then block? This is because the fetch request itself might not fail, but rather complete successfully and yield a response. However, the response received may not be what our app expected. To handle this case, we check the response status codes.

Now when a bad URL is passed or the API returns an unexpected response, the page will relay that information to the user.

## "Loading" state
In the same way we added an error value in state to check for errors, we can also add a loading value to check whether the request is resolved or not.

```jsx
const Image = () => {
  const [imageURL, setImageURL] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/photos", { mode: "cors" })
      .then((response) => {
        if (response.status >= 400) {
          throw new Error("server error");
        }
        return response.json();
      })
      .then((response) => setImageURL(response[0].url))
      .catch((error) => setError(error))
      .finally(() => setLoading(false));
  }, []);

  if (error) return <p>A network error was encountered</p>;
  if (loading) return <p>Loading...</p>;

  return (
    <>
      <h1>An image</h1>
      <img src={imageURL} alt={"placeholder text"} />
    </>
  );
};
```

## Using custom hooks
We can separate out the fetching logic altogether into a custom hook. This will allow us to make the logic reusable and easily testable.

Here is how we could create a `useImageURL` custom hook for our example:

```jsx
import { useState, useEffect } from "react";

const useImageURL = () => {
  const [imageURL, setImageURL] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/photos", { mode: "cors" })
      .then((response) => {
        if (response.status >= 400) {
          throw new Error("server error");
        }
        return response.json();
      })
      .then((response) => setImageURL(response[0].url))
      .catch((error) => setError(error))
      .finally(() => setLoading(false));
  }, []);

  return { imageURL, error, loading };
};

const Image = () => {
  const { imageURL, error, loading } = useImageURL();

  if (error) return <p>A network error was encountered</p>;
  if (loading) return <p>Loading...</p>;

  return (
    <>
      <h1>An image</h1>
      <img src={imageURL} alt={"placeholder text"} />
    </>
  );
};
```

## Managing multiple fetch requests
### How to handle race conditions
Many apps use Effects to kick off data fetching. It is quite common to write a data fetching Effect like this:

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

It doesn’t matter where `page` and `query` come from. While this component is visible, you want to keep `results` synchronized with data from the network for the current `page` and `query`. This is why it’s an Effect.

However, the code above has a bug. Imagine you type "hello" fast. Then the `query` will change from "h", to "he", "hel", "hell", and "hello". This will kick off separate fetches, but there is no guarantee about which order the responses will arrive in. For example, the "hell" response may arrive after the "hello" response. Since it will call setResults() last, you will be displaying the wrong search results. This is called a “race condition”: two different requests “raced” against each other and came in a different order than you expected.

**To fix the race condition, you need to add a cleanup function to ignore stale responses:**

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

This ensures that when your Effect fetches data, all responses except the last requested one will be ignored.

Handling race conditions is not the only difficulty with implementing data fetching. You might also want to think about caching responses (so that the user can click Back and see the previous screen instantly), how to fetch data on the server (so that the initial server-rendered HTML contains the fetched content instead of a spinner), and how to avoid network waterfalls (so that a child can fetch data without waiting for every parent).

If you don’t use a framework (and don’t want to build your own) but would like to make data fetching from Effects more ergonomic, consider extracting your fetching logic into a custom Hook like in this example:

```jsx
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

You’ll likely also want to add some logic for error handling and to track whether the content is loading. You can build a Hook like this yourself or use one of the many solutions already available in the React ecosystem. 

Also, in general, whenever you have to resort to writing Effects, keep an eye out for when you can extract a piece of functionality into a custom Hook with a more declarative and purpose-built API like useData above. The fewer raw useEffect calls you have in your components, the easier you will find to maintain your application.

### How to handle network waterfalls
Let’s start with laying out components first, then wire the data fetching afterward. We’ll have the App component itself, it will render Sidebar and Issue, and Issue will render Comments.

```jsx
const App = () => {
  return (
    <>
      <Sidebar />
      <Issue />
    </>
  );
};

const Sidebar = () => {
  return; // some sidebar links
};

const Issue = () => {
  return (
    <>
      // some issue data
      <Comments />
    </>
  );
};

const Comments = () => {
  return; // some issue comments
};
```

Now to the data fetching. Let’s first extract the actual fetch and useEffect and state management into a nice hook, to simplify the examples:

```jsx
export const useData = (url) => {
  const [state, setState] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await (await fetch(url)).json();

      setState(data);
    };

    dataFetch();
  }, [url]);

  return { data: state };
};
```

Now we will call the useData hook from the components themselves. And would also want to show the loading state while we’re waiting of course!

```jsx
const Comments = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData('/get-comments');

  // show loading state while waiting for the data
  if (!data) return 'loading';

  // rendering comments now that we have access to them!
  return data.map((comment) => <div>{comment.title}</div>);
};
```

And exactly the same code for Issue, only it will render Comments component after loading:

```jsx
const Issue = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData('/get-issue');

  // show loading state while waiting for the data
  if (!data) return 'loading';

  // render actual issue now that the data is here!
  return (
    <div>
      <h3>{data.title}</h3>
      <p>{data.description}</p>
      <Comments />
    </div>
  );
};
```

And the App itself:

```jsx
const App = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData('/get-sidebar');

  // show loading state while waiting for the data
  if (!data) return 'loading';

  return (
    <>
      <Sidebar data={data} />
      <Issue />
    </>
  );
};
```

What we implemented here is a classic waterfall of requests. In React, only components that are actually **returned** will be mounted, rendered, and as a result, will trigger useEffect and data fetching in it. In our case, every single component returns "loading" state while it waits for data. And only when data is loaded, do they switch to a component next in the render tree, it triggers its own data fetching, returns “loading” state and the cycle repeats itself.

Waterfalls like that are not the best solution when you need to show the app as fast as possible. Likely, there are a few ways to deal with them.

#### The `Promise.all` solution
The first and easiest solution is to pull all those data fetching requests as high in the render tree as possible. In our case, it’s our root component App. But there is a catch there: you can’t just “move” them and leave as-is. We can’t just do something like this:

```jsx
useEffect(async () => {
  const sidebar = await fetch('/get-sidebar');
  const issue = await fetch('/get-issue');
  const comments = await fetch('/get-comments');
}, []);
```

This is just yet another waterfall, only co-located in a single component: we fetch sidebar data, await for it, then fetch issue, await, fetch comments, await. The time when all the data will be available for render will be the sum of all those waiting times: 1s + 2s + 3s = 6 seconds. Instead, we need to fire them all at the same time, so that they are sent in parallel. That way we will be waiting for all of them no longer than the longest of them: 3 seconds. 50% performance improvement!

One way to do it is to use Promise.all:

```jsx
useEffect(async () => {
  const [sidebar, issue, comments] = await Promise.all([
    fetch('/get-sidebar'),
    fetch('/get-issue'),
    fetch('/get-comments'),
  ]);
}, []);
```

and then save all of them to state in the parent component and pass them down to the children components as props:

```jsx
const useAllData = () => {
  const [sidebar, setSidebar] = useState();
  const [comments, setComments] = useState();
  const [issue, setIssue] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      // waiting for allthethings in parallel
      const result = (
        await Promise.all([
          fetch(sidebarUrl),
          fetch(issueUrl),
          fetch(commentsUrl),
        ])
      ).map((r) => r.json());

      // and waiting a bit more - fetch API is cumbersome
      const [sidebarResult, issueResult, commentsResult] =
        await Promise.all(result);

      // when the data is ready, save it to state
      setSidebar(sidebarResult);
      setIssue(issueResult);
      setComments(commentsResult);
    };

    dataFetch();
  }, []);

  return { sidebar, comments, issue };
};

const App = () => {
  // all the fetches were triggered in parallel
  const { sidebar, comments, issue } = useAllData();

  // show loading state while waiting for all the data
  if (!sidebar || !comments || !issue) return 'loading';

  // render the actual app here and pass data from state to children
  return (
    <>
      <Sidebar data={sidebar} />
      <Issue
        comments={comments}
        issue={issue}
      />
    </>
  );
};
```

## Conclusion
When we are fetching data with React we should, at the minimum,<br>
handle:
+ data state
+ loading state
+ error state

handle:
+ race conditions
+ network waterfalls
+ caching

We have already learned how to use the fetch API and handle the state for the requested data, and also states for loading and possible errors. We've also learned how to solve race conditions by returning clean-ups in our useEffect calls, also lifting our fetch calls up to parent components and putting them into a Promise.all in order to prevent network waterfalls. As we see, solving all these problems (also caching and more) require extensive customization of data fetching logic of our apps. Luckily there are already built solutions for these problems. One of the popular ones is the **[TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview "tanstack.com") (formerly known as React Query)**.

---
### Knowledge Check

**Q1.** How can you fetch data from an API in React?

**A1.** 

```jsx
import { useEffect, useState } from "react";

const Image = () => {
  const [imageURL, setImageURL] = useState(null);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/photos", { mode: "cors" })
      .then((response) => response.json())
      .then((response) => setImageURL(response[0].url))
      .catch((error) => console.error(error));
  }, []);

  return (
    imageURL && (
      <>
        <h1>An image</h1>
        <img src={imageURL} alt={"placeholder text"} />
      </>
    )
  );
};

export default Image;
```

**Q2.** Why should you manually throw errors in fetch requests?

**A2.** Working over the network is inherently unreliable. The API you’re making a request to might be down, there could be network connectivity issues, or the response you receive could contain errors. A multitude of things can go wrong, and if you don’t preemptively plan for errors, your website can break or appear unresponsive to users.

To simulate a network error, scroll up to the previous code snippet and change the fetch URL to something random. After a refresh of your browser window, the page will remain a blank white screen, without giving the user any indication that the page has finished loading or that there was an error.

To fix this, we need to check for *something* before `Image` component returns JSX. We’ll call it: `error`.

```jsx
if (error) return <p>A network error was encountered</p>

return (
  imageURL && (
    <>
      <h1>An image</h1>
      <img src={imageURL} alt={"placeholder text"} />
    </>
  )
);
```

To set this `error`, we’ll add it to the component’s state.

```jsx
const [imageURL, setImageURL] = useState(null);
const [error, setError] = useState(null);
```

And finally, we modify the `catch` call accordingly:

```jsx
useEffect(() => {
  fetch("https://jsonplaceholder.typicode.com/photos", { mode: "cors" })
    .then((response) => {
      if (response.status >= 400) {
        throw new Error("server error");
      }
      return response.json();
    })
    .then((response) => setImageURL(response[0].url))
    .catch((error) => setError(error));
}, []);
```

Note: Notice how we also handle errors in the then block? This is because the fetch request itself might not fail, but rather complete successfully and yield a response. However, the response received may not be what our app expected. To handle this case, we check the response status codes.

Now when a bad URL is passed or the API returns an unexpected response, the page will relay that information to the user.

**Q3.** How can you avoid waterfalling requests?

**A3.** 

Let’s start with laying out components first, then wire the data fetching afterward. We’ll have the App component itself, it will render Sidebar and Issue, and Issue will render Comments.

```jsx
const App = () => {
  return (
    <>
      <Sidebar />
      <Issue />
    </>
  );
};

const Sidebar = () => {
  return; // some sidebar links
};

const Issue = () => {
  return (
    <>
      // some issue data
      <Comments />
    </>
  );
};

const Comments = () => {
  return; // some issue comments
};
```

Now to the data fetching. Let’s first extract the actual fetch and useEffect and state management into a nice hook, to simplify the examples:

```jsx
export const useData = (url) => {
  const [state, setState] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      const data = await (await fetch(url)).json();

      setState(data);
    };

    dataFetch();
  }, [url]);

  return { data: state };
};
```

Now we will call the useData hook from the components themselves. And would also want to show the loading state while we’re waiting of course!

```jsx
const Comments = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData('/get-comments');

  // show loading state while waiting for the data
  if (!data) return 'loading';

  // rendering comments now that we have access to them!
  return data.map((comment) => <div>{comment.title}</div>);
};
```

And exactly the same code for Issue, only it will render Comments component after loading:

```jsx
const Issue = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData('/get-issue');

  // show loading state while waiting for the data
  if (!data) return 'loading';

  // render actual issue now that the data is here!
  return (
    <div>
      <h3>{data.title}</h3>
      <p>{data.description}</p>
      <Comments />
    </div>
  );
};
```

And the App itself:

```jsx
const App = () => {
  // fetch is triggered in useEffect there, as normal
  const { data } = useData('/get-sidebar');

  // show loading state while waiting for the data
  if (!data) return 'loading';

  return (
    <>
      <Sidebar data={data} />
      <Issue />
    </>
  );
};
```

What we implemented here is a classic waterfall of requests. In React, only components that are actually **returned** will be mounted, rendered, and as a result, will trigger useEffect and data fetching in it. In our case, every single component returns "loading" state while it waits for data. And only when data is loaded, do they switch to a component next in the render tree, it triggers its own data fetching, returns “loading” state and the cycle repeats itself.

Waterfalls like that are not the best solution when you need to show the app as fast as possible. Likely, there are a few ways to deal with them.

#### The `Promise.all` solution
The first and easiest solution is to pull all those data fetching requests as high in the render tree as possible. In our case, it’s our root component App. But there is a catch there: you can’t just “move” them and leave as-is. We can’t just do something like this:

```jsx
useEffect(async () => {
  const sidebar = await fetch('/get-sidebar');
  const issue = await fetch('/get-issue');
  const comments = await fetch('/get-comments');
}, []);
```

This is just yet another waterfall, only co-located in a single component: we fetch sidebar data, await for it, then fetch issue, await, fetch comments, await. The time when all the data will be available for render will be the sum of all those waiting times: 1s + 2s + 3s = 6 seconds. Instead, we need to fire them all at the same time, so that they are sent in parallel. That way we will be waiting for all of them no longer than the longest of them: 3 seconds. 50% performance improvement!

One way to do it is to use Promise.all:

```jsx
useEffect(async () => {
  const [sidebar, issue, comments] = await Promise.all([
    fetch('/get-sidebar'),
    fetch('/get-issue'),
    fetch('/get-comments'),
  ]);
}, []);
```

and then save all of them to state in the parent component and pass them down to the children components as props:

```jsx
const useAllData = () => {
  const [sidebar, setSidebar] = useState();
  const [comments, setComments] = useState();
  const [issue, setIssue] = useState();

  useEffect(() => {
    const dataFetch = async () => {
      // waiting for allthethings in parallel
      const result = (
        await Promise.all([
          fetch(sidebarUrl),
          fetch(issueUrl),
          fetch(commentsUrl),
        ])
      ).map((r) => r.json());

      // and waiting a bit more - fetch API is cumbersome
      const [sidebarResult, issueResult, commentsResult] =
        await Promise.all(result);

      // when the data is ready, save it to state
      setSidebar(sidebarResult);
      setIssue(issueResult);
      setComments(commentsResult);
    };

    dataFetch();
  }, []);

  return { sidebar, comments, issue };
};

const App = () => {
  // all the fetches were triggered in parallel
  const { sidebar, comments, issue } = useAllData();

  // show loading state while waiting for all the data
  if (!sidebar || !comments || !issue) return 'loading';

  // render the actual app here and pass data from state to children
  return (
    <>
      <Sidebar data={sidebar} />
      <Issue
        comments={comments}
        issue={issue}
      />
    </>
  );
};
```
