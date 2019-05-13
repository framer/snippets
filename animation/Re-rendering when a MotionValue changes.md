# Re-render a component when a `MotionValue` changes

`MotionValue`s are designed to facilitate high-performance user interfaces. Which is why, when they change, they don't trigger React re-renders.

This can be confusing if you expect to use the output of a `MotionValue` as a text value, as when the `MotionValue` changes the text doesn't update.

We can fix this by using React's `useState` hook, and calling its `set` function using the `MotionValue`'s `onChange` method.

```jsx
import * as React from "react";
import { useState, useEffect } from "react";
import { Frame, useMotionValue } from "framer";

export function MyComponent() {
  // Create the MotionValue
  const x = useMotionValue(0);

  // Set-up React state for the MotionValue - this will be initalised as `0`
  const [xState, setXState] = useState(x.get());

  // Bind the state's set function to the MotionValue. It's important we return
  // the unsubscribe function so when the component unmounts it unsubscribes
  // the state from the MotionValue.
  // This could be written as
  // useEffect(() => x.onChange(setXState), [])
  // for short.
  useEffect(() => {
    const unsubscribe = x.onChange(setXState);
    return unsubscribe;
  }, []);

  // Pass our `x` MotionValue to our Frame and output its latest value as text.
  return (
    <Frame x={x} animate={{ x: 100 }}>
      {xState}
    </Frame>
  );
}
```
