Imagine your app’s state is described as a plain object. For example, the state of a todo app might look like this:

```javascript
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
```

To change `state`, you need to `dispatch` an `action`.
An "action" is a plain javascript object that describes what happened:

```javascript
  { type: 'ADD_TODO', text: 'Go to swimming pool' }
  { type: 'TOGGLE_TODO', index: 1 }
  { type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

**Why?**

Because enforcing that every change is described as an action lets us have a clear understanding of what’s going on in the app.
If something changed, we know why it changed.

To tie "actions" and "state" together, we need to write a function called a "reducer":

```javascript
// "app/reducers/visibilityFilter.js"
function reducer(visibilityFilter = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter;
  }

  return visibilityFilter;
}

// "app/reducers/todos.js"
function reducer(todos = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      const todo = {
        text: action.text,
        completed: false
      };

      return [
        todo,
        ...todos,
      ];

    case 'TOGGLE_TODO':
      return todos.map((todo, index) => {
        if (action.index === index) {
          const completed = !todo.completed;
          return {
            completed,
            ...todo,
          };
        }

        return todo;
      });
  }

  return todos;
}

// "app/reducers/index.js"
export { default as visibilityFilter } from 'app/reducers/visibilityFilter';
export { default as todos } from 'app/reducers/todos';
```

And then we write a "root reducer" that manages the entire state of our app by calling those two reducers for the corresponding state keys:

```javascript
import { reducers } from 'app/reducers';

function rootReducer(state = {}, action) {
  return {
    todos: reducers.todos(state.todo, action),
    visibilityFilter: reducers.visibilityFilter(state.visibilityFilter, action),
  };
};
```

And this is the main idea behind Redux: that you describe how your state is updated over time in response to action objects.

-----

## **The three principles**

### Single source of truth

The state of your whole application is stored in single a object tree called `store`.

>  **A store is not a class. It's just an object with a few methods on it.**
> To create it, pass your "rootReducer" function to `createStore`.

* **Store Methods**

1. `getState()`
2. `dispatch(action)​`
3. `subscribe(listener)​`
4. `replaceReducer(nextReducer)​`

_**`getState()​`**_

Returns the current state tree of your application.

```js
console.log(store.getState());

// {
//   visibilityFilter: 'SHOW_ALL',
//   todos: [
//     {
//       text: 'Consider using Redux',
//       completed: true,
//     },
//     {
//       text: 'Keep all state in a single tree',
//       completed: false
//     }
//   ]
// }
```

_**`dispatch(action)​`**_

Dispatches an action. This is the only way to trigger a state change.
The store's "rootReducer" function will be called with the current `getState()`'s result and the given action synchronously.

_**`​subscribe(listener)​`**_

Register a function to be called on state changes (whenever a `dispatch` is fired).

_Usually used to triggers React `render()` method._

_**`​replaceReducer(nextReducer)​`**_

Replaces the reducer currently used by the store to calculate the state.

_It's an advanced API primaraly created to implement code splitting and hot reloading._


### State is read-only

The only way to change state is by emitting an `action`, an object that describes what happened.

This ensures that neither network calls or any view's logic will ever write directly to the state.
Instead, they express an intent to transform the state.

Since actions are plain objects, they can be serialized, stored and replayed for debugging and testing purposes.

```js
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1,
});

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED',
});
```

### Changes are made with pure functions

To specify how the state tree is transformed by actions, you write pure `reducers`.

Reducers are just pure functions that takes the previous `state`, an `action` and return the next state.
And the most important: **don't mutate the "state" object passed to the reducer**, return a new state object instead.

```js
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBLITY_FILTER':
      return action.filter;
    default:
      return state;
  }
}

function todos(state = [], action) {
  // ...
}

import { combineReducers, createStore } from 'redux';
const rootReducer = combinaReducers({ visibilityFilter, todos });
const store = createStore(rootReducer);
```

Just remember that the reducer must be pure. Given the same arguments, it should calculate the next state and return it.
_No surprises. No side effects. No API calls. No mutations. Just a calculation._
