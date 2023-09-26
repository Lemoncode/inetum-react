# 17 Use Debug Value

The internal hook _useDebugValue_ is useful for debugging custom hooks using the React DevTools. This hook allows you to display a label for a custom hook.

# Steps

- We will take as a starting point the example _16-memo-predicate_. Copy the contents of the project to a new folder and run _npm install_.

```bash
npm install
```

- Let's open the file _app.tsx_. Let's add the following content.
  _./src/app.tsx_

```tsx
import React from "react";
import { MyComponent } from "./demo";
import "./styles.css";

export const App = () => {
  return (
    <div className="App">
      <MyComponent />
    </div>
  );
};
```

- Let's also open _styles.css_. Add this content.

```css
.App {
  font-family: sans-serif;
  text-align: center;
}
```

- Let's open the file _demo.tsx_. Let's add the content that appears just below (a custom hook that saves a user and stores their TO-DO's), you can test this by entering in the
  text box a number like 1, 2, 3 (user Ids) then the list of todo's for that given user
  will appear.

_./src/demo.tsx_

```tsx
import React from "react";

const useUserTodos = () => {
  const [user, setUser] = React.useState("");
  const [userTodos, setUserTodos] = React.useState([]);

  const loadTodos = () => {
    fetch(`https://jsonplaceholder.typicode.com/todos?userId=${user}`)
      .then((response) => response.json())
      .then((json) => {
        console.log(userTodos);
        setUserTodos(json);
      });
  };

  return { user, setUser, userTodos, loadTodos };
};

export const MyComponent = () => {
  const { user, setUser, userTodos, loadTodos } = useUserTodos();

  React.useEffect(() => {
    loadTodos();
  }, [user]);

  return (
    <div>
      <input value={user} onChange={(e) => setUser(e.target.value)} />
      <h3>User {user} TO-DO's:</h3>
      <ul>
        {userTodos.map((todo, index) => (
          <li key={index}>
            {todo.title}
            {todo.completed ? <>&#10004;</> : <>&#10006;</>}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

- Now let's check what _useDebugValue_ can do for us. In this case we will show a label indicating the content of _user_.

_./src/demo.tsx_

```diff
import React from "react";

const useUserTodos = () => {
  const [user, setUser] = React.useState("");
  const [userTodos, setUserTodos] = React.useState([]);

  const loadTodos = () => {
    fetch(`https://jsonplaceholder.typicode.com/todos?userId=${user}`)
      .then(response => response.json())
      .then(json => {
        console.log(userTodos);
        setUserTodos(json);
      });
  };

+ React.useDebugValue(user !== "" ? `User ${user}` : 'No user');

  return { user, setUser, userTodos, loadTodos };
};

export const MyComponent = () => {
  const { user, setUser, userTodos, loadTodos } = useUserTodos();

  React.useEffect(() => {
    loadTodos();
  }, [user]);

  return (
    <div>
      <input value={user} onChange={e => setUser(e.target.value)} />
      <h3>User {user} TO-DO's:</h3>
      <ul>
        {userTodos.map((todo, index) => (
          <li key={index}>
            {todo.title}
            {todo.completed ? <>&#10004;</> : <>&#10006;</>}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

- Execute the example

```bash
npm start
```

- Let's open the React DevTools in Chrome or Firefox. In the _hooks_ section you can check the debug value of the custom hook.

![01-dev-tools](./resources/01-dev-tools.png)

# About Basefactor + Lemoncode

We are an innovating team of Javascript experts, passionate about turning your ideas into robust products.

[Basefactor, consultancy by Lemoncode](http://www.basefactor.com) provides consultancy and coaching services.

[Lemoncode](http://lemoncode.net/services/en/#en-home) provides training services.

For the LATAM/Spanish audience we are running an Online Front End Master degree, more info: http://lemoncode.net/master-frontend
