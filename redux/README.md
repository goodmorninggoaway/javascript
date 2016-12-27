# Redux Style Guide

*A mostly reasonable approach to Redux*

## Table of Contents

1. [Actions](#actions)
1. [Action Types](#action-types)
1. [Actions Creators](#action-creators)
1. [Reducers](#reducers)
1. [Connecting React Components](#connecting-react-components)


## Actions
Action objects should follow the [Flux Standard Action](https://github.com/acdlite/flux-standard-action) schema.

## Action Types

### Naming Actions
Action types should be named `NOUN_VERB`
```js
// bad
const FETCHED_USERS = 'FETCHED_USERS';
const FETCHING_USERS = 'FETCHING_USERS';
const FETCH_USERS_ERROR = 'FETCH_USERS_ERROR';

// good
export const USERS_FETCH = 'USERS_FETCH';
export const USERS_FETCH_SUCCEEDED = 'USERS_FETCH_SUCCEEDED';
export const USERS_FETCH_FAILED = 'USERS_FETCH_FAILED';
```

If you need a composite noun, use `NOUN_NOUN_...NOUN_VERB`
```js
// good
export const MADE_IN_USA_APPLIANCES_FETCH = 'MADE_IN_USA_APPLIANCES_FETCH';
```
Consider [namespacing](#namespaced-values) your action types if you notice many composite action names.

#### HTTP Requests
Most actions related to HTTP requests should have start, success, and error actions.

```js
export const USERS_FETCH = 'USERS_FETCH';
export const USERS_FETCH_SUCCEEDED = 'USERS_FETCH_SUCCEEDED';
export const USERS_FETCH_FAILED = 'USERS_FETCH_FAILED';
```

This enables UI state such as loading indicators and flash notifications on error.

### Casing
Keep action types in UPPER_SNAKE_CASE. This prevents spelling errors (eg. upper_Snake_Case).
```js
// bad
export const usersFetch = 'usersFetch';

// good
export const USERS_FETCH = 'USERS_FETCH'
```

### Value
Unless namespaced, the value of the string should be the same as the variable name
```js
// bad
export const USERS_FETCH = 'This action means the GET /users request has started';

// good
export const USERS_FETCH = 'USERS_FETCH'
```

### Namespaced Values
If the action type is namespaced, the convention is `npm-module-or-app/reducer/ACTION_TYPE`. 
```js
// bad
export const USERS_FETCH = 'users-USERS/FETCH';

// good
export const USERS_FETCH = 'myApp/users/USERS_FETCH'
```

See ["Ducks" Proposal](https://github.com/erikras/ducks-modular-redux)

### Using Action Types
Namespacing and creating HTTP action types can create a lot of boilerplate. 

NEVER use inline (magic) strings for action types. Either export action types as constants OR use a function to generate actions. Prefer constants until you prove a good use case for functions.


```js
// bad
function actionCreator() {
	return { type: 'SID_SCIENCE_KID' }
}

// reducer.js
...
case 'SID_SCIENCE_KID': {
	return {...state, pancakes: 1};
}


// good

// constants.js
export SID_SCIENCE_KID = 'SID_SCIENCE_KID';


// actionCreators.js
import {SID_SCIENCE_KID} from './constants.js';

function actionCreator() {
	return { type: SID_SCIENCE_KID };
}

// reducer.js
import {SID_SCIENCE_KID} from './constants.js';
...
case SID_SCIENCE_KID: {
	return {...state, pancakes: 1};
}

```


## Action Creators

- Group related action creators in one file.
- Use [redux-thunk](https://github.com/gaearon/redux-thunk) for asynchronous actions
- Expose each action creator as a named export.
- Don't use a default export.

```js
// actions/something.js
export function doSomething() {
  return {
    type: 'DO_SOMETHING',
    payload: { foo: 'bar' },
    meta: { baz: 'qux' }
  };
}
```

Action creators are functions which return an action object. An action object contains a type and optionally a payload
and metadata. Action objects should follow the [Flux Standard Action] schema.

We could also use the createAction helper of redux-actions, but other than enforcing FSA it doesn't do much here in
terms of reducing boilerplate. In fact createAction is less readable and doesn't let us easily export named functions
(exporting anonymous or arrow functions should be avoided). Note that only `type` is mandatory.


## Reducers

- Expose reducers as the default export.
- Use constants instead of inline strings for action types.
- Always define an `initialState` variable.

```js
// using redux-actions

// usersReducer.js
import { USERS_FETCH_SUCCEEDED } from './constants';

const initialState = {};

const handlers = {
	[USERS_FETCH_SUCCEEDED]: (state, action) => ({...action.payload})
};

```

A reducer handles incoming actions. All reducers are triggered for all actions, so it's up to each reducer to decide
whether or not to act on the incoming action. This is commonly done by switching on the action type. Reducers receive
a current state and an action object and must return a new (updated or not) state. Their function signature is as
follows:

```js
(state, action) => state
```

Reducers in Redux are pure functions, which means they cannot access outside information other than the arguments they
are given. It's also not allowed to mutate outside state, be it by referencing outer scope or by mutating objects which
they receive as arguments. In other words they must take the data they are given and return a new state without using
mutation. This can often be achieved by using the array and object spread operators.

The simplest way to create a reducer is to use the handleActions method of redux-actions. This avoids having to write a
switch statement and a default handler. Another benefit is that is enforces the use of FSA-compliant action objects. The
final reducer should be exposed as the default export. Individual handler functions can be exposed as named exports
in order to simplify unit testing.


## Connecting React components

- Prefer a state selector function that returns the minimal state subset you need.
- Expose the connected component as default export.
- Expose the unconnected component as named export for unit testing.

```js
export class TodoList extends React.Component { ... }

const stateSelector = ({ todos, filters }) => ({ todos, filters });
export default connect(stateSelector)(TodoList);
```

The [reselect](https://github.com/rackt/reselect) library works well for more complex selectors and to achieve better performance.

