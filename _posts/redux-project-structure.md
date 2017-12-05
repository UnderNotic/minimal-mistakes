---
title: "React / Redux project structure"
excerpt: "Project structure, file names etc."
header:
 teaser: ""
tags: 
  - react
  - reactjs
  - redux
  - project structure
  - js
  - frontend
--- 
### Not so hot way
Typical structure for example project looks something like this:

```
components/
    TodoList.js
containers/
    TodoContainer.js
reducers/
    todoReducer.js
actions/
    todoActions.js
constants/
    todoContants.js
selectors/
    todoSelectors.js
sagas/
    todoSaga.js
index.js
```
Files are grouped by technology roles not by features.

It is completely fine to use it for small projects but as project grows this won't scale up.
Having more than 10 files in directory makes searching difficult, also import autocomplete becomes a hassle with dozen of options available.

### Features oriented over everything else
Personally I use below structure:
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
            te/sts
            .
            .
            .
    css/ -> css related files if not using css-in-js
    image/ -> images
    utils/ -> code that doesn't fit into any other place
    index.js -> entry point
    configureStore.js -> redux store configuration
    reducers.js -> combine reducers
.env -> contains 'NODE_PATH=src/', needed for absolute imports
package.json
README.md
```

Everything is divided by features, it scales by adding new directories per feature and not having +10 files in directories.
It uses index.js files because typically a feature in redux is seen as a single container file, this way you can easily import something by doing `import Todo from containers/Todo`. Other files in feature directory are most of the times imported withing this directory.

I also use duck convention so redux constants, action creators and reducers are in one file under specific feature.
It eliminates context switching and I don't like having 10 lines of imports in each redux file. 
More about it
[here](https://github.com/erikras/ducks-modular-redux/).
Of course selectors or sagas are optional and if I'm using redux-thunk,
it logic should be place in duck files under action creators section. 

Additionaly to make `absolute imports` work I have .env file in root directory:
```
NODE_PATH=src/
```   
This file is used by default in create-react-app so if your using it nothing `webpack'y` has to be configured.

### File naming convention
In the past I was using kebab-case for both files and directories no matter what was inside. I wanted to have same convention for server-side code (nodejs) and front-end code (react). 
But I took a different direction and went with community, deciding to use a convention that will tell from just watching at the name of a file or directory what is inside.
- If a file is exporting something that can be instantiated (class, component) then use PascalCase
- index.js files are lower-case
- everything else is camelCase
- test files are pascalCase.test.js
- directory that contains index.js, which fullfil point number 1, shoulkd be PascalCase
- every other directory should be camelCase