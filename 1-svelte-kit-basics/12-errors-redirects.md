# Errors & Redirects

## Table of Contents
- [Types of Errors](#types-of-errors)
- [Custom Error Handling](#custom-error-handling)
- [Error Pages](#error-pages)
- [Redirects](#redirects)

--

## Types of Errors

There are two types of errors in SvelteKit — expected errors and unexpected errors.

An expected error is one that was created with the error helper from `@sveltejs/kit`, as in `src/routes/expected/+page.server.js`:

```javascript
// src/routes/expected/+page.server.js
import { error } from '@sveltejs/kit';

export function load() {
    throw error(420, 'Enhance your calm');
}
```

Any other error, such as the one in `src/routes/unexpected/+page.server.js`, is treated as unexpected:

```javascript
// src/routes/unexpected/+page.server.js
export function load() {
    throw new Error('Kaboom!');
}
```

When you throw an expected error, you're telling SvelteKit, "don't worry, I know what I'm doing here." An unexpected error, by contrast, is assumed to be a bug in your app. When an unexpected error is thrown, its message and stack trace will be logged to the console.

In a later chapter, we'll learn about how to add custom error handling using the `handleError` hook.

If you click the links in this app, you'll notice an important difference: the expected error message is shown to the user, whereas the unexpected error message is redacted and replaced with a generic 'Internal Error' message and a 500 status code. That's because error messages can contain sensitive data.

--

## Custom Error Handling

When something goes wrong inside a load function, SvelteKit renders an error page.

The default error page is somewhat bland. We can customize it by creating a `src/routes/+error.svelte` component:

```svelte
<!-- src/routes/+error.svelte -->
<script>
    import { page } from '$app/stores';
    import { emojis } from './emojis.js';
</script>

<h1>{$page.status} {$page.error.message}</h1>
<span style="font-size: 10em">
    {emojis[$page.status] ?? emojis[500]}
</span>
```

Notice that the `+error.svelte` component is rendered inside the root `+layout.svelte`. We can create more granular `+error.svelte` boundaries:

```svelte
<!-- src/routes/expected/+error.svelte -->
<h1>this error was expected</h1>
```

This component will be rendered for `/expected`, while the root `src/routes/+error.svelte` page will be rendered for any other errors that occur.

--

## Redirects

We can also use the throw mechanism to redirect from one page to another.

Create a new load function in `src/routes/a/+page.server.js`:

```javascript
// src/routes/a/+page.server.js
import { redirect } from '@sveltejs/kit';

export function load() {
    throw redirect(307, '/b');
}
```

Navigating to `/a` will now take us straight to `/b`.

You can throw `redirect(...)` inside load functions, form actions, API routes, and the `handle` hook, which we'll discuss in a later chapter.

The most common status codes you'll use:

- 303 — for form actions, following a successful submission
- 307 — for temporary redirects
- 308 — for permanent redirects

--

**Note:** Some web hosts expect a `404.html` file. This would apply if you're baking out your site as a set of static HTML pages. Some web servers let you specify `404.html`, which will basically be for everything that doesn't have a file already created. If you're using one of those web servers, then you're probably going to use something called `adapter static`, which we'll bring up the docs for.

So, you can create a fallback page and specify a `404.html` when using `adapter static`. This page will be invoked for everything that doesn't match.