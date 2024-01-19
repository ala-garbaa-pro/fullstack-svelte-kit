# Table of Contents
- [Layout Data](#layout-data)
  - [Updating the Layout File](#updating-the-layout-file)
  - [Adding a Sidebar to the Post Page](#adding-a-sidebar-to-the-post-page)
  - [Inheritance of Data](#inheritance-of-data)
  - [Efficient Navigation](#efficient-navigation)

# Layout Data

Just as `layout.svelte` files create UI for every child route, `layout.server.js` files load data for every child route.

## Updating the Layout File

Suppose we'd like to add a 'more posts' sidebar to our blog post page. We could return summaries from the load function in `src/routes/blog/[slug]/+page.server.js`, like we do in `src/routes/blog/+page.server.js`, but that would be repetitive.

Instead, let's rename `src/routes/blog/+page.server.js` to `src/routes/blog/+layout.server.js`. Notice that the `/blog` route continues to work — `data.summaries` is still available to the page.

## Adding a Sidebar to the Post Page

Now, add a sidebar in the layout for the post page:

```svelte
src/routes/blog/[slug]/+layout.svelte
<script>
	export let data;
</script>

<div class="layout">
	<main>
		<slot />
	</main>

	<aside>
		<h2>More posts</h2>
		<ul>
			{#each data.summaries as { slug, title }}
				<li>
					<a href="/blog/{slug}">{title}</a>
				</li>
			{/each}
		</ul>
	</aside>
</div>

<style>
	@media (min-width: 640px) {
		.layout {
			display: grid;
			gap: 2em;
			grid-template-columns: 1fr 16em;
		}
	}
</style>
```

## Inheritance of Data

The layout (and any page below it) inherits `data.summaries` from the parent `+layout.server.js`.

## Efficient Navigation

When we navigate from one post to another, we only need to load the data for the post itself — the layout data is still valid. See the documentation on invalidation to learn more.