# recloud

[![npm version](https://badge.fury.io/js/recloud.svg)](https://badge.fury.io/js/recloud)

Mixin that listens to prop/state changes and only reloads data from the server when necessary. Branch from the Parse-specific [React-Cloud-Code-Mixin](https://github.com/rubencodes/react-cloud-code-mixin) to be able to load data from any endpoint. Inspired by [ParseReact's Mixin](https://github.com/ParsePlatform/ParseReact/blob/master/docs/api/Mixin.md) and [Reselect](https://github.com/reactjs/reselect).


## How to Use

The first step is to include the mixin in your component like so:

```javascript
var MyComponent = React.createClass({
  mixins: [ Recloud ],
  /*...*/
});
```

This mixin requires you to define a `loadData` function (analagous to ParseReact's observe function). It should return an object where the keys are the data keys where the results will be stored, and the value is an object with instructions on how and when to reload data (details below). For example:

```javascript
var MyComponent = React.createClass({
  mixins: [ Recloud ],
  loadData: function(props, state) {
    return {
      MyData: {
        load: () => {
          return MyAPI.loadData({
            foo: "foo",
            bar: "bar"
          });
        }
      }
    };
  },
  /*...*/
  render: function() {
    //example render function
    return (
      <h1>{ this.data.MyData }</h1>
    );
  }
});
```
Whenever the component mounts, it will issue a call to my `loadData` API and the results will be stored in `this.data.MyData`. Any time props or state change, the function will be called again (unless otherwise configured—details below).

The objects associated with the data keys can take the following parameters:

- `load`: a function that returns a promise that will load data from your server.
- `propDeps`: an array with the names of each `prop` whose value change should cause a reload (e.g. the component will only pull data and re-render if one of these changed).*
- `stateDeps`: an array with the names of each `state` whose value change should cause a reload (e.g. the component will only pull data and re-render if one of these changed).*

Example:
```javascript
var MyComponent = React.createClass({
  mixins: [ Recloud ],
  loadData: function(props, state) {
    return {
      Users: {
        load: () => {
          return MyAPI.getUsers({
            age: props.userAge
          });
        },
        propDeps: ['userAge']
      }
    };
  },
  /*...*/
  render: function() {
    //example render function
    return (
      <h1>{ this.data.Users }</h1>
    );
  }
});
```

In the example, the `getUsers` API is called with the parameter `age` set to `props.userAge`, and resulting data is stored in `this.data.Users`. Our `propDeps` specify that any time that `this.props.userAge` changes, the data is pulled again from the server and the component is reloaded. If this prop has not changed, nothing is reloaded.

The `propDeps` and `stateDeps` declarations are the biggest departure from the ParseReact mixin; ParseReact reloads in response to any prop or state change. To get this behavior, simply omit the `propDeps` and `statedDeps` keys. This will force a reload with any prop or state change. If you don't want to ever reload data in response to props or state, this can also be specified by defining `propDeps` and `stateDeps` to be empty arrays.

*Array and Object values not yet supported for `propDeps` and `stateDeps` 

## Additional Component Methods
###`this.pendingQueries()` 
Returns an array containing the names of calls that are currently waiting for results.
###`this.queryErrors()` 
Returns a map of call names to the error they encountered on the last request, if there was one.
###`this.reloadData([callList])` 
Forces a refresh of Cloud Code calls with names in the `callList`, if provided, else refreshes data from all the calls. Note: the names in the `callList` should be the name of the data key in which the data is stored (e.g. the key part of the key-value pairs returned in `loadData`.

## Install

```
npm install recloud --save
```

Then, in your React project:

```javascript
var Recloud = require('recloud');
```

Or, if you're using ES6/ECMA2015 syntax:
```javascript
import Recloud from 'recloud'
```

## Depencies
None! Well, except Parse. But that's a given.

## F.A.Q.
**Does this replace ParseReact Mixin?**

It can, but it doesn't have to. This project started when my company switched from doing Parse queries client-side to making API calls in order to reduce client-side logic and network usage. We use this in our app now instead of ParseReact, but the two can operate side by side (as long as you don't name two data params the same).

**Does this replace React-Cloud-Code-Mixin?**

It can, but it doesn't have to. The two projects are effectively the same, where Recloud's load function returns a call to Parse.Cloud.run.

**Is this production ready?**

Like with most open-source software, I can't make any guarantees, but I can tell you it's stable enough that the company I work at is using actively it in production.

**How can I contribute?**

Feel free to fork this project and create a pull request! I'll merge in anything useful. Thanks!

## License

MIT
