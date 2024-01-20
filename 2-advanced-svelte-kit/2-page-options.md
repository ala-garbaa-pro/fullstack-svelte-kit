# Table of Contents
1. [Page Options](#page-options)
   - [Server-Side Rendering (SSR)](#server-side-rendering-ssr)
   - [Client-Side Rendering (CSR)](#client-side-rendering-csr)
   - [Prerendering](#prerendering)
   - [Trailing Slash](#trailing-slash)
   - [Progressive Web Apps (PWAs) with SvelteKit](#progressive-web-apps-pwas-with-sveltekit)
   - [Adapter Static and Prerendering](#adapter-static-and-prerendering)
   - [Dynamic Route Manifest](#dynamic-route-manifest)

# Page Options

In the chapter on loading data, we explored exporting load functions from `+page.js`, `+page.server.js`, `+layout.js`, and `+layout.server.js` files. Additionally, various page options can be exported from these modules:

## Server-Side Rendering (SSR)

Server-side rendering (SSR) involves generating HTML on the server, the default behavior in SvelteKit. SSR is crucial for performance, resilience, and search engine optimization (SEO). Disabling SSR may be necessary for components that rely on immediate access to browser globals like `window`. To disable SSR, set `ssr` to false in the `+page.server.js` file:

```svelte
// src/routes/+page.server.js
export const ssr = false;
```

Setting `ssr` to false in the root `+layout.server.js` effectively turns the entire app into a single-page app (SPA).

## Client-Side Rendering (CSR)

Client-side rendering (CSR) enables interactivity on the page, allowing features like incrementing a counter on button clicks without a full-page reload. To disable client-side rendering, set `csr` to false in the `+page.server.js` file:

```svelte
// src/routes/+page.server.js
export const csr = false;
```

While uncommon, disabling CSR can be useful for assessing the usability of your application for users who cannot use svelte.

## Prerendering

Prerendering involves generating HTML for a page at build time, providing cheap and performant static data serving. To enable prerendering, set `prerender` to true in the `+page.server.js` file:

```svelte
// src/routes/+page.server.js
export const prerender = true;
```

Prerendering has trade-offs, such as longer build times and the need to deploy a new version for content updates. Not all pages can be prerendered, and the `prerender.entries` configuration specifies pages with dynamic route parameters.

Setting `prerender` to true in the root `+layout.server.js` effectively turns SvelteKit into a static site generator (SSG).

## Trailing Slash

The `trailingSlash` option determines how trailing slashes in URLs are handled. By default, SvelteKit strips trailing slashes. To enforce a trailing slash or ignore it, set `trailingSlash` accordingly:

```svelte
// src/routes/always/+page.server.js
export const trailingSlash = 'always';
```

```svelte
// src/routes/ignore/+page.server.js
export const trailingSlash = 'ignore';
```

The default value is `'never'`. Trailing slashes impact prerendering, with URLs saved differently based on their presence.

## Progressive Web Apps (PWAs) with SvelteKit

It is possible to build Progressive Web Apps (PWAs) with SvelteKit. SvelteKit provides tools for adding a service worker and supports a library called Vite Plugin PWA, simplifying PWA development.

## Adapter Static and Prerendering

When using `adapter static`, both setting `prerender` to true in the `+layout.server.js` and using `adapter static` are necessary. This combination is crucial for handling non-prerendered routes. However, not every page needs to be prerendered, allowing flexibility in building fully client-rendered single-page apps.

## Dynamic Route Manifest

A forthcoming feature, proposed by Elliott Johnson, introduces a new export called `entries` in `page.server.js` files. This function, defined per route with dynamic parameters, returns an array of valid paths for that page. This feature simplifies the process of specifying prerendered routes, reducing the need to update entries data in the SvelteKit config.