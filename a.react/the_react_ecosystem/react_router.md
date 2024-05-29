# React Router
Up until this point in the curriculum, we have been building one-page applications. However, for any larger scale application, we are going to have multiple pages. Thankfully, the browser allows client-side JavaScript to manage the way a user can navigate, with the `History API`. We can leverage the power of this to manage routing in React with the help of a package like React Router.

## Client-side routing
Client-side routing refers to the process of managing navigation within a web application entirely on the client side, without the need to reload the entire page from the server. This is achieved by using JavaScript to manipulate the browser's URL and update the content displayed on the page dynamically.

Here's how client-side routing typically works:

1. **Initial Page Load**: When a user visits a website for the first time or reloads the page, the server sends the initial HTML, CSS, and JavaScript files required for the application to run.

2. **JavaScript Router Initialization**: Once the initial page has loaded, JavaScript code on the page initializes a router. This router is responsible for intercepting URL changes and rendering the appropriate content without requesting a new page from the server.

3. **Navigation Events**: When a user clicks on a link or interacts with elements that trigger a navigation event (e.g., clicking a button that changes the URL), the router intercepts this event and prevents the default browser behavior of requesting a new page from the server.

4. **URL Manipulation**: Instead of requesting a new page from the server, the router updates the URL in the browser's address bar to reflect the new location within the application. This manipulation of the URL is often achieved using the HTML5 History API, which allows developers to modify the browser history programmatically.

5. **Rendering Content**: After updating the URL, the router determines which component or page to render based on the new URL. It may match the URL against a predefined set of routes or patterns to determine the appropriate content to display.

6. **Updating the DOM**: Once the router has determined the content to display, it updates the DOM (Document Object Model) of the page to reflect the new content. This could involve replacing the entire page content, or just certain sections of it, depending on the design of the application.

7. **State Management**: Alongside routing, client-side applications often utilize state management techniques (such as React's state or Redux) to manage the application's state. This allows for efficient rendering and updating of components based on user interactions and changes in data.

Advantages of client-side routing include:

- **Faster Navigation**: Since only the necessary content is loaded and rendered, navigation within the application can be faster compared to traditional server-side navigation.
- **Smoother User Experience**: With client-side routing, transitions between pages or sections of an application can be smoother, as there's no need to wait for server responses.
- **Single Page Application (SPA) Support**: Client-side routing is essential for building single-page applications (SPAs) where all interactions occur within a single HTML page.

However, it's essential to consider some drawbacks, such as initial loading times, search engine optimization (SEO) challenges, and potential issues with browser history and bookmarking. Additionally, client-side routing relies heavily on JavaScript, so users with JavaScript disabled may experience limited functionality.

## A Reactive Solution `(React Router)`
While client-side routing allows for nicer, app-like interactions (since you are controlling the routes, you can make fancy CSS animations across route changes), a lot of caveats can be missed. When a browser reloads, it notifies screen-readers of new content to read, but in the case of client-side routing, you will need to notify screen-readers of route updates manually. However, with the help of a robust library, you can often address these concerns!

React Router is a popular library for implementing client-side routing in React applications. It provides a declarative way to manage the application's routes, allowing developers to define how different components are rendered based on the URL.

Here are some key features and concepts of React Router:

1. **Declarative Routing**: React Router allows developers to define routes using a declarative syntax, typically within the main component of the application. Routes are defined using special components provided by React Router, such as `<Route>` and `<Switch>`.

2. **Nested Routes**: React Router supports nested routes, allowing developers to define routes within other routes. This is useful for creating complex application layouts with nested components.

3. **Route Parameters**: React Router allows developers to define dynamic route segments using parameters. These parameters can be accessed within components to render content based on the current URL.

4. **Programmatic Navigation**: React Router provides APIs for programmatic navigation, allowing developers to navigate to different routes imperatively using JavaScript code. This is useful for handling navigation in response to user actions or application logic.

5. **URL Parameters**: React Router allows developers to access URL parameters and query parameters within components, enabling dynamic content rendering based on the current URL.

6. **History Management**: React Router integrates with the HTML5 History API to manage browser history. It allows developers to control browser navigation, manipulate the browser history, and handle browser back and forward buttons.

7. **Code Splitting**: React Router supports code splitting, allowing developers to split their application into smaller chunks and load them dynamically based on the current route. This can help improve performance by reducing the initial load time of the application.

Overall, React Router simplifies the process of implementing client-side routing in React applications, making it easier for developers to create single-page applications with complex navigation and dynamic content rendering. It has become a standard choice for routing in React projects due to its flexibility, powerful features, and active community support.

React Router is a standard routing library for React applications. By using React Router, we can specify React components, that can be rendered based on the route, and so much more. Let’s dive in!

### Creating a router
Create a new Profile.jsx file with the following component:

```jsx
// Profile.jsx

const Profile = () => {
  return (
    <div>
      <h1>Hello from profile page!</h1>
      <p>So, how are you?</p>
    </div>
  );
};

export default Profile;
```

Add the following content to the App.jsx file:

```jsx
// App.jsx

const App = () => {
  return (
    <div>
      <h1>Hello from the main page of the app!</h1>
      <p>Here are some examples of links to other pages</p>
      <nav>
        <ul>
          <li>
            <a href="profile">Profile page</a>
          </li>
        </ul>
      </nav>
    </div>
  );
};

export default App;
```

Now it’s time to add the router! There’s a couple of ways of defining our app’s routes, but in React Router v6.7.0 or higher, it is recommended to add routes as objects.

We can install the React Router package with the following bash command:

`npm install react-router-dom`

Finally we add the following to the main.jsx file:

```jsx
// main.jsx

import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile",
    element: <Profile />,
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

1. We import `createBrowserRouter` and `RouterProvider` from React Router.
1. `createBrowserRouter` is used to create the configuration for a router by passing arguments in the form of an array of routes.
1. The configuration array contains objects with two mandatory keys, the `path` and the corresponding `element` to be rendered.
1. This generated configuration is then rendered in, by passing it to the `RouterProvider` component.

### The `<Link>` element
You may notice, when we click the links in the navbar, the browser is reloading for the next URL instead of using React Router. This isn’t what was promised! To help with this, **React Router exports a custom `Link` element** to be used instead of the regular `a` tag. We can replace the `a` tag in our navbar with the `Link` element.

```jsx
// App.jsx

import { Link } from "react-router-dom";

const App = () => {
  return (
    <div>
      <h1>Hello from the main page of the app!</h1>
      <p>Here are some examples of links to other pages</p>
      <nav>
        <ul>
          <li>
            <Link to="profile">Profile page</Link>
          </li>
        </ul>
      </nav>
    </div>
  );
};

export default App;
```

And now, we don’t get the browser reloading every time we click the link on the navbar!

### Nesting routes and the `<Outlet>` Component
What if you want to render a section of a page differently, based on different URLs? This is where nested routes come into play! We can add routes nested as the children of one another to ensure that the child gets rendered alongside the parent. We create the Popeye.jsx and Spinach.jsx as example:

```jsx
// Popeye.jsx

import { Link } from "react-router-dom";

const Popeye = () => {
  return (
    <>
      <p>Hi, I am Popeye! I love to eat Spinach!</p>
      <Link to="/">Click here to go back</Link>
    </>
  );
};

export default Popeye;
```

```jsx
// Spinach.jsx

import { Link } from "react-router-dom";

const Spinach = () => {
  return (
    <>
      <p>Hi, I am Spinach! Popeye loves to eat me!</p>
      <Link to="/">Click here to go back</Link>
    </>
  );
};

export default Spinach;
```

Now we rewrite the main.jsx file as the following:

```jsx
// main.jsx

import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";
import Spinach from "./Spinach";
import Popeye from "./Popeye";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile",
    element: <Profile />,
    children: [
      { path: "spinach", element: <Spinach /> },
      { path: "popeye", element: <Popeye /> },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

This allows us to render the child component alongside the parent, through an `Outlet` component. We can rewrite the `Profile` component to add an `Outlet` which will get replaced by the various profiles when that route is visited!

```jsx
// Profile.jsx

import { Outlet } from "react-router-dom";

const Profile = () => {
  return (
    <div>
      <h1>Hello from profile page!</h1>
      <p>So, how are you?</p>
      <hr />
      <h2>The profile visited is here:</h2>
      <Outlet />
    </div>
  );
};

export default Profile;
```

Check out the `/profile`, `/profile/popeye` and `/profile/spinach` pages. The `<Outlet />` component gets replaced with the children component when their paths are visited.



If you want to render something as a default component when no path is added to Profile, you can add an index route to the children.

To do this, we create a DefaultProfile component:

```jsx
// DefaultProfile.jsx

const DefaultProfile = () => {
  return <p>Oh, nothing to see here!</p>;
};

export default DefaultProfile;
```

Now, add an `index` tag with the `DefaultProfile` as a child to the `/profile` route.

```jsx
// main.jsx


import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";
import DefaultProfile from "./DefaultProfile";
import Spinach from "./Spinach";
import Popeye from "./Popeye";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile",
    element: <Profile />,
    children: [
      { index: true, element: <DefaultProfile /> },
      { path: "spinach", element: <Spinach /> },
      { path: "popeye", element: <Popeye /> },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

If you visit the /profile path now, you should be able to see some default content where the Outlet is rendered when the index path is rendered.

### The `useOutletContext()` hook
As we've already learned, we can nest routes as children of a parent route, allowing us to use an `<Outlet />` in the parent to render the appropriate element based on the rest of the path.

Often parent routes manage state or other values you want shared with child routes. You can create your own **context provider** if you like, but this is such a common situation that it's built-into `<Outlet />`:

```jsx
// The <Outlet /> component has a built in "context" prop

function Parent() {
  const [count, setCount] = React.useState(0);
  return <Outlet context={[count, setCount]} />;
}
```

```jsx
// We can use the useOutletContext() hook to use the provided context

import { useOutletContext } from "react-router-dom";

function Child() {
  const [count, setCount] = useOutletContext();
  const increment = () => setCount((c) => c + 1);
  return <button onClick={increment}>{count}</button>;
}
```

### Dynamic Segments with React Router
We can render content according to the their URLs dynamically, with "dynamic segments" of `react router`:

```jsx
// main.jsx

import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile/:name",
    element: <Profile />,
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

The colon (:) turns the path section after it into a “dynamic segment”. Dynamic segments will match dynamic (changing) values in that position of the URL, like the `name`. These can also be called “URL params” or “params” in short. These can be used with the help of the `useParams` hook. We can thus write the Profile component as the following:

```jsx
// Profile.jsx

import { useParams } from "react-router-dom";
import DefaultProfile from "./DefaultProfile";
import Spinach from "./Spinach";
import Popeye from "./Popeye";

const Profile = () => {
  const { name } = useParams();

  return (
    <div>
      <h1>Hello from profile page!</h1>
      <p>So, how are you?</p>
      <hr />
      <h2>The profile visited is here:</h2>
      {name === "popeye" ? (
        <Popeye />
      ) : name === "spinach" ? (
        <Spinach />
      ) : (
        <DefaultProfile />
      )}
    </div>
  );
};

export default Profile;
```

### Handling Bad URLs
In our above example, when no params are passed to the /profile path, the application shows an error. This makes sense since we did not specify any component to render when we navigate to /profile without a "name" parameter.

We can show a default/error page in case the user visits a wrong or unused path:

```jsx
// ErrorPage.jsx

import { Link } from "react-router-dom";

const ErrorPage = () => {
  return (
    <div>
      <h1>Oh no, this route doesn't exist!</h1>
      <Link to="/">
        You can go back to the home page by clicking here, though!
      </Link>
    </div>
  );
};

export default ErrorPage;
```

Now we will add the `errorElement` to the configuration, and verify that it renders an error page by going to the `/profile` path or any unmentioned paths.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";
import ErrorPage from "./ErrorPage";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
    errorElement: <ErrorPage />,
  },
  {
    path: "profile/:name",
    element: <Profile />,
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

### Protected Routes and Navigation
Often, you will need to decide whether a certain route should be rendered or not. One example is authentication, where you render certain routes based on if the user is logged in or not. If they are logged in, you show some information about the user like here at The Odin Project dashboard page. Otherwise, they are redirected to the sign-in page (this could be any page). While there are many ways to do so, one of the easiest ways is to conditionally create a config for the router.

You will often come across the need to reroute the user to a different URL programmatically. This is where we use the `<Navigate />` component. The `<Navigate />` component reroutes the user to the desired URL when it is rendered. It is a wrapper around the `useNavigate` hook that lets you navigate programmatically, to URLs, or even go back down the user’s history.

```jsx
// A <Navigate> element changes the current location when it is rendered. It's a component wrapper around useNavigate, and accepts all the same arguments as props.

import * as React from "react";
import { Navigate } from "react-router-dom";

class LoginForm extends React.Component {
  state = { user: null, error: null };

  async handleSubmit(event) {
    event.preventDefault();
    try {
      let user = await login(event.target);
      this.setState({ user });
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    let { user, error } = this.state;
    return (
      <div>
        {error && <p>{error.message}</p>}
        {user && (
          <Navigate to="/dashboard" replace={true} />
        )}
        <form
          onSubmit={(event) => this.handleSubmit(event)}
        >
          <input type="text" name="username" />
          <input type="password" name="password" />
        </form>
      </div>
    );
  }
}
```

---
### Knowledge Check

**Q1.** What does client-side routing mean?

**A1.** Client-side routing refers to the process of managing navigation within a web application entirely on the client side, without the need to reload the entire page from the server. This is achieved by using JavaScript to manipulate the browser's URL and update the content displayed on the page dynamically.

**Q2.** How do you set up a basic router?

**A2.** 

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile",
    element: <Profile />,
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

**Q3.** What should be used in place of “a” tags to enable client-side routing?

**A3.** React Router exports a custom `Link` element to be used instead of the regular `a` tag.

```jsx
import { Link } from "react-router-dom";

const App = () => {
  return (
    <div>
      <h1>Hello from the main page of the app!</h1>
      <p>Here are some examples of links to other pages</p>
      <nav>
        <ul>
          <li>
            <Link to="profile">Profile page</Link>
          </li>
        </ul>
      </nav>
    </div>
  );
};

export default App;
```

**Q4.** How do you create nested routes?

**A4.** We can add routes nested as the children of one another to ensure that the child gets rendered alongside the parent.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";
import Spinach from "./Spinach";
import Popeye from "./Popeye";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile",
    element: <Profile />,
    children: [
      { path: "spinach", element: <Spinach /> },
      { path: "popeye", element: <Popeye /> },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

This allows us to render the child component alongside the parent, through an `Outlet` component. We can rewrite the Profile component to add an Outlet which will get replaced by the various profiles when that route is visited.

```jsx
import { Outlet } from "react-router-dom";

const Profile = () => {
  return (
    <div>
      <h1>Hello from profile page!</h1>
      <p>So, how are you?</p>
      <hr />
      <h2>The profile visited is here:</h2>
      <Outlet />
    </div>
  );
};

export default Profile;
```

Check out the `/profile`, `/profile/popeye` and `/profile/spinach` pages. The `<Outlet />` component gets replaced with the children component when their paths are visited.

**Q5.** What do you mean by dynamic segments or URL params?

**A5.** With dynamic segments, we can render content dynamically from the component itself.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
  },
  {
    path: "profile/:name",
    element: <Profile />,
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

The colon (:) turns the path section after it into a “dynamic segment”. Dynamic segments will match dynamic (changing) values in that position of the URL, like the `name`. These can also be called “URL params” or “params” in short. These can be used with the help of the `useParams` hook. We can thus rewrite the Profile component as the following:

```jsx
import { useParams } from "react-router-dom";
import DefaultProfile from "./DefaultProfile";
import Spinach from "./Spinach";
import Popeye from "./Popeye";

const Profile = () => {
  const { name } = useParams();

  return (
    <div>
      <h1>Hello from profile page!</h1>
      <p>So, how are you?</p>
      <hr />
      <h2>The profile visited is here:</h2>
      {name === "popeye" ? (
        <Popeye />
      ) : name === "spinach" ? (
        <Spinach />
      ) : (
        <DefaultProfile />
      )}
    </div>
  );
};

export default Profile;
```

**Q6.** How do you add a “catch-all” route?

**A6.** In our above example, when no params are passed to the /profile path, the application shows an error. This makes sense since we did not specify any component to render when we navigate to /profile without a "name" parameter.

We can show a default/error page in case the user visits a wrong or unused path:

```jsx
// ErrorPage.jsx

import { Link } from "react-router-dom";

const ErrorPage = () => {
  return (
    <div>
      <h1>Oh no, this route doesn't exist!</h1>
      <Link to="/">
        You can go back to the home page by clicking here, though!
      </Link>
    </div>
  );
};

export default ErrorPage;
```

Now we will add the `errorElement` to the configuration, and verify that it renders an error page by going to the `/profile` path or any unmentioned paths.

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import App from "./App";
import Profile from "./Profile";
import ErrorPage from "./ErrorPage";

const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
    errorElement: <ErrorPage />,
  },
  {
    path: "profile/:name",
    element: <Profile />,
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

**Q7.** How do you pass data from parent to child through an `<Outlet />` component?

**A7.** As we've already learned, we can nest routes as children of a parent route, allowing us to use an `<Outlet />` in the parent to render the appropriate element based on the rest of the path.

Often parent routes manage state or other values you want shared with child routes. You can create your own **context provider** if you like, but this is such a common situation that it's built-into `<Outlet />`:

```jsx
// The <Outlet /> component has a built in "context" prop

function Parent() {
  const [count, setCount] = React.useState(0);
  return <Outlet context={[count, setCount]} />;
}
```

```jsx
// We can use the useOutletContext() hook to use the provided context

import { useOutletContext } from "react-router-dom";

function Child() {
  const [count, setCount] = useOutletContext();
  const increment = () => setCount((c) => c + 1);
  return <button onClick={increment}>{count}</button>;
}
```

**Q8.** How do you create protected routes?

**A8.** Often, you will need to decide whether a certain route should be rendered or not. One example is authentication, where you render certain routes based on if the user is logged in or not. If they are logged in, you show some information about the user; otherwise, they are redirected to the sign-in page. While there are many ways to achieve this behaviour, one of the easiest ways is to conditionally create a config for the router, thus changing the provided router configuration whether the user is logged in or not.