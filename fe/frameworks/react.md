## JSX

- JSX = Javascript XML.
- JSX allows writing HTML elements inside Javascript code. It converts HTML tags (`<h1>`, `<MyButton>`, etc.) into React elements and place them in the DOM by calling `React.createElement`, `React.appendChild`, etc.
- The above conversion from JSX to native Javascript can be done by a transpiler like Babel.
- React and JSX are 2 separate things. React doesn't require JSX, but it's helpful as a visual aid when working with UI inside Javascript code.

## Fragment

Fragment is an empty tag (`<> </>`), used to group things without leaving any trace in the browser HTML tree.

## Purity

Pure functions:
- Does not mutate already existing variables.
- Given same input, produces same output.

React components rendering process should be pure function:
- Does not mutate already existing variables: they does not change preexisting variables (props, state, context, DOM nodes, etc.).
- Given same input, produces same output: this can imply many things:
  - Reading preexisting mutable variables (since they can be changed elsewhere), so you shouldn't even read (let alone write) mutable variables (like `ref.current`) during rendering.

Benefits of pure React components:
- The component can run in different environments.
- Enabling rendering skipping when component's inputs are not changed.

## Event propagation

Each event propagates in 3 phases:
- It travels down from parent to clicked element, calling all event capture handlers (e.g. `onClickCapture`).
- It travels upwards from clicked element to parent, calling all event handlers (e.g. `onClick`).

Event propagation does not work for `onScroll` which only called on the JSX tag you attach it to.

You can stop the propagation of an event by using `event.stopPropagation()`.

## How useState works

```typescript
let componentHooks = [];
let currentHookIndex = 0;

// How useState works inside React (simplified).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // This is not the first render,
    // so the state pair already exists.
    // Return it and prepare for next Hook call.
    currentHookIndex++;
    return pair;
  }

  // This is the first time we're rendering,
  // so create a state pair and store it.
  pair = [initialState, setState];

  function setState(nextState) {
    // When the user requests a state change,
    // put the new value into the pair.
    pair[0] = nextState;
    updateDOM();
  }

  // Store the pair for future renders
  // and prepare for the next Hook call.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Counter() {
  const [count, setCount] = useState(0);

  function handleOnClick() {
    setCount(count + 1);
    setTimeout(() => {
      alert(count);
    }, 5000);
  }
  
  return {
    onClick: handleOnClick,
    counter: `Clicked ${count} times`,
  };
}

function updateDOM() {
  // Reset the current Hook index
  // before rendering the component.
  currentHookIndex = 0;
  let output = Counter();
  
  // Update the DOM to match the output snapshot.
  // This is the part React does for you.
  counter.textContent = output.counter;
  counter.onClick = output.onClick;
}

let counter = document.getElementById('counter');
```

- When calling `useState`, React gives you a snapshot of the state for that render (`output`).
- Variables and event handlers don't "survive" re-renders, every render has its own event handlers. Every render (and functions inside it) will always see the snapshot of the state that React gave to that render.
	- This means event handlers created in the past have the state values from the render in which they where created. This explains why `alert` shows `count` instead of `count + 1` even after 5 seconds which is the time by then the `Counter` component should have already been re-rendered.

## React rendering process

- Triggering a render.
	- On initial render.
	- On re-render (props/state change).
- Rendering the component: React recursively calling your components. The JSX you return from that functional component is like a snapshot of the UI in time.
	- On initial render: calling the root component.
	- On re-render: calling the function component whose state changed.
- Committing to the DOM: React modifying the DOM.
	- On initial render: React uses `appendChild()` DOM API to put all the DOM nodes it has created on the screen.
	- On re-render: React applies the minimal necessary operations (calculated in the 2nd step above) to make the DOM match the latest rendering snapshot. React only changes the DOM nodes if there's a differences between re-renders.

After that, the browser will repaint the screen, this is known as "browser rendering".

## Batching

React waits until all code in the event handlers has run before processing your state updates. This means if there're multiple `setState` calls inside an event handler, they are batched.

React does not batch across multiple intentional events (like clicks), each event handler is handled separately.

React batches `setState` calls by put them into a queue:
- `setState(v)`: adding `replace with v` to the queue.
- `setState(v => v + 1)`: adding the updater function `v => v + 1` to the queue.

```typescript
const [n, setN] = useState(0);

// Inside some event handler:

// In next render, n = 6
setN(n + 5);
setN(n => n + 1);

// In next render, n = 42
setN(n + 5);
setN(n => n + 1);
setN(42);
```

## State

Component identity = component type (e.g. `Counter`, `h1`, etc.) + its position/structure in render tree (e.g. `root>div>div>Counter`).

Same components = same identity.

A component's state is tied to its identity: React preserves a component's state for as long as it's being rendered with same identity. If its identity changes (or when the component gets un-rendered, see `key` usage below), React destroys/resets its state.

Consequences:
- Same component type at same position preserves state:
```jsx
// Counter's state is preserved when isFancy changes since
// Counter preserves its identity regardless of change in isFancy:
// - Position: still first child of <div />
// - Component type: still Counter at this position.
function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
	  <div>
		  {isFancy ? (
			  <Counter isFancy={true} />
		  ) : (
		      <Counter isFancy={false} />
		  )}
	  </div>
  );
}
```
- Different component types at same position reset state:
```jsx
// Counter's state is reset when Counter is re-rendered
// after isFancy changes from false to true and back to false.
// This is because the component identity changes during that change:
// Position: first child of <div />
// Component type: change from Counter to p.
function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
	  <div>
		  {isFancy ? (
			  <p> Fancy counter placeholder </p>
		  ) : (
		      <Counter isFancy={false} />
		  )}
	  </div>
  );
}
```

There will be cases where you want to reset state of component with same identity, you can use `key` to override the React's default way of figuring out a component identity. Within a same parent, `key` tells React that it should identify a component based on (component type + `key`), rather than (component type + position):
```jsx
// Switching between isFancy true/false does NOT preserve
// the state of Counter since you gave them different keys,
// even they have same component type and are rendered at same
// position.
//
// Behavior:
// - "counter2" is (newly created and) shown as 0, increase it to 1.
// - switch isFancy to true.
// - "counter2" gets un-rendered (removed from the DOM).
// - "counter1" is (newly created and) shown as 0 (not 1, since different identity), increase it to 2.
// - switch isFancy to false.
// - "counter1" gets un-rendered (removed from the DOM).
// - "counter2" is (newly created and) shown as 0 (not 1), this means its state got reset, it's because previously it was un-rendered.
function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
	  <div>
		  {isFancy ? (
			  <Counter key="counter1" isFancy={true} />
		  ) : (
		      <Counter key="counter2" isFancy={false} />
		  )}
	  </div>
  );
}
```

## `useReducer`

### Writing reducers

- Reducers must be pure, since similar to state updater functions, they run during rendering.
- Each action describes a single user interaction, even if that leads to multiple changes in the data.

### Comparing `useState` and `useReducer`

- Code size: with `useState` you don't have to write a lot of (boilerplate) code upfront.
- Debugging: with `useReducer` it can be easier to tell where and why the state was set incorrectly by examining the state and the action that triggered the state update.
- Testing: a reducer is a pure function and does not depend on your component. This means you can export and test it separately.

## `useRef`

### Managing the DOM

DOM manipulation is the most common use case for refs. Sometimes you might need access to the DOM elements managed by React (e.g. to focus a node, scroll to it, or measured its size and position):

```jsx
// In your component:
const myRef = useRef(null);
<div ref={myRef}>
```

When React creates a DOM node for the `<div>`, it will assign a reference to this node to `myRef.current` during the commit phase.

You can also pass a `ref callback` to the `ref` prop, the callback will be called with the DOM node when it is time to set the ref, and with null when it's time to clear it.

```jsx
<div ref={(node) => {
  myRef.current = node;
}}>
```

Since refs are an escape hatch that should be used sparingly, React by default does not let a component access the DOM nodes of other components, not even for its own children. That's why if you try to put a ref on your own component, by default you will get `null`:

```jsx
function MyInput(props) {
  return <input {...props} />;
} 

function MyForm() {
  const myRef = useRef(null);
  return <MyInput ref={myRef} />;
}
```

Instead, the component that wants to expose their DOM node(s) (`MyInput`) have to opt in to that behavior using `forwardRef`:
```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

In the above code, the parent component of `MyInput` can do anything with `input` through the ref, such as changing its CSS. `useImperativeHandle` can be used to restrict the exposed functionality:

```jsx
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    // Only expose focus and nothing else
    focus() {
      realInputRef.current.focus();
    },
  }));

  return <input {...props} ref={realInputRef} />;
});
```

React sets `ref.current` during the commit phase. Before updating the DOM, React sets the affected `ref.current` values to `null`; after updating the DOM, React immediately sets them to the corresponding DOM nodes. Usually you will access refs from event handlers. That explains why in the following code, it scrolls to the todo item that was just before the last added one:
```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  function handleAdd() {
    const newTodo = {};

    // This is queued instead of immediately triggering a re-render/DOM update.
    setTodos([ ...todos, newTodo ]);

    // listRef is still pointing to old DOM list.
    listRef.current.lastChild.scrollIntoView(/*...*/);
  }
}
```

To fix the issue, you can use `flushSync` from `react-dom` to force React to update the DOM synchronously right after the code wrapped in `flushSync` executes.

```typescript
flushSync(() => {
  setTodos([ ...todos, newTodo ]);
});

// listRef is now pointing to new DOM list.
listRef.current.lastChild.scrollIntoView(/*...*/);
```

Best practices:
- Avoid changing DOM nodes managed by React: for example modifying, adding children to, or removing children from elements that are managed by React can lead to inconsistent visual results or crashes.
- If you must do it, do it with caution. You can safely modify parts of the DOM that React has no reason to update.

## `useEffect`

### Types of React components logic

3 types:
- Rendering logic: calculate JSX from props/states. Rendering logic must be pure.
- Event handlers:
  - Side effects triggered by user actions.
- Effects:
  - Side effects triggered by rendering/changes in [reactive values](#reactive-values).
  - Used to synchronize an external system to the current props and state.
  - **Effects run at end of commit phase after screen updates**.

### `useEffect` behavior

It was tempting to think of Effects as "callbacks" or "lifecycle events" that fire at a specific time like "after a render" or "before unmount". This way of thinking gets complicated very fast thus should be avoided.

Instead, always focus on a single start/stop synchronization cycle at a time.

```typescript
useEffect(() => {
  // Synchronization start:
  // This runs after every render.
  return () => {
    // Synchronization stop (cleanup function):
    // This runs each time before the Effect runs again,
    // and one final time when the component unmounts (gets removed).
  };
});

useEffect(() => {
  // This runs only on mount (when the component appears).
  return () => {
    // Same as above.
  };
}, []);

useEffect(() => {
  // This runs on mount and also if either a or b have changed since the last render.
  return () => {
    // Same as above.
  };
}, [a, b]);
```

Note that in development mode, React intentionally remounts your components once immediately after its initial mount and whenever you save a file.

### Fetching data in Effects

Downsides:
- Effects don't run on servers: The initial-rendered HTML only includes a loading state with no data. Client has to download all JavaScript and render your app just to discover that it now needs the data.
- Network waterfalls: rendering parent compnent -> fetching parent data -> rendering child -> fetching child data. This can be significantly slower than fetching all data in parallel.
- Can suffer from race condition, fixes requite some boilerplates:
  ```typescript
  useEffect(() => {
    const fetchData = async () => {
      const repsonse = await fetch(`${props.id}`);
      setData(response.json());
    };

    fetchData();
  }, [props.id]);  // if `id` changes fast enough, the component might display incorrect data.
  ```

  Fix #1: boolean flag

  ```typescript
  useEffect(() => {
    let abort = false;

    const fetchData = async () => {
      const repsonse = await fetch(`${props.id}`);
      if (!abort)
        setData(response.json());
    };

    fetchData();

    return () => {
      // Aborting the result originated from previous requests,
      // only update the component with result from last request.
      // Problem: we abort unsued results, not unused requests,
      // they are still in-flight.
      abort = true;
    };
  }, [props.id]);
  ```

  Fix #2: associate each request with some identity for abortion.

  ```typescript
  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      try {
        const repsonse = await fetch(`${props.id}`, { signal: abortController.signal });
        setData(response.json());
      } catch (e) {
        if (e.name === 'AbortError') {
          // Aborting a fetch throws an error
        }
      }
    };

    fetchData();

    return () => {
      abortController.abort();
    };
  }, [props.id]);
  ```

Recommendations:
- If you use a Framework, use its built-in data fetching mechanism.
- Otherwise, using/building a client-side cache: React Query, useSWR, React Router 6.4+, etc. In case developing your own, you would use Effects under the hood but add logic for deduplication requests, caching responses, avoiding network waterfalls by preloading data or hoisting data requirements to routes.

### You might not need an Effect
- When something can be calculated from existing props/states, calculate it during rendering. If the calculation is expensive, `useMemo` can be used to cache the result.
- When you want to reset the state when a prop changes, consider giving your component an identity by passing `key`:
  ```typescript
  function ProfilePage({ userId }) {
    const [comment, setComment] = useState('');
    
    // Bad: comment only gets reset after component render.
    // This means ProfilePage has a flash of stale comment
    // data when userId changes, and takes 1 extra render
    // (triggered by setComment) to reset comment:

    // userId changes
    // -> ProfilePage rendered with old comment
    // -> useEffect triggered
    // -> setComment('') called
    // -> ProfilePage rendered with '' comment.
    useEffect(() => {
      setComment('');
    }, [userId]);

    return /* rendering user profile with comment */;
  }
  ```

  Instead:

  ```typescript
  function ProfilePage({ userId }) {
    return <Profile userId={userId} key={userId}>;
  }

  function Profile({ userId }) {
    const [comment, setComment] = useState('');
    // ...
  }
  ```
- Reset/adjust the state (or part of it) when a props changes, but not all: consider "unstate"/restructure the state of the variables which depend on the props and directly calculate it from the props during rendering.
- Notify parent components about state changes:
  ```typescript
  function Toggle({ onChange }) {
    const [isOn, setIsOn] = useState(false);

    // 🔴 Bad: the effect only runs after the screen gets updated with new isOn.
    useEffect(() => {
      onChange(isOn);
    }, [isOn]);

    function handleClick() {
      setIsOn(!isOn);
      // ✅ Good: Perform all updates during the event that caused them.
      onChange(!isOn);
    }
  }
  ```

### Other best practices
- Each Effect in your code should represent/be responsible for a separate and independent synchronization process.
- When there's some logic in an Effect that access/read (the latest) reactive values, but you don't want the whole Effect reacts to that values, there are 2 (or 3) solutions:
  - Exclude those reactive values from the dependency array, however depending on your lint configuration this may cause a linting error (plus, supressing it is never recommended, since you are lying to React about your dependency list).
  - Extract that logic into an Effect Event using `useEffectEvent`, some notes:
    - Only call Effect Events from inside Effects.
    - Never pass them to other components or Hooks.
  - If you are reading some state in order to calculate its next value, use the updater function:
  ```typescript
  useEffect(() => {
    setCount(count => count + 2);
  }, []);  // count no need to be in the dependency list, thus change in count
           // does not trigger the Effect.
  ```
- Subscribe to an external store: consider using `useSyncExternalStore` instead.
- You don't choose what to put on the dependency list, the list *describes* your code. Thus if you want to change the dependency list, change the code first.
- Since objects and functions declared inside the components change during re-renders, you should try to avoid them as your Effect's dependencies. Instead, try moving them outside the component, inside the Effect, or extracting primitives values out of them.
- Effects let you step out of React and synchronize with external system. When you write custom hooks that utilize `useEffect`, you can refactor the custom hooks in a way that extracts their logic out into some classes/modules that act as the external system. This lets your custom hooks/Effects stay simple because they only need to send messages to the system you have moved outside React.

### Reference

React compares the dependency values using `Object.is` comparision.


## Passing data

Some ways to pass data to children components:
- Passing props: when the component in need of the data is not distant from the data-provider component.
- Extract components and pass JSX as `children`: when the component in need of the data is distant from the data-provider component and intermmediate components do not need the data.
- Using context: when the data is needed by distant components in different parts of the tree.

## Context

Context use cases:
- Theming.
- Current account.
- Routing.
- Managing state: you can combine reducers and context together to manage state of a complex screen (create the context, then put state and dispatch into the context, finally use context anywhere in the tree).

## Reactive values

Reactive values are values that can change due to a re-render:
- Props, state.
- All variables declared/calculated in the component body, since they can change on a re-render.

Some values you can omit from the dependency array since they have a stable identity:
- Refs: same `useRef` call returns same object on every render.
- The set functions returned by `useState`.