# Redux-Basic-Reference (for React)

WHAT ?
---
Redux is a predictable state container for Javascript Apps. In other words, its a collection of all the data that describes the application.

React vs Redux

- Redux serves to construct the applications state.
- Whereas React provides views to display and present that state.


## Key terminology:
- #### State:

    Everything that changes in your app including data and user interactions is contained in a single object referred to as the 'state' or the 'state tree'

    When talking about state with redux, this is referring specifically to application-level state (ie the state of the entire app), as opposed to the more localised component-level state.

- #### Reducers:
    A reducer is a function that returns a piece of the applications state. If there are many pieces of state, there can be many different reducers.

    ```js
    // reducers/reducer_books.js

    export default function() {
      return [
        {title: 'Javascript the good parts'},
        {title: 'Harry Potter'},
        {title: 'Eloquent Ruby'},
        {title: 'The Dark Tower'}
      ]
    };

    ```
    Since its likely that you'll have several different bits of state, you'll likely have several reducers, so in order to combine them, you'll want to use an index page that merges them all together like so:

    ```js
    // reducers/index.js

    // here, BooksReducer returns the array of book objects from the above example.

    import { combineReducers } from 'redux';
    import BooksReducer from './reducer_books';

    const rootReducer = combineReducers({
      // once mapped to a components props using connect,
      // this key will be accessible within a containers props
      books: BooksReducer
    });

    export default rootReducer;
    ```


- #### Containers:
    A container is a smart component that has been given access to the state that is produced by Redux.

    You can have multiple containers in one app, but you should only ever make a component a container when you need it to concern itself with a piece of state.

    Redux architecture revolves around a strict unidirectional data flow. Downwards data flow is therefore a popular principal, in which only the parent-most component in an application is responsible for fetching data, which can then be passed in a single direction downwards, to its child components.

    In light of this downwards data-flow principal, only the parent-most component that needs to care about a particular piece of state needs to be a container. This doesn't always mean the index or app.js module, you may want various child components to be containers as well / instead.

    React and Redux are two separate libraries, and its only through a third library called [React-Redux](https://github.com/reactjs/react-redux) that we can combine the two, thereby creating a react component, which is aware of the state stored in Redux.

- #### Actions and Action Creators    
    In short, Actions and Action Creators are used for changing state.

    An Action, is an object that describes a user event that should change some state. An action always contains a ```type``` property, which describes the action being triggered. If necessary, it **might** also contain additional information, providing further context or details relating to the action, which is typically stored inside a ```payload```.

    For example, selecting a book from a list by clicking on it. Here, the user event (clicking a book item) triggers an action creator ( ```selectingBook()```...), which produces an action ```{type: BOOK_SELECTED, payload: {title: 'Eloquent Javascript'}}```.

HOW ?
---

#### Redux Data Flow:

- 1) User event triggers an action creator function, possibly also passing info about the data / item on which the event occurred (ie the book being selected).

- 2) The action creator will dispatch an action object, containing a ```type``` key, describing the purpose of the action, and possibly a ```payload``` key, used to provide further context of the action (such as the book being selected).

- 3) The action is automatically sent to all reducers, which either respond to the action with an updated state object, or ignore the action, returning the current state, un-mutated.

- 4) All reducers process the action, and return a new state, assembled from all reducers. The new state then notifies containers of any changes.

- 5) Any containers with updated state then re-render, adapting to the new state.

### SETUP:

This set-up guide is based on create-react-app

```bash
$ npm i redux react-redux --save
```

1) Place your parent-most component inside a Provider Store, which will pass redux state **downwards** into your app. ([READ more about Provider stores here](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store))

src/index.js:
```js
// react set-up
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import registerServiceWorker from './registerServiceWorker';

// redux set-up
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';
import reducers from './reducers';

const createStoreWithMiddleWare = applyMiddleware()(createStore)

ReactDOM.render(
  // Pass state down from your reducers into your app
  <Provider store={createStoreWithMiddleWare(reducers)}>
    <App />
  </Provider>,
  document.getElementById('root')
);
registerServiceWorker();
```

2) a) Write some reducers to provide state to your app

src/reducers/reducer_albums.js
```js
export default () => {
  return [
    {title: 'Illmatic', artist: 'Nas', released: 1994},
    {title: '2001', artist: 'Dr Dre', released: 1999},
    {title: 'The Score', artist: 'Fugees', released: 1996}
  ]
}
```
src/reducers/reducer_active_album.js

```js
export default (state = null, action) => {
  switch(action.type) {
    case 'ALBUM_SELECTED':
      return action.payload;
  }
  return state;
}
```

2) b) ...and combine them into a rootReducer, using redux's ```combineReducers``` method.

src/reducers/index.js
```js
import { combineReducers } from 'redux';
import AlbumsReducer from './reducer_albums';
import ActiveAlbum from './reducer_active_album';

const rootReducer = combineReducers({
  albums: AlbumsReducer,
  activeAlbum: ActiveAlbum
});

export default rootReducer;

```
3) Write an action creator:

src/actions/index.js
```js
export function selectAlbum(album) {
  return {
    type: 'ALBUM_SELECTED',
    payload: album
  }
}
```

4) Upgrade a smart component to a container to complete the redux data flow.

There are several steps here:

  - a) Import:
    - ``` {connect}``` from react-redux
    - your action creator(s)
    - and ```{bindActionCreators}``` from redux

  - b) Map your state to your container's props using ```connect```.
    - This is done using a react-redux ```connect()``` invocation; taking a mapStateToProps function as an argument, and your smart component as a curried argument:
      ```connect(mapStateToProps)(BookList)```
    - A mapStateToProps function takes ```state``` as an argument, and return an object that represents that state:
    
    ```js
    // whatever is returned from this function will show in the container's props
    const mapStateToProps = (state) => {
      return {
        // whatever key being referenced here
         // must be defined as as a key in the combineReducers index module
        books: state.books
      };
    }
    ```


  src/containers/album_list.js

    ```js

    ```

#### Ingredients:
- ```import {connect} from 'react-redux'```
- A smart / class-based component, which makes use of the state via ```this.props```.
- A [state-mapping function](#state-mapping-function), which takes an argument 'state' and returns an object which represents some state ({books: state.books}).
- Finally - a call to **connect**
  - ```export default connect(mapStateToProps)(mySmartComponent)```

The redux (state) and react (views) libraries are disconnected and independent of one another, and it is only through **react-redux** that they become connected and collaborative.

A **container** is a normal react component that gets bonded to the applications state via the above process.

The container is created by taking a class component, and bonding it to the apps state using the state mapping function, together with the ```connect``` function imported from 'react-redux' module.

#### State Mapping Function:
A state-mapping function is written to take ```state``` as an argument, and map a particular value from state to a relevant key in a new object which is returned.
```js

// receives state as an argument
const mapStateToProps = (state) => {
  // returns a new object...
  return {
    // ...which has a particular value from state
    // assigned to a sensible key reference
    books: state.books
  };
}
```
The state-mapping function is used in conjunction with the connect function to pass the state object to the container's props.

```js
// containers/book-list.js
import React, {Component} from 'react';
// connect from react-redux is used together with mapStateToProps
// to bond the state returned from
import { connect } from 'react-redux';

class BookList extends Component {

  renderList() {
    // redux state is accessed using 'this.props'
    return this.props.books.map((book) => {
      return (
        <li key={book.title} className="list-group-item">{book.title}</li>
      )
    })
  }

  render() {
    return (
      <ul className="list-group col-sm-4">
        {this.renderList()}
      </ul>
    )
  }
}

// STATE-MAPPING FUNCTION
// whatever is returned from this function will show as props in the BookList container
const mapStateToProps = (state) => {
  return {
    // whatever key being referenced here
     // must be defined as as a key in the combineReducers index module
    books: state.books
  };
}


// here, the connect function uses the mapStateToProps function
// to bond the state object (returned by the your reducers)
// to the BookList component's props
export default connect(mapStateToProps)(BookList)
// the returned value here is your **container**

```

Redux state is accessed within a container using ```this.props...```

Similarly to React smart components, whenever our application state changes, our container will re-render, and cause all child components to re-render also.

Actions and Action Creators
---

**Actions and Action Creators are used for changing state.**

An action creator is a function that returns an action (object).

```js
// action creator, triggered by user events.
function selectBook(book) {
  // selectBook is an action creator that needs to return an action;
  // an object with a type property
  return {
    type: 'BOOK_SELECTED',
    payload: book
  }
}
```

```js
// action object
{
  type: 'BOOK_SELECTED',
  payload: {title: 'Harry Potter'}
}
```
The action object, returned by the action creator is passed through all reducers, which update the state according to the nature and contents of the action.


Using a switch statement, you can determine the state that is returned, based on the type of any action it is passed.


```js

switch(action) {
  case BOOK_SELECTED:
  return action.book
  default:
  // ignore this action and just return the currentState (unchanged)
  return currentState  
}

```

In order to trigger actions from user interactions or other ongoing events, you need to import the action, and map it to your containers props (similarly to mapping state to props), using redux's ```bindActionCreators``` function.

This ```bindActionCreators``` function is used to make sure the action flows through all the application's reducers.

```js
import { selectBook } from '../actions/index';
import { bindActionCreators } from 'redux';
```

Once imported, you then need to write a mapping function, that takes an object

WRITE NOTES ON Upgrading a smart component to a container, and the action reducers data flow setup, using:

```js
mapStateToProps
```
```js
mapDispatchToProps
```
```js
connected
```

```js
bindActionCreators
```

- Middleware

  Redux-Promise package takes promisified ajax responses, and intervenes between the action and the reducer. If the action payload is a promise, it converts it into the response data value, and passes that data on to the reducer instead of the promise.

### Avoid State Mutations in Reducers
Never mutate state in your reducers, instead, return a completely new piece of state.

```js
// BAD :(
export default function(state = [], action) {

  switch (action.type) {
    case FETCH_WEATHER:
    // mutative
      return state.push(action.payload.data);
  }
  return state;
}
```

```js
// GOOD :)
export default function(state = [], action) {

  switch (action.type) {
    case FETCH_WEATHER:
    // non-mutative
      return state.concat([action.payload.data]);
  }
  return state;
}
```
