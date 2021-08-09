## How to make work this in callbacks?
You have to be careful about the meaning of this in JSX callbacks. In JavaScript, class methods are not bound by default. If you forget to bind this.handleClick and pass it to onClick, this will be undefined when the function is actually called.

This is not React-specific behavior; it is a part of how functions work in JavaScript. Generally, if you refer to a method without () after it, such as onClick={this.handleClick}, you should bind that method. There are three options to resolve it.

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

---