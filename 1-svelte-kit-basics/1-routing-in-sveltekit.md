## Table of Contents
- [Routing in SvelteKit](#routing-in-sveltekit)
  - [Filesystem-Based Routing](#filesystem-based-routing)
  - [Creating Pages](#creating-pages)
  - [Adding a Second Page](#adding-a-second-page)
  - [Navigating Between Pages](#navigating-between-pages)
  - [Updating Page Contents](#updating-page-contents)
  - [Utilizing Layouts](#utilizing-layouts)

---
Sure, here are the official documentation quotes for the mentioned topics:

> [Routing / Layouts • Svelte Tutorial](https://learn.svelte.dev/tutorial/layouts)


> [Routing / Pages • Svelte Tutorial](https://learn.svelte.dev/tutorial/pages)

---

These quotes are sourced directly from the official Svelte Tutorial documentation.
## Routing in SvelteKit

SvelteKit employs filesystem-based routing, where the routes of your app — indicating what the app should do when a user navigates to a particular URL — are determined by the directories in your codebase.

### Filesystem-Based Routing

Every `+page.svelte` file inside `src/routes` creates a page in your app. Currently, there is one page in the app, located at `src/routes/+page.svelte`, which corresponds to the root (`/`). Navigating to `/about` results in a 404 Not Found error.

### Creating Pages

To resolve this, let's add a second page: `src/routes/about/+page.svelte`. Copy the contents of `src/routes/+page.svelte` and update it as follows:

```svelte
src/routes/about/+page.svelte
<nav>
	<a href="/">home</a>
	<a href="/about">about</a>
</nav>

<h1>about</h1>
<p>this is the about page.</p>
```

### Adding a Second Page

Now, you can navigate between `/` and `/about`.

### Navigating Between Pages

In contrast to traditional multi-page apps, navigating to `/about` and back updates the contents of the current page, mimicking a single-page app. This combines fast server-rendered startup with instant navigation, although this behavior can be configured.

### Updating Page Contents

SvelteKit utilizes filesystem-based routing, meaning that the routes of your app are determined by the directories in your codebase. Each `+page.svelte` file inside `src/routes` creates a corresponding page in your app.

In this app, there is currently one page at `src/routes/page.svelte`, mapping to the root of the application. Navigating to the about page results in a 404 Not Found error because we haven't defined that route yet.

To address this, add a second page by creating a new file `about/page.svelte`. Copy the homepage content into this file, and update the navigation links.

Now, you can navigate between home and about. Unlike traditional multi-page apps, content is updated within the current document when navigating between these routes, avoiding full document reloads.

### Utilizing Layouts

To handle common UI elements across different routes, use layouts. Create a new file, `src/routes/+layout.svelte`, and move duplicated content from `+page.svelte` files into `+layout.svelte`. The `<slot />` element is used to render page content:

```svelte
src/routes/+layout.svelte
<nav>
	<a href="/">home</a>
	<a href="/about">about</a>
</nav>

<slot />
```

A `+layout.svelte` file applies to every child route, including sibling `+page.svelte` (if it exists). You can nest layouts to arbitrary depth, providing flexibility for structuring your app.