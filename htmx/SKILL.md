---
name: htmx-development
description: Build, refactor, and debug hypermedia-driven web applications using htmx. Trigger whenever htmx attributes (hx-*), response headers (HX-*), out-of-band swaps, server-rendered HTML fragments, or hypermedia API patterns are added, modified, or reviewed in a codebase.
---
# htmx Development Skill

Write idiomatic, accessible, and maintainable hypermedia-driven code using htmx. This skill enforces the Hypermedia As The Engine Of Application State (HATEOAS) architecture based on the official htmx documentation and reference.

## When to Use

- Adding interactive UI behaviors (search, inline editing, modals, infinite scroll, lazy loading, polling) using `hx-*` attributes.
- Writing backend endpoints that serve HTML fragments or respond to htmx requests.
- Coordinating multi-part UI updates using Out-of-Band (`hx-swap-oob`) swaps or `HX-Trigger` headers.
- Configuring progressive enhancement (`hx-boost`), client-side validation, request synchronization (`hx-sync`), or history support.
- Migrating or reviewing htmx 2.x and htmx 4.x implementations.

---

## Core Architectural Principles

1. **Return HTML, Not JSON**: Endpoints targeted by htmx must return rendered HTML fragments (partials), never JSON payloads to be parsed on the client. Keep state transitions and template rendering on the server.
2. **Use Semantic HTTP Verbs**:
   - `hx-get`: Safe, idempotent reads and filtering. Never mutate server state on a `GET` request.
   - `hx-post` / `hx-put` / `hx-patch` / `hx-delete`: State mutations.
3. **Respect Progressive Enhancement & Semantics**: Use real `<form>` elements for multi-input submissions and `<button>` or `<a>` elements for clickable actions rather than attaching `hx-post` to non-interactive `<div>` or `<span>` elements.
4. **Minimize Client-Side State**: Let the DOM represent current state. When an action affects multiple decoupled areas of the page, prefer event-driven updates (`HX-Trigger`) or Out-of-Band swaps (`hx-swap-oob`) over complex JavaScript glue code.

---

## Core Reference & Patterns

### 1. Issuing & Triggering Requests

- **HTTP Verbs**: `hx-get`, `hx-post`, `hx-put`, `hx-patch`, `hx-delete`
- **Natural Default Triggers** (omit `hx-trigger` when using defaults):
  - `<input>`, `<textarea>`, `<select>` -> triggers on `change`
  - `<form>` -> triggers on `submit`
  - Everything else -> triggers on `click`
- **Trigger Modifiers (`hx-trigger`)**:
  - `changed`: Only issue request if the input value actually changed.
  - `delay:<time>`: Debounce the request (e.g., `delay:300ms`); timer resets on rapid repeat events.
  - `throttle:<time>`: Rate-limit the request (e.g., `throttle:500ms`); fires once per interval.
  - `once`: Only trigger the request one time.
  - `from:<selector>`: Listen for the event on another DOM node (e.g., `from:body`, `from:window`).
  - `target:<selector>`: Filter events to those originating from a matching child selector.
  - `consume`: Stop event propagation (`stopPropagation()`).
  - `queue:<first|last|all|none>`: Control how concurrent events on the same element are queued.
- **Trigger Filters**: Place boolean JS expressions in square brackets right after the event name:
  `hx-trigger="keyup[key=='Enter']"` or `hx-trigger="click[ctrlKey]"`
- **Special Events**:
  - `load`: Fires once when the element is initially loaded into the DOM.
  - `revealed`: Fires once when the element scrolls into the viewport (ideal for infinite scroll).
  - `intersect`: Fires when the element intersects the viewport (supports `root:<selector>` and `threshold:<0.0-1.0>`).
  - `every <timing>`: Polls at fixed intervals (e.g., `hx-trigger="every 5s"`). Stop polling from the server by returning HTTP status **`286`**.

### 2. Targeting Elements (`hx-target`)

Specify where the returned HTML fragment will be placed. Defaults to the triggering element itself if omitted.

- Standard CSS selector: `#results-list`, `.notification-banner`
- `this`: Explicitly target the element bearing the `hx-target` attribute (useful when inherited).
- `closest <selector>`: Closest ancestor matching the selector (e.g., `closest tr`, `closest div.card`).
- `find <selector>`: First descendant matching the selector (e.g., `find .status-badge`).
- `next <selector>` / `previous <selector>`: Next or previous element in the DOM matching the selector.

### 3. Swapping Strategies (`hx-swap`)

Controls how the returned HTML replaces the target content. Defaults to `innerHTML`.

| Strategy | Behavior |
| :--- | :--- |
| `innerHTML` | Replaces the inner children of the target element (default). |
| `outerHTML` | Replaces the entire target element itself with the response. |
| `textContent` | Replaces text content without parsing HTML tags (safe for raw text). |
| `beforebegin` | Inserts response immediately before the target element (sibling). |
| `afterbegin` | Prepends response inside the target before its first child. |
| `beforeend` | Appends response inside the target after its last child. |
| `afterend` | Inserts response immediately after the target element (sibling). |
| `delete` | Deletes the target element after response completes, ignoring response content. |
| `none` | Does not append or swap content from response (OOB swaps and headers still process). |

**Swap Modifiers** (append to `hx-swap`, e.g., `hx-swap="innerHTML swap:100ms settle:200ms show:top"`):
- `transition:true`: Wrap the DOM update in the browser View Transitions API.
- `swap:<time>` / `settle:<time>`: Pause before swapping old content out or settling new attributes in.
- `scroll:top` / `scroll:bottom` / `show:top` / `show:bottom`: Control viewport or container scrolling (accepts selector prefix like `show:#header:top`).
- `focus-scroll:true|false`: Enable or suppress automatic scroll to focused inputs after swap.
- `ignoreTitle:true`: Prevent `<title>` tags in the response from updating `document.title`.

### 4. Selecting & Updating Multiple Elements

- **Response Filtering (`hx-select`)**: Extract only a subset of the returned HTML via a CSS selector (e.g., `hx-select="#table-container"`).
- **Out-of-Band Swaps (`hx-swap-oob` & `hx-select-oob`)**:
  Include extra top-level elements in the server's HTML response with `hx-swap-oob="true"` (or a specific swap strategy like `hx-swap-oob="beforeend:#notifications"`). Any element with a matching `id` in the current DOM will be updated simultaneously alongside the primary target.
  ```html
  <!-- Primary target response -->
  <div id="cart-items">...updated items...</div>

  <!-- Out-of-band update piggybacked in the same HTTP response -->
  <span id="cart-badge" hx-swap-oob="true">4</span>
  ```

### 5. Request Data, Forms & Synchronization

- **Forms**: Inside a `<form>`, any htmx request automatically serializes and submits all inputs in the form.
- **`hx-include`**: Include input values from outside the triggering element/form using extended selectors (e.g., `hx-include="#global-filter, closest tr"`).
- **`hx-vals`**: Append extra parameters as JSON (`hx-vals='{"source": "modal"}'`). Prefix with `js:` ONLY for trusted client expressions (`hx-vals='js:{"tz": Intl.DateTimeFormat().resolvedOptions().timeZone}'`).
- **`hx-params`**: Filter submitted parameters (`*` for all, `none` for none, `not <param1,param2>`, or `<param1,param2>`).
- **`hx-encoding="multipart/form-data"`**: Required on forms or elements that upload `<input type="file">`.
- **`hx-sync`**: Resolve race conditions between elements (e.g., an input's live validation vs. form submission):
  - `hx-sync="closest form:abort"`: Abort the input's in-flight request if the parent form submits.
  - `hx-sync="this:replace"`: Cancel any previous in-flight request from this element and replace it with the new one.
  - `hx-sync="this:drop"`: Ignore new triggers while a request is currently in-flight.

### 6. UX, Loading States & disabling Elements

- **`hx-indicator`**: Point to a spinner or container via CSS selector (`#spinner`). htmx adds the `htmx-request` class to the indicator while the request is in flight. Elements with class `.htmx-indicator` automatically transition from `opacity: 0` to `opacity: 1`.
- **`hx-disabled-elt`**: Automatically set the `disabled` attribute on target elements during an active request to prevent double-submissions (e.g., `hx-disabled-elt="this"` on a button, or `hx-disabled-elt="find button"` on a form).
- **`hx-confirm`**: Display a confirmation dialog before issuing the request. Hook into the `htmx:confirm` event (`e.preventDefault()`, then call `e.detail.issueRequest(true)`) to replace native `window.confirm()` with a custom modal.
- **`hx-preserve`**: Add `hx-preserve="true"` to elements with a stable `id` (such as `<video>`, `<audio>`, or third-party iframes) so they retain playback state and DOM identity across swaps.

---

## Server-Side Integration & HTTP Headers

Always inspect request headers on the backend to differentiate partial updates from full-page loads, and use response headers to instruct client behavior.

### Request Headers (Sent by htmx -> Read by Server)

| Header | Purpose |
| :--- | :--- |
| `HX-Request` | Always `"true"` for htmx requests. Use to return a partial template instead of the full layout. |
| `HX-Boosted` | `"true"` when the request comes from an element enhanced with `hx-boost="true"`. |
| `HX-Target` | The `id` of the target element on the client. |
| `HX-Trigger` | The `id` of the triggered element on the client. |
| `HX-Trigger-Name` | The `name` attribute of the triggered element. |
| `HX-Current-URL` | The current browser URL where the htmx action originated. |
| `HX-Prompt` | User text input entered when `hx-prompt` was used. |
| `HX-History-Restore-Request` | `"true"` when the request is restoring a missed history cache entry on back/forward navigation. |

**Important (`Vary` Header)**: Whenever a single URL returns a partial HTML fragment if `HX-Request: true` is present and a full HTML page otherwise, **always include `Vary: HX-Request`** (or `Vary: HX-Request, HX-Boosted`) in the HTTP response headers. Without this, browsers or CDNs will cache the partial fragment and serve raw unstyled fragments on full-page refreshes.

### Response Headers (Sent by Server -> Processed by htmx)

| Header | Purpose |
| :--- | :--- |
| `HX-Location` | Trigger a client-side htmx navigation (AJAX redirect) without a full browser reload. Can be a URL string or JSON `{"path": "/items", "target": "#main"}`. |
| `HX-Redirect` | Perform a hard browser-level redirect (`window.location`) to a new URL. |
| `HX-Refresh` | If `"true"`, forces a full client-side refresh of the current page. |
| `HX-Push-Url` | Pushes a new URL into `history.pushState` (or `"false"` to prevent pushing). |
| `HX-Replace-Url` | Replaces the current URL in the browser location bar via `history.replaceState`. |
| `HX-Retarget` | CSS selector that overrides `hx-target` on the client (vital for validation errors that need to swap the form instead of a list). |
| `HX-Reswap` | Overrides the `hx-swap` strategy for this response (e.g., `HX-Reswap: none` or `outerHTML`). |
| `HX-Reselect` | CSS selector that overrides `hx-select` for this response. |
| `HX-Trigger` | Fires client-side events immediately on response receipt (`HX-Trigger: itemSaved` or JSON `{"showToast": {"message": "Saved!", "level": "info"}}`). |
| `HX-Trigger-After-Swap` | Fires client-side events immediately after the DOM swap step completes. |
| `HX-Trigger-After-Settle` | Fires client-side events after the DOM settling phase completes. |

---

## Canonical Implementation Patterns

### Pattern 1: Active Search with Debounce & Race-Condition Protection
```html
<div class="search-header">
  <input type="search"
         name="q"
         placeholder="Search users..."
         hx-get="/users/search"
         hx-trigger="input changed delay:300ms, keyup[key=='Enter']"
         hx-target="#user-results"
         hx-indicator="#search-spinner"
         hx-sync="this:replace">
  <span id="search-spinner" class="htmx-indicator" aria-hidden="true">Loading...</span>
</div>
<tbody id="user-results">
  <!-- Server renders <tr> rows here -->
</tbody>
```

### Pattern 2: Form Submission with Validation & Error Retargeting
When creating an item that appends to a list on success (`200 OK`), but re-renders the form with field errors on failure (`422 Unprocessable Entity`):
```html
<form id="create-todo-form"
      hx-post="/todos"
      hx-target="#todo-list"
      hx-swap="beforeend"
      hx-disabled-elt="find button[type='submit']"
      hx-on::after-request="if(event.detail.successful) this.reset()">
  <input type="text" name="title" required>
  <button type="submit">Add Todo</button>
</form>
<ul id="todo-list">...</ul>
```
**Server Response on Validation Error**:
- Return status `200` (or `422` if `responseHandling` is configured to swap `422`) with headers:
  - `HX-Retarget: #create-todo-form`
  - `HX-Reswap: outerHTML`
- Body: The re-rendered `<form id="create-todo-form">...</form>` containing inline error messages and preserved user inputs.

### Pattern 3: Decoupled Updates via Server-Triggered Events
Avoid hardcoding complex OOB selectors when multiple independent components depend on state changes:
1. Server mutation endpoint (`POST /items`) returns header: `HX-Trigger: itemsUpdated`
2. Any component on the page listens for that event on `body`:
```html
<div id="metrics-panel"
     hx-get="/metrics/summary"
     hx-trigger="itemsUpdated from:body">
  <!-- Refreshes automatically whenever any endpoint emits HX-Trigger: itemsUpdated -->
</div>
```

---

## Security & Gotchas

1. **XSS & Untrusted HTML (`hx-disable`)**:
   htmx parses and executes HTML returned from the server. Always escape user-supplied content in backend templates. When rendering user-generated HTML that should *never* have htmx attributes interpreted, wrap it in a container with `hx-disable`:
   ```html
   <div hx-disable>
     {{ raw_user_comment_html }}
   </div>
   ```
2. **CSRF Token Injection**:
   Include CSRF tokens on state-changing requests (`POST`, `PUT`, `PATCH`, `DELETE`) globally either via `<body hx-headers='{"X-CSRF-Token": "TOKEN_VALUE"}'>` or an event listener on `document.body`:
   ```javascript
   document.body.addEventListener('htmx:configRequest', (event) => {
     event.detail.headers['X-CSRF-Token'] = document.querySelector('meta[name="csrf-token"]')?.content;
   });
   ```
3. **4xx and 5xx Error Swapping**:
   By default in **htmx 2.x**, HTTP `4xx` and `5xx` responses do **not** swap content into the DOM. If your server returns `422 Unprocessable Entity` with validation HTML, either configure `htmx.config.responseHandling` in `<meta name="htmx-config">`, handle it in `htmx:beforeSwap` (`event.detail.shouldSwap = true`), or return `200 OK` with `HX-Retarget` / `HX-Reswap`.
4. **Attribute Inheritance Pitfalls**:
   In htmx 2.x, attributes like `hx-target`, `hx-swap`, `hx-confirm`, `hx-boost`, and `hx-include` are inherited by child elements automatically. Placing `hx-target="#main"` or `hx-confirm="Are you sure?"` on a parent container will unexpectedly apply to every child button or link inside it unless overridden or explicitly stopped with `hx-disinherit="*"` (or `hx-disinherit="hx-confirm"`).
   *(Note: If the codebase uses **htmx 4.x**, implicit inheritance is disabled by default and requires explicit `:inherited` modifiers such as `hx-target:inherited="#main"` unless `htmx.config.implicitInheritance = true` is set).*
5. **CSS Transitions Require Stable IDs**:
   For CSS transitions to animate smoothly across swaps, the element being modified must keep the **exact same `id`** in both the old DOM and the newly swapped HTML response, so htmx can transition its `class` or `style` during the settling phase.
6. **Dynamic Third-Party DOM Initialization**:
   Never initialize JS widgets strictly on `DOMContentLoaded` if they can be loaded via htmx swaps. Always initialize inside `htmx.onLoad(function(content) { ... })` scoped to the newly swapped `content` fragment, and clean up instances on `htmx:beforeCleanupElement`.
7. **Inline Event Handling (`hx-on`)**:
   In htmx 2.x+, use the double-colon syntax for htmx lifecycle events (`hx-on::before-request="..."`, `hx-on::after-request="..."`) and single-colon for standard DOM events (`hx-on:click="..."`). Avoid deprecated `hx-on="htmx:afterRequest: ..."`.

