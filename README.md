## How to make work "this" in callbacks?
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

---

## How to set local state with props and avoid issues with update state

If you set local state from props you may face issue when new props are not reflect in
your state. So you need to add some additional code in "componentDidUpdate" method.

```
class ProfileStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      statusText: this.props.status,
    }
  }

  componentDidUpdate(prevProps, prevState) {
    if(prevProps.status !== this.props.status) {        // without this condition you will have endless loop
      this.setState({ statusText: this.props.status })
    }
  }
  
  // other code
}
```

---

## How to set one handler to several inputs

<details>
<summary>Example with a class component</summary>

```
    class Reservation extends React.Component {
      constructor(props) {
        super(props);
        this.state = {
          isGoing: true,
          numberOfGuests: 2
        };
    
        this.handleInputChange = this.handleInputChange.bind(this);
      }
    
      handleInputChange(event) {
        const target = event.target;
        const value = target.type === 'checkbox' ? target.checked : target.value;
        const name = target.name;
    
        this.setState({
          [name]: value
        });
      }
    
      render() {
        return (
          <form>
            <label>
              Is going:
              <input
                name="isGoing"
                type="checkbox"
                checked={this.state.isGoing}
                onChange={this.handleInputChange} />
            </label>
            <br />
            <label>
              Number of guests:
              <input
                name="numberOfGuests"
                type="number"
                value={this.state.numberOfGuests}
                onChange={this.handleInputChange} />
            </label>
          </form>
        );
      }
    }
    
    ReactDOM.render(
      <Reservation />,
      document.getElementById('root')
    );

```

</details>

[Source](https://reactjs.org/docs/forms.html#handling-multiple-inputs)

<details>
<summary>Example with a functional component</summary>

```
    export default function Form() {
      const [person, setPerson] = useState({
        firstName: 'Barbara',
        lastName: 'Hepworth',
        email: 'bhepworth@sculpture.com'
      });
    
      function handleChange(e) {
        setPerson({
          ...person,
          [e.target.name]: e.target.value
        });
      }
    
      return (
        <>
          <label>
            First name:
            <input
              name="firstName"
              value={person.firstName}
              onChange={handleChange}
            />
          </label>
          <label>
            Last name:
            <input
              name="lastName"
              value={person.lastName}
              onChange={handleChange}
            />
          </label>
          <label>
            Email:
            <input
              name="email"
              value={person.email}
              onChange={handleChange}
            />
          </label>
          <p>
            {person.firstName}{' '}
            {person.lastName}{' '}
            ({person.email})
          </p>
        </>
      );
    }

```
</details>

[Source](https://beta.reactjs.org/learn/updating-objects-in-state#)

---

## How to read data from &lt;input type="file" /> and send it with axios

1. Reading

```
const ProfileAvatar = () => {
    const handleAvatarChange = (e) => {
        if (e.target.files.length) {
            const file = e.target.files[0];  //  reading
            props.updateAvatar(file);
        }
    }

    return <input type="file" onChange={handleAvatarChange} />
};

export default ProfileAvatar;
```

2. Sending

```
  updateAvatar(avatarFile) {
    let data = new FormData();          // 1) create FormData instance
    data.append('image', avatarFile);   // 2) append your file
    debugger;
    return axiosInstance.put(`profile/photo`, data, {
      headers: {
        'Content-Type': `multipart/form-data`,  // 3) specify headers
      }
    });
  },
```

---

## How to add css class in map method conditionally

```const TodoList = (props) => {
    // ...
    return (
        <ul>
            {todos.map(todo => {        // {id: 1234, title: 'Todo title', completed: true }
                const classes = ['todo']
                if (todo.completed) {
                    classes.push('completed')
                }
                
                return (
                    <li className={classes.join(' ')} key={todo.id}>
                        // ...
                    </li>
                )
            })}
        </ul>
    )
}
```

[Source](https://github.com/vladilenm/react-typescript-intro/blob/d5ff18c6622cbf13f4f2ab9abc292c4b499a7853/src/components/TodoList.tsx#L27)


---

## How to get params from url with react-router

Before react-router v5.1 was released (until 2019)

```
function BlogPost({ match }) {
  let { slug } = match.params
  // ...
}
```

After react-router v5.1 was released (after 2019)

```
import { useParams } from 'react-router-dom'
function BlogPost() {
  let { slug } = useParams()
  // ...
}
```

[Source](https://reacttraining.com/blog/react-router-v5-1)

---

## How to specify conditional parameters in route

Before react-router v6

```
<Route path="/edit/:todoId?" element={<EditItem />} />
```

After react-router v6

```
<Route path="/edit">
  <Route index element={<EditItem />} />
  <Route path=":todoId" element={<EditItem />} />
</Route>
```
or
```
<Route path="/edit" element={<EditItem />} />
<Route path="/edit/:todoId" element={<EditItem />} />
```

[Source](https://reacttraining.com/blog/react-router-v5-1)
