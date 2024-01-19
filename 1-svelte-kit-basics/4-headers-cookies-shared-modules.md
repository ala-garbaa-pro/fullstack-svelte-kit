# Headers, Cookies, & Shared Modules

## Table of Contents
- [Setting Headers and Cookies](#setting-headers-and-cookies)
- [Using Cookies API](#using-cookies-api)
- [Handling Input-Dependent Data](#handling-input-dependent-data)
- [Organizing Code with $lib](#organizing-code-with-lib)

## Setting Headers and Cookies

Inside a load function (as well as in form actions, hooks, and API routes, which we'll learn about later), you have access to a `setHeaders` function, which can be used to set headers on the response.

Most commonly, you'd use it to customize caching behavior with the `Cache-Control` response header. However, for the sake of this tutorial, we'll demonstrate changing the content type of the response to `text/plain`. (You may need to reload the iframe to see the effect.)

```javascript
// src/routes/+page.server.js
export function load({ setHeaders }) {
    setHeaders({
        'Content-Type': 'text/plain'
    });
}
```

## Using Cookies API

The `setHeaders` function cannot be used with the `Set-Cookie` header. Instead, you should use the cookies API. To read a cookie, use `cookies.get(name, options)`:

```javascript
// src/routes/+page.server.js
export function load({ cookies }) {
    const visited = cookies.get('visited');

    return {
        visited: visited === 'true'
    };
}
```

To set a cookie, use `cookies.set(name, value, options)`. It's strongly recommended to explicitly configure the path when setting a cookie:

```javascript
// src/routes/+page.server.js
export function load({ cookies }) {
    const visited = cookies.get('visited');

    cookies.set('visited', 'true', { path: '/' });

    return {
        visited: visited === 'true'
    };
}
```

Now, if you reload the iframe, "Hello stranger!" becomes "Hello friend!"

## Handling Input-Dependent Data

When working with input-dependent data, the approach depends on the desired UI. If you want results to stream in as the user types, handle the logic inside the component asynchronously. Alternatively, you can load data inside a load function triggered by a form submission.

## Organizing Code with $lib

Because SvelteKit uses directory-based routing, it's easy to place modules and components alongside the routes that use them. A good rule of thumb is to "put code close to where it's used." For shared code accessed by multiple routes, the `src/lib` directory is a suitable location. Use the `$lib` alias to access these shared resources.

Example of importing a file from `$lib`:

```javascript
// src/routes/a/deeply/nested/route/+page.svelte
<script>
    import { message } from '$lib/message.js';
</script>

<h1>a deeply nested route</h1>
<p>{message}</p>
```

Consistently use `$lib` to avoid issues with the number of dot-dot slashes when referencing shared modules or components across different routes. This is a convenient place to store utility components, `utils.js` files, and similar shared resources.