# Advanced Routing

## Table of Contents
1. [Optional Parameters](#optional-parameters)
2. [Rest Parameters](#rest-parameters)
3. [Matchers](#matchers)
4. [Route Groups](#route-groups)
5. [Layouts](#layouts)
6. [Layout Hierarchy](#layout-hierarchy)
7. [Data Preloading](#data-preloading)
8. [Layout Resets](#layout-resets)
9. [Named Layouts](#named-layouts)

## Optional Parameters

In the first chapter on routing, we learned how to create routes with dynamic parameters.

Sometimes it's helpful to make a parameter optional. A classic example is when you use the pathname to determine the locale — `/fr/...`, `/de/...`, and so on — but you also want to have a default locale.

To do that, we use double brackets. Rename the [lang] directory to [[lang]].

The app now fails to build because `src/routes/+page.svelte` and `src/routes/[[lang]]/+page.svelte` would both match `/`. Delete `src/routes/+page.svelte`. (You may need to reload the app to recover from the error page).

Lastly, edit `src/routes/[[lang]]/+page.server.js` to specify the default locale:

```javascript
// src/routes/[[lang]]/+page.server.js
const greetings = {
  en: 'hello!',
  de: 'hallo!',
  fr: 'bonjour!'
};

export function load({ params }) {
  return {
    greeting: greetings[params.lang ?? 'en']
  };
}
```

## Rest Parameters

To match an unknown number of path segments, use a `[...rest]` parameter, so named for its resemblance to rest parameters in JavaScript.

Rename `src/routes/[path]` to `src/routes/[...path]`. The route now matches any path.

Other, more specific routes will be tested first, making rest parameters useful as 'catch-all' routes. For example, if you needed a custom 404 page for pages inside `/categories/...`, you could add these files:

```
src/routes/
├ categories/
│ ├ animal/
│ ├ mineral/
│ ├ vegetable/
│ ├ [...catchall]/
│ │ ├ +error.svelte
│ │ └ +page.server.js
```

Inside the `+page.server.js` file, throw `error(404)` inside `load`.

Rest parameters do not need to go at the end — a route like `/items/[...path]/edit` or `/items/[...path].json` is totally valid.

## Matchers

To prevent the router from matching on invalid input, you can specify a matcher. For example, you might want a route like `/colors/[value]` to match hex values like `/colors/ff3e00` but not named colors like `/colors/octarine` or any other arbitrary input.

First, create a new file called `src/params/hex.js` and export a match function from it:

```javascript
// src/params/hex.js
export function match(value) {
  return /^[0-9a-f]{6}$/.test(value);
}
```

Then, to use the new matcher, rename `src/routes/colors/[color]` to `src/routes/colors/[color=hex]`.

Now, whenever someone navigates to that route, SvelteKit will verify that `color` is a valid hex value. If not, SvelteKit will try to match other routes, before eventually returning a 404.

Matchers run both on the server and in the browser.

## Route Groups

Sometimes it's useful to have layouts that don't affect the route. For example, you might have `/app` and `/account` routes, and those need to be behind authentication while your `/about` route page is open to the world. This can be achieved with a route group, which is a directory in parentheses.

Create an `(authed)` group by renaming `account` to `(authed)/account` then renaming `app` to `(authed)/app`.

Now we can control access to these routes by creating `src/routes/(authed)/+layout.server.js`:

```javascript
// src/routes/(authed)/+layout.server.js
import { redirect } from '@sveltejs/kit';

export function load({ cookies, url }) {
  if (!cookies.get('logged_in')) {
    throw redirect(303, `/login?redirectTo=${url.pathname}`);
  }
}
```

If you try to visit these pages, you'll be redirected to the `/login` route, which has a form action in `src/routes/login/+page.server.js` that sets the `logged_in` cookie.

We can also add some UI to these two routes by adding a `src/routes/(authed)/+layout.svelte` file:

```html
<!-- src/routes/(authed)/+layout.svelte -->
<slot />

<form method="POST" action="/logout">
  <button>log out</button>
</form>
```

## Layouts

As we saw in the routing introduction, layouts are a way to share UI and data loading logic between different routes.

Sometimes it's useful to use layouts without affecting the route. For example, you might need your `/app` and `/account` routes to be behind authentication, while your `/about` page is open to the world. We can do this with a route group, which is a directory in parentheses.

Create an `(authed)` group by renaming `account` to `(authed)/account` then renaming `app` to `(authed)/app`.

Now we can control access to these routes by creating `src/routes/(authed)/+layout.server.js`:

```javascript
// src/routes/(authed)/+layout.server.js
import { redirect } from '@sveltejs/kit';

export function load({ cookies, url }) {
  if (!cookies.get('logged_in')) {
    throw redirect(303, `/login?redirectTo=${url.pathname}`);
  }
}
```

If you try to visit these pages, you'll be redirected to the `/login` route, which has a form action in `src/routes/login/+page.server.js` that sets the `logged_in` cookie.

We can also add some UI to these two routes by adding a `src/routes/(authed)/+layout.svelte` file:

```html
<!-- src/routes/(authed)/+layout.svelte -->
<slot />

<form method="POST" action="/logout">
  <button>log out</button>
</form>
```

## Layout Hierarchy

Ordinarily, a page inherits every layout above it, meaning that `src/routes/a/b/c/+page.svelte` inherits four layouts:

- `src/routes/+layout.svelte`
- `src/routes/a/+layout.svelte`
- `src/routes/a/b/+layout.svelte`
- `src/routes/a/b/c/+layout.svelte`

Occasionally, it's useful to break out of the current layout hierarchy. We can do that by adding the `@` sign followed by the name of the parent segment to 'reset' to — for example `+page@b.svelte` would put `/a/b/c` inside `src/routes/a/b/+layout.svelte`, while `+page@a.svelte` would put it inside `src/routes/a/+layout.svelte`.

Let's reset it all

 the way to the root layout, by renaming it to `+page@.svelte`.

The root layout applies to every page of your app; you cannot break out of it.

## Data Preloading

In this falcate docs regarding the data SvelteKit preload code, hover and tap have this same description, what's their difference?

The difference between hover and tap is that if a link has the `data-sveltekit-preload` data attribute set to `hover`, then when the mouse comes to rest over that link, SvelteKit will begin fetching the code and data for the next page.

The assumption being that if the mouse has come to rest over the link, there's a very high chance that the user is about to initiate a navigation. But if you don't want that, then you can specify `tap`. In that case, the preloading won't begin until there's a pointer down then.

That doesn't give you quite as much of a head start, but it does reduce the number of false positives that you'll encounter. So if you're trying to save data, then that's a good thing for you to do. But if you're trying to maximize the speed of navigation, then you'll want to use hover.

Obviously, hover only applies to desktop because on a mobile device, you don't have a mouse; there are no hover events. So in that case, it will fall back to the tap behavior in all cases. If you do have hover set, then if for whatever reason the hover hasn't yet been triggered because the mouse clicks on the link while it's still moving over the link, then it will also initiate navigation on top.

## Layout Resets

Is the `+page@.svelte` the same as `+layout@.svelte`?

Yeah, so you can apply a layout reset to your layout components as well as to your page components if you want a whole group of routes to be reset to an earlier layout separately, altogether, I guess. Yeah, I hope that makes sense.

## Named Layouts

Is there a way to have named layouts?

The way that you name a layout is just by having it inside a segment. So wherever you are in the tree, you can refer to a layout above the route that you're currently at just by referring to the name of the directory in which that layout is.

To be clear, a layout that you can refer to that isn't route-based?

No, because if you had something like that, then the layouts would no longer be bound to the hierarchy, and the behavior would be very difficult to reason about. So the name of the layout is always connected to the directory in which that layout lives.

If we started having two ways of naming layouts, then it would just add confusion, but you wouldn't actually gain any expressive power, so we don't do that.