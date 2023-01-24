---
title: "In order to understand Redux, you must understand Redux"
date: 2023-01-23T18:01:53-05:00
draft: false
tags:
  - JavaScript
  - React
  - Redux
---

I started learning **Redux** this week and while doing so, I realized that before I learn the core Redux library and the functions it has to offer, I'd be better off learning about the Redux as a design pattern for global state management. With a thorough understanding of the Redux design pattern, I can utilize the best tools available to implement the pattern (i.e. the Redux toolkit and Redux library) to their full potential.

> Yes, before Redux is a library, it is a design pattern. Redux has reducers and actions. Reducers store the state and actions change the state. 

Redux can be described in three fundamental principles: 

## Store: The single source of truth 

The global state of your application is stored in a large, single object, called redux store.

The Redux store itself is a simple observable/pub-sub implementation, with a single "change/updated" event emitter. The use of actions and reducers has some similarities to CQRS and event sourcing as well. See https://redux.js.org/introduction/motivation and https://redux.js.org/introduction/prior-art .

```javascript
console.log(store.getState())

/* Prints
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
*/
```

## Actions: State is read-only

The global state of the application stored as an object is immutable. The only way to change the state is to emit an action, an object describing what happened.

```javascript
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

## Reducers: The modifications are done with pure functions called Reducers

```javascript
function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}
```
