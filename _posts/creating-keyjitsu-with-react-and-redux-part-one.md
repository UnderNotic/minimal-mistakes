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

## Basic setup
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

After this package.json will be:

```json
{
  "name": "keyjitsu",
  "version": "0.1.0",
  "author": "Piotr Szymura <undernotic@gmail.com>",
  "dependencies": {
    "bootstrap": "^4.0.0-beta.2",
    "styled-components": "2.3.0",
    "react": "^16.2.0",
    "react-dnd": "^2.5.4",
    "react-dom": "^16.2.0",
    "redux": "^3.7.2",
    "react-redux": "^5.0.6",
    "react-router": "4.2.0"
  },
  "devDependencies": {
    "react-scripts": "1.0.17",
    "babel-jest": "21.2.0",
    "enzyme": "^3.2.0",
    "enzyme-adapter-react-16": "1.1.0",
    "enzyme-to-json": "3.3.0",
    "redux-mock-store": "^1.3.0",
    "sinon": "^4.1.3"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}

```

I will follow project structure I usually use, more about it here ({% post_url 2017-12-08-choosing-the-proper-redux-project-structure %}):

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

Let's proceed with initial hello world using whole stack.
Add Home Container to `containers/Home/index.js`

```javascript
import React from 'react';

export default () => 
    <p>Hello World</p>;

```

Wrap react app with provider that connects it to redux:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './globalStyle';
import Home from 'containers/Home';
import registerServiceWorker from './registerServiceWorker';
import {Provider} from 'react-redux';
import configureStore from './configureStore';

ReactDOM.render( 
    <Provider store={configureStore()}>
        < Home/> 
    </Provider>
    ,document.getElementById('root'));
registerServiceWorker();
```
Create /configureStore.js file, which will create redux store:

```javascript
import {createStore} from 'redux';
import createReducer from './reducers';

export default function configureStore(){
    const reducers = createReducer();
    return createStore(reducers);
}
```
and /reducers.js to combine all reducers into one:

```javascript
import { combineReducers} from 'redux';
import mainReducer from 'containers/Main/indexDuck';

export default () => 
        combineReducers(
            {
            }
        )
    ;
```

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
We will be using Lato font from google, to do so add following to index.html into the `<head>` section.
```html    
<link href="https://fonts.googleapis.com/css?family=Lato" rel="stylesheet">
``` 


Now start with `npm start` and You should see Hello world on grey background.

Let check basic test setup:

# mocking redux store with redux wrapper 

# jest tests with enzyme

# it runs