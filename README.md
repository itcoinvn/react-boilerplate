# How to use TypeScript with React 18 ?

## **Creating a React app with TypeScript**

Let’s create ourselves a vanilla React TypeScript app with Create React App:

```bash
yarn create react-app my-app --template typescript
```

Now let’s upgrade the version of React to `@next`:

```bash
yarn add react@next react-dom@next
```

This will leave you with entries in the `package.json` that use React 18. It will likely look something like this:

```json
{
    "dependencies": {
            "react": "^18.0.0",
            "react-dom": "^18.0.0",
        }
}
```

If we run `yarn start`, we’ll find ourselves running a React 18 app

## **Using the new APIs**

So let’s try using the [ReactDOM.createRoot](https://github.com/reactwg/react-18/discussions/5) API. It’s this API that opts our application in to using React 18’s new features. We’ll open up `index.tsx` and make this change:

```javascript
-ReactDOM.render(
-  <React.StrictMode>
-    <App />
-  </React.StrictMode>,
-  document.getElementById('root')
-);

+const root = ReactDOM.createRoot(document.getElementById('root'));
+
+root.render(
+  <React.StrictMode>
+    <App />
+  </React.StrictMode>
+);
```

If we were running JavaScript alone, this would work. Because we’re using TypeScript as well, however, we’re now confronted with an error:

{% hint style="info" %}
_Property 'createRoot' does not exist on type 'typeof import("/code/my-app/node\_modules/@types/react-dom/index")'. TS2339_
{% endhint %}

This is the TypeScript compiler complaining that it doesn’t know anything about `ReactDOM.createRoot`. This is because the type definitions that are currently in place in our application don’t have that API defined.

Let’s upgrade our type definitions:

```bash
yarn add @types/react @types/react-dom
```

We might reasonably hope that everything should work now — alas, it does not. The same error is presenting. TypeScript is not happy.

## **Telling TypeScript about the new APIs**

If we take a look at the [PR that added support for the APIs](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/53685), we’ll find some tips. If you look at one for [next.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/a07e9cfb005682fb6be0a2e85113eac131c3006f/types/react/next.d.ts), you’ll find this info, courtesy of [Sebastian Silbermann](https://twitter.com/sebsilbermann): __&#x20;

````javascript
/**
 * These are types for things that are present in the upcoming React 18 release.
 *
 * Once React 18 is released they can just be moved to the main index file.
 *
 * To load the types declared here in an actual project, there are three ways. The easiest one,
 * if your `tsconfig.json` already has a `"types"` array in the `"compilerOptions"` section,
 * is to add `"react/next"` to the `"types"` array.
 *
 * Alternatively, a specific import syntax can to be used from a typescript file.
 * This module does not exist in reality, which is why the {} is important:
 *
 * ```ts
 * import {} from 'react/next'
 * ```
 *
 * It is also possible to include it through a triple-slash reference:
 *
 * ```ts
 * /// <reference types="react/next" />
 * ```
 *
 * Either the import or the reference only needs to appear once, anywhere in the project.
 */
````

et’s try the first item on the list. We’ll edit our `tsconfig.json` and add a new entry to the `"compilerOptions"` section:

```json
"types": ["react/next", "react-dom/next"]
```

If we restart our build with `yarn start`, we’re now presented with a different error:

{% hint style="info" %}
Argument of type 'HTMLElement | null' is not assignable to parameter of type 'Element | Document | DocumentFragment | Comment'.\_
{% endhint %}

{% hint style="info" %}
Type 'null' is not assignable to type 'Element | Document | DocumentFragment | Comment'. TS2345\_
{% endhint %}

Now this actually has nothing to do with the issues with our new React type definitions. They are fine. This is TypeScript saying, “It’s not guaranteed that `document.getElementById('root')` returns something that is not `null`. Since we’re in `strictNullChecks` mode, you need to be sure `root` is not null.”

We’ll deal with that by testing whether we do have an element in play before invoking `ReactDOM.createRoot`:

```javascript
import {createRoot} from 'react-dom/client';

const rootElement = document.getElementById('root');
if (!rootElement) throw new Error('Failed to find the root element');

const root = createRoot(rootElement);

root.render(
    <React.StrictMode>
     <App />
    </React.StrictMode>
);
```

Good luck!
