# SvelteKit Stores

## Table of Contents
- [Page Store](#page-store)
- [Navigating Store](#navigating-store)
- [Updated Store](#updated-store)

### Page Store
As we learned earlier, Svelte stores are a place to put data that doesn't belong to an individual component. SvelteKit provides three readonly stores via the `$app/stores` module — page, navigating, and updated. The most commonly used one is the page store.

The page store provides information about the current page, including:
- `url` — the URL of the current page
- `params` — the current page's parameters
- `route` — an object with an id property representing the current route
- `status` — the HTTP status code of the current page
- `error` — the error object of the current page, if any (error handling details will be covered later)
- `data` — the data for the current page, combining the return values of all load functions
- `form` — the data returned from a form action

As with any other store, you can reference its value in a component by prefixing its name with the $ symbol. For example, to access the current pathname, you can use `$page.url.pathname`.

```svelte
src/routes/+layout.svelte
<script>
    import { page } from '$app/stores';
</script>

<nav>
    <a href="/" aria-current={$page.url.pathname === '/'}>
        home
    </a>

    <a href="/about" aria-current={$page.url.pathname === '/about'}>
        about
    </a>
</nav>

<slot />
```

### Navigating Store
The navigating store represents the current navigation. When a navigation starts due to a link click, back/forward navigation, or programmatic goto, the value of `navigating` becomes an object with properties such as `from` and `to`, indicating the source and destination, and `type`, representing the type of navigation (e.g., link, popstate, or goto).

You can use the navigating store to show a loading indicator for long-running navigations. In the layout component, import the navigating store and add a message to the nav bar:

```svelte
src/routes/+layout.svelte
<script>
    import { page, navigating } from '$app/stores';
</script>

<nav>
    <a href="/" aria-current={$page.url.pathname === '/'}>
        home
    </a>

    <a href="/about" aria-current={$page.url.pathname === '/about'}>
        about
    </a>

    {#if $navigating}
        navigating to {$navigating.to.url.pathname}
    {/if}
</nav>

<slot />
```

### Updated Store
The updated store contains a boolean value indicating whether a new version of the app has been deployed since the page was first opened. To make it work, configure your `svelte.config.js` file with the `kit.version.pollInterval` option.

Version changes only occur in production, and `$updated` will always be false during development. You can manually check for new versions by calling `updated.check()`.

```svelte
src/routes/+layout.svelte
<script>
    import { page, navigating, updated } from '$app/stores';
</script>

{#if $updated}
    <div class="toast">
        <p>
            A new version of the app is available
            <button on:click={() => location.reload()}>
                reload the page
            </button>
        </p>
    </div>
{/if}
```