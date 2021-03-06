# typed-reducer

This library is a framework-agnostic attempt to create typed reducers with Typescript following a class-based approach. Due to this, it plays particularly nicely with @ngrx. 

## Install

    npm i typed-reducer --save 

## Usage

```javascript
import { createReducer, Action } from 'typed-reducer';

const ACTION_TYPE = 'ACTION_TYPE';

// define an action
class MyAction {
    public type = ACTION_TYPE;
    constructor(public payload: string) {}
}

// create a class, each method is a branch of what we would normally do with a switch statement
export class Reducer {
    @Action
    public someMethod(state: string[], action: MyAction): State {
        return [ ...state, action.payload];
    }
}

export const reducer = createReducer(Reducer);
```

### Full example

```javascript
import { createReducer, Action } from 'typed-reducer';
import { CreateTodoAction, MarkTodoDoneAction, ArchiveTodoAction } from './todo.actions';
import { createTodo } from './create-todo';

export class TodoReducer {
    @Action
    public createTodo(state: Todo[], action: CreateTodoAction): Todo[] {
        return [ ...state, createTodo(action.payload)];
    }

    @Action
    public markDone(state: Todo[], action: MarkTodoDoneAction): Todo[] {
        return state.map(todo => {
            return todo.id === action.payload ? { ...todo, done: true } : todo
        });
    }

    @Action
    public archiveTodo(state: Todo[], action: ArchiveTodoAction): Todo[] {
        return state.filter(todo => todo.id !== action.payload);
    }
}

export const reducer = createReducer(TodoReducer);
```

### Use with Ngrx and AOT

```javascript

import { reducer } from './todos.reducer.ts';

// initial state of the store's tree
const initialState = {
  todos: []
};

// create your store tree
export const rootReducer = (
  state: AppState = initialState,
  action: Action
) => {
  return {
    todos: reducer(state.todos, action),
    // here go your other reducers
  };
};

// export a factory to be AOT compliant
export function reducerFactory() {
  return rootReducer;
};

// let's pass the factory function to StoreModule
@NgModule({
  // module's configuration ...
  imports: [
    StoreModule.forRoot(undefined, { reducerFactory })
  ]
})
export class AppModule { }
```

## Options

### Freeze
In order to prevent errors related to mutating the state, you can pass the option `freeze`, so that the state will be frozen using Object.freeze. Do this only in dev mode.

```javascript
const options = { freeze: true };
export const todos = createReducer(TodoReducer, options);
```

### Log

It is possible to log every action by setting the property log to `true`. This has been inspired by [ngrx-store-logger](https://github.com/btroncone/ngrx-store-logger/blob/master/src/index.ts). Do this only in dev mode.

![Logging](https://raw.githubusercontent.com/Gbuomprisco/typed-reducer/master/example-app/log.png)

```javascript
const options = { log: true };
export const todos = createReducer(TodoReducer, options);
```

