# Setting Up A React Environment
There are multiple ways to start using React in your projects, from attaching a set of \<script> tags which serve React from a CDN(Content delivery network), to robust toolchains and frameworks that are highly configurable and allow for increased scalability and optimization.

Some examples of these toolchains include:

+ Vite’s React Config
+ Gatsby
+ NextJS
+ Create React App (Deprecated).

Why do we need these toolchains? Can’t we just make our own as we see fit?

Yes, but it’s hard. React is a complex beast and there are many moving parts. Before you can start writing any sort of code that provides functionality, you would need to configure at least the following:

+ Package Management (NPM, Yarn)
+ Module bundling (Webpack, Parcel)
+ Compilation (Babel)
+ *React* itself

All of this, and sometimes much more is required to get a React project and development environment up and running.

**A note on Create React App:** Create React App, or CRA, was the official way to scaffold new React projects since its introduction in 2016. Unfortunately, owing to many reasons, CRA was deprecated in early 2023. Due to CRA’s popularity, you’ll see it mentioned in many tutorials and guides. However, it’s no longer recommended to use for new projects.

### Simplifying the process
Now that you understand what is involved with starting a React project from scratch, you can breathe a sigh of relief to learn that we can get started with a *single terminal command*.

`Vite` builds frontend tools for developers and it leverages the latest technologies under the hood to provide a great developer experience. Fortunately, it also caters to the React ecosystem. We will use Vite’s CLI to quickly create a template React project. It requires minimal configuration and provides extremely useful tools right out of the box, allowing us to get straight to the learning. Let’s get started!

### Creating a React app
Please make sure that you are using the LTS version of Node, otherwise errors may occur. Fire up a terminal instance, cd into the folder containing your projects, and enter the following command:

```bash
npm create vite@latest my-first-react-app -- --template react
```

If you see the following output, enter y and then press enter:

```bash
Need to install the following packages:
  create-vite@5.X.X
Ok to proceed? (y)
```

Once the command has executed, it should output the next steps for you to follow:

```bash
cd my-first-react-app
npm install
npm run dev
```

Provided everything has gone according to plan, head over to localhost:5173.

Congratulations! You’ve created your first React app.

Note: As you might’ve noticed by now, you can replace "my-first-react-app" with the name of your project.

### Chrome "React Developer Tools" Extension
React Developer Tools is a browser extension that helps developers debug and inspect React applications more effectively. It provides various features and functionalities to enhance the development experience when working with React-based projects. Some of its key features include:

1. Component Tree: React Developer Tools allows developers to view the component hierarchy of their React application in a tree-like structure. This makes it easier to understand the structure of the application and how components are nested within each other.

2. Props and State Inspection: Developers can inspect the props and state of each component in real-time. This is particularly useful for debugging and understanding how data is passed between components.

3. Component Highlighting: The extension highlights React components directly in the browser, making it easier to identify and locate specific components on the webpage.

4. Performance Monitoring: React Developer Tools includes performance monitoring features that allow developers to analyze the rendering performance of their React application. This can help identify performance bottlenecks and optimize the application for better performance.

5. Redux Integration: If the application uses Redux for state management, React Developer Tools seamlessly integrates with Redux DevTools to provide a comprehensive debugging experience for Redux-powered applications.

Overall, React Developer Tools serves as a valuable tool for React developers, helping them streamline the development process, debug issues more efficiently, and optimize the performance of their applications.

---
### Knowledge Check

**Q1.** What are some of the ways we can start a new React project?

**A1.** There are multiple ways to start using React in your projects, from attaching a set of \<script> tags which serve React from a CDN, to robust toolchains and frameworks that are highly configurable and allow for increased scalability and optimization.

Some examples of these toolchains include:

+ Vite’s React Config
+ Gatsby
+ NextJS
+ Create React App (Deprecated)

**Q2.** Why should we initially be using pre-made toolchains instead of making our own?

**A2.** Why do we need the above toolchains? Can’t we just make our own as we see fit?

Yes, but it’s hard. React is a complex beast and there are many moving parts. Before you can start writing any sort of code that provides functionality, you would need to configure at least the following:

+ Package Management (NPM, Yarn)
+ Module bundling (Webpack, Parcel)
+ Compilation (Babel)
+ React itself

All of this, and sometimes much more is required to get a React project and development environment up and running.

**Q3.** What is Vite and why would we use it?

**A3.** Vite is a modern build tool for front-end development, primarily focusing on JavaScript-based projects. It stands out for its speed and efficiency in the development process, particularly for building Vue.js and React applications. The name "Vite" comes from the French word for "fast" or "quick."

Vite requires minimal configuration and provides extremely useful tools right out of the box.

**Q4.** What command can we run to scaffold a new React project using Vite?

**A4.** The following bash command can be used to create a React project with Vite:

```bash
npm create vite@latest my-first-react-app -- --template react
```

**Q5.** What is in the "public" folder?

**A5.** In a Vite project, the "public" folder typically contains static assets and files that you want to be served directly to the client without any processing by the build tool. These assets include things like images, fonts, HTML files, and other resources that your application might need.

When you build a Vite project, the contents of the "public" folder are typically copied as-is into the output directory without undergoing any transformation or bundling. This means that the structure and contents of the "public" folder in your project will be preserved in the final build.

For example, if you have an image file named "logo.png" in the "public" folder of your Vite project, it will be accessible at the same path relative to the root of your deployed application after building. So if you access your deployed application at `https://example.com`, the "logo.png" file would be accessible at `https://example.com/logo.png`.

In summary, the "public" folder in a Vite build contains static assets and files that are served directly to the client without any processing by the build tool.

**Q6.** What is in the "src" folder?

**A6.** The "src" folder in a Vite project typically contains the source code of your application. This is where you'll find the JavaScript, TypeScript, CSS, SCSS, or other files that make up your project's codebase.

Here's a breakdown of what you might find in the "src" folder:

1. **JavaScript/TypeScript Files**: These files contain the logic and functionality of your application. They may include scripts for handling user interactions, making API requests, managing state, and more.

2. **CSS/SCSS Files**: If your project includes stylesheets, you'll likely find them in this folder. These files define the visual appearance and layout of your application's components and UI elements.

3. **Component Files**: If you're using a component-based framework like Vue.js or React, you may have separate files for each component within the "src" folder. These files typically include the template (HTML-like structure), JavaScript/TypeScript logic, and possibly CSS/SCSS styles for each component.

4. **Assets**: This folder may contain additional assets such as images, fonts, or other media files used by your application. These assets can be imported into your JavaScript/TypeScript files and referenced in your HTML or CSS/SCSS.

5. **Other Supporting Files**: Depending on your project setup and requirements, you may have additional files in the "src" folder for things like routing configurations, global utilities, data models, or any other necessary components of your application architecture.

Overall, the "src" folder is where you'll spend most of your time writing and organizing the source code of your Vite project. It serves as the central location for all the files that contribute to the functionality and appearance of your application.

**Q7.** Why are the React Developer Tools useful?

**A7.** React Developer Tools is a browser extension that helps developers debug and inspect React applications more effectively. It provides various features and functionalities to enhance the development experience when working with React-based projects. Some of its key features include:

1. Component Tree: React Developer Tools allows developers to view the component hierarchy of their React application in a tree-like structure. This makes it easier to understand the structure of the application and how components are nested within each other.

2. Props and State Inspection: Developers can inspect the props and state of each component in real-time. This is particularly useful for debugging and understanding how data is passed between components.

3. Component Highlighting: The extension highlights React components directly in the browser, making it easier to identify and locate specific components on the webpage.

4. Performance Monitoring: React Developer Tools includes performance monitoring features that allow developers to analyze the rendering performance of their React application. This can help identify performance bottlenecks and optimize the application for better performance.

5. Redux Integration: If the application uses Redux for state management, React Developer Tools seamlessly integrates with Redux DevTools to provide a comprehensive debugging experience for Redux-powered applications.

Overall, React Developer Tools serves as a valuable tool for React developers, helping them streamline the development process, debug issues more efficiently, and optimize the performance of their applications.