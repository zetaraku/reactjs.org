---
title: hydrateRoot
---

<Intro>

`hydrateRoot` lets you display React components inside a browser DOM node whose HTML content was previously generated by [`react-dom/server`.](/apis/react-dom/server)

```js
const root = hydrateRoot(domNode, reactNode, options?)
```

</Intro>

<InlineToc />

---

## Usage {/*usage*/}

### Hydrating server-rendered HTML {/*hydrating-server-rendered-html*/}

If your app's HTML was generated by [`react-dom/server`](/apis/react-dom/client/createRoot), you need to *hydrate* it on the client.

```js [[1, 3, "document.getElementById('root')"], [2, 3, "<App />"]]
import {hydrateRoot} from 'react-dom/client';

hydrateRoot(document.getElementById('root'), <App />);
````

This will hydrate the server HTML inside the <CodeStep step={1}>browser DOM node</CodeStep> with the <CodeStep step={2}>React component</CodeStep> for your app. Usually, you will do it once at startup. If you use a framework, it might do this behind the scenes for you.

To hydrate your app, React will "attach" your components' logic to the initial generated HTML from the server. Hydration turns the initial HTML snapshot from the server into a fully interactive app that runs in the browser.

<Sandpack>

```html public/index.html
<!--
  HTML content inside <div id="root">...</div>
  was generated from App by react-dom/server.
-->
<div id="root"><h1>Hello, world!</h1><button>You clicked me <!-- -->0<!-- --> times</button></div>
```

```js index.js active
import './styles.css';
import {hydrateRoot} from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document.getElementById('root'),
  <App />
);
```

```js App.js
import { useState } from 'react';

export default function App() {
  return (
    <>
      <h1>Hello, world!</h1>
      <Counter />
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      You clicked me {count} times
    </button>
  );
}
```

</Sandpack>

You shouldn't need to call `hydrateRoot` again or to call it in more places. From this point on, React will be managing the DOM of your application. If you want to update the UI, your components can do this by [using state.](/apis/react/useState)

<Pitfall>

The React tree you pass to `hydrateRoot` needs to produce **the same output** as it did on the server.

This is important for the user experience. The user will spend some time looking at the server-generated HTML before your JavaScript code loads. Server rendering creates an illusion that the app loads faster by showing the HTML snapshot of its output. Suddenly showing different content breaks that illusion. This is why the server render output must match the initial render output on the client during hydration.

The most common causes leading to hydration errors include:

* Extra whitespace (like newlines) around the React-generated HTML inside the root node.
* Using checks like `typeof window !== 'undefined'` in your rendering logic.
* Using browser-only APIs like [`window.matchMedia`](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia) in your rendering logic.
* Rendering different data on the server and the client.

React can recover from some hydration errors, but **you must fix them like other bugs.** In the best case, they'll lead to a slower app; in the worst case, event handlers would get attached to the wrong elements.

</Pitfall>

---

### Hydrating an entire document {/*hydrating-an-entire-document*/}

Apps fully built with React can render the entire document from the root component, including the [`<html>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html) tag:

```js {3,10}
function App() {
  return (
    <html>
      <head>
        <title>My app</title>
      </head>
      <body>
        <Page />
      </body>
    </html>
  );
}
```

To hydrate the entire document, pass the [`document`](https://developer.mozilla.org/en-US/docs/Web/API/Window/document) global as the first argument to `hydrateRoot`:

```js {5}
import {hydrateRoot} from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document,
  <App />
);
```

---

### Updating a hydrated root component {/*updating-a-hydrated-root-component*/}

After the root has finished hydrating, you can call [`root.render`](#root-render) to update the root React component. **Unlike with [`createRoot`](/apis/react-dom/client/createRoot), you don't usually need to do this because the initial content was already rendered as HTML.**

If you call `root.render` at some point after hydration, and the component tree structure matches up with what was previously rendered, React will [preserve the state.](/learn/preserving-and-resetting-state) Notice how you can type in the input, which means that the updates from repeated `render` calls every second in this example are not destructive:

<Sandpack>

```html public/index.html
<!--
  All HTML content inside <div id="root">...</div> was
  generated by rendering <App /> with react-dom/server.
-->
<div id="root"><h1>Hello, world! <!-- -->0</h1><input placeholder="Type something here"/></div>
```

```js index.js active
import {hydrateRoot} from 'react-dom/client';
import './styles.css';
import App from './App.js';

const root = hydrateRoot(
  document.getElementById('root'),
  <App counter={0} />
);

let i = 0;
setInterval(() => {
  root.render(<App counter={i} />);
  i++;
}, 1000);
```

```js App.js
export default function App({counter}) {
  return (
    <>
      <h1>Hello, world! {counter}</h1>
      <input placeholder="Type something here" />
    </>
  );
}
```

</Sandpack>

It is uncommon to call [`root.render`](#root-render) on a hydrated root. Usually, you'll [update state](/apis/react/useState) inside one of the components instead.


---
## Reference {/*reference*/}

### `hydrateRoot(domNode, options?)` {/*hydrate-root*/}

Call `hydrateRoot` to “attach” React to existing HTML that was already rendered by React in a server environment.

```js
const domNode = document.getElementById('root');
const root = hydrateRoot(domNode, reactNode);
```

React will attach to the HTML that exists inside the `domNode`, and take over managing the DOM inside it. An app fully built with React will usually only have one `hydrateRoot` call with its root component.

[See examples above.](#usage)

#### Parameters {/*parameters*/}

* `domNode`: A [DOM element](https://developer.mozilla.org/en-US/docs/Web/API/Element) that was rendered as the root element on the server.

* `reactNode`: The "React node" used to render the existing HTML. This will usually be a piece of JSX like `<App />` which was rendered with a `ReactDOM Server` method such as `renderToPipeableStream(<App />)`.

* **optional** `options`: A object contain options for this React root.

  * `onRecoverableError`: optional callback called when React automatically recovers from errors.
  * `identifierPrefix`: optional prefix React uses for IDs generated by [`useId`.](/apis/react/useId) Useful to avoid conflicts when using multiple roots on the same page. Must be the same prefix as used on the server.

#### Returns {/*returns*/}

`hydrateRoot` returns an object with two methods: [`render`](#root-render) and [`unmount`.](#root-unmount)

#### Caveats {/*caveats*/}

* `hydrateRoot()` expects the rendered content to be identical with the server-rendered content. You should treat mismatches as bugs and fix them.
* In development mode, React warns about mismatches during hydration. There are no guarantees that attribute differences will be patched up in case of mismatches. This is important for performance reasons because in most apps, mismatches are rare, and so validating all markup would be prohibitively expensive.
* You'll likely have only one `hydrateRoot` call in your app. If you use a framework, it might do this call for you.
* If your app is client-rendered with no HTML rendered already, using `hydrateRoot()` is not supported. Use [`createRoot()`](/apis/react-dom/client/createRoot) instead.

---

### `root.render(reactNode)` {/*root-render*/}

Call `root.render` to update a React component inside a hydrated React root for a browser DOM element.

```js
root.render(<App />);
```

React will update `<App />` in the hydrated `root`.

[See examples above.](#usage)

#### Parameters {/*root-render-parameters*/}

* `reactNode`: A "React node" that you want to update. This will usually be a piece of JSX like `<App />`, but you can also pass a React element constructed with [`createElement()`](/apis/react/createElement), a string, a number, `null`, or `undefined`.


#### Returns {/*root-render-returns*/}

`root.render` returns `undefined`.

#### Caveats {/*root-render-caveats*/}

* If you call `root.render` before the root has finished hydrating, React will clear the existing server-rendered HTML content and switch the entire root to client rendering.

---

### `root.unmount()` {/*root-unmount*/}

Call `root.unmount` to destroy a rendered tree inside a React root.

```js
root.unmount();
```

An app fully built with React will usually not have any calls to `root.unmount`.

This is mostly useful if your React root's DOM node (or any of its ancestors) may get removed from the DOM by some other code. For example, imagine a jQuery tab panel that removes inactive tabs from the DOM. If a tab gets removed, everything inside it (including the React roots inside) would get removed from the DOM as well. In that case, you need to tell React to "stop" managing the removed root's content by calling `root.unmount`. Otherwise, the components inside the removed root won't know to clean up and free up global resources like subscriptions.

Calling `root.unmount` will unmount all the components in the root and "detach" React from the root DOM node, including removing any event handlers or state in the tree. 


#### Parameters {/*root-unmount-parameters*/}

`root.unmount` does not accept any parameters.


#### Returns {/*root-unmount-returns*/}

`render` returns `null`.

#### Caveats {/*root-unmount-caveats*/}

* Calling `root.unmount` will unmount all the components in the tree and "detach" React from the root DOM node.

* Once you call `root.unmount` you cannot call `root.render` again on the root. Attempting to call `root.render` on an unmounted root will throw a "Cannot update an unmounted root" error.
