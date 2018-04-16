Build a SSR App With Preact, Unistore and Preact Router
=======================================================
[https://scotch.io/tutorials/build-a-ssr-app-with-preact-unistore-and-preact-router]

_Yomi Eluwande April 03_

In this tutorial, we are going to explore how to build a server-side rendered app with [Preact](https://preactjs.com/). [Preact Router](https://github.com/developit/preact-router) will be used for routing, [Unistore](https://github.com/developit/unistore) for state management and [Webpack](https://webpack.js.org/) for JS bundling. Some existing knowledge of Preact, Unistore, and Webpack might be needed.

 1. [Why Server-Side Rendered Apps?](#Why-Server-Side-Rendered-Apps?)
 2. [Build a SSR app with Preact](#Build-a-SSR-app-with-Preact)
 3. [Build the Preact App](#Build-the-Preact-App)
 4. [Build the Node Server](#Build-the-Node-Server)
 5. [Adding CSS Styling](#Adding-CSS-Styling)
 6. [Conclusion](#Conclusion)

# Why Server-Side Rendered Apps?

We can all agree that Single Page Apps are a very popular way of building modern web applications. When it comes to SPAs, there are two ways in which you can render the content of the app to your users; Client Side Rendering and Server-Side Rendering.

With Client Side Rendering, whenever a user opens up the app, it sends a request to load up the layout, HTML, CSS, and JavaScript. This is all fine. But in cases (all the time) which the content of the application is dependent on the completion of successfully loading the JS scripts, this can be a problem. This means users would be forced to view an empty or a preloader while waiting for the scripts to finish loading.

Server-Side Rendering doesn't operate like that, with SSR, your initial request straight up loads the page, layout, CSS, JavaScript, and content. SSR makes sure that data is properly initialized at render time.

Another advantage Server-Side Rendering has over Client Side Rendering is SEO.

Technologies Let's go over the technologies we'll be using to build the Server-Side Rendered App;

Preact `Preact` is a 3kb alternative to React with the same API. It aims to offer a development experience similar to React, albeit with some features stripped away such as [PropTypes and Children](https://preactjs.com/guide/differences-to-react). Unistore Unistore is a 650b centralized state container with component bindings for React and Preact. It's small size means it complements Preact nicely. Preact Router `Preact Router` helps to manage route in Preact applications. provides a component that conditionally renders its children when the URL matches their path. Webpack `Webpack` is bundler that helps to bundle JavaScript files for usage in a browser. Enough said. preact-render-to-string

# Build a SSR app with Preact

Building this app would be divided into two. We'll first build the server-side of the code which will be in Node and Express and then we'll code the Preact part of the code.

The idea is to create a Preact app as it were and hook it up to a Node server using the `preact-render-to-string` package. It allows for rendering JSX and Preact components to an HTML string which can then be used in a server. This means we'll be creating Preact components in a `src` folder and hook it up to the Node server file.

The first thing to do would be to create the directory for the project and create the different folders that would be needed. Create a folder named `preact-unistore-ssr` and run the command `npm init --y` inside the folder. That creates a minimal `package.json` and an accompanying `package-lock.json`.

Next, let's install some of the tools we'll be needing for this project. Open up the `package.json` file and edit with the code below, then run the `npm i` command.

```JSON
{
  "name": "preact-unistore-ssr",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-cli": "^6.26.0",
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-plugin-transform-react-jsx": "^6.24.1",
    "babel-preset-env": "^1.6.1",
    "file-loader": "^1.1.11",
    "url-loader": "^1.0.1",
    "webpack": "^3.11.0",
    "webpack-cli": "^2.0.13"
  },
  "dependencies": {
    "express": "^4.16.2",
    "preact": "^8.2.6",
    "preact-render-to-string": "^3.7.0",
    "preact-router": "^2.6.0",
    "unistore": "^3.0.4"
  }
}
```

That will install all the packages needed for this application. In the `devDependencies` object, we have some babel packages that will help with transpiling ES6 code. `file-loader` and `url-loader` are Webpack plugins that help with importing files, assets, modules e.t.c.

In the dependencies object, we install packages like Express, Preact, `preact-render-to-string`, `preact-router`, and `unistore`.

Next, let's create a Webpack config file, create a file named `webpack.config.js` in the root of the project and edit it with the code below:

```bash
# Create files 
touch webpack.config.js .babelrc
```

```JavaScript
// webpack.config.js
const path = require("path");

module.exports = {
    entry: "./src/index.js",
    output: {
        path: path.join(__dirname, "dist"),
        filename: "app.js"
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: "babel-loader",
            }
        ]
    }
};
```

In the webpack config above, we define the entry point to be a `src/index.js` file and the output to be `dist/app.js` and also set the rules for using babel. The entry point file does not exist yet but that will be created later.

Since we're using Babel, we'll need to create a `.babelrc` file in the root of the project and put in our config.

```JSON
//.babelrc
{
    "plugins": [
        ["transform-react-jsx", { "pragma": "h" }]
    ],
    "presets": [
        ["env", {
            "targets": {
                "node": "current",
                "browsers": ["last 2 versions"]
            }
        }]
    ]
}
```

# Build the Preact App

Next up, we'll begin to create files for the Preact side of things. Create a `src` folder and create the following files in it.

```bash
# mkdir ./src ./src/store && touch ./src/store/store.js ./src/About.js ./src/App.js ./src/index.js ./src/router.js

store/store.js
About.js
App.js
index.js
router.js
```

Let's begin to edit the files with the necessary code. We'll start with the `store.js` file. This will contain the store data and actions.

```JavaScript
import createStore from 'unistore'

export let actions = store => ({
    increment(state) {
        return { count: state.count + 1 }
    },
    decrement(state) {
        return { count: state.count - 1 }
    }
})

export default initialState => createStore(initialState)
```

in the code block above, we are exporting some set of actions, which basically increments and decrements the value of the `count` by 1. The actions will always receive state as first parameter and any other params may come next. The `createStore` function which is used to initialize the store in `Unistore` is also exported.

Next up, we'll edit the `router.js` file. This contains the setup for the routes we'll be using in the app.

```JavaScript
import { h } from 'preact'
import Router from 'preact-router'

import { App } from "./App";
import { About } from "./About";

export default () => (
    <Router>
        <App path="/" />
        <About path="/about" />
    </Router>
)
```

`preact-router` makes it so easy to define routes. all you have to do is import the routes and make them the children of the `Router` component. You can then set a `prop` of `path` to each component so that `preact-router` knows which component to serve for a route.

There are mainly just two routes in the application, the `App.js` component serves as the home route and the `About.js` component serves as the about page.

Next is the `About.js` file and you can proceed to edit with the following:

```JavaScript
import { h } from 'preact';
import { Link } from 'preact-router/match';

export const About = () => (
    <div>
        <p>This is a Preact app being rendered on the server. It uses Unistore for state management and preact-router for routing.</p>
        <Link href="/">Home</Link>
    </div>
);
```

It's a simple component that has a short description and a `Link` component that leads to the home route.

Let's open up `App.js` which serves as the home route and edit with the necessary code.

```JavaScript
import { h } from 'preact'
import { Link } from 'preact-router'
import { connect } from 'unistore/preact'

import { actions } from './store/store'

export const App = connect('count', actions)(
    ({ count, increment, decrement }) => (
      <div class="count">
        <p>{count}</p>
        <button class="increment-btn" onClick={increment}>Increment</button>
        <button class="decrement-btn" onClick={decrement}>Decrement</button>
        <Link href="/about">About</Link>
      </div>
    )
  )
```

So what's happening here? The `connect` function is imported, as well as the `actions` function. In the `App` component, the state value, `count` is exposed as well as the `increment` and `decrement` actions. The `increment` and `decrement` actions are both hooked up to different buttons with the `onClick` event handler.

The `index.js` file is the entry point for Webpack if you remember too well, it's going to serve as the parent component for all other components in the Preact app. Open up the file and edit with the code below.

```JavaScript
// index.js
import { h, render } from 'preact'
import { Provider } from 'unistore/preact'
import Router from './router'

import createStore from './store/store'

const store = createStore(window.__STATE__)

const app = document.getElementById('app')

render(
    <Provider store={store}>
        <Router />
    </Provider>,
    app,
    app.lastChild
)
```

In the code block above, the `Provider` component is imported. It's important to specify the working environment if it's preact or react. We also import the `Router` component from the `router.js` file and the `createStore` function is equally imported from the `store.js` file.

The `const store = createStore(window.__STATE__)` line is used to pass initial state from the server to client since we're building a SSR app afterall.

Finally, in the `render` function, we wrap the `Router` component inside the `Provider` component thereby making the store available to all child components.

That's a wrap on the client side of things. We'll now move to the server-side of the app.

# Build the Node Server

To start with, a `server.js` file will be created, this will house the Node app that will be used for the server-side rendering.

```bash
touch server.js
```

```JavaScript
// server.js
const express = require("express");
const { h } = require("preact");
const render = require("preact-render-to-string");
import { Provider } from 'unistore/preact'
const { App } = require("./src/App");
const path = require("path");

import Router from './src/router'
import createStore from './src/store/store'

const app = express();

const HTMLShell = (html, state) => `
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.6.2/css/bulma.min.css">
            <title> SSR Preact App </title>
        </head>
        <body>
            <div id="app">${html}</div>
            <script>window.__STATE__=${JSON.stringify(state).replace(/<|>/g, '')}</script>
            <script src="./app.js"></script>
        </body>
    </html>`

app.use(express.static(path.join(__dirname, "dist")));

app.get('**', (req, res) => {
    const store = createStore({ count: 0, todo: [] })

    let state = store.getState()

    let html = render(
        <Provider store={store}>
            <Router />
        </Provider>
    )

    res.send(HTMLShell(html, state))
})

app.listen(4000);
```

Let's break this down into bits.

```JavaScript
const express = require("express");
const { h } = require("preact");
const render = require("preact-render-to-string");
import { Provider } from 'unistore/preact'
const { App } = require("./src/App");
const path = require("path");

import Router from './src/router'
import createStore from './src/store/store'

const app = express();
```

In the code block above, we import the packages needed for the Node server, such as `express`, `path`, we also import `preact`, the `Provider` component from `unistore` and most importantly the `preact-render-to-string` package which enables us to do server-side rendering. The routes and store is also imported from their respective files.

```JavaScript
const HTMLShell = (html, state) => `
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.6.2/css/bulma.min.css">
            <title> SSR Preact App </title>
        </head>
        <body>
            <div id="app">${html}</div>
            <script>window.__STATE__=${JSON.stringify(state).replace(/<|>/g, '')}</script>
            <script src="./app.js"></script>
        </body>
    </html>`
```

In the code block above, we create the base HTML that will be used for the app. In the HTML code, the state is initialiazed in the `script` section. The `HTMLShell` function accepts two params, the first one is `html`, this will be the output gotten from the `preact-render-to-string`, `html` is then injected inside the HTML code. The second param is the state.

```JavaScript
app.use(express.static(path.join(__dirname, "dist")));

app.get('**', (req, res) => {
    const store = createStore({ count: 0})

    let state = store.getState()

    let html = render(
        <Provider store={store}>
            <Router />
        </Provider>
    )

    res.send(HTMLShell(html, state))
})

app.listen(4000);
```

In the first LOC, we tell Express to use the `dist` when serving static files, if you remember too well, the `app.js` is inside the `dist` folder. Next, we set a route for any request that comes into the app with `app.get(**)` and this is where the work lies. This first thing to do is to initialize the store and it's state and then create a variable that holds the value of the state.

Then, `preact-render-to-string` which was imported as `render` is used to render the client side Preact app alongside the `Router` which holds the route and `Provider` which provides the store to every child component.

With that done, we can finally run the app and see what it looks like, before you do that, add the code block below to the `package.json` file.

```JavaScript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start:client": "webpack -w",
    "start:server": "babel-node server.js",
    "dev": "npm run start:client & npm run start:server"
  },
```

These are scripts that allows us to get the app up and running, run the command `npm run dev` in your terminal and go to [http://localhost:4000](http://localhost:4000). The app should be up and running and you should get a display similar to the one below.

# Adding CSS Styling

Now that are views are done and the client is hooked up to the server, let's add some styling to the app. We'll need to let Webpack know that it needs to bundle CSS files. To do that, `style-loader` and `css-loader` needs to be added to the app and both can be installed by running the command below.

`npm i css-loader style-loader --save-dev`

Once the installation is done, head over to the `webpack.config.js` file and add the the code below inside the `rules` array.

```JavaScript
{
    test: /\.css$/,
    use: [ 'style-loader', 'css-loader' ]
}
```

We can now create a `index.css` file inside the `src` folder and edit with the following code:

```css
body {
  background-image: linear-gradient(to right top, #2b0537, #820643, #c4442b, #d69600, #a8eb12);
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}
a {
  display: block;
  color: white;
  text-decoration: underline;
}
p {
  color: white
}
.count p {
  color: white;
  font-size: 60px;
}
button:focus {
  outline: none;
}
.increment-btn {
  background-color: #1A2C5D;
  border: none;
  color: white;
  border-radius: 3px;
  padding: 10px 20px;
  font-size: 14px;
  margin: 0 10px;
}
.decrement-btn {
  background-color: #BC1B1B;
  border: none;
  color: white;
  border-radius: 3px;
  padding: 10px 20px;
  font-size: 14px;
  margin: 0 10px;
}
```

In the `index.js` file, add this code at the top of the file too. `import './index.css';`

Now the page should display nicely like this. 

# Conclusion

In this tutorial, we've seen how to create a Server-Side Rendered Preact app and explored the advantages of building server-side rendered apps. We also saw how to use Unistore for basic state management and how to hook up state from the server to the frontend using window.__STATE__.

You should now have an idea on how to render a Preact app on the server. The basic gist is to initially render the app on the server FIRST and then render the components on the browser.

The code for this tutorial can be viewed on [Github](https://github.com/yomete/preact-ssr) for your perusal.