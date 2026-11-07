# experimental-runtime

> **Research prototype (2022-era):** This repository explores in-place JavaScript re-execution in an iframe to preserve state during code edits. It is **not** the current [VizHub runtime library](https://github.com/vizhub-studio/vizhub-runtime) and makes no claim of production-grade isolation for untrusted code.

## Reproduce the experiment

The root `package.json` defines Rollup, Mocha and a local static server:

```bash
npm install
npm run build
npm test         # Rebuilds, then runs Mocha
npm run dev      # Starts local server and Rollup watcher
```

- [`src/`](src/) contains the implementation and [`test/`](test/) the checks.
- [`public/`](public/) contains the demo assets served by the development script.
- Re-execution assumes **idempotent rendering and cleanup**; persistent DOM and application state can otherwise accumulate side effects.
- The iframe mechanism described below is an experiment, not a security specification. Untrusted code requires an independently reviewed sandbox origin, permissions and embedding policy.
- This project has an older development toolchain; record any dependency/environment changes separately rather than assuming the examples run unchanged on current Node.js.

## Original experiment notes


A runtime environment for in-browser live coding using Web technologies.

The fundamental idea behind this project is that JavaScript is executed in an iframe when the code is changed, _without_ re-evaluating the entire iframe. This is similar to [hot reloading](https://stackoverflow.com/questions/41428954/what-is-the-difference-between-hot-reloading-and-live-reloading-in-react-native) and [hot module replacement](https://webpack.js.org/concepts/hot-module-replacement/). This approach has a number of advantages over re-evaluating the entire iframe (setting `srcdoc`) on each code change, including:

- Application state can remain in memory between re-executions of JavaScript.
- DOM can remain in memory between re-executions of JavaScript.
- JavaScript libraries need only be loaded and parsed when dependencies change.
- It opens the door to dynamically injected state and CSS.

The main disadvantage of this approach is that one must be sure to always write idempotent rendering logic. The `main` function will be executed again and again within the same iframe DOM context, when the code changes and also when the state changes. The code must account for this and clean up any DOM elements created "the last time around" rather than simply appending new DOM elements every time. With React, idempotent rendering logic is baked in. With D3, patterns like `g.append('text')` must be updated to use an idempotent pattern like `g.selectAll('text.x-axis-label').data([null]).join('text').attr('class', 'x-axis-label')`.

Whatever editors use this runtime may want to consider an escape hatch to allow users to force re-evaluation of the iframe. Users may opt to use this as a sort of "master reset button" if they mess up the state of the DOM or have a memory leak or something otherwise goes haywire/fubar.

Inspired by:

- [Bret Victor - Inventing on Principle](https://www.youtube.com/watch?v=PUv66718DII)
- [Dan Abramov - Live React: Hot Reloading with Time Travel](https://www.youtube.com/watch?v=xsSnOQynTHs)

Related work:

- [GitHub: yangsu/inventing-on-principle](https://github.com/yangsu/inventing-on-principle)
