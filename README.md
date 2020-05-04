# React State Hooks

This may help you to get a better understanding of how React Hooks -- such as useState, useReducer, and useContext -- can be used in combination for impressive state management in React applications. We will almost reach the point where these hooks mimic sophisticated state management libraries like Redux for globally managed state.

## Contents

1. React useState for simple state management
2. React useReducer for complex state management
3. React useContext for global state management

## useState

```
import React, {useState} from 'react';
import uuid from 'uuid/v4';

const initialTodos = [
    {
        id: uuid(),
        task: 'Build React UI',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Connect Firebase',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Learn GraphQL',
        complete: true,
    },
];

const App = ()=> {
    const [todos, setTodos] = useState(initialTodos);
    const [task, setTask] = useState('');

    const handleChangeInput = event => {
        setTask(event.target.value);
    };
    const handleSubmit = event =>{
        if(task) setTodos(todos.concat({id: uuid(), task, complete: false}));
        setTask('');
        event.preventDefault();
    };

    const handleChangeCheckbox = id => {
        setTodos(
            todos.map(todo=> {
                if(todo.id===id){
                    return {...todo, complete: !todo.complete};
                } else{
                    return todo;
                }
            })
        );
    };

    return (
    <div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <label>
              <input
                type="checkbox"
                checked={todo.complete}
                onChange={() => handleChangeCheckbox(todo.id)}
              />
              {todo.task}
            </label>
          </li>
        ))}
      </ul>
     <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={task}
          onChange={handleChangeInput}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
};

export default App;

```

## useReducer

Once you run into more complex state objects or state transitions--which you want to keep maintainable and predictable, the useReducer hook is a great candidate to manage them. Let's see how we can add buttons to filter our list of todos for showing all todo items, only complete todo items and only incomplete todo items.

```
import React, {useState, useReducer} from 'react';
import uuid from 'uuid/v4';

const initialTodos = [
    {
        id: uuid(),
        task: 'Build React UI',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Connect Firebase',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Learn GraphQL',
        complete: true,
    },
];

const filterReducer = (state, action)=> {
    switch(action.type){
        case 'SHOW_ALL':
            return 'ALL';
        case 'SHOW_COMPLETE':
            return 'COMPLETE';
        case 'SHOW_INCOMPLETE':
            return 'INCOMPLETE';
        default:
            throw new Error();
    }
};

const todoReducer = (state, action) => {
  switch (action.type) {
    case 'DO_TODO':
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, complete: true };
        } else {
          return todo;
        }
      });
    case 'UNDO_TODO':
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, complete: false };
        } else {
          return todo;
        }
      });
    case 'ADD_TODO':
      return state.concat({
        task: action.task,
        id: uuid(),
        complete: false,
      });
    default:
      throw new Error();
  }
};


const App = ()=> {
    const [todos, dispatchTodos] = useReducer(todoReducer,initialTodos);
    const [filter, dispatchFilter] = useReducer(filterReducer, 'ALL');
    const [task, setTask] = useState('');

    const handleChangeInput = event => {
        setTask(event.target.value);
    };
    const handleSubmit = event =>{
        if(task) dispatchTodos({ type: 'ADD_TODO', task, id: uuid() });
        setTask('');
        event.preventDefault();
    };

    const handleChangeCheckbox = todo => {
        dispatchTodos({
            type: todo.complete ? 'UNDO_TODO' : 'DO_TODO',
            id: todo.id,
        });
    };

    const handleShowAll = () => {dispatchFilter({type: 'SHOW_ALL'})};
    const handleShowComplete = () => {dispatchFilter({type: 'SHOW_COMPLETE'})};
    const handleShowIncomplete = () => {dispatchFilter({type: 'SHOW_INCOMPLETE'})};

    const filteredTodos = todos.filter(todo => {
        if(filter === "ALL"){
            return true;
        }

        if(filter === "COMPLETE" && todo.complete){
            return true;
        }

        if(filter === "INCOMPLETE" && !todo.complete){
            return true;
        }
        return false;
    });

    return (
    <div>
      <div>
        <button type="button" onClick={handleShowAll}>Show All</button>
        <button type="button" onClick={handleShowComplete}>Show Complete</button>
        <button type="button" onClick={handleShowIncomplete}>Show Incomplete</button>
      </div>
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <label>
              <input
                type="checkbox"
                checked={todo.complete}
                onChange={() => handleChangeCheckbox(todo)}
              />
              {todo.task}
            </label>
          </li>
        ))}
      </ul>
     <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={task}
          onChange={handleChangeInput}
        />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  );
};

export default App;

```

A reducer function always receives the current state and an action as arguments. Depending on the mandatory type of the action, it decides what task to perform in the switch case statement, and returns a new state based on the implementation details.

Now you can use the reducer function in a useReducer hook. It takes the reducer function and an initial state and returns the filter state and the dispatch function to change it.

Everytime a button is clicked an action with an action type is dispatched for the reducer function. The reducer function then computes the new state. Often the current state from the reducer function's argument is used to compute the new state with the incoming action.

Now everything that has been managed by useState for our todo items is managed by useReducer now. The reducer describes what happens for each state transition and how this happens by moving the implementation details in there.

## useContext

We can take our state management one step further. At the moment, the state is managed co-located to the component. That's because we only have 1 component after all. What if we would have a deep component tree? How could we dispatch state changes from anywhere?

Let's dive into React's Context API and the useContext hook to mimic more a Redux's philosophy by making state changes available in the whole component tree.

```
import React, {useState, useReducer, useContext, createContext} from 'react';
import uuid from 'uuid/v4';

const TodoContext = createContext(null);

const initialTodos = [
    {
        id: uuid(),
        task: 'Build React UI',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Connect Firebase',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Learn GraphQL',
        complete: true,
    },
];

const filterReducer = (state, action)=> {
    switch(action.type){
        case 'SHOW_ALL':
            return 'ALL';
        case 'SHOW_COMPLETE':
            return 'COMPLETE';
        case 'SHOW_INCOMPLETE':
            return 'INCOMPLETE';
        default:
            throw new Error();
    }
};

const todoReducer = (state, action) => {
  switch (action.type) {
    case 'DO_TODO':
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, complete: true };
        } else {
          return todo;
        }
      });
    case 'UNDO_TODO':
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, complete: false };
        } else {
          return todo;
        }
      });
    case 'ADD_TODO':
      return state.concat({
        task: action.task,
        id: uuid(),
        complete: false,
      });
    default:
      throw new Error();
  }
};

const App = ()=> {
    const [todos, dispatchTodos] = useReducer(todoReducer,initialTodos);
    const [filter, dispatchFilter] = useReducer(filterReducer, 'ALL');

    const filteredTodos = todos.filter(todo => {
        if(filter === "ALL"){
            return true;
        }

        if(filter === "COMPLETE" && todo.complete){
            return true;
        }

        if(filter === "INCOMPLETE" && !todo.complete){
            return true;
        }
        return false;
    });

    return (
        <div>
        <TodoContext.Provider value={dispatchTodos}>
            <Filter dispatch={dispatchFilter}/>
            <TodoList todos={filteredTodos}/>
            <AddTodo />
        </TodoContext.Provider>
        </div>
  );
};

const Filter = ({dispatch}) => {
    const handleShowAll = () => {dispatch({type: 'SHOW_ALL'})};
    const handleShowComplete = () => {dispatch({type: 'SHOW_COMPLETE'})};
    const handleShowIncomplete = () => {dispatch({type: 'SHOW_INCOMPLETE'})};
    return (
        <button type="button" onClick={handleShowAll}>Show All</button>
        <button type="button" onClick={handleShowComplete}>Show Complete</button>
        <button type="button" onClick={handleShowIncomplete}>Show Incomplete</button>
    );
};

const TodoList = ({todos}) => {
    <ul>
        {todos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
    </ul>
};

const TodoItem = ({todo}) => {
    const dispatch = useContext(TodoContext);
    const handleChange = () =>  dispatch({
      type: todo.complete ? 'UNDO_TODO' : 'DO_TODO',
      id: todo.id,
    });

    return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={todo.complete}
          onChange={handleChange}
        />
        {todo.task}
      </label>
    </li>
  );
};

const AddTodo = ()=>{
    const dispatch = useContext(TodoContext);
    const [task, setTask] = useState('');

    const handleSubmit = event =>{
        if(task) dispatch({ type: 'ADD_TODO', task, id: uuid() });
        setTask('');
        event.preventDefault();
    };

    const handleChange = event => setTask(event.target.value);

    return (
        <form onSubmit={handleSubmit}>
            <input type="text" value={task} onChange={handleChange} />
            <button type="submit">Add Todo</button>
        </form>
    );
};


export default App;

```

In the end, we have a component tree whereas each component receives state as props dispatch functions to alter the state. Most of the state is managed by the parent App component. That's it for the refactoring. Now, the component tree isn't very deep and it isn't difficult to pass props down. However, in larger applications it can be a burden to pass down everything several levels. That's why React came up with the idea of the context container. First, we create the context, second, the app can use the context Provider method to pass implicitly a value down the component tree.

## Bonus

There are several React Hooks that make state management in React Components possible. We covered them above. This bonus material pushes it to the next level by implementing one global state container with useReducer and useContext.

There are 2 caveats with useReducer as to why it cannot be used as one global state container: First every reducer function operates on one independent state. There is not one state container. And second, every dispatch function operates only on one reducer function. There is no global dispatch function which pushes actions through every reducer.

So far, we have an application that uses useReducer and useState to manage state and React's Context API to pass information such as the dispatch function and state down the component tree. State and state update function(dispatch) could be made available in all components by using useContext.

Since we have 2 useReducer functions, both dispatch functions are independent. Now we could either pass both dispatch functions down the component tree with React's Context API or implement one global dispatch function which dispatches actions to all reducer functions. It would be one universal dispatch function which calls all the independent dispatch functions given by our useReducer hooks.

Instead of having a React Context for each dispatch function, let's have one universal context for our new global dispatch function. In our app component, we merged all dispatch functions from our reducers into one dispatch function and pass it down via our new context provider. The global dispatch function iterates through all dispatch functions and executes everyone of them by passing the incoming action object to it. Now the dispatch function from the context can be used everywhere the same; in the TodoItem and AddTodo components, but also in the Filter component.

In the end we only need to adjust our reducers, so that they don't throw and error anymore in case of an incoming action type that isn't matching one of the cases, because it can happen that not all reducers are interested in the incoming action now. Now all reducers receive the incoming actions when actions are dispatched, but not all care about them. However, the dispatch function is one global function, accessible anywhere via React's context, to alter the state in different reducers.

In the last step, we want to hide everything behind one custom React hook called useCombinedReducers:

As before, we want to have access to one global state container(state) and one universal dispatch function(dispatch). That's what our custom hook returns. Parameters our custom hook receives are each returned array from our useReducer calls allocated by object keys. This custom hook looks very similar to Redux's _combineReducers_ function.

Basically, that's it. You have one custom hook which takes in the return values from all your useReducer hooks at a top-level component of your application. The new hook returns the global state object in addition to the global dispatch function. Both can be passed down by React's Context API to be consumed from anywhere in your application.

```
import React, {
  useState,
  useReducer,
  useContext,
  createContext,
} from 'react';
import uuid from 'uuid/v4';

const DispatchContext = createContext(null);

const initialTodos = [
    {
        id: uuid(),
        task: 'Build React UI',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Connect Firebase',
        complete: false,
    },
    {
        id: uuid(),
        task: 'Learn GraphQL',
        complete: true,
    },
];

const filterReducer = (state, action) => {
  switch (action.type) {
    case 'SHOW_ALL':
      return 'ALL';
    case 'SHOW_COMPLETE':
      return 'COMPLETE';
    case 'SHOW_INCOMPLETE':
      return 'INCOMPLETE';
    default:
      return state;
  }
};

const todoReducer = (state, action) => {
  switch (action.type) {
    case 'DO_TODO':
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, complete: true };
        } else {
          return todo;
        }
      });
    case 'UNDO_TODO':
      return state.map(todo => {
        if (todo.id === action.id) {
          return { ...todo, complete: false };
        } else {
          return todo;
        }
      });
    case 'ADD_TODO':
      return state.concat({
        task: action.task,
        id: action.id,
        complete: false,
      });
    default:
      return state;
  }
};

const useCombinedReducer = useReducers => {
  // Global State
  const state = Object.keys(useReducers).reduce(
    (acc, key) => ({ ...acc, [key]: useReducers[key][0] }),
    {}
  );

  // Global Dispatch Function
  const dispatch = action =>
    Object.keys(useReducers)
      .map(key => useReducers[key][1])
      .forEach(fn => fn(action));

  return [state, dispatch];
};

const App = () => {
  const [state, dispatch] = useCombinedReducer({
    filter: useReducer(filterReducer, 'ALL'),
    todos: useReducer(todoReducer, initialTodos),
  });

  const { filter, todos } = state;

  const filteredTodos = todos.filter(todo => {
    if (filter === 'ALL') {
      return true;
    }

    if (filter === 'COMPLETE' && todo.complete) {
      return true;
    }

    if (filter === 'INCOMPLETE' && !todo.complete) {
      return true;
    }

    return false;
  });

  return (
    <DispatchContext.Provider value={dispatch}>
      <Filter />
      <TodoList todos={filteredTodos} />
      <AddTodo />
    </DispatchContext.Provider>
  );
};

const Filter = () => {
  const dispatch = useContext(DispatchContext);

  const handleShowAll = () => {
    dispatch({ type: 'SHOW_ALL' });
  };

  const handleShowComplete = () => {
    dispatch({ type: 'SHOW_COMPLETE' });
  };

  const handleShowIncomplete = () => {
    dispatch({ type: 'SHOW_INCOMPLETE' });
  };

  return (
    <div>
      <button type="button" onClick={handleShowAll}>
        Show All
      </button>
      <button type="button" onClick={handleShowComplete}>
        Show Complete
      </button>
      <button type="button" onClick={handleShowIncomplete}>
        Show Incomplete
      </button>
    </div>
  );
};

const TodoList = ({ todos }) => (
  <ul>
    {todos.map(todo => (
      <TodoItem key={todo.id} todo={todo} />
    ))}
  </ul>
);

const TodoItem = ({ todo }) => {
  const dispatch = useContext(DispatchContext);

  const handleChange = () =>
    dispatch({
      type: todo.complete ? 'UNDO_TODO' : 'DO_TODO',
      id: todo.id,
    });

  return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={todo.complete}
          onChange={handleChange}
        />
        {todo.task}
      </label>
    </li>
  );
};

const AddTodo = () => {
  const dispatch = useContext(DispatchContext);

  const [task, setTask] = useState('');

  const handleSubmit = event => {
    if (task) {
      dispatch({ type: 'ADD_TODO', task, id: uuid() });
    }

    setTask('');

    event.preventDefault();
  };

  const handleChange = event => setTask(event.target.value);

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={task} onChange={handleChange} />
      <button type="submit">Add Todo</button>
    </form>
  );
};

export default App;
```
