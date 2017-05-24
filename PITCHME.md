![redux-logo](https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67)
---

Why?

+++

![](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-03.svg)

+++

### 3 princípios

+++
##### Single source of truth
Todo o estado da sua aplicação é armazenado em uma única árvore de objetos, dentro de uma única store. Qualquer acesso ao state, é feito através de referência ao dado armazenado na store. Essa prática evita que você tenha dados duplicados, e uma vez que um dado é atualizado, a alteração se propaga para toda a aplicação.

+++
##### State is read-only
A única forma de alterar o estado da sua aplicação é emitindo uma action, um objeto descrevendo o que aconteceu.
Para acessar o state, você pode utilizar o método getState da store, o mesmo retorna todo o estado da aplicação, mas somente para a leitura.

+++
##### Changes are made with pure functions
Para descrever como o state da aplicação será alterado pelas actions, nós escrevemos pure reducers.
Reducers são funções que são chamadas toda vez que uma action é disparada e recebem como parâmetros o state atual e a action, e devolvem um novo state.

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

![](https://image.slidesharecdn.com/reactjs-reduxadvanced-160718135632/95/workshop-22-reactredux-middleware-15-638.jpg?cb=1470751997)

+++

### Adicionando middleware

```javascript
import { createStore, combineReducers, applyMiddleware } from 'redux'
import logger from './mymiddleware'

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

Pure functions

+++
A função abaixo possui inputs e outputs bem definidos:
```javascript
function square(x) {
    return x * x;
}

square(2); // 4
```

+++
A função abaixo porém, não possui inputs e outputs tão bem definidos:
```javascript
function generateDate() {
    var date = new Date();
    generate(date);
}

generateDate(); // ???
```

+++

### Memoization
É o cache do resultado de uma função, baseado nos parâmetros de entrada.
Ao chamar uma função com determinados parâmetros, se o resultado pedido já estiver no cache, retorna ele, ao invés de calcular/fazer tudo novamente.

+++

Imutabilidade

+++
mutável
```javascript
let a = { b: 1 }
let c = a;
c.b = 2
console.log(a.b) // 2
a === c // true
```

+++
imutável
```javascript
let a = { b: 1 }
let c = { ...a };
c.b = 2
console.log(a.b) // 1
a === c // false
```

+++
PureComponent
```javascript
shouldComponentUpdate(nextProps) {
  return nextProps.a !== this.props.a;
}
```

---

  ### redux thunk

+++

Por padrão, actions creators return actions (object: type, payload).<br/>
Redux thunk, retorna uma function, que recebe function(dispatch, getState).<br/>
Possibilitando:
```javascript
  function incrementAsync() {
    return dispatch => {
      setTimeout(() => {
        dispatch({ type: 'EVENT' });
      }, 1000);
    };
  }
```

+++

```javascript
  function requestData() {
    return (dispatch, getState) => {
      const state = getState()

      return fetch(`/url/${state.id}`)
        .then(res => dispatch({
          type: 'REQUEST_DATA',
          payload: res.data,
        }))
    };
  }
```


---

  ### redux saga
  redux-saga is a library that aims to make side effects

+++

```javascript
  import { call, put, select, takeLatest } from 'redux-saga/effects'

  function* fetchUser(action) {
    const state = yield select();
    const user = yield call(fetch, `/url/${state.id}`);

    yield put({
      type: 'USER_FETCH_SUCCEEDED', user: user
    });
  }

  function* mySaga() {
    yield takeLatest('USER_FETCH_REQUESTED', fetchUser);
  }

  export default mySaga;
```

---

  ### reselect
  * Selectors can compute derived data, allowing Redux to store the minimal possible state.
  * Selectors are efficient. A selector is not recomputed unless one of its arguments change.
  * Selectors are composable. They can be used as input to other selectors.

+++

createSelector(...inputSelectors | [inputSelectors], resultFunc)

```javascript
  import { createSelector } from 'reselect'

  const mySelector = createSelector(
    state => state.values.value1,
    state => state.values.value2,
    (value1, value2) => value1 + value2
  )

```

---
![](https://cdn.pbrd.co/images/acCT3I0IO.png)

---

#### Redux devtools
* Lets you inspect every state and action payload
* Lets you go back in time by “cancelling” actions
* If you change the reducer code, each “staged” action will be re-evaluated
* If the reducers throw, you will see during which action this happened, and what the error was
* With persistState() store enhancer, you can persist debug sessions across page reloads

+++
![](https://i.imgur.com/Tz7sCdB.png)
+++
![](https://i.imgur.com/vKzQqJZ.png)

---
The End :)
