# Table of Contents

1. [Loading Data on the Server](#loading-data-on-the-server)
    - [Dynamic Parameters in Routes](#dynamic-parameters-in-routes)
    - [SvelteKit's Core Functions](#sveltekits-core-functions)
    - [Loading Data for Pages](#loading-data-for-pages)
    - [Loading Page with Spinner](#loading-page-with-spinner)
    - [Svelte Native for Mobile](#svelte-native-for-mobile)

## Loading Data on the Server

To create routes with dynamic parameters, use square brackets around a valid variable name. For example, a file like `src/routes/blog/[slug]/+page.svelte` will create a route that matches `/blog/one`, `/blog/two`, `/blog/three`, and so on.

Let's create that file:

```markdown
src/routes/blog/[slug]/+page.svelte
```

```svelte
<h1>blog post</h1>
```

We can now navigate from the `/blog` page to individual blog posts. In the next chapter, we'll see how to load their content.

Multiple route parameters can appear within one URL segment, as long as they are separated by at least one static character: `foo/[bar]x[baz]` is a valid route where `[bar]` and `[baz]` are dynamic parameters.

---

## Dynamic Parameters in Routes

Very often your routes will have dynamic parameters. That means that you'll have a single route that can serve multiple pages. So here for `/.page.svelte`. That creates a root that will match `blog/one`, `blog/two`, `blog/three`, and so on. At the moment, that file doesn't exist, so let's create it.

But for now, we'll just put in some static placeholder content, And if we navigate to any of the pages that are defined by that route, then we'll see that same static content. Now the way that dynamic parameters are defined is actually very flexible, you can have multiple parameters in a single segment.

You can have multiple parameters in a route, and we'll learn more about complex routing patterns later on in the workshop. So SvelteKit's job boils down to three things.

---

## SvelteKit's Core Functions

At its core, SvelteKit's job boils down to three things:

1. **Routing** — figure out which route matches an incoming request
2. **Loading** — get the data needed by the route
3. **Rendering** — generate some HTML (on the server) or update the DOM (in the browser)

We've seen how routing and rendering work. Let's talk about the middle part — loading.

---

## Loading Data for Pages

Every page of your app can declare a load function in a `+page.server.js` file alongside the `+page.svelte` file. As the file name suggests, this module only ever runs on the server, including for client-side navigations. Let's add a `src/routes/blog/+page.server.js` file so that we can replace the hard-coded links in `src/routes/blog/+page.svelte` with actual blog post data:

```javascript
src/routes/blog/+page.server.js
```

```javascript
import { posts } from './data.js';

export function load() {
    return {
        summaries: posts.map((post) => ({
            slug: post.slug,
            title: post.title
        }))
    };
}
```

For the sake of the tutorial, we're importing data from `src/routes/blog/data.js`. In a real app, you'd be more likely to load the data from a database or a CMS, but for now, we'll do it like this.

We can access this data in `src/routes/blog/+page.svelte` via the `data` prop:

```svelte
src/routes/blog/+page.svelte
```

```svelte
<script>
    export let data;
</script>

<h1>blog</h1>

<ul>
    <li><a href="/blog/one">one</a></li>
    <li><a href="/blog/two">two</a></li>
    <li><a href="/blog/three">three</a></li>
    {#each data.summaries as { slug, title }}
        <li><a href="/blog/{slug}">{title}</a></li>
    {/each}
</ul>
```

Now, let's do the same for the post page:

```javascript
src/routes/blog/[slug]/+page.server.js
```

```javascript
import { posts } from '../data.js';

export function load({ params }) {
    const post = posts.find((post) => post.slug === params.slug);

    return {
        post
    };
}
```

```svelte
src/routes/blog/[slug]/+page.svelte
```

```svelte
<script>
    export let data;
</script>

<h1>blog post</h1>
<h1>{data.post.title}</h1>
<div>{@html data.post.content}</div>
```

There's one last detail we need to take care of — the user might visit an invalid pathname like `/blog/nope`, in which case we'd like to respond with a 404 page:

```javascript
src/routes/blog/[slug]/+page.server.js
```

```javascript
import { error } from '@sveltejs/kit';
import { posts } from '../data.js';

export function load({ params }) {
    const post = posts.find((post) => post.slug === params.slug);

    if (!post) throw error(404);

    return {
        post
    };
}
```

We'll learn more about error handling in later chapters.

---

## Loading Page with Spinner

Can you have a loading page that shows with a spinner before the `+page.tsload` function finishes and `+page.svelte` is rendered?

> Yes, you can. Unfortunately, because of a technical limitation with web containers, which is the technology that powers this website, I cannot demonstrate streaming in this tutorial. What I can do is show a little demo that has been put together, this [sveltekit-on-the-edge](https://example.com/sveltekit-on-the-edge). So this is a page that is being rendered dynamically in an edge function.

Is Osseo near here? I don't know if that's the nearest point of presence that this page is being rendered at. And if I click on the streamed link here, then it's actually gonna get some data from the server that is gonna be delayed. So we're actually able to stream promises from the edge function, or from the serverless function, or from your server, and they will be rendered in the browser when the promise resolves using streaming.

So that's how we approach that problem. We don't have a dedicated loading page or something like that. Instead, you return promises from your data, and in that way you're able to render the UI that is most appropriate for your case.

---

## Svelte Native for Mobile

There was another question that came in around, Kind of, there's Svelte Native for mobile but have you seen any traction on that?

Not knowing too many people talking about Svelte Mobile, just wondering if that's a good route, or if you're thinking about mobile?

> I haven't personally used Svelte Native. Some people have, I believe it is an

 up-to-date and functional project, and it's the only one that we have, as far as I'm aware.

So if you do wanna build a native app, then I definitely recommend giving Svelte Native a look. But it is not an officially supported project that comes from the community. It's something that if we have time and bandwidth to do, we would like bring into the organization at some point and have that as a first-class project.

But right now it's a little bit, your mileage may vary.