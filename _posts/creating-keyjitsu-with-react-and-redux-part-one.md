---
title: "Creating KeyJitsu with React and redux - part I"
excerpt: "Creating production like website with react and redux"
header:
 teaser: ""
tags: 
  - react
  - reactjs
  - redux
--- 

### Basic setup
Let starts with scallafolding boilerpalte with create-react-app:

```
npm install -g create-react-app
create-react-app keyjitsu
```

and add dependencies:

```
npm install --save bootstrap@next, styled-components, react-dnd, redux, react-redux, react-router
```

and dev deps:
```
npm install --save-dev babel-jest, enzyme, enzyme-adapter-react-16, enzyme-to-json, redux-mock-store, sinon
```

Reading this You should know most of the dependencies but briefly:
   - bootstrap - css framework 
   - styled-components css-in-js solution
   - react-dnd - drag and drop with react
   - redux - state management
   - react-redux - redux bindings for react
   - react-router - routing for react

   - enzyme - testing utilities for react (rendering components)
   - enzyme-adapter-react-16 - need for enzyme to work with react 16
   - enzyme-to-json - snapshot testing with enzyme
   - redux-mock-store - mock redux store to to easily write assertions on it
   - sinon - mocking library

Fortunately with above dependencies We don't have to eject create-react-app and write something custom for webpack. 
Jest is already baked in create-react-app so there is no need for adding into package.json as another dependency.



I will follow project structure I usually use, more about it here (https://deaddesk.top/redux-project-structure/):

```
src/
    components/ -> common dumb components that can be reused
        Header/
            index.js
        Menu/
            index.js
    containers/
        Todo/
            tests/
                todoContainer.test.js
            index.js -> container component 
            TodoList -> dumb component that is used only for todo
            todoDuck.js -> action constants, action creators, reducer
            selectors.js -> reselect code
            saga.js -> redux-saga
        AnotherFeature/
            .
            .
            .
    css/ -> css related files if not using css-in-js
    images/ -> images
    utils/ -> code that doesn't fit into any other place
    index.js -> entry point
    configureStore.js -> redux store configuration
    reducers.js -> combine reducers
.env -> contains 'NODE_PATH=src/', needed for absolute imports
package.json
README.md
```.

Remove what is not needed first:
    - src/index.css 
    - src/logo.svg
    - src/App.js
    - src/App.test.js
    - src/App.css]

Let's create global style definition:
    - create file src/globalStyle.js
    - inside that file define css that will style doms that are wrapping our app so html, body and #root div

```javascript
import { injectGlobal } from 'styled-components';

injectGlobal`
  html,
  body,
  #root {
    height: 100%;
    width: 100%;
    margin: 0;
    background-color: #222222;
  }

  body {
    font-family: "Lato", "Helvetica Neue", Helvetica, Arial, sans-serif;
    color: #ffffff;
    font-size: 15px;
  }
`;
```