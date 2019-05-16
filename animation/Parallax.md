# Parallax scroll

Although the implementation details differ slightly whether we're in Framer X or production, the basic process of making a parallax animation is:

1. Use a `MotionValue` to record scroll position
2. Create another `MotionValue` using `useTransform` to convert from the scroll position into another value, like `y`.
3. Pass the `y` `MotionValue` to a component we want to animate.

With Framer, there are two primary methods for getting a scroll position. In Framer X, via the `Scroll` component, and in production via the `useViewportScroll` hook.

## Prototype

### Via a code component

```jsx
import { Scroll } from "framer";

export function Component() {
  // Create a `MotionValue` to track a `Scroll` component's vertical offset.
  const contentOffsetY = useMotionValue(0);

  // We can use the `useTransform` hook to translate that into any other value.
  // Using `scrollY` as a base, we want to convert its flat pixel value either
  // into faster movement (for objects closer to the user) or slower movement
  // (further from the user). Here we'll make it move slower by translating
  // `[0, -1]` into `[0, 0.5]`, ie for every pixel the user scrolls, `y` will
  // move by half a pixel.
  const y = useTransform(contentOffsetY, [0, -1], [0, 0.5]);

  return (
    <>
      // Bind `contentOffsetY` to the `Scroll` component.
      <Scroll contentOffsetY={contentOffsetY} />
      // Bind `y` to a `Frame`.
      <Frame y={y} />
    </>
  );
}
```

### Via overrides

```jsx
// Example taken from Giels Perry https://gist.github.com/perrysmotors/aa5d5bb7031a69ad17ef7a4ef29554ce
import { Override, motionValue, useTransform } from "framer";

// Create a `MotionValue` to record scroll position. We use the `motionValue`
// function here as we're outside a React component or override and therefore
// don't need to handle the component's lifecycle.
const contentOffsetY = motionValue(0);

// Apply this override to a single `Scroll` component to bind the `MotionValue`
export function TrackScroll(): Override {
  return { contentOffsetY: contentOffsetY };
}

// Apply this override to any `Frame` you want to move in parallax.
export function ParallaxLayer(): Override {
  // We're making one pixel of scroll movement convert into half a pixel of
  // `y` offset for this `Frame`. If you want different amounts of movement for
  // different depth effects you could duplicate this override and change `0.5`
  // into something else. `2` would be double movement, for instance.
  const y = useTransform(contentOffsetY, [0, -1], [0, 0.5], {
    clamp: false
  });
  return {
    y: y
  };
}
```

## Production

```jsx
import { motion, useTransform, useViewportScroll } from "framer-motion";

export function Component() {
  // `scrollY` is a `MotionValue` that emits the current vertical scroll
  // position of the page.
  const { scrollY } = useViewportScroll();

  // We can use the `useTransform` hook to translate that into any other value.
  // Using `scrollY` as a base, we want to convert its flat pixel value either
  // into faster movement (for objects closer to the user) or slower movement
  // (further from the user). Here we'll make it move slower by translating
  // `[0, 1]` into `[0, 0.5]`, ie for every pixel the user scrolls, `y` will
  // move by half a pixel.
  //
  // We make `clamp: false` so we also transform values outside of these ranges.
  const y = useTransform(scrollY, [0, 1], [0, 0.5], { clamp: false });

  // We bind `y` to a `motion` component. You can pass it to more than one if
  // you're animating multiple components in the same way. But bear in mind
  // that if you affect the `y` value in another way, for instance an animation
  // or dragging, what affects one `y` will affect them all.
  return <motion.div style={{ y }} />;
}
```
