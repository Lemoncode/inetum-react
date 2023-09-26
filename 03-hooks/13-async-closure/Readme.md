# 13 Async closure
## Summary

This example takes the _12-set-state-func_ example as starting point.

In the previous example we saw how to solve the issue of handling
asynchronous calls and cope with frozen values, this can be a good method
if we only depend on one state field, if we are facing more
complex scenarios, we will need a more ellaborated solution,
let's see how _useRef_ hook can help to tackle on this issues.

## Steps

- First, let's copy the previous example, and do a _npm install_

```bash
npm install
```

- Let's get started, what are we going to do?

Regarding data:

- We are going to create a second counter, we save it in the state.
- We will display a message to show how many seconds have passed.

Regarding functionality:

- When we mount the component for the first time, the value of the seconds state will be 0.
- When one second passes, we will set the value of seconds state to 1.
- When two seconds pass we will set the value of the message
  (this message will display the value stored in seconds state).

In the component:

- We show the number of seconds.
- We show the message.

> This case is made just to check how _useRef_ works you could
> solve this using other approaches.

```tsx
import React from "react";

export const MyComponent = () => {
  const [message, setMessage] = React.useState("initial message");
  const [seconds, setSeconds] = React.useState(0);

  React.useEffect(() => {
    setTimeout(() => {
      console.log(seconds);
      setSeconds(1);
    }, 1000);

    setTimeout(() => {
      setMessage(`Total seconds: ${seconds}`);
    }, 2000);
  }, []);

  return (
    <>
      <h3>{message}</h3>
      <h4>{seconds}</h4>
    </>
  );
};
```

If we weren't aware of the problem with closures, we would expect
that the final message was "Total seconds: 1", but we will execute it and see that
the message that appears on the screen is "Total seconds: 0"

To solve this, Facebook chaps provide us with the _useRef_ hook, this hook:

- Stores a initial value (same as with useState).
- Returns an object.
- This object exposes a _current_ property which is a mutable variable
  (the value of the seconds would be stored here), if we modify this value
  in a future render, it will be available in past async call.
- When another render is executed, _useRef_ returns the same instance of the object.

Let's check this in action:

_./src/demo.tsx_

```diff
import React from "react";

export const MyComponent = () => {
  const [message, setMessage] = React.useState("initial message");
  const [seconds, setSeconds] = React.useState(0);

+ const secondsRef = React.useRef(seconds);

  React.useEffect(() => {
    setTimeout(() => {
      console.log(seconds);
      setSeconds(1);
+      secondsRef.current = 1;
    }, 1000);

    setTimeout(() => {
-      setMessage(`Total seconds: ${seconds}`);
+      setMessage(`Total seconds: ${secondsRef.current}`);
    }, 2000);
  }, []);

  return (
    <>
      <h3>{message}</h3>
      <h4>{seconds}</h4>
    </>
  );
};
```

- If we execute it, we will see how get now the expected behavior.

```bash
npm start
```

- That's very cool, but if you have to use _useRef_ it can be a
  code bad smell, there could be easier way to achieve what you
  need, for instance we could just wrap the message, plus the
  seconds in the same object / state:

```diff
export const MyComponent = () => {
-  const [message, setMessage] = React.useState("initial message");
-  const [seconds, setSeconds] = React.useState(0);
-  const secondsRef = React.useRef(seconds);
+  const [info, setInfo] = React.useState({
+    message: 'initial message',
+    seconds: 0,
+ });
```

- We are now going to change the content of the useEffect, pay attention to the setState
  that we are going to use it as a function, this will provide us with the last value:

```diff
- const secondsRef = React.useRef(info.seconds);

  React.useEffect(() => {
    setTimeout(() => {
-      console.log(seconds);
-      setSeconds(1);
-      secondsRef.current = 1;
+      console.log(info.seconds);
+      setInfo(info => ({...info, seconds: 1}))
    }, 1000);

    setTimeout(() => {
-      setMessage(`Total seconds: ${secondsRef.current}`);
+      setInfo(info => ({...info, message: `Total seconds: ${info.seconds}`}));
    }, 2000);
  }, []);
```

- Now that we are using the object we have to update the places where we bind the data in the markup.

```diff
  return (
    <>
-      <h3>{message}</h3>
-      <h4>{seconds}</h4>
+      <h3>{info.message}</h3>
+      <h4>{info.seconds}</h4>

    </>
  );
```

- If we execute it we will see it working as well.

```bash
npm start
```

# About Basefactor + Lemoncode

We are an innovating team of Javascript experts, passionate about turning your ideas into robust products.

[Basefactor, consultancy by Lemoncode](http://www.basefactor.com) provides consultancy and coaching services.

[Lemoncode](http://lemoncode.net/services/en/#en-home) provides training services.

For the LATAM/Spanish audience we are running an Online Front End Master degree, more info: http://lemoncode.net/master-frontend
