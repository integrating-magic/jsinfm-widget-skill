---
name: jsinfm-widget
description: Scaffolds and builds self-contained JavaScript or React widgets that run inside a FileMaker web viewer — charts, tables, formatted displays, or full React+TanStack Query widgets that fetch live FileMaker data via FMGofer and send events back with WV Event. Use whenever the user wants to build, practice, or debug a FileMaker web-viewer JS widget, a FileMaker JS/HTML widget, or otherwise write JavaScript that runs inside FileMaker.
---

# FM Widget Builder

You are helping a FileMaker developer build a self-contained JavaScript widget that runs inside a FileMaker web viewer. The developer may not know React, JavaScript, or modern web tooling — for most builds you handle all of that automatically. (The one exception is **practice mode** below, where the point is for the user to write the JS themselves.)

## First: which kind of build? (always ask this before anything else)

Before any intake or scaffolding, ask the user which of these three they want. Their answer routes the entire rest of this skill:

1. **Simple widget** — a small, mostly self-contained widget (a chart, a table, a formatted display). Often built best with a traditional JS library like **ApexCharts** or **DataTables** rather than React. → **Mode 1**.
2. **Practice / learning** — the user wants to practice a JavaScript concept, or practice using JS inside FileMaker — not ship a production widget. → **Mode 2**.
3. **Complex, data-fetching widget** — a reactive widget that fetches live FileMaker data, manages state, and sends events back. This is the full React + TanStack Query path. → **Mode 3** (the bulk of this document, starting at "What you need from the user").

If the user is unsure, ask a clarifying question or two about what they're trying to accomplish, then recommend a mode. Modes 1 and 2 share the same lightweight vanilla-JS environment; only Mode 3 uses the React scaffold.

## FileMaker script constants (always true — never ask)

These two FileMaker script names and their parameter shapes are fixed conventions. Never ask the user for them and never invent alternatives:

- **`Fetch Data`** — the script the widget calls to **read** data. Always invoked with a plain object carrying two properties: **`type`** (identifies the calling widget, since the script is shared) and **`query`** (the search/filter criteria; omit or leave empty for load-all). The config fetch is the one variant: `{ type: "load" }`.
- **`WV Event`** — the script the widget calls whenever a **click runs a FileMaker script** (or otherwise sends something back). Always invoked with an object carrying two properties: **`event`** (what happened) and **`data`** (the payload). FileMaker branches on `event`.

## Modes 1 & 2 — the lightweight environment

Both the simple-widget and practice modes use the **vanilla-JS** environment (Vite, no React, no TanStack Query):

```
https://github.com/integrating-magic/js-dev-environment-new.git
```

Setup:
```
git clone https://github.com/integrating-magic/js-dev-environment-new.git <widgetName>
rm -rf <widgetName>/.git
cd <widgetName> && npm install
```

Then edit the **values** in `widget.config.cjs`. ⚠️ This environment uses **CommonJS** (`module.exports`), unlike the React scaffold's ESM `export default` — keep it CommonJS, don't convert it:
```js
module.exports = {
  widgetName: "<widgetName>",
  server: "$",
  file: "<FileMakerFileName>",
  uploadScript: "UploadToHTML",
};
```

Files you'll work in: `index.html` (entry), `src/index.js` (your code), `src/style.css`, and `src/sampleData.js` (put any sample data here). Install any libraries you need with `npm install <pkg>` (e.g. `apexcharts`, `datatables.net`). The npm scripts match Mode 3's names — `npm start` (dev server at http://localhost:1234/), `npm run deploy-to-fm`, `npm run upload`, `npm run generate-script-steps`.

## Mode 1 — Simple widget

1. Ask the user to describe what they want to build — what it should display and how it should behave.
2. From their description, **suggest a JS library when one fits**, and say briefly why:
   - **ApexCharts** — charts/graphs (line, bar, donut, sparkline, etc.)
   - **DataTables** — sortable / searchable / paginated tables
   - plain DOM / vanilla JS when the widget is simple enough not to need a library
   Each library has its own setup; install it and follow its init pattern.
3. Scaffold the lightweight environment (above), install the chosen library, and build the widget in `src/index.js` + `src/style.css`.
4. Provide useful **sample data** in `src/sampleData.js` so the widget renders immediately during development.
5. If the widget needs live FileMaker data instead of sample data, you can still use the `Fetch Data` / `WV Event` conventions (see "FileMaker script constants" above) — but keep the calls simple; there is no TanStack Query here. Use `FMGofer.PerformScript("Fetch Data", { type: "<widgetName>" }).json()` directly (install `fm-gofer` if needed).

## Mode 2 — Practice / learning

The goal is for the user to **learn**, not for you to hand them a finished solution.

1. Ask what they want to practice. If they're unsure, **suggest a JS concept** suited to their level — e.g. array methods (`map`/`filter`/`reduce`), `fetch` / promises / `async-await`, closures, event handling, DOM manipulation, `localStorage`, ES modules — **or** practicing **JS inside FileMaker** specifically (calling FileMaker scripts from JS, reading script results, the web-viewer ↔ FileMaker bridge, `FileMaker.PerformScript` / FMGofer).
2. For FileMaker-specific practice, **look up current examples and docs** (web search / fetch) so your guidance is accurate, and cite what you find.
3. Scaffold the lightweight environment and set it up to support the exercise: a clear starting point in `src/index.js`, any needed library installed, and **sample data in `src/sampleData.js`** to work against.
4. **Do not solve the exercise for them.** Provide scaffolding, `// TODO` markers, hints, and an explanation of the concept — but leave the actual JS for the user to write. Offer to review their attempt or nudge them when they get stuck.

---

## Mode 3 — Complex data-fetching widget (React + TanStack Query)

Everything from here down is the Mode 3 path.

## What you need from the user before starting

If any of these are missing, ask for them before writing any code:

1. **What kind of widget** — ask the user to describe the widget they want to build: what it should display and how it should behave.
2. **FileMaker file name** — the `.fmp12` file the widget will live in (no extension needed)
3. **Widget name** — a short camelCase name (e.g. `coursesList`, `invoiceDetail`)
4. **Sample JSON** — the output the `Fetch Data` script returns for this widget. Ask them to paste it directly into the chat.
5. **Data scope** — ask whether the widget needs to load **all** the data or just **some** of it.
   - If just some, ask **what the widget will send to FileMaker in the `query` property** — i.e. the search/filter parameter(s) the widget passes to `Fetch Data` to narrow down which records get returned.
6. **Boot mode** — ask whether this is a **self-booting** widget or a **load-by-script** widget:
   - **Self-booting** → `index.jsx` renders the app immediately on load. If the widget needs a config (question 7), it fetches it from FileMaker via a query with `{ type: "load" }` — **separate from the data fetch**.
   - **Load-by-script** → keep the `loadApp` function on `window` so FileMaker invokes it via a script step, passing the config in.
7. **Config** — ask whether the widget needs a config object brought in from FileMaker (settings, labels, IDs, colors, environment values, etc.). If yes, ask **what properties the config contains**. If no config is needed, skip all config plumbing — no `useConfig()` hook, no `{ type: "load" }` fetch, no `config` prop.
8. **WV Event behavior** — only relevant if the widget sends anything back to FileMaker (a click that runs a script, a checkout, a form submission, a selection). Before asking the technical question, interrogate the user about what the widget should *do* after it fires the event. Ask:
   - *"After the widget sends an event to FileMaker, what should happen next in the widget?"* — e.g. clear the cart, show a success message, navigate to a different view, reload the data, close the web viewer, do nothing.
   - Use their answer to figure out whether widget state needs to update, and whether FM needs to send anything back.
   - Then ask: *"What does the `WV Event` script need to do: fire the event only, or fire the event and wait for a response from FileMaker?"*
     - **Fire only** → use `FileMaker.PerformScript("WV Event", JSON.stringify({ event, data }))` — fire and forget, no await needed. Handle any widget state updates (clear cart, show confirmation, etc.) immediately in JS without waiting for FM.
     - **Fire and wait** → use `await FMGofer.PerformScript("WV Event", { event, data }).json()` — waits for FileMaker to finish and return a result. Use the response to drive the next widget state (e.g. show a confirmation number FM generated, reload with updated data).
   - **Prompt the user if they haven't thought it through** — if they say "just send the data," ask: *"Should the widget give the user any feedback after that — like clearing the form, showing a success message, or resetting to a fresh state?"* Many users don't realize the widget needs to handle its own post-send state.

## The stack (Mode 3 — React + TanStack Query)

Mode 3 widgets always use **React with TanStack Query** for data fetching. (Modes 1 and 2 use the vanilla-JS environment instead — see above.) Always scaffold from:

```
https://github.com/integrating-magic/js-dev-react.git
```

This scaffold already includes `@tanstack/react-query` and `fm-gofer` as dependencies, so `npm install` brings them in — no separate install step.

## Setup steps (do these first, in order)

1. Clone the scaffold (skip if the project already exists locally), then remove its git history so the widget starts fresh:
   ```
   git clone https://github.com/integrating-magic/js-dev-react.git <widgetName>
   rm -rf <widgetName>/.git
   ```

2. Edit the **values** in `widget.config.js` (the scaffold already has it in the right ESM `export default` shape — just change the contents, don't rewrite the structure):
   ```js
   export default {
     widgetName: "<widgetName>",
     server: "$",
     file: "<FileMakerFileName>",
     uploadScript: "UploadToHTML",
   };
   ```
   Leave `scripts/upload.js` as-is — the scaffold ships a correct ESM version that reads this config. Do **not** rewrite it, do **not** switch the config to `module.exports`/`require()` (the package is ESM and they will throw), and do **not** create a `.cjs` copy; one config file only.

3. Run `npm install` — this pulls in `@tanstack/react-query` and `fm-gofer` along with the rest, since they're already in the scaffold's `package.json`. (If you cloned an older copy that predates that, run `npm install @tanstack/react-query fm-gofer` to add them.)

## Architecture rules (always follow these)

- **Config is opt-in** (intake question 7) — only build config plumbing if the developer said the widget needs a config. The config object carries widget settings, not the main record data; the record data always comes reactively via TanStack Query + FMGofer. When a config is needed, delivery depends on boot mode (intake question 6):
  - **Self-booting** → FileMaker never calls `loadApp`. `index.jsx` renders the app immediately, and the widget fetches its own config with `FMGofer.PerformScript("Fetch Data", { type: "load" }).json()` — the same `Fetch Data` script as the data fetch, which branches on `type: "load"` to return the config object. This is a **separate query from the data fetch**.
  - **Load-by-script** → expose `loadApp` on `window`; FileMaker calls it via a script step with the config JSON, which is parsed and passed to `<App config={config} />`.
- **Always use real FM fetch** — the widget runs inside FileMaker's web viewer even during development, so FMGofer works. Do not add dev-mode sample data fallbacks in the queryFn.
- **FMGofer call pattern**: `FMGofer.PerformScript("Fetch Data", param).json()` — the `.json()` parses the result. `param` is a **plain object**, passed as-is. Do **not** `JSON.stringify` it first: FMGofer serializes it and nests it under the `"parameter"` key of its envelope. Pre-stringifying double-encodes it, so the FM script's `JSONGetElement(...; "type")` reads an empty string.
- **The fetch param always includes a `type`** — set `type` to the widget name (intake question 3). `Fetch Data` is shared across widgets, so it branches on `type` to know which widget is calling and what to return. Put the search/filter criteria (from the **Data scope** answer) in the `query` property alongside it:
  - Load-all: `{ type: "<widgetName>" }`
  - Subset: `{ type: "<widgetName>", query: { ... } }`
  - The one exception is the self-booting config fetch (only when a config is needed — intake question 7), which sends `{ type: "load" }`.
- **Wrap the app in QueryClientProvider** in `index.jsx`
- **Title renders immediately** — show skeleton pulse cards while loading, an error callout on failure, then the real data

## File structure to create

```
src/
  index.jsx              # Defines loadApp(config); self-boots or exposes on window (boot mode)
  App.jsx                # Receives config prop. Layout: title, loading skeletons, error, data list
  style.css              # @import "tailwindcss" (already present)
  sampleData.js          # Shape reference only — not wired into queryFn
  hooks/
    useConfig.js         # Only if config needed (q8) + self-booting: { type: "load" } fetch
    use<Entity>.js       # TanStack Query + FMGofer hook
  components/
    <Entity>Card.jsx     # Card component for each record
```

## Hook pattern

```js
import { useQuery } from "@tanstack/react-query";
import FMGofer from "fm-gofer";

// `type` identifies this widget to the shared `Fetch Data` script.
const TYPE = "<widgetName>";

// Load-all version:
export function use<Entity>() {
  return useQuery({
    queryKey: ["<entity>"],
    queryFn: () =>
      FMGofer.PerformScript("Fetch Data", { type: TYPE }).json(),
  });
}

// Search/subset version — pass the criteria in the `query` property:
export function use<Entity>(query) {
  return useQuery({
    queryKey: ["<entity>", query],
    queryFn: () =>
      FMGofer.PerformScript("Fetch Data", { type: TYPE, query }).json(),
  });
}
```

**Config hook** — build this **only** for self-booting widgets that need a config (intake question 7); skip it entirely otherwise. Same `Fetch Data` script, but `type: "load"` returns the config object. Separate query from the data fetch:

```js
// hooks/useConfig.js
export function useConfig() {
  return useQuery({
    queryKey: ["config"],
    queryFn: () =>
      FMGofer.PerformScript("Fetch Data", { type: "load" }).json(),
  });
}
```

## index.jsx pattern

**Self-booting** — render immediately; there is no config prop. If a config is needed (intake question 7), App fetches it via `useConfig()`:

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
const queryClient = new QueryClient();
const root = createRoot(document.getElementById("root"));

root.render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
);
```

In `App.jsx`, call the entity hook for the records. If the widget needs a config (intake question 7), also call `useConfig()` — two separate queries; if the data query needs values from the config (e.g. search criteria), gate it with `enabled` until the config arrives. If no config is needed, `App.jsx` only has the data query. Either way, render the title immediately and skeletons while queries are pending.

**Load-by-script** — `loadApp(config)` receives the config object from FileMaker and renders the app with it:

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
const queryClient = new QueryClient();
const root = createRoot(document.getElementById("root"));

function loadApp(config) {
  root.render(
    <QueryClientProvider client={queryClient}>
      <App config={config} />
    </QueryClientProvider>
  );
}

window.loadApp = loadApp;
```

Use only the variant that matches the chosen boot mode (intake question 6) — note `App`'s props differ between them.

## Styling

Tailwind v4 is already configured. `src/style.css` only needs `@import "tailwindcss"`. No config file needed.

## Deploy commands

| Command | What it does |
|---|---|
| `npm start` | Dev server at http://localhost:1234/ |
| `npm run deploy-to-fm` | Build + upload to FileMaker |
| `npm run upload` | Upload only, no rebuild |
| `npm run generate-script-steps` | Outputs FM script steps to paste into FileMaker |

## Sending data back to FileMaker

When a click runs a FileMaker script — or the widget otherwise needs to send something back (a checkout, a form submission, a selection) — always use the script name **`WV Event`**, called with an object of **`event`** and **`data`**. `event` names what happened; `data` is the payload. The call pattern depends on whether the widget needs to wait for a response (intake question 8):

**Fire only** — fire and forget, no response expected:
```js
FileMaker.PerformScript("WV Event", JSON.stringify({
  event: "<eventName>",
  data: { /* whatever the widget is sending */ },
}));
```

**Fire and wait** — awaits a result back from FileMaker:
```js
const result = await FMGofer.PerformScript("WV Event", {
  event: "<eventName>",
  data: { /* whatever the widget is sending */ },
}).json();
```

In FileMaker, the `WV Event` script reads `Get(ScriptParameter)`, branches on `event` to know what the widget did, then pulls from `data` with `JSONGetElement`.

## Feature discussion rule

When the sample JSON reveals fields or portal data beyond what the user explicitly asked to display, **surface 2-3 feature ideas and ask before building**. Do not implement features speculatively.

## What the user needs to do in FileMaker

The `Fetch Data` script must exit with a JSON script result that is a **plain array** of record objects:

```json
[
  {
    "recordId": "1",
    "fieldData": { "FieldName": "value" },
    "portalData": {
      "PortalName": [{ "Portal::Field": "value" }]
    }
  }
]
```

FMGofer's `.json()` returns this array directly — access it as `data ?? []`, **not** `data?.data ?? []`.

Run `npm run generate-script-steps` to get the exact FileMaker script steps to build this.
