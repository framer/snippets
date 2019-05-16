# Long press

Framer X and React don't currently have native support for the long press gesture. Here's a snippet to implement it with a React hook.

## Make it reusable

### The hook

```jsx
// use-long-press.ts
import { useRef } from "react";

function useLongPress(
  callback: (e: MouseEvent | TouchEvent) => void,
  duration: number = 500
) {
  // This will be a reference to our `setTimeout` counter, so we can clear it
  // if the user moves or releases their pointer.
  const timeout = useRef(null);

  // Create an event handler for mouse down and touch start events.
  function onPressStart(event: MouseEvent | TouchEvent) {
    // Prevent the browser's default response to this event. On mobile browsers
    // long presses are used . This will also block touch scrolling - a more
    // robust implementation will take this into account, but this is fine
    // for prototyping.
    event.preventDefault();

    // Start a timeout that, after the provided `duration`, will fire the
    // supplied callbacl.
    timeout.current = setTimeout(() => callback(event), duration);
  }

  // This function, when called, will cancel the timeout and thus end the
  // gesture.
  function cancelTimeout() {
    clearTimeout(timeout.current);
  }

  return {
    // Initiate the gesture on mouse down or touch start
    onMouseDown: onPressStart,
    onTouchStart: onPressStart,

    // Cancel the gesture if the pointer is moved. This is quite an aggressive
    // approach so you might want to make an alternative function here that
    // detects how far the pointer has moved from its origin using `e.pageX`
    // for `MouseEvent`s or `e.touches[0].pageX` for `TouchEvent`s.
    onMouseMove: cancelTimeout,
    onTouchMove: cancelTimeout,

    // Cancel the timeout when the pointer session is ended.
    onMouseUp: cancelTimeout,
    onTouchEnd: cancelTimeout
  };
}
```

### How to use in your component

```jsx
// MyComponent.tsx
import { useLongPress } from "./use-long-press";

export function MyComponent() {
  // Create a long press version of your gesture by passing it to `useLongPress`.
  const gestures = useLongPress(() => {
    console.log("long press!");
  });

  // Provide the gesture callbacks to the component using the spread syntax
  return <Frame {...gestures} />;
}
```

The `useLongPress` hook we created makes multiple gestures, and you might want to use some of them for other purposes. You can manually write event handlers that call the ones returned from `useLongPress`.

```jsx
const longPress = useLongPress(() => {
  console.log("long press!");
});

const onMouseDown = (e: MouseEvent | TouchEvent) => {
  longPress.onMouseDown(e);
  doOtherThings();
};

// Make sure you pass your custom event handlers **after** the ones generated
// by `useLongPress` to overwrite them!
return <Frame {...longPress} onMouseDown={onMouseDown} />;
```
