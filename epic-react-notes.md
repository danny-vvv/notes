# Epic React notes

# React Fundamentals

## [Javascript to Know For React](https://kentcdodds.com/blog/javascript-to-know-for-react)

Dynamic import of react component (`const BigComponent = React.lazy(() => import('./big-component'))`)

> “A word of caution around this is that if you find yourself doing **?.** a lot in your code, you might want to consider the place where those values originate and make sure they consistently return the values they should.”

## [Imperative vs Declarative Programming](https://ui.dev/imperative-vs-declarative-programming/)

Imperative is the step-by-step logic for doing something (the "how"), declarative is what you want to do (the "what").
- Imperative: "stand up, turn 180 degrees, walk forward, turn right..."
- Declarative: "go to the kitchen" (`me.goToKitchen()`)

## React / ReactDOM packages

* React: creation of React elements
* ReactDOM: managing React elements in DOM, i.e. inserting elements

## [Babel repl](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=App&corejs=3.6&spec=false&loose=false&code_lz=MYewdgzgLgBArgSxgXhgHgCYIG4D40QAOAhmLgBICmANtSGgPRGm7rNkDqIATtRo-3wMseAFBA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=react&prettier=true&targets=&version=7.16.6&externalPlugins=&assumptions=%7B%7D)

Babel transpiles JSX into React api calls

JSX:

```jsx
const ui = <div><span>Hello</span> <span>World</span></div>
```

JS:

```js
const ui = /*#__PURE__*/ React.createElement(
  "div",
  null,
  /*#__PURE__*/ React.createElement("span", null, "Hello"),
  " ",
  /*#__PURE__*/ React.createElement("span", null, "World")
);
```

## React Fragments

Components can only render one child because React uses `React.createElement` to create each element, which can only create one element at a time (this is the same as how creating elements with Javascript DOM methods works).

```javascript
React.createElement(
	‘div’, 
	{prop: ‘hello’},
	React.createElement(‘span’, {children: ‘world’})
)
```

## Forms

### Uncontrolled Form inputs

> [...] the browser is maintaining the state of the input by itself and we can be notified of changes and “query” for the value from the DOM node.

### Controlled Form inputs

> [...] React allows us to programmatically set the `value` prop on the input like so:
>
> ```jsx
> <input value={myInputValue} />
> ```
>
> Once we do that, React ensures that the value of that input can never differ from the value of the `myInputValue` variable.

### SyntheticEvent

> 2:44 Synthetic event is actually not a real event from the browser. It's an object that React creates for us that looks and behaves exactly like a regular event.
>
> 2:55 Most of the time, you won't know that you're interacting with a fake event or a SyntheticEvent from React. You'll just assume that you're interacting with the native event. They do this for performance reasons. React also uses event delegation, if that's something that you've heard of, all to just improve the performance of your application.

## Rendering arrays

```js
const list = ['One', 'Two', 'Three']
const listUI = list.map(listItem => <li>{listItem}</li>)
// notice that listUI is an array
const ui = <ul>{listUI}</ul>
```

> If you re-render that list with an added item, React doesn’t really know whether you added an item in the middle, beginning, or end. And the same goes for when you remove an item (it doesn’t know whether that happened in the middle, beginning, or end either

React can make a "best-guess", which is ok in simple cases.

> However, if any of those React elements represent a component that is maintaining state, that can be pretty problematic

Explanation: [Understanding React's Key Prop](https://kentcdodds.com/blog/understanding-reacts-key-prop)

Using the element's index as the key removes the warnings because the keys will all be unique, but this doesn't prevent bugs. The indexes of the mapped elements are updated on rerender, so they're not really unique identifiers for each item.

```jsx
items.map((item, index) => <div key={index} />)
```

For example, say our map has crated an element with `key={4}`. If an earlier element is removed from the array, then on rerender that key would no longer be `4`, it would be `3`. In this scenario we haven't actually provided React with a way to track the elements in the array, we've merely circumvented the key warning. 

In other words, we *have* provided keys and they *are* unique - they just happen to be completely useless.

Ideally, all our data should have unique ids which can then be used as keys.

See: 

* [Use the key prop when Rendering a List with React](https://egghead.io/lessons/react-use-the-key-prop-when-rendering-a-list-with-react) 
* [Why React needs a key prop](https://epicreact.dev/why-react-needs-a-key-prop/)

# React Hooks

## Lazy state initialisation

> [...] React’s useState hook allows you to pass a function instead of the actual value, and then it will only call that function to get the state value when the component is rendered the first time. So you can go from this: `React.useState(someExpensiveComputation())` To this: `React.useState(() => someExpensiveComputation())`
>
> And the `someExpensiveComputation` function will only be called when it’s needed!

[lazy state initialization](https://kentcdodds.com/blog/use-state-lazy-initialization-and-function-updates)

> ```
> const getInitialState = () => Number(window.localStorage.getItem('count'))
> const [count, setCount] = React.useState(getInitialState)
> ```
>
> Creating a function is fast. Even if what the function does is computationally expensive. So you only pay the performance penalty when you *call* the function. So if you pass a function to `useState`, React will only call the function when it needs the initial value (which is when the component is initially rendered).
>
> This is called "lazy initialization." It's a performance optimization. You shouldn't have to use it a whole lot, but it can be useful in some situations, so it's good to know that it's a feature that exists and you can use it when needed. I would say I use this only 2% of the time. It's not really a feature I use often.

Example:

```js
// Accesses local storage on every re-render.

const [name, setName] = React.useState(
  window.localStorage.getItem('name') ?? initialName,
)
```

```js
// Creates a function every re-render. The function is only called on mount. Therefore local storage is only accessed once. 

const [name, setName] = React.useState(
  () => window.localStorage.getItem('name') ?? initialName,
)
```

This is useful when the state initialisation is computationally expensive or involves an IO action (i.e. accessing local storage). Creating a function every re-render is cheaper by comparison.

## [Function updates](https://kentcdodds.com/blog/use-state-lazy-initialization-and-function-updates#dispatch-function-updates)

> **Any time I need to compute new state based on previous state, I use a function update**.

> ```jsx
> function DelayedCounter() {
>   const [count, setCount] = React.useState(0)
>   const increment = async () => {
>     await doSomethingAsync()
>     setCount(previousCount => previousCount + 1)
>   }
>   return <button onClick={increment}>{count}</button>
> }
> ```

> You can click that as frequently as you like and it'll manage updating the count for every click. This works because we no longer worry about accessing a value that may be "stale" but instead we get access to the latest value of the variable we need. So even though the `increment` function we're running in has an older version of `count`, our function updater receives the most up-to-date version of the state.

## Custom hooks

> [...] what makes a custom hook a custom hook isn't that the function starts with "use", but that it uses other hooks inside of it. **All a custom hook is is a function that uses hooks.** 

A basic `useLocalStorageState` hook:

```js
function useLocalStorageState(key, defaultValue) {
  const [state, setState] = React.useState(
    () => getFromLocalStorage(key) || defaultValue,
  )

  React.useEffect(() => {
    saveToLocalStorage(key, state)
  }, [key, state])

  return [state, setState]
}

// Utility functions
function getFromLocalStorage(key) {
  return JSON.parse(window.localStorage.getItem(key))
}

function saveToLocalStorage(key, value) {
  window.localStorage.setItem(key, JSON.stringify(value))
}
```

Usage (in a *tic-tac-toe* game):

```js
const [squares, setSquares] = useLocalStorageState(
  'squares',
  Array(9).fill(null),
)
```

## Hooks flow

![flow chart](https://raw.githubusercontent.com/donavon/hook-flow/master/hook-flow.png)

## Lifting state

> As a community we’re pretty good at lifting state. It becomes natural over time. One thing that we typically have trouble remembering to do is to push state back down (or [colocate state](https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster)).

## useState

> - **Managed State:** State that you need to explicitly manage
> - **Derived State:** State that you can calculate based on other state

[My tic-tac-toe game exercise on Code Sandbox](https://codesandbox.io/s/hopeful-jones-h023z-tic-tac-toe-h023z?file=/src/App.js)

## useRef

Refs are used to provide access to DOM nodes. When we create DOM nodes in JSX, all we're doing is writing syntax which is used to call `React.createElement()` - we're not actually creating DOM elements directly, so we don't have direct access to them once they're created.

This needs to be considered when using packages which require access to the DOM directly. For instance, a package may use vanilla javascript to add event listeners to DOM nodes.

Below is an example using a package called VanillaTilt:

```jsx
function Tilt({children}) {
  React.useEffect(() => {
    const tiltNode = tiltRef.current
    // Access the DOM node directly via it's ref:
    VanillaTilt.init(tiltNode, {})

    // Clean-up function so event listeners aren't left in DOM
    // after component unmount:
    return () => tiltNode.vanillaTilt.destroy()
  }, [])
  
  // The <div> below is a call to React.createElement().
  // It isn't the DOM node itself.
  return (
    <div ref={tiltRef}>
      <div>{children}</div>
    </div>
  )
}
```

## async useEffect

> One important thing to note about the `useEffect` hook is that you cannot return anything other than the cleanup function. This has interesting implications with regard to async/await syntax:
>
> ```javascript
> // this does not work, don't do this:
> React.useEffect(async () => {
>   const result = await doSomeAsyncThing()
>   // do something with the result
> })
> ```
>
> The reason this doesn’t work is because when you make a function async, it automatically returns a promise (whether you’re not returning anything at all, or explicitly returning a function). This is due to the semantics of async/await syntax. So if you want to use async/await, the best way to do that is like so:
>
> ```javascript
> React.useEffect(() => {
>   async function effect() {
>     const result = await doSomeAsyncThing()
>     // do something with the result
>   }
>   effect()
> })
> ```
>
> This ensures that you don’t return anything but a cleanup function.
>
> 🦉 I find that it’s typically just easier to extract all the async code into a utility function which I call and then use the promise-based `.then` method instead of using async/await syntax:
>
> ```javascript
> React.useEffect(() => {
>   doSomeAsyncThing().then(result => {
>     // do something with the result
>   })
> })
> ```

### Error handling:

```javascript
// option 1: using .catch
fetchPokemon(pokemonName)
  .then(pokemon => setPokemon(pokemon))
  .catch(error => setError(error))

// option 2: using the second argument to .then
fetchPokemon(pokemonName).then(
  pokemon => setPokemon(pokemon),
  error => setError(error),
)
```

> Using `.catch` means that you’ll handle an error in the `fetchPokemon` promise, but you’ll *also* handle an error in the `setPokemon(pokemon)` call as well. This is due to the semantics of how promises work.
>
> Using the second argument to `.then` means that you will catch an error that happens in `fetchPokemon` only.

### Using a status state

[Stop using isLoading booleans](https://kentcdodds.com/blog/stop-using-isloading-booleans)

### Using a state object

```js
const [{status, pokemon, error}, setState] = React.useState({
  status: 'idle',
  pokemon: null,
  error: null
})
```

```js
setState({status: 'pending' })

fetchPokemon(pokemonName).then(
  pokemon => {
  setState({pokemon: pokemon, status: 'resolved'})
  },
  error => {
  setState({status: 'rejected', error: error})
  },
)
```

> This is a really important point. Whatever you pass to your state updater function is exactly the value that the state is going to be on the next render. In our case, that means that our state is going to be this object.

So if we have:

```
setState({pokemon: pokemon, status: 'resolved'})
setState({status: 'error'})
```

...then `state` will end up being `{status: 'error'}`, *not* `{pokemon: pokemon, status: 'error'}`. This is helpful because it gives us greater control over different bits of state. If we separated these states out into separate useState hooks, then we'd need to ensure that we always set all the different states when something changes.

For example:

```js
if (status === 'pending') {
  setPokemon(null)
  setError(null)
}

if (status === 'error') {
  setError(error)
  setStatus('resolved')
  setPokemon(null)
}
```

This could quickly get complicated; it's not very easy to scale, and leaves us prone to introducing bugs. What if a developer comes in and adds a new call to `setPokemon`, but doesn't set the status and error at the appropriate points? Then we'd be relying on tests (which may or may not be written to cover the particular case) or a reviewer to spot this in PR.

## Error boundaries

React apps crash when a runtime errors occurs. The React team chose to display a blank page in these cases rather than a potentially broken UI. 

This was deemed the safest solution, as a broken UI could pose problems, i.e. users sending a message to the wrong person in Messenger, or banking apps showing the wrong figures.

> While runtime errors like this are problematic and you should probably fix them, they do sometimes happen even if you're using typescript [...]
>
> It would be better to show a useful error than a white screen of death where somebody has to pull up the developer tools to figure out what's going on.

Error boundaries catch runtime errors and can be used to render content when an error occurs, instead of breaking the whole application.

Currently, error boundaries can only be written as class components.

```jsx
class ErrorBoundary extends React.Component {
  state = {error: null}
  static getDerivedStateFromError(error) {
    return {error}
  }

  render() {
    const {error} = this.state
    if (error) {
      return <this.props.FallbackComponent error={error} />
    }
    return this.props.children
  }
}

function App() {
  return (
    <ErrorBoundary key={key} FallbackComponent={ErrorFallback}>
      <HelloWorld />
    </ErrorBoundary>)
}
```

In the above example, an `ErrorBoundary` component is used to wrap the `HelloWorld` component. When a runtime error occurs in `HelloWorld`, React will look for the closest parent error boundary, which is in this case the `ErrorBoundary` wrapper.

If an error occurs in ErrorBoundary, React will look for `ErrorBoundary`'s closest parent ErrorBoundary. If one isn't found, the error won't be handled and we will instead get a white screen.

### Re-mounting the error boundary

We also need to implement a way to reset the error state of the `ErrorBoundary` component. One quick way to do this is by giving the `ErrorBoundary` component a unique key prop, as in the above example.

When the key changes, `ErrorBoundary` will be re-mounted and `getDerivedStateFromError` will run again. Assuming there's no longer any errors, the newly mounted `ErrorBoundary` won't have an error state.

This may not work in all scenarios, but it is a relatively ok solution otherwise.

### react-error-boundary

[GitHub](https://github.com/bvaughn/react-error-boundary) | [NPM](https://www.npmjs.com/package/react-error-boundary) | [Yarn](https://yarnpkg.com/package/react-error-boundary)

While we can create our own error boundary components and logic, it'd be better if we didn't need to think about writing and maintaining it ourselves.

react-error-boundary is a package that provides an `ErrorBoundary` component we can use out of the box:

```jsx
import { ErrorBoundary } from 'react-error-boundary'

function App() {
  return (
    <ErrorBoundary key={key} FallbackComponent={ErrorFallback}>
      <HelloWorld />
    </ErrorBoundary>)
}
```

All we've done here is removed the `ErrorBoundary` class component we created and replaced it's usage with the imported `ErrorBoundary`.

### Resetting the error boundary

The problem with re-mounting the error boundary component is we end up re-mounting it's child components too, in this case `HelloWorld`. Depending on the children, this could cause problems with the UI.

Re-mounting the `ErrorBoundary` and it's children when we have an error is a legitimate solution, but we don't need to re-mount everything when we're just changing the key prop to render different content.

This might be explained better with a more realistic use case:

```jsx
<ErrorBoundary key={user.id} FallbackComponent={ErrorFallback}>
  <User user={user} />
</ErrorBoundary>)
```

When we change the `user`, the `ErrorBoundary`'s key changes, so everything is re-mounted. That's ok if there was an error, but it's unnecessary when no error occurred.

react-error-boundary provides a way to reset the error state instead of relying on re-mounting. There are [some examples in the usage section of their GitHub readme](https://github.com/bvaughn/react-error-boundary#usage).

This is a fairly simple example based on the above code:

```jsx
import { ErrorBoundary } from 'react-error-boundary'

function ErrorFallback({error, resetErrorBoundary}) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  )
}

function App() {
  const [user, setUser] = React.useState({ id: null })
  
  // Simulating some interaction that would change the user:
  setUser({ id: 1 })
  
  function handleReset() {
    setUser({id: null})
  }
  
  return (
    <ErrorBoundary 
      key={user.id} 
      FallbackComponent={ErrorFallback}
      onReset{handleReset}
    >
      <User user={user} />
    </ErrorBoundary>)
}
```

#### Reset keys

react-error-boundary also provides a `resetKeys` prop:

```jsx
<ErrorBoundary resetKeys={[user.id]} />
```

`resetKeys` takes an array. When any of the values in the array change, the error boundary's state will be reset. This can be used to remove error messages from the UI if the user does something else, for example selecting a different user.

# Advanced React hooks

## useReducer

> [Should I useState or useReducer?](https://kentcdodds.com/blog/should-i-usestate-or-usereducer)

> [How to implement useState with useReducer](https://kentcdodds.com/blog/how-to-implement-usestate-with-usereducer)

The `useReducer` hooks looks like this:

`const [state, dispatch] = React.useReducer(reducer, initialState)`

* `state`: our state, like with `useState`
* `dispatch`: our dispatch function (or state setter) for updating the state
* `reducer`: a function which will return the new state
* `initialState`: our initial state, the same as in `useState`

Below is an example of using `useReducer` for a simple counter component:

```jsx
function countReducer(state, action) {
  const {type, step} = action

  switch (type) {
    case 'INCREMENT':
      return {count: state.count + step}
    default:
      throw new Error(`Unhandled action type "${type}"`)
  }
}

function Counter({initialCount = 0, step = 1}) {
  const [state, dispatch] = React.useReducer(countReducer, {
    count: initialCount,
  })
  const {count} = state
  const increment = () => dispatch({type: 'INCREMENT', step})

  return <button onClick={increment}>{count}</button>
}
```

Note that the use of an `action.type` and the switch statement in the reducer function are conventions made popular by Redux. We don't have to follow these conventions at all.

For example:

```jsx
function countReducer(state, newState) {
  return newState
  // or for more complex state objects:
  // return {...state, ...newState}
}

function Counter() {
  const [state, dispatch] = React.useReducer(countReducer, {
    count: 0,
  })
  const {count} = state
  const increment = () => dispatch({count: count + 1})

  return <button onClick={increment}>{count}</button>
}
```

### Lazy initialisation

A function can be passed as a third argument to `useReducer` to enable lazy initialisation of the state. This third argument is passed `useReducer`'s' the second argument.

Example 1:

```jsx
function init(initialCount) {
  return initialCount // => 0
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, 0, init)
}
```

In the above case, `init` is called at the lazy initialisation stage with an argument of `0` and returns the initial state.

Example 2:

```jsx
function init(initialState) {
  return {
    count: initialState.count,
    somethingElse: someExpensiveCalculation()
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, {count: 0}, init)
}
```

The above gives perhaps a clearer view of how `init` can be used other than simply returning the `initialState`. Now we're getting some more initial state, but isolating an expensive function call to the lazy state initialisation stage of the React lifecycle.

## Memoization

> Memoization: a performance optimization technique which eliminates the need to recompute a value for a given input by storing the original computation and returning that stored value when the same input is provided. Caching is a form of memoization.

> Another interesting aspect to memoization is the fact that the cached value you get back is *the same one* you got last time.

See: [Memoization and React](https://epicreact.dev/memoization-and-react/)

## useCallback

Say we have a `useEffect` which calls a function:

```jsx
function updateLocalStorage() {
  return window.localStorage.setItem('count', count)
}

React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage]) // <-- function as a dependency
```

Every time the component re-renders, `updateLocalStorage` is recreated, therefore it is a different function on every render. The consequence of this is that the `useEffect` will run on every re-render, because it's dependency of `updateLocalStorage` is changing each time.

This is what `useCallback` solves.

```javascript
const updateLocalStorage = React.useCallback(
  () => window.localStorage.setItem('count', count),
  [count] // <-- dependency list
)
React.useEffect(() => {
  updateLocalStorage()
}, [updateLocalStorage])
```

In this case, React's `useCallback` hook only gives us a new `updateLocalStorage` function if it's dependencies (`[count]`) change, so the `useEffect` won't run on every re-render. 

Instead, the `useEffect` will only run when `useCallBack` gives us back a new `updateLocalStorage` function, which will only happen when `count` changes.

> [...]  `useCallback` is just a shortcut to using `useMemo` for functions:
>
> ```jsx
> // the useMemo version:
> const updateLocalStorage = React.useMemo(
>   // useCallback saves us from this annoying double-arrow function thing:
>   () => () => window.localStorage.setItem('count', count),
>   [count],
> )
> 
> // the useCallback version
> const updateLocalStorage = React.useCallback(
>   () => window.localStorage.setItem('count', count),
>   [count],
> )
> ```
>
> 🦉 A common question with this is: "Why don’t we just wrap every function in `useCallback`?" You can read about this in my blog post [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback).

Kent C Dodds's article linked above ([When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)) does a good job of explaining how using `useCallback` isn't always the best option. While `useCallback` is used for performance gains, `useCallback` comes with it's own costs which can in fact outweigh any benefits when it is used unnecessarily. Kent provides the following examples to demonstrate this point:

No `useCallback`:

```jsx
const dispense = candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}
```

With `useCallback`:

```jsx
const dispense = React.useCallback(candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy))
}, [])
```

In the above cases, the first example which doesn't use  `useCallback` is actually more performant. While `setCandies` and `filter` are run and `dispense` recreated on each rerender, this is less costly than the memoisation that comes into play when using `useCallback`.



