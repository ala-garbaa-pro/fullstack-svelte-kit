# Advanced Loading

## Table of Contents
- [Introduction](#introduction)
- [Universal Load Functions](#universal-load-functions)
- [Combining Server and Universal Load Functions](#combining-server-and-universal-load-functions)
- [Accessing Parent Data in Load Functions](#accessing-parent-data-in-load-functions)
- [Performance Considerations](#performance-considerations)
- [Manual Invalidation with `invalidate`](#manual-invalidation-with-invalidate)
- [Custom Invalidation Keys](#custom-invalidation-keys)
- [Nuclear Option: `invalidateAll`](#nuclear-option-invalidateall)

## Introduction

In the previous section on loading, data was loaded from the server using `+page.server.js` and `+layout.server.js` files. While this is convenient for various server-related tasks, there are scenarios where loading data on the client side makes more sense. This includes cases such as fetching data from an external API, using in-memory data, delaying navigation, or returning non-serializable items.

## Universal Load Functions

To address the non-serializable return issue, the server load functions in `src/routes/red/+page.server.js`, `src/routes/green/+page.server.js`, and `src/routes/blue/+page.server.js` are transformed into universal load functions by renaming them to `+page.js`. This enables the functions to run both on the server during server-side rendering and in the browser during client-side navigation.

```javascript
// src/routes/red/+page.js
// src/routes/green/+page.js
// src/routes/blue/+page.js
export async function load() {
  // ... (returning a component constructor)
}
```

Now, these components can be used like any other value, even inside `src/routes/+layout.svelte`.

```javascript
// src/routes/+layout.svelte
<nav
  class:has-color={!!$page.data.color}
  style:background={$page.data.color ?? 'var(--bg-2)'}
>
  <!-- Navigation links -->
  {#if $page.data.component}
    <svelte:component this={$page.data.component} />
  {/if}
</nav>
```

Read the documentation to learn more about the distinction between server load functions and universal load functions, and when to use each.

## Combining Server and Universal Load Functions

Sometimes, it's necessary to use both server load functions and universal load functions together. For instance, returning data from the server while also including non-serializable values like a component.

```javascript
// src/routes/+page.js
export async function load({ data }) {
  const module = data.cool
    ? await import('./CoolComponent.svelte')
    : await import('./BoringComponent.svelte');

  return {
    component: module.default,
    message: data.message
  };
}
```

## Accessing Parent Data in Load Functions

Load functions in SvelteKit components have access to data returned by their parent load functions. This can be achieved using `await parent()`.

```javascript
// src/routes/+layout.server.js
export function load() {
  return { a: 1 };
}

// src/routes/sum/+layout.js
export async function load({ parent }) {
  const { a } = await parent();
  return { b: a + 1 };
}

// src/routes/sum/+page.js
export async function load({ parent }) {
  const { a, b } = await parent();
  return { c: a + b };
}
```

## Performance Considerations

In SvelteKit, route-based code splitting is used. The JavaScript required for the initial page load includes what's needed for interactions on that page. Subsequent route code is fetched as needed during navigation, optimizing the amount of code downloaded.

## Manual Invalidation with `invalidate`

Sometimes, load functions need to be rerun manually to update data. This can be achieved using the `invalidate` function.

```javascript
// src/routes/[...timezone]/+page.svelte
<script>
  import { onMount } from 'svelte';
  import { invalidate } from '$app/navigation';

  export let data;

  onMount(() => {
    const interval = setInterval(() => {
      invalidate('/api/now');
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  });
</script>
```

## Custom Invalidation Keys

Dependencies for load functions can be specified manually using `depends`. Custom keys like `data:now` can be created for more granular control.

```javascript
// src/routes/+layout.js
export async function load({ depends }) {
  depends('data:now');

  return {
    now: Date.now()
  };
}
```

## Nuclear Option: `invalidateAll`

For a comprehensive rerun of all load functions on the current page, `invalidateAll()` can be used.

```javascript
// src/routes/[...timezone]/+page.svelte
<script>
  import { onMount } from 'svelte';
  import { invalidateAll } from '$app/navigation';

  export let data;

  onMount(() => {
    const interval = setInterval(() => {
      invalidateAll();
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  });
</script>
```

Note: `invalidate(() => true)` and `invalidateAll` serve different purposes, as `invalidateAll` reruns load functions without any URL dependencies.