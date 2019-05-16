# Double tap

Framer X and React don't currently have native support for the double click/tap gesture. Here's a snippet to implement it with a React hook.

## Make it reusable

```jsx
// use-double-tap.ts
import { useRef } from "react";

function useDoubleTap(
  callback: (e: MouseEvent | TouchEvent) => void,
  timeout: number = 300 // 300 milliseconds
) {
  // Maintain the previous timestamp in a ref so it persists between renders
  const prevClickTimestamp = useRef(0);

  // Returns a function that will only fire the provided `callback` if it's
  // fired twice within the defined `timeout`.
  return (e: MouseEvent | TouchEvent) => {
    // performance.now() is a browser-specific function that returns the
    // current timestamp in milliseconds
    const clickTimestamp = performance.now();

    // We can get the time since the previous click by subtracting it from
    // the current timestamp. If that duration is than `timeout`, fire our callback
    if (clickTimestamp - prevClickTimestamp.current <= timeout) {
      callback(e);

      // Reset the previous timestamp to `0` to prevent users triggering
      // further double clicks by clicking in rapid succession
      prevClickTimestamp.current = 0;
    } else {
      // Otherwise update the previous timestamp to the latest timestamp.
      prevClickTimestamp.current = clickTimestamp;
    }
  };
}
```

```jsx
// MyComponent.tsx
export function MyComponent() {
  // Create a double tap/click version of your callback by providing it to
  // `useDoubleTap`. We could provide a different `timeout` as the second argument
  const onClick = useDoubleTap(() => {
    console.log("double click!");
  });

  // Provide the new callback to our `Frame` or other component.
  return <Frame onClick={onClick} />;
}
```
