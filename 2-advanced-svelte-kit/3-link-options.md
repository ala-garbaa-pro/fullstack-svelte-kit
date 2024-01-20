# Link Options

## Table of Contents
- [Introduction](#introduction)
- [Anticipating Navigations](#anticipating-navigations)
- [Customizing Preloading Behavior](#customizing-preloading-behavior)
- [Programmatic Preloading](#programmatic-preloading)
- [Disabling Client-Side Routing](#disabling-client-side-routing)
- [Additional Resources](#additional-resources)

## Introduction

In this exercise, we explore various link options in SvelteKit, focusing on improving navigation speed and customizing preloading behavior.

## Anticipating Navigations

In the /slow-a and /slow-b routes, artificial delays in their load functions result in slow navigation. To enhance user experience, we can use the `data-sveltekit-preload-data` attribute on anchor elements to initiate navigation on hover (desktop) or tap (mobile). This can significantly reduce the perceived delay, making navigation snappier.

```html
src/routes/+layout.svelte
<nav>
	<a href="/">home</a>
	<a href="/slow-a" data-sveltekit-preload-data>slow-a</a>
	<a href="/slow-b">slow-b</a>
</nav>
```

## Customizing Preloading Behavior

The `data-sveltekit-preload-data` attribute can be customized with the following values:
- "hover" (default, falls back to "tap" on mobile)
- "tap" — preloading only on tap
- "off" — disabling preloading

This attribute may lead to false positives, and as an alternative, the `data-sveltekit-preload-code` attribute can be used to preload JavaScript without eagerly loading data.

```javascript
import { preloadCode, preloadData } from '$app/navigation';

// preload the code and data needed to navigate to /foo
preloadData('/foo');

// preload the code needed to navigate to /bar, but not the data
preloadCode('/bar');
```

## Programmatic Preloading

To initiate preloading programmatically, import `preloadCode` and `preloadData` from `$app/navigation`. This allows more control over when and what to preload.

## Disabling Client-Side Routing

In certain cases, disabling client-side routing may be necessary. Adding the `data-sveltekit-reload` attribute on individual links or containing elements can achieve this.

```html
src/routes/+layout.svelte
<nav data-sveltekit-reload>
	<a href="/">home</a>
	<a href="/about">about</a>
</nav>
```

## Additional Resources

For more information on available link options and their values, consult the [link options documentation](https://kit.svelte.dev/docs/link-options) on the SvelteKit website.