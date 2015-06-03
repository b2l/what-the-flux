# What the Flux?

## What is Flux?
A Javascript application pattern.  
Promoted by Facebook along with React

## Why Flux?

* Easy to reason about
* Predictable
* Scalable
* Easy to test

## Flux:

### Dispatcher

It's the central hub of the application

It just dispatch the actions to the stores

Ensure that only one action is processed at a time

It manage the dependencies between stores

### Stores

Represent a domain of your application

Manage the state of this domain

Broadcast an event when the state has changed

```
todos = []


TodosStore = {
    getTodos: function() {
        return todos;
    }
}

Dispatcher.register(function(action) {
    switch(action.type) {
        case 'ADD_TODO':
            todos.push(action.data.text);
            TodosStore.emit();
            break;
    }
});


todos = []


TodosStore = {
    getTodos: function() {
        return todos;
    }
}

Dispatcher.register(function(action) {
    switch(action.type) {
        case 'ADD_TODO':
            todos.push(action.data.text);
            TodosStore.emit();
            break;
    }
});
```

### Views & Controller-Views

Controller-views listen to store for changes

Query the store for its state/data

Re-render the view tree with the new data


### Actions

Message with payload of data to send to the dispatcher

Wrapper into a helper to call from the view, like `addTodo(todoText)`

Actions are usually called from view in response to event

May also come from other places, such as the server

```
var Actions = {

    addTodo: function(text) {
        Dispatcher.dispatch({
            type: 'ADD_TODO',
            data: {
                text: text
            }
        });
    }

}
```

## Pitfalls & Patterns

### Backend communication (Ajax)

#### What we usually do:
```
TodosStore = {
    getTodos: function() {
        return todos;
    }
}

Dispatcher.register(function(action) {
    switch(action.type) {
        case 'ADD_TODO':
            saveTodo(action.data.text)
                .then(function(err, response) {
                    todos.push(action.data.text);
                    TodosStore.emit();
                });
            break;
    }
});
```

But now there is a mutation of our data without an action

Hard to debug

Harder to reason about the app

#### Fire an action
```
Dispatcher.register(function(action) {
    switch(action.type) {
        case 'ADD_TODO':
            saveTodo(action.data.text)
                .then(function(err, response) {
                    if(err)
                        Actions.addTodoFailed(err)
                    else
                        Actions.addTodoCompleted(response)
                });
            break;
        case 'ADD_TODO_COMPLETED':
            todos.push(action.data.text);
            TodosStore.emit();
            break;
    }
});
```

Always fire an action at the end of an async request

#### Keep your store sync

```
saveTodo(action.data.text)
  .then(function(err, response) {
    if(err)
      Actions.addTodoFailed(err)
    else
      Actions.addTodoCompleted(response)
  });
```

It's a lot easier to read and to reason about your store if they are synchronous.

Stores don't have to be coupled with the network layer


### Store dependencies

#### What we may do:

```
var FiltersStore = require('./filters_store');

FiltersStore.on('CHANGE', function(filter) {
    todos = todos.filter(function(todo) {
        return todo.prop == filter;
    });
});
```

Temptation to listen to another store

And mutate the state according to the dependency

#### What we should do

```
var FiltersStore = require('./filters_store');

var todos = [];
TodosStore = {
}

TodosStore.dispatchToken = Dispatcher.register(function(action) {

    switch(action.type) {
        case "FILTER":
            Dispatcher.waitFor(FiltersStore.dispatchToken);
            /* Do your buisness */
            break;
    }

});
```

The actions are not advanced setter.

Multiple store can listen to the same action

Handle the dependencies / dispatch order with waitFor()

### Actions

#### Actions are neither command nor elaborate setters

Those are commands:
```
Actions.SHOW_NOTIFICATION
Actions.FETCH_FROM_SERVER
```

Actions should be like newspaper:
```
Actions.TODO_CREATED_SUCCESS
Actions.TODO_SYNC_REQUESTED
```

Actions represent the changes in your app

If your action are named and represent a change, you can easily debug your app when there is a problem.

Or work on a new part and make change with confidence.

```
TODO_CREATED "something"
TODO_REMOVED 1
TODO_UPDATED 0, "something else"
TODO_SYNC_REQUESTED
TODO_SYNC_COMPLETED
TODO_CREATED ""
TODO_CREATION_ERROR "empty text"
```

Inspired by  @JeremyMorell talk: Those who forget the past...  are doomed to debug it
