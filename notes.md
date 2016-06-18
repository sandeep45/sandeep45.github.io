### Push vs Concat

````
var a = [1,2,3];
console.log(a.push(4)); // 4

var a = [1,2,3];
console.log(a.concat(4)); // [1,2,3,4]
````

### Spread

#### In an Array

````
x = [1,2]
[1, 2]
a = [...x, 3]
[1, 2, 3]
````

#### In a function

````
var a = function(...args) { console.log(args) };
a(1,2,3); // [1,2,3]
a(1,2); // [1,2]
````

### Deconstructing

````
var a = function({x,z}){
 console.log(x, z);
}

a({
  w: "a",
  x:"b",
  y:"c",
  z:"d"
})
````

### Typing with React

putting value on a field in react makes it readonly and wont change from user input

### Naming Callbacks

Name all callbacks with the `on` prefix. This enabled distinguising them from the `props`

### `Bind`

#### Parameters are prepended

`Bind` function has been used for ages to setup the value of `this` and `paramteres` of a function. It might Interesting to some that it doesn't set the parameters. it instead sets up the parameters to be prepended to the actualy paramters passed when the function is called.

````
var a = function(...args) { console.log (args); }
a = a.bind(null, 1);
a(); // [1]
a(2,3); // [1,2,3]
````

#### autobind `this` in methods

A common thing we see in React is that a child component needs to call the parent components callback function. In the callback function this is being referred. The value of `this` needs to refer to the parent component, but will not be so when called as callback from the props.

To ensure that `this` in methods of a class refer to the class when called from a callback we can:

1. Use decorator `@autobind`.
Reference: http://technologyadvice.github.io/es7-decorators-babel6/

````
@autobind
class App extends React.Component {
  constructor(...args){
    super(...args);
  }

  editNote(){
    console.log(this); // App
  }
}
````

2. Bind the `this` to the class object in the constructor

````
class App extends React.Component {
  constructor(...args){
    super(...args);
    this.editNote = this.editNote.bind(this);
  }

  editNote(){
    console.log(this); // App
  }
}
````

3. Use Fat-Arrow functions, where `this` is resolved lexically.

To do this we will need to be able to setup class properties. This a stage-1 feature and can be enabled with babel.

Reference:
- http://www.ian-thomas.net/autobinding-react-and-es6-classes/
- https://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#autobinding
- http://babeljs.io/docs/plugins/transform-class-properties/
- http://babeljs.io/docs/plugins/preset-stage-1/

````
class App extends React.Component {
  constructor(...args){
    super(...args);
  }

  editNote = () => {
    console.log(this); // App
  }
}
````

4. `Object.assign`

Copies owned and enumerable properties over.

````
var obj = Object.assign({}, {a: 1}, {b:2});
console.log(obj); // {a:1, b:2}
````

5. GUI for cookies & `localstorage`

It turns out that the resources tab in chrome inspector shows localstorage of all domains loaded.

6. localstorage access variables directly.

````
localStorage.x = "y"
console.log(localStorage.x); // "y"
console.log(localStorage.getItem("x")); // "y"
````

7. React starter template

https://github.com/webpack/react-starter

8. Webpack Cookbook

https://christianalfoni.github.io/react-webpack-cookbook/index.html

9. Watch me code

https://sub.watchmecode.net/series/

10. Webpack React Book

http://survivejs.com/webpack_react/react_and_flux/

11. Flux

It means: "one way" data flow.

Action --> Dispatcher --> Store --> View

In the view, we have user interactions, which trigger actions and this circle complets

Action --> Dispatcher --> Store --> View --> Action

### Action: Are simple objects containing the new data and identifying `type` property

`Action Creators` are Helper methods, from library.
1. they create an action from method parameters,
2. assign a type to the action and
3. provide it to the dispatcher.

It is the payload of data we pass to the store via the dispatcher.

They can come from the server like on initialization or from the view when the user interacts with it.

The creator will will add a type like _TODO-UPDATE-TEXT_

ActionCreater is a simpler helper file which encapsulates the work to create the payload of a certain action and delivers it to the dispatcher. This makes the code clean as the view can just say things like: `TodoActionCreator.create("abc")` and now the creator know what payload to build and it passes to the dispatcher. Writing `create` is much cleaner and intutive than writing code which builds payload etc.


### Dispatcher: Traffic Control / Central Hub / Singleton (Facebook Dispatcher)

until the store layer is completely done, the view or anything else can not process anything through it to the store. Synchronous updates are managed by the dispatcher

It sends every action to all stores via the callbacks the stores register with the dispatcher.

When change of one object can change another object and so-on, it then becomes very difficult to predict what would change as result of a user interaction. In Flux, an action can only change data within a single round, making the whole system more predictable.

It is a registry of callbacks into the stores. It is a mechanism for distributing actions in to stores. Each store registers itself and provides a callback. When an action creator provides the dispatcher with a new action, ALL STORES receive the action via the callback in the registry.

It can be used to manage dependencies within stores by invoking callbacks of stores in a specific order. Stores can also declaratively wait for other stores to finish updating, and then update themselves.

Dispatcher used by Facebook: https://www.npmjs.com/package/flux

It is able to manage dependencies between stores. It has a `waitFor` method.

In the store we could use `Dispatcher.waitFor` and pass other  stores' dispatch token. this makes it wait till the other stores state has updated.

As application grows, a time will come when storeA will say that i need storeB to update first. To solve this we have `waitFor`. The dispatch method, call each registered calback synchronously. When the registered callback method of the store calls `waitFor`, its exection stops, callbacks of its dependencies are run and then the original callback continues to execute.

### Store: Data Layer (EventEmitter)

They update themselves in response to an action which they get from the dispatcher.

They then emit a change event for the view.

Stores contain both application state and logic. Like the M in MVC but they manage state of many objects. Stores manage application state for a particular domain within the application.

Store's registered callback with the dispatcher is called. The callback has the `action` as a parameter. In the callback there is a big switch statement based on the action's type it determines if and how which internal store methods need to be called. These update the store and thent the store broadcasts that its state has changed and now the views may query the new state and update themselves.

### Views and Controller-Views: (React)

They listen for change events on the store, retrieve the new data from the stores and update state which in turn updates the state.

At the top of view hierarchy we have a controller view. this is a special kind of view, which listersn for events that are broadcassted by the storeS it depends on. Usually we can have one of these controller views governing any significatn section of the page.

When it gets an event from the store, it requests the data via the stores getter method, sets its own state, causing its `render` and `render` of all of its decendents to run.

Often it passes the ENTIRE STATE of the store down the chain of views in a single object. this keeps things simple as it reduces the number of props, and each view can take what they need from the object and be as pure as possible.

Private methods of a view, follow convention to start with underscore: _. These are usually used as callbacks to event handlers.


--- --- --- --- --- --- --- --- --- --- --- --- --- ---


So if something changes in the view, lets say added a note, then that doesnt mean that now the noteStore has one more item like it would have been in a MVC architecture. In Flux, there are now backward arrows. The view on adding a new note, will trigger an action, which will update the stores listening for it and then their corresponding view can update.

Describing UI at any point of time. For any fixed set of data, the UI will always be the same.

--- --- --- --- --- --- --- --- --- --- --- --- --- ---

Boiler plate of Flux App

First we have `/js` directory with:
1. actions
2. components
3. constants
4. dispatcher
5. stores

Then inside the js directory we have:
1. app.js
2. bundle.js

We also have `index.html` which uses bundle.js and `package.json` which has scripts/process to build the bundle file.

###AppDispatcher.js

This file requires up the dispatcher given by facebook's `flux` library. It then builds a single dispatcher for the app and exports it

````
var Dispatcher = require('flux').Dispatcher
var AppDispatcher = new Dispatcher();
module.exports = AppDispatcher;
````


###TodoStore.js

This file has the store. It:
1. gets callbacks from the dispatcher,
2. uses the constants and processes the actions
3. emits a change
4. lets views query for update data
5. maintains internal state/object of data
6. build on top of eventEmitter

Is an object which has owned and enurmerable properties of EventEmitter.prototype. It also has these boiler plate methods added:
1. `getXXX` - this method can be called by the view to get the data
2. `emitChange` - this is called by methods in the store, so the listening views know that somehting has changed.
3. `addChangeListener` - which takes a callback to be called when something has changed
4. `removeChangeListener` - which takes a callback to not be called when something has changed

It also has boiler plate code to register itself with the dispatcher and process all events it cares about

````
  AppDispatcher.register(function(action){

  });
````

In the callback it has a massive `switch` statement where it process events it cares for. In the processing it takes the action data, calls internal method, which updates the interal state/object it keeps and then calls emitChange.

````
 case TodoConstants.TODO_CREATE:
      text = action.text.trim();
      if (text !== '') {
        create(text);
        TodoStore.emitChange();
      }
      break;
````

###TodoActions.js

This is the helper file for creating actions with the correct payload. The payload has the action type and the data. It also calls the dispatcher with the payload.

````
var TodoActions = {

  /**
   * @param  {string} text
   */
  create: function(text) {
    AppDispatcher.dispatch({
      actionType: TodoConstants.TODO_CREATE,
      text: text
    });
  }
````

###TodoConstants.js

It creates action types so they can be easily passed arround. Uses `keymirror` for ease.

````
var constants = keyMirror({
  TODO_CREATE: null,
  TODO_COMPLETE: null
});
````

###TodoApp.react.js

This is the controller view, which subscirbes to get changes from the store. It sets all data in its state and passes the whole thing down to the child views.

It also triggers off an action on user interactions.

Has a method to get data from store.

````
function getTodoState() {
  return {
    allTodos: TodoStore.getAll(),
    areAllComplete: TodoStore.areAllComplete()
  };
}
````

It gets store data in `getInitialState`.
On mount it subscribes to get changes.
On unmount it unsubscribe to get changes
On change it updates it state

````
  getInitialState: function() {
    return getTodoState();
  },

  componentDidMount: function() {
    TodoStore.addChangeListener(this._onChange);
  },

  componentWillUnmount: function() {
    TodoStore.removeChangeListener(this._onChange);
  }
````

````
_onChange: function() {
    this.setState(getTodoState());
  }
````

### Style
Looks like its using the two brace syntax, but really react jsx needs only one brace and the other is because style accepts an objec



--- --- --- --- --- --- --- --- --- --- --- --- --- ---

https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a#.rk3xjbj9h
