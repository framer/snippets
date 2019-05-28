# `Page`: Externally control `Page` components

[Steve Ruiz](https://twitter.com/steveruizok) created some code that can be used to create overrides that turn normal components into controls for your `Page` component.

## The code

Create a new file in Framer X called `CreatePageControls.ts` and replace its contents by copy/pasting the following code:

```javascript
import * as React from "react"

export function createPageControls(options?: {
    currentPage?: number
    loop?: boolean
    history?: number[]
}) {
    const { currentPage = 0, loop = false } = options
    const { history = [currentPage] } = options

    // Initial Store
    let store = {
        connected: null,
        progress: 0,
        currentPage,
        pages: [],
        loop,
        history,
    }

    // Store
    const storeSetters = new Set()
    const setStoreState = (changes: any) => {
        store = { ...store, ...changes }
        store.progress = store.currentPage / (store.pages.length - 1) || 0
        storeSetters.forEach(setter => setter(store))
    }

    // Hook
    const usePageControls = (props?: any) => {
        // Connect to store
        const [state, setState] = React.useState(store)
        React.useEffect(() => () => storeSetters.delete(setState), [])
        storeSetters.add(setState)
        const set = (values: any) => setStoreState(values)

        // If this
        if (props) {
            React.useEffect(() => {
                const componentProps = props.children[0].props

                setStoreState({
                    currentPage: componentProps.currentPage,
                    pages: componentProps.children,
                })
            }, [props.currentPage, props.pages])
        }

        /**
         * Ensure that the next page is an actual page.
         */
        const validatePageIndex = (index: number) => {
            const { pages } = state

            let nextPage: number

            if (state.loop) {
                nextPage = (pages.length + index) % pages.length
            } else {
                nextPage = index
            }

            return Math.max(Math.min(nextPage, pages.length - 1), 0)
        }

        /**
         * Updates the hook when the user changes the Page component's current page.
         */
        const onChangePage = (currentPage: number) => {
            if (currentPage !== state.currentPage) {
                snapToPage(currentPage)
            }
        }

        /**
         * Snaps the page component to the page at the provided index. Defaults to
         * `0`.
         */
        const snapToPage = (index: number = 0) => {
            const { pages, history, currentPage } = state

            const nextPage = validatePageIndex(index)

            if (nextPage !== currentPage) {
                set({
                    currentPage: nextPage,
                    history: [...history, currentPage],
                })
            }
        }

        /**
         * Snaps the page component to the next page in a given direction,
         * either `"right"` or `"left"`. Defaults to `"right"`.
         */
        const snapToNextPage = (direction: "right" | "left" = "right") => {
            const next = nextPage(direction)
            if (next === null) return
            snapToPage(next)
        }

        /**
         * Snaps the page component to the previous page in the hook's "history"
         * of visited pages.
         */
        const snapToPreviousPage = () => {
            const { history, currentPage } = state

            if (history.length <= 1) return

            const h = [...history]

            const nextPage = validatePageIndex(h.pop())

            if (nextPage !== currentPage) {
                set({
                    currentPage: nextPage,
                    history: h,
                })
            }
        }

        /**
         * Snaps the page component to the nearest page to a given "progress"
         * value, where `0` is the Page component's first page and `1` is the last.
         */
        const snapToProgress = (progress: number) => {
            const { pages } = state

            const index = Math.round((pages.length - 1) * progress)

            snapToPage(index)
        }

        /**
         * Returns the index of the next page in the given direction, or else
         * `null` if no page exists in that direction.
         */
        const nextPage = (direction: "right" | "left" = "right") => {
            const { pages, currentPage } = state
            const offset = direction === "right" ? 1 : -1

            if (state.loop) {
                return (pages.length + (currentPage + offset)) % pages.length
            } else {
                const next = currentPage + offset

                return next < 0 || next > pages.length - 1 ? null : next
            }
        }

        /**
         * Returns the index of the previous page in the hook's "history" of
         * visited pages, or else `null` if no page exists.
         */
        const previousPage = () => {
            const { history } = state

            const h = [...history]

            if (h.length <= 1) {
                return null
            }

            return validatePageIndex(h.pop())
        }

        return {
            pages: store.pages,
            currentPage: store.currentPage,
            totalPages: store.pages.length,
            progress: store.progress,
            history: store.history,
            nextPage,
            previousPage,
            snapToNextPage,
            snapToPage,
            snapToPreviousPage,
            snapToProgress,
            onChangePage,
        }
    }
    return usePageControls
}
```

## Usage

This file now exports a function called `createPageControls`. This function is used to create a special hook that can, in turn, be used to turn many different components into controls for the **same** `Page` component.

### Create page controls hook

Create a new file called `PageControls.ts`.

Import `createPageControls`:

```javascript
import { createPageControls } from "./CreatePageControls";
```

Create our page controls hook like this:

```javascript
const usePageControls = createPageControls();
```

### Bind controls to `Page` component

Before we can make any controls, we need to create an override that binds these controls to a `Page` component:

```javascript
export function PageComponent(props): Override {
  const { currentPage, onChangePage } = usePageControls(props);

  return {
    currentPage,
    onChangePage
  };
}
```

In your code, you can now click on a `Page` component, and choose File: `PageControls` and Override: `PageComponent`.

### Create controls

You can now copy/paste more overrides into the `PageControls`. For instance here are some overrides to create next and previous controls:

```javascript
// Move to the next page (right)
export function NextButton(props): Override {
  const { snapToNextPage, nextPage } = usePageControls();

  return {
    opacity: nextPage("right") === null ? 0.3 : 1,
    onTap: () => snapToNextPage()
  };
}

// Move to the next page (left)
export function PrevButton(props): Override {
  const { snapToNextPage, nextPage } = usePageControls();

  return {
    opacity: nextPage("left") === null ? 0.3 : 1,
    onTap: () => snapToNextPage("left")
  };
}
```

If you now apply the `NextButton` override to a component, clicking that component will paginate the `Page` component by one page.

Some other helpful overrides:

```javascript
// Show the current page number
export function PageNumber(props): Override {
  const { currentPage, nextPage } = usePageControls();

  return {
    text: currentPage + 1
  };
}

// Jump to start (progress 0)
export function ToStart(props): Override {
  const { snapToProgress } = usePageControls();

  return {
    onTap: () => snapToProgress(0)
  };
}

// Jump to end (progress 1)
export function ToEnd(props): Override {
  const { snapToProgress } = usePageControls();

  return {
    onTap: () => snapToProgress(1)
  };
}

// Show the previous page (in history)
export function Undo(props): Override {
  const { snapToPreviousPage, previousPage } = usePageControls();

  return {
    opacity: previousPage() === null ? 0.3 : 1,
    onTap: () => snapToPreviousPage()
  };
}

// Show a progress bar
export function ProgressBar(props): Override {
  const { progress, currentPage, pages } = usePageControls();

  const background =
    pages[currentPage] && pages[currentPage].props.background.initialValue;

  return {
    animate: {
      background,
      width: props.width * progress
    }
  };
}
```
