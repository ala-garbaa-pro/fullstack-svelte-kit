# SvelteKit Hooks and Advanced Techniques

## Table of Contents
1. [Introduction](#introduction)
2. [Handle Hook](#handle-hook)
3. [Transform Page Chunk](#transform-page-chunk)
4. [Creating New Routes](#creating-new-routes)
5. [Event Object](#event-object)
6. [Adding Data to Event.Locals](#adding-data-to-eventlocals)
7. [Event.Fetch Method](#eventfetch-method)
8. [HandleFetch Hook](#handlefetch-hook)
9. [Error Handling](#error-handling)
10. [HandleError Hook](#handleerror-hook)
11. [Customizing Error Responses](#customizing-error-responses)

## Introduction
Okay, so we've finished part 3, the tutorial. We've learned all of the basics of SvelteKit, and it's now time to move on to part 4 where we're gonna learn some advanced techniques. We're gonna begin with hooks. SvelteKit provides several hooks, which are ways to intercept the framework's default behavior.

## Handle Hook
The most elementary hook is `handle`, which lives in `src/hooks.server.js`. It receives an event object along with a resolve function and returns a Response. The `resolve` function is where the magic happens, as SvelteKit matches the incoming request URL to a route of your app.

```svelte
src/hooks.server.js
export async function handle({ event, resolve }) {
    return await resolve(event);
}
```

## Transform Page Chunk
For pages (as opposed to API routes), you can modify the generated HTML with `transformPageChunk`:

```svelte
src/hooks.server.js
export async function handle({ event, resolve }) {
    return await resolve(event, {
        transformPageChunk: ({ html }) => html.replace('<body', '<body style="color: hotpink"')
    });
}
```

## Creating New Routes
You can also create entirely new routes:

```svelte
src/hooks.server.js
export async function handle({ event, resolve }) {
    if (event.url.pathname === '/ping') {
        return new Response('pong');
    }

    return await resolve(event, {
        transformPageChunk: ({ html }) => html.replace('<body', '<body style="color: hotpink"')
    });
}
```

## Event Object
The `event` object passed into `handle` is the same object — an instance of a `RequestEvent` — that is passed into API routes in `+server.js` files, form actions in `+page.server.js` files, and load functions in `+page.server.js` and `+layout.server.js`.

## Adding Data to Event.Locals
A useful pattern is to add some data to `event.locals` in `handle` so that it can be read in subsequent load functions:

```svelte
src/hooks.server.js
export async function handle({ event, resolve }) {
    event.locals.answer = 42;
    return await resolve(event);
}
```

```svelte
src/routes/+page.server.js
export function load(event) {
    return {
        message: `the answer is ${event.locals.answer}`
    };
}
```

## Event.Fetch Method
The `event` object has a `fetch` method that behaves like the standard Fetch API, but with superpowers:

- It can be used to make credentialed requests on the server.
- It can make relative requests on the server.
- Internal requests go directly to the handler function when running on the server.

## HandleFetch Hook
Its behavior can be modified with the `handleFetch` hook:

```svelte
src/hooks.server.js
export async function handleFetch({ event, request, fetch }) {
    return await fetch(request);
}
```

For example, you could respond to requests for `src/routes/a/+server.js` with responses from `src/routes/b/+server.js` instead.

```svelte
src/hooks.server.js
export async function handleFetch({ event, request, fetch }) {
    const url = new URL(request.url);
    if (url.pathname === '/a') {
        return await fetch('/b');
    }

    return await fetch(request);
}
```

## Error Handling
The `handleError` hook lets you intercept unexpected errors and trigger some behavior, like pinging a Slack channel or sending data to an error logging service.

```svelte
src/hooks.server.js
export function handleError({ event, error }) {
    console.error(error.stack);
}
```

## HandleError Hook
You can customize error responses using the `handleError` hook:

```svelte
src/hooks.server.js
export function handleError({ event, error }) {
    console.error(error.stack);

    return {
        message: 'everything is fine',
        code: 'JEREMYBEARIMY'
    };
}
```

## Customizing Error Responses
You can now reference properties other than `message` in a custom error page:

```svelte
src/routes/+error.svelte
<script>
    import { page } from '$app/stores';
</script>

<h1>{$page.status}</h1>
<p>{$page.error.message}</p>
<p>Error code: {$page.error.code}</p>
```
