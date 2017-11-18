# React.js best practices and conventions
=====

## Table of Contents

1. [Scope](#scope)
1. Organization
  1. [Component Organization](#component-organization)
  1. [Formatting Props](#formatting-props)
1. Patterns
  1. [Computed Props](#computed-props)
  1. [Compound State](#compound-state)
  1. [Sub-render](#sub-render)
  1. [View Components](#view-components)
  1. [Container Components](#container-components)
1. Anti-patterns
  1. [Cached State in render](#cached-state-in-render) 
  1. [Compound Conditions](#compound-conditions)
  1. [Existence Checking](#existence-checking)
  1. [Setting State from Props](#setting-state-from-props)
1. Practices
  1. [Naming Handle Methods](#naming-handler-methods)
  1. [Naming Events](#naming-events)
  1. [Using PropTypes](#using-proptypes)
  1. [Using Entities](#using-entities)
1. Gotchas
  1. [Tables](#tables)
1. Other
  1. [ClassName](#classname)
  1. [JSX](#jsx)
  1. [ES6 Harmony](#es6-harmony)
  1. [flux](#flux)
  1. [Do more in render](#do-more-in-render)
  1. [Mixins are awesome](#mixins-are-awesome)

---

## Scope

This is how we write [React.js](https://facebook.github.io/react/).

**[Back to top](#table-of-contents)**

---

## Component Organization

Always set propTypes propTypes at the top of the component for validation and a useful guide to a components expected usage [see section below for more on this](#using-proptypes), followed by the mixins because it is handy to initially know exactly what external behaviours the component is using/dependent on.

Split the life-cycle methods into those that occur before an instance of the component is created (e.g. `getInitialState`, `getDefaultProps`) and those which occur during the mounting/updating/mounted cycle (e.g. `componentWillMount`, `shouldComponentUpdate`). Declaring the lifecycle methods in order of execution also makes the component easier to reason about.

Have any custom methods follow the lifecycle methods and be prefixed with an underscore to make them easier to identify. Generally group them by utility (parsers, event handlers, etc).

Render method should always be last. The render method is a mandatory lifecycle method and it’s almost always the function you need to find first when opening a file. Consequently, it’s pragmatic to have it in a consistent location across all of my components.

In general, my mixins will follow the same conventions as regular components.

* displayName
* propTypes
* mixins
* static methods
* get methods
* state methods
* lifecycle events
* event handlers
* "private" methods
* render

```javascript
React.createClass({
        displayName: '',

        propTypes: {},
        mixins : [],

        statics: {},

        getInitialState: function() {},
        getDefaultProps: function() {},

        componentWillMount : function() {},
        componentDidMount: function() {},
        shouldComponentUpdate: function() {},
        componentWillUpdate: function() {},
        componentWillReceiveProps: function() {},
        componentDidUpdate: function() {},
        componentWillUnmount : function() {},

        _parseData : function() {},
        _onSelect : function() {},

        render : function() {}
    })
```

Example:
```javascript
module.exports = React.createClass({
  displayName: 'UserAccount',
  
  propTypes: {
    name: React.PropTypes.string
  },

  getInitialState: function() {
    return {
      loggedIn: false
    };
  },
  getDefaultProps: function() {
    return {
      name: 'Guest'
    };
  },

  componentWillMount: function() {
    // add event listeners (Flux Store, WebSocket, document, etc.)
  },
  componentDidMount: function() {
    // React.getDOMNode()
  },
  componentWillUnmount: function() {
    // remove event listeners (Flux Store, WebSocket, document, etc.)
  },

  handleClick: function() {
    this.setState({loggedIn: !this.state.loggedIn});
  },

  _doSomethingGross: function() {
    // These really aren't private but it's a sign the method could stand
    // improvement or has unideal implementation.
  },

  render: function() {
    return (
      <div
       className="Person"
       onClick={this.handleClick}>
        {this.props.name} {this.state.loggedIn ? "is logged in" : ""}
      </div>
    );
  },

});
```

Place `get` methods ([computed props](#computed-props)) after React's `getInitialState` and `getDefaultProps`.

Place `has`/`is`/`can` methods ([compound state](#compound-state)) after that, respectively.

**[Back to top](#table-of-contents)**

## Formatting Props

Indentation and new line for component attributes.

When there are enough attributes on a component that displaying them inline becomes untidy usually more than one, I always display them on multiple lines and indent each one. i.e.

(*Hint*: Don't separate props with commas)

```javascript
<UserAccount
    firstName={...}
    lastName={...}
    occupation={...} />
```
Rather than…
```javascript
<UserAccount firstName={...} lastName={...} occupation={...} />
```

**[Back to top](#table-of-contents)**

## Computed Props

Name computed prop methods with the `get` prefix.

```javascript
  // bad
  firstAndLastName() {
    return `{this.props.firstName} {this.props.lastname}`;
  }

  // good
  getFullName() {
    return `{this.props.firstName} {this.props.lastname}`;
  }
```

See: [Cached State in render](#cached-state-in-render) anti-pattern

**[Back to top](#table-of-contents)**

## Compound State

Name compound state methods with the `is`, `has` or `can` prefix.

```javascript
// bad
happyAndKnowsIt() {
  return this.state.happy && this.state.knowsIt;
}
```

```javascript
// good
isWillingSongParticipant() {
  return this.state.happy && this.state.knowsIt;
},

hasWorrysomeBehavior() {
  return !this.isWillingSongParticipant() && this.props.punchesKittens;
},

canPetKittens() {
  return this.hasHands() && this.props.kittens.length;
}
```

These methods should return a `boolean` value.

~~See: [Compound Conditions](#compound-conditions) anti-pattern~~

**[Back to top](#table-of-contents)**

## Sub-render

Use sub-`render` methods to isolate logical chunks of component UI.

```javascript
// bad
render() {
  return <div>{this.props.name} {this.state.loggedIn ? "is logged in" : ""}</div>;
}
```

```javascript
// good
renderLoggedInStatement() {
  if (this.state.loggedIn) {
    return "is logged in";
  }

  return "";
},

render() {
  return <div>{this.props.name} {this.renderSmilingStatement()}</div>;
}
```

**[Back to top](#table-of-contents)**

## View Components

Compose components into views. Don't create one-off components that merge layout and domain components.

```javascript
// bad
var PeopleWrappedInBSRow = React.createClass({
  render() {
    return (
      <div className="row">
        <People people={this.state.people} />
      </div>
    );
  }
});
```

```javascript
// good
var BSRow = React.createClass({
  render() {
    return <div className="row">{this.props.children}</div>;
  }
});

var SomeView = React.createClass({
  render() {
    return (
      <BSRow>
        <People people={this.state.people} />
      </BSRow>
    );
  }
});
```

This works nicely for complex components—like Tabs or Tables—where you you might need to iterate over children and place them within a complex layout.

**[Back to top](#table-of-contents)**

## Container Components

> A container does data fetching and then renders its corresponding
> sub-component. That's it. &mdash; Jason Bonta

#### Bad

```javascript
// CommentList.js

var CommentList = React.createClass({
  getInitialState() {
    return { comments: [] }
  },

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  },

  render() {
    return <ul> {this.state.comments.map(renderComment)} </ul>;
  },

  renderComment({body, author}) {
    return <li>{body}—{author}</li>;
  }
});
```

#### Good

```javascript
// CommentList.js

var CommentList = React.createClass({
  render() {
    return <ul> {this.props.comments.map(renderComment)} </ul>;
  },

  renderComment({body, author}) {
    return <li>{body}—{author}</li>;
  }
});
```

```javascript
// CommentListContainer.js

var CommentListContainer = React.createClass({
  getInitialState() {
    return { comments: [] }
  },

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  },

  render() {
    return <CommentList comments={this.state.comments} />;
  }
});
```

[Read more](https://medium.com/@learnreact/container-components-c0e67432e005)
[Watch more](https://www.youtube.com/watch?v=KYzlpRvWZ6c&t=1351)

**[Back to top](#table-of-contents)**

## Cached State in `render`

Do not keep state in `render`

```javascript
// bad
render() {
  var name = 'Mrs. ' + this.props.name;
  return <div>{name}</div>;
}

// good
render() {
  return <div>{'Mrs. ' + this.props.name}</div>;
}
```

```javascript
// best
getFancyName() {
  return `Mrs. {this.props.name}`;
},

render() {
  return <div>{this.getFancyName()}</div>;
}
```

See: [Computed Props](#computed-props) pattern

**[Back to top](#table-of-contents)**

## Compound Conditions

Do not put compound conditions in `render`.

```javascript
// bad
render() {
  return <div>{if (this.state.happy && this.state.knowsIt) { return "Clapping hands" }</div>;
}
```

```javascript
// better
isTotallyHappy() {
  return this.state.happy && this.state.knowsIt;
},

render() {
  return <div>{if (this.isTotallyHappy() { return "Clapping hands" }}</div>;
}
```

```javascript
// betterer
getHappinessMessage() {
  if (this.isTotallyHappy()) {
    return "Clapping hands";
  }
},

isTotallyHappy() {
  return this.state.happy && this.state.knowsIt;
},

render() {
  return <div>{this.getHappinessMessage()}</div>;
}
```

The best solution for this would use a [container component](#container-components) to manage state and pass new state down as props.

See: [Compound State](#compound-state) pattern

**[Back to top](#table-of-contents)**

## Existence Checking

Do not check existence of `prop` objects.

```javascript
// bad
render() {
  if (this.props.person) {
    return <div>{this.props.person.firstName}</div>;
  } else {
    return null;
  }
}
```

```javascript
// good
getDefaultProps() {
  return {
    person: {
      firstName: 'Guest'
    }
  };
},

render() {
  return <div>{this.props.person.firstName}</div>;
}
```

This is only where objects or arrays are used. Use PropTypes.shape to clarify the types of nested data expected by the component.

**[Back to top](#table-of-contents)**

## Setting State from Props

Do not set state from props without obvious intent.

```javascript
// bad
propTypes: {
  items: React.PropTypes.array
},

getInitialState() {
  return {
    items: this.props.items
  };
}
```

```javascript
// good
propTypes: {
  initialItems: React.PropTypes.array
},

getInitialState() {
  return {
    items: this.props.initialItems
  };
}
```

Read: ["Props in getInitialState Is an Anti-Pattern"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[Back to top](#table-of-contents)**

## Naming Handler Methods

Name the handler methods after their triggering event.

```javascript
// bad
punchABadger() {},

render() {
  return <div onClick={this.punchABadger}> ... </div>;
}
```

```javascript
// good
handleClick() {},

render() {
  return <div onClick={this.handleClick}> ... </div>;
}
```

Handler names should:

- begin with `handle`
- end with the name of the event they handle (eg, `Click`, `Change`)
- be present-tense

If you need to disambiguate handlers, add additional information between `handle` and the event name. For example, you can distinguish between `onChange` handlers:  `handleNameChange` and `handleAgeChange`. If you do this, ask yourself if you should create another component class.

**[Back to top](#table-of-contents)**

## Naming Events

Use custom event names for components Parent-Child event listeners.

```javascript
var Parent = React.createClass({
  handleCry() {
    // handle Child's cry
  },

  render() {
    return (
      <div className="Parent">
        <Child onCry={this.handleCry} />
      </div>
    );
  }
});

var Child = React.createClass({
  render() {
    return (
      <div
       className="Child"
       onChange={this.props.onCry}>
        ...
      </div>
    );
  }
});
```

**[Back to top](#table-of-contents)**

## Using PropTypes

Use PropTypes to communicate expectations and log meaningful warnings.

```javascript
propTypes: {
        arrayProp: React.PropTypes.array,
        boolProp: React.PropTypes.bool,
        funcProp: React.PropTypes.func,
        numProp: React.PropTypes.number,
        objProp: React.PropTypes.object,
        stringProp: React.PropTypes.string,
    }
```

Use PropTypes to communicate expectations and log meaningful warnings.

```javascript
var MyValidatedComponent = React.createClass({
  propTypes: {
    name: React.PropTypes.string
  },

  ...
});
```
This component will log a warning if it receives `name` of a type other than `string`.


```html
<Person name=1337 />
// Warning: Invalid prop `name` of type `number` supplied to `MyValidatedComponent`, expected `string`.
```

Components may require `props`

```javascript
var MyValidatedComponent = React.createClass({
  propTypes: {
    name: React.PropTypes.string.isRequired
  },

  ...
});
```

This component will now validate the presence of name.

```html
<Person />
// Warning: Required prop `name` was not specified in `Person`
```

Read: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[Back to top](#table-of-contents)**

## Using Entities

Use React's `String.fromCharCode()` for special characters.

```javascript
// bad
<div>PiCO · Mascot</div>

// nope
<div>PiCO &middot; Mascot</div>

// good
<div>{'PiCO ' + String.fromCharCode(183) + ' Mascot'}</div>

// better
<div>{`PiCO {String.fromCharCode(183)} Mascot`}</div>
```

Read: [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html#html-entities)

**[Back to top](#table-of-contents)**

## Tables

The browser thinks you're dumb. But React doesn't. Always use `tbody` in your `table` components.

```javascript
// bad
render() {
  return (
    <table>
      <tr>...</tr>
    </table>
  );
}

// good
render() {
  return (
    <table>
      <tbody>
        <tr>...</tr>
      </tbody>
    </table>
  );
}
```

The browser is going to insert `tbody` if you forget. React will continue to insert new `tr`s into the `table` and confuse the heck out of you. Always use `tbody`.

**[Back to top](#table-of-contents)**

## className

Use `classNames()` to manage conditional classes in your app:

```javascript
// bad
render() {
  var classes = ['MyComponent'];
  if (this.state.active) {
    classes.push('MyComponent--active');
  }

  return <div className={classes.join(' ')} />;
},

getClassName() {
  var classes = ['MyComponent'];
  if (this.state.active) {
    classes.push('MyComponent--active');
  }
  return classes.join(' ');
}

// good
render() {
  var classes = {
    'MyComponent': true,
    'MyComponent--active': this.state.active
  };

  <div className={classNames(classes)} />;
}
```

Read: [Class Name Manipulation](http://facebook.github.io/react/docs/class-name-manipulation.html)

**[Back to top](#table-of-contents)**

## JSX

Multi-line JSX
No matter how few elements are being returned, write any JSX which contains nested elements across multiple lines with indentation to enhance readability, i.e:

```javascript
return (
        <div>
            <ComponentOne />
            <ComponentTwo />
        </div>
    );
```

Rather than…

```javascript
return (<div><ComponentOne /><ComponentTwo /></div>);
```

Furthermore, while the parenthesis are not technically required with a single line JSX statement, still use them for the sake of consistency (and because unrestrained elements floating around in JS will makes your left eye twitch uncontrollably).

Conditional JSX
Conditional elements that needs to be returned depending on state, props, or another condition, you should declare an empty variable at the top of the render function and only populate it with JSX if the condition is met. When the variable is returned in the render method return statement, it’ll either render the conditional elements, or nothing at all.

```javascript
var optionalElement;

    if (this.props.condition) {
        optionalElement = (<div> … </div>);
    }

    return (
        <div>
            …
            {optionalElement}
            …
        </div>
    );
```

In-line list iteration
Where possible, iterate over lists of data in-line in the returned JSX unless its internal logic is sufficiently complex enough to warrant moving outside of the return statement and populating an array for rendering.

```javascript
return (
        <div>
            {this.props.list.map(function(data, i) {
                return (<Component data={data} key={i} />)
            })}
        </div>
    );
```

**[Back to top](#table-of-contents)**

## ES6 Harmony

**[Back to top](#table-of-contents)**

## flux

Flux is great for storing application state that either doesn’t logically belong to any one specific component or state that should persist after an unmount. A common bit of advice is to never use `this.state` and to put everything in Flux stores. However, this isn’t quite right. You should feel free to use `this.state` for component specific state that isn’t relevant after something unmounts. An example of such state is the `isCurrentlyOpen` state of `DropdownToggle`.

Flux is also quite verbose, which makes it inconvenient for data state, state that is persisted to the server. We currently use a global Backbone model for data fetching and saving but we’re also use REST APIs.

For all other state, we could gradually introduce Flux into our code base. It’s great because it doesn’t require a rewrite; you can use it where you want. It’s easy to unit test, easy to scale, and can provided some cool benefits like resolving circular dependencies in our core modules. It has also removed the need for hacky singleton components.

**[Back to top](#table-of-contents)**

## Do more in render

If you find yourself putting lots of logic into `componentWillReceiveProps` or `componentWillMount`, instead, see if you can move that logic to render(). If you find yourself wanting to do pre-processing on props, or write a method named `setComputedState()`, simply do that processing in `render()`. This will also prevent you from duplicating code in `componentWillMount` and `componentWillReceiveProps`.

```javascript
// bad
componentWillMount: function () {
    this.setState({
        computedFoo: this.props.foo
    });
},
componentWillReceiveProps: function (nextProps) {
    this.setState({
        computedFoo: nextProps.foo
    });
},
render: function () {
    return React.DOM.div({className: this.state.computedFoo});
}


// better
render: function () {
    var computedFoo = compute(this.props.foo);
    return React.DOM.div({className: computedFoo});
}
```

This strategy can also help you minimize the number of fields on props an state, and even help you avoid state entirely. Having fewer fields leads to simpler components.

Also, don't be afraid to have your `render()` method call other methods that return components. Do more in the render phase, but avoid having long functions by having the actual `render()` method delegate out.

**[Back to top](#table-of-contents)**

## Mixins are awesome

Mixins are handy ways to create re-usable chunks of functionality that multiple components will use. One cool feature of mixins is that they do not override component lifecycle methods, but rather, they are called in addition. This means a mixin can do setup in its own `componentWillMount()` method and clean up in `componentWillUnmount()` without conflicting with the host component's `componentWillMount`. In general, mixins are useful for isolating related chunks of functionality.

**[Back to top](#table-of-contents)**
