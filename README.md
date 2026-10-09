# inertiax-viewstack

Stacked navigation for [InertiaX](https://github.com/buhrmi/inertiax), driven by browser history.

[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![svelte](https://img.shields.io/badge/svelte-%5E5.0-ff3e00.svg)](https://svelte.dev)
[![built for InertiaX](https://img.shields.io/badge/built%20for-InertiaX-9553e9.svg)](https://github.com/buhrmi/inertiax)

With Inertia X View Stack you can push and pop pages similar to native mobile navigation. Each view in the stack is rendered inside an Inertia X [`<Frame>`](https://github.com/buhrmi/inertiax), so every level loads and manages its own page props, and the browser back button and page refresh keep working.

It exports two components and one action:

- `<ViewStack>` - a stack of panes. Each entry is loaded in an InertiaX `<Frame>` and animated in. Clicks on links inside the stack, or inside its child content, are intercepted and pushed onto the stack.
- `<Modal>` - the same stack rendered as a modal overlay, opened with the `use:modal` action.

## Features

- Panes slide in and out with Svelte transitions.
- Every level is a real browser history entry, so back, forward and refresh work. The URL does not change, which means deep links are not supported - see [URLs and deep links](#urls-and-deep-links).
- Views are InertiaX frames, so routing, props, `restore` and loading states come from InertiaX. There is no second data layer.
- Render `<Modal />` once in your layout, then add `use:modal` to any link.
- Styles are scoped and framework-free, and can be themed with CSS custom properties.
- Two components, one action, no configuration.

## Requirements

| Package | Version |
| --- | --- |
| [`svelte`](https://svelte.dev) | `^5.0.0` (runes) |
| [`inertiax-svelte`](https://github.com/buhrmi/inertiax) | `^11.0.0` |
| [`inertiax-core`](https://github.com/buhrmi/inertiax) | `^11.0.0` |

The package is built specifically for InertiaX: it renders panes with `<Frame>` and uses `shouldIntercept` from `inertiax-core` to decide which clicks to intercept.

## Installation

```bash
npm install inertiax-viewstack
# or
bun add inertiax-viewstack
# or
pnpm add inertiax-viewstack
```

## Quick start

### 1. Render the modal host once

Add `<Modal />` to your root layout. It renders nothing until a modal is opened.

```svelte
<!-- src/layouts/default.svelte -->
<script>
  import { Modal } from 'inertiax-viewstack'
</script>

{@render children()}

<Modal />
```

### 2. Open a modal from any link

```svelte
<script>
  import { modal } from 'inertiax-viewstack'
</script>

<a href="/holders" use:modal>Holders</a>
```

Clicking the link pushes `/holders` into the modal stack. It is fetched and rendered in an InertiaX `<Frame>`. Clicking the overlay, the Back button or the browser back button closes it.

The `href` is all it needs: the target is a normal InertiaX page or partial. The address bar does not change, see [URLs and deep links](#urls-and-deep-links).

### 3. Or build an inline stack

Use `<ViewStack>` directly when you want a stack that is not a modal, for example a drawer where the first frame stays put:

```svelte
<script>
  import { Frame } from 'inertiax-svelte'
  import { ViewStack } from 'inertiax-viewstack'
</script>

<ViewStack id="user">
  <Frame src="/balances" historyState={false} />

  <!-- Links in here are intercepted and pushed onto the stack -->
  <a href="/orders">Orders</a>
</ViewStack>
```

## How it works

Each stack is stored as an array of `<Frame>` props under a key in `history.state`:

```js
history.state = {
  user: [{ src: '/balances' }],
  modal: [{ src: '/holders' }]
}
```

- Intercepting a link pushes a `{ src }` entry onto `history.state[id]` and calls `history.pushState` with no URL, so the address bar keeps its current value.
- The browser back button returns to the previous entry, and a `popstate` listener syncs the visible stack.
- `close()` is just `history.go(-stack.length)`, so closing a modal pops every frame it opened in one go.

Because the stack lives in the history entry, a refresh reconstructs it, with each pane restoring through `<Frame restore>`.

## URLs and deep links

Every push creates a new history entry, but it does not change the URL: `pushState` is called without a URL, and non-top `<Frame>`s default to `updateBrowserUrl: false`. Two consequences:

- Back, forward and refresh work, because the stack is saved in the history entry and rebuilt on load.
- Deep links are not supported. The URL is identical for every level, so it cannot be used to rebuild a stack. Opening a stacked URL directly, or sharing it, shows the ordinary top-level page rather than the pane or modal.

If you need shareable, per-level URLs, that is app-level work. This package deliberately keeps the URL unchanged while navigating within a stack.

## API

### `<ViewStack>`

| Prop | Type | Default | Description |
| --- | --- | --- | --- |
| `id` | `string` | - | The `history.state` key this stack uses. Required, and must be unique per stack. |
| `children` | `Snippet` | - | Optional root content. Rendered once, beneath every pushed pane. Clicks inside it are intercepted. |
| `stack` | `object[]` (bindable) | `[]` | The frame props currently on the stack. Bind to it (`bind:stack`) if you need to read or drive the stack yourself. |

When `children` is omitted, the first pushed pane becomes the bottom of the stack. When it is present, panes slide in above it.

### `<Modal>`

The modal overlay. It takes no props and needs no configuration. Render it once, usually in your root layout. It hosts a `<ViewStack id="modal">` and adds a blurred, click-to-close backdrop.

### `modal` (action)

```svelte
<a href="/holders" use:modal>Holders</a>
```

A Svelte action that turns the element's click into a modal-stack push. Apply it to a link, or to a wrapper to intercept all links inside it. It removes its listener when the element is destroyed.

## Styling

The components ship with scoped CSS and no framework dependency. Theme them with CSS custom properties:

| Variable | Default | Applies to |
| --- | --- | --- |
| `--ivs-surface` | `#0d0d13` | Background of a pane and of the modal surface. |
| `--ivs-border-color` | `var(--color-border, #d5d5dd22)` | Border of the modal surface. |

```css
:root {
  --ivs-surface: #14141c;
  --ivs-border-color: #33333d;
}
```

## License

[MIT](./LICENSE)
