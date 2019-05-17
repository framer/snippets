# Using the animation API in a Class component

We wrote the Framer animation and gesture APIs to be simple and future-resilient, which meant going all-in on React Hooks.

Some of the functionality provided by hooks like , `useMotionValue`, `useTransform` and `useAnimation` can also be used in Class components with a little wiring and manual handling of lifecycle events.

These underlying APIs weren't designed for outside the context of a hook but will work perfectly fine with the right set-up.

## `useMotionValue`

```jsx
import * as React from "react";
import { Frame, motionValue } from "framer";

export class MyComponent extends React.Component {
  // This will create a `MotionValue` when a new instance of this component
  // is created, that lasts for the duration of the component's lifecycle.
  x = motionValue(0);

  render() {
    // Pass the `MotionValue` to `Frame` as usual.
    return <Frame x={this.x} />;
  }
}
```

## `useTransform`

```jsx
import * as React from "react";
import { Frame, motionValue, transform } from "framer";

export class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    // This time we create `this.x` in the component's constructor to
    // guarantee it's created before `this.y`
    this.x = motionValue(0);

    // We can use the `transform` function in the same way as `useTransform`,
    // except it returns a function rather than a new `MotionValue`. This
    // function here, if called `transformXToY(1)` will return `2`.
    const transformXToY = transform([0, 1], [0, 2], { clamp: false });

    // We can create a new `MotionValue` as a child of another with the
    // `addChild` method. This is usually abstracted by `useTransform`.
    this.y = this.x.addChild({ transformer: transformXToY });
  }

  render() {
    return <Frame x={this.x} y={this.y} />;
  }
}
```

## `useAnimation`

```jsx
import * as React from "react";
import { Frame, AnimationControls } from "framer";

export class MyComponent extends React.Component {
  // Create `AnimationControls` by invoking the class directly, rather than
  // via the `useAnimation` hook
  controls = new AnimationControls();

  componentDidMount() {
    // Tell the controls that the component has mounted, which means that the
    // DOM elements are now safe and ready to animate.
    this.controls.mount();
  }

  componentWillUnmount() {
    // This will ensure all animations are stopped when the component unmounts,
    // and that the controls can't be used to trigger any further animations.
    this.controls.unmount();
  }

  render() {
    // Pass the controls to the `Frame.animate` prop as usual.
    return <Frame animate={this.controls} />;
  }
}
```
