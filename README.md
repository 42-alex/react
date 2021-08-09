## How to make work this in callbacks?
You have to be careful about the meaning of this in JSX callbacks. 
In JavaScript, class methods are not bound by default. If you forget
to bind this.handleClick and pass it to onClick, this will be 
undefined when the function is actually called.

This is not React-specific behavior; it is a part of how functions 
work in JavaScript. Generally, if you refer to a method without () 
after it, such as onClick={this.handleClick}, you should bind that 
method. There are three options to resolve it.

### Option 1. Bind "this" in constructor

```
class Toggle extends React.Component {
  
  constructor(props) {
    super(props);

    this.handleClick = this.handleClick.bind(this);    // this binding
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

---

### Option 2. Use experimental syntax to define class methods

```
class Toggle extends React.Component {

  handleClick = () => {		// 	this syntax also ensures "this" is bound within handleClick
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

---

### Option 3. Use an arrow function in the callback

```
class Toggle extends React.Component {

  handleClick = () => {		
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={() => this.handleClick()}> 	// 	arrow function in the callback
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```

The problem with this syntax is that a different callback is created each time 
the Toggle renders. In most cases, this is fine. However, if this callback is 
passed as a prop to lower components, those components might do an extra 
re-rendering. React documentation generally recommend binding in the constructor 
or using the class fields syntax, to avoid this sort of performance problem.

---

## How to pass arguments to Event Handlers (event and anyOfYourArg)

Inside a loop, it is common to want to pass an extra parameter to an event 
handler. For example, if id is the row ID, either of the following would work:

```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

The above two lines are equivalent, and use arrow functions and Function.prototype.bind respectively.

In both cases, the e argument representing the React event will be passed as a second argument after the ID.
With an arrow function, we have to pass it explicitly, but with bind any further arguments are automatically forwarded.
