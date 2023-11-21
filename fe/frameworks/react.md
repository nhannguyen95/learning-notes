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

React components rendering process should be pure function: they does not change preexisting variables (props, state, context).

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