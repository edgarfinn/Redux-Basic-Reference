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
    A container is a smart component that has access to the state that is produced by Redux.

    React and Redux are two separate libraries, and its only through a third library called [React-Redux](https://github.com/reactjs/react-redux) that we can combine the two, thereby creating a react component, which is aware of the state stored in Redux.

    You can have multiple containers in one app, but you should only ever make a component a container when you need it to concern itself with a piece of state.

    Redux architecture revolves around a strict unidirectional data flow. Downwards data flow is therefore a popular principal, in which only the parent-most component in an application is responsible for fetching data, which can then be passed in a single direction downwards, to its child components.

    In light of the downwards data flow principal, only the parent-most component that needs to care about a particular piece of state needs to be a container / smart component. This doesn't always mean the index or app.js module, you may want various child components to be containers as well / instead.

HOW ?
---

### Connecting React to Redux:
#### Ingredients:
- ```import {connect} from 'react-redux'```
- A smart / class-based component, which makes use of the state via ```this.props```.
- A [state function](#statefunction), which takes an argument 'state' and returns an object which represents some state ({books: state.books}).
- Finally - a call to **connect**
  - ```export default connect(stateFunction)(mySmartComponent)```

The redux (state) and react (views) libraries are disconnected and independent of one another, and it is only through **react-redux** that they become connected and collaborative.

A **container** is a normal react component that gets bonded to the applications state via the above process.

The container is created by taking a class component, and bonding is to the apps state using the state function together with the ```connect``` function imported from 'react-redux' module.

#### StateFunction:
A state function is written to take ```state``` as an argument, and map a particular value from it to a relevant to the value of a key in a new object which is returned.
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
The state function is used in conjunction with the connect function to pass the state object to the container's props.

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

// whatever is returned from this function
// will show as props in BookList
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

#### IMPORTANT:

- Redux state is accessed within a container using ```this.props...```

Similarly to React smart components, whenever our application state changes, our container will re-render, and cause all child components to re-render also.

Actions and Action Creators
---

An Action, is an object that describes a user interaction that should change the state. For example, clicking on a book item, in a list of books, thereby selected the book clicked on.

Actions and Action Creators are used for changing state.

An action creator is a function that returns an object (which is the action). That action object contains a 'type' property, which describes the action being triggered, as well as any specific data relating to the action:

```js
// action creator, triggered by user events.
function() {
  return {
    type: 'BOOK_SELECTED',
    book: {title: 'Harry Potter'}
  }
}
```

The action object, returned by the action creator is passed through all reducers:

```js
// action object
{
  type: 'BOOK_SELECTED',
  book: {title: 'Harry Potter'}
}
```
If your reducers are be written with a switch statement, you can determin the state that is returned, based on the type of any action it is passed.

```js

switch(action) {
  case BOOK_SELECTED:
  return action.book
  default:
  // ignore this action and just return the currentState unchanged
  return currentState  
}

```
