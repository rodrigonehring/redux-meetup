![redux-logo](https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67)
---

Why?

+++

![](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-03.svg)

+++

### Three Principles

##### Single source of truth
The state of your whole application is stored in an object tree within a single store.

##### State is read-only
The only way to change the state is to emit an action, an object describing what happened.

##### Changes are made with pure functions
To specify how the state tree is transformed by actions, you write pure reducers.

---

Basics - redux

+++
### Actions
Actions are payloads of information that send data from your application to your store. They are the only source of information for the store. You send them to the store using store.dispatch().

+++
```javascript
const addTodo = (message) => ({
  type: 'ADD_TODO',
  payload: message
})

store.dispatch(
  addTodo('Build my first Redux app')
)
```

+++
### Reducers
Actions describe the fact that something happened, but don't specify how the application's state changes in response. This is the job of reducers.

+++
```javascript
function(state, action) {
	switch (action.type) {
		case 'ADD_TODO':
			return {
				...state,
				todos: [].concat(state.todos, action.payload),
			}
		default:
			return state
	}
}
```


+++
### initialState Reducer
```javascript

const initialState = {
	todos: ['todo1', 'todo2']
}

function(state = initialState, action) {
	switch (action.type) {
		...
		default:
			return state
	}
}
```

+++
### store
```javascript

import { createStore } from 'redux'
import todoApp from './reducers'

const store = createStore(todoApp)

```

+++
### store - persist data
```javascript

import { createStore } from 'redux'
import todoApp from './reducers'

const store = createStore(todoApp, window.STATE_FROM_SERVER)

// or

const store = createStore(todoApp, window.STATE_FROM_LOCAL_STORAGE)

```

+++
### store - combineReducers
permite criar estruturas mais "complexas"
```javascript
{
  app: {...},
  user: {...},
  posts: {...},
  todos: {...},
}

```

---

Basics - react-redux

+++
### Provider
Makes the Redux store available to the connect() calls in the component hierarchy below. Normally, you can’t use connect() without wrapping a parent or ancestor component in <Provider>.

+++
```javascript
import { Provider } from 'react-redux'
import configureStore from './configureStore'
import MyRootComponent from './MyRootComponent'

const store = configureStore()

ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
)
```

+++
### React Context
Todo componente react, seja class ou function, recebe props e context;  

```javascript
const MyComponent = (props, context) => {
  context.store.getState().todos...
}

MyComponent.contextTypes = {
  store: PropTypes.object
};
```

+++
### React-redux connect()

```javascript
import { connect } from 'react-redux'

const MyComponent = ({ todos }) => {
  ...
}

function mapStateToProps(state) {
  return {
    todos: state.todos,
  }
}

export default connect(mapStateToProps)(MyComponent)
```
+++
### React-redux connect()

```javascript
import { connect } from 'react-redux'
import { makeActionX } from './actions'

const MyComponent = ({ makeActionX }) => (
  <button onClick={makeActionX} />
)

function mapDispatchToProps(dispatch) {
  return {
    makeActionX: (event) => dispatch(makeActionX(event))
  }
}

export default connect(null, mapDispatchToProps)(MyComponent)

```

---

## Redux Middleware

+++

Colocar code entre dispatch de uma action e antes do reducer
![](https://image.slidesharecdn.com/reactjs-reduxadvanced-160718135632/95/workshop-22-reactredux-middleware-15-638.jpg?cb=1470751997)

+++

### Adicionando middleware

```javascript
import { createStore, combineReducers, applyMiddleware } from 'redux'

let todoApp = combineReducers(reducers)
let store = createStore(
  todoApp,
  // applyMiddleware() tells createStore()
  // how to handle middleware
  applyMiddleware(logger, crashReporter)
)
```

+++

### Estrutura middleware

```javascript
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

+++

### Exemplo emit socket

```javascript
import socket from './socket'

const socketMiddleware = store => next => action => {
  if (action.socket) {
    socket.emit(action.type, action.payload)
  }

  return next(action)
}
```
 
+++

### Exemplo emit socket

```javascript
const enterRoom = (room) => ({
  type: 'WS_ENTER_ROOM',
  payload: room,
  socket: true,
});


store.dispatch(enterRoom(1))
```

+++

### Exemplo socket.on()

```javascript
import socket from './socket'

const socketMiddleware = store => {

  socket.on('channelName', data => {
    store.dispatch(data.type, data.payload)
  })
  
  return next => action => {
    return next(action)
  }
}
```

+++
### Exemplo final
```javascript
const socketMiddleware = io => store => {

  io.on('connect', () => store.dispatch({ type: 'WS_CONNECTED' }));

  io.on('CMD_PUBLISH', ({ event, payload }) => {
    store.dispatch({ type: event, payload });
  });

  return next => action => {
    if (action.socket) {
      const event = action.socket.event;
      io.emit(event, action.payload);
    }
    return next(action);
  };
};
```



---
  # Tools
+++
#### Redux devtools
* Lets you inspect every state and action payload
* Lets you go back in time by “cancelling” actions
* If you change the reducer code, each “staged” action will be re-evaluated
* If the reducers throw, you will see during which action this happened, and what the error was
* With persistState() store enhancer, you can persist debug sessions across page reloads
---

The End :)
