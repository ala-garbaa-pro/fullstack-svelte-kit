# Progressive Enhancement

## Table of Contents
1. [Introduction](#introduction)
2. [Enhancing Forms](#enhancing-forms)
3. [Transitions](#transitions)
4. [Handling JavaScript Failures](#handling-javascript-failures)
5. [Delete Icon Display](#delete-icon-display)

## Introduction

In this post, we'll explore the concept of progressive enhancement in the context of a web application. By leveraging the `<form>` element, our app remains functional even in scenarios where JavaScript is disabled, ensuring resilience.

## Enhancing Forms

To progressively enhance the user experience when JavaScript is available, we use the `enhance` function from `$app/forms`. Below is an example of how to implement this in SvelteKit:

```svelte
<script>
	import { enhance } from '$app/forms';

	export let data;
	export let form;
</script>

<form method="POST" action="?/create" use:enhance>
<!-- ... -->

<form method="POST" action="?/delete" use:enhance>
<!-- ... -->
```

This simple addition enables the emulation of browser-native behavior, updating the form prop, handling successful responses, navigation on redirect, and rendering error pages as needed.

## Transitions

To add a touch of elegance to the user interface, transitions can be implemented using Svelte's transition features. Here's an example using `fly` and `slide`:

```svelte
<script>
	import { fly, slide } from 'svelte/transition';
	import { enhance } from '$app/forms';

	export let data;
	export let form;
</script>

<li in:fly={{ y: 20 }} out:slide>...</li>
```

These transitions provide a visually pleasing experience when adding or deleting items from the list.

## Handling JavaScript Failures

The importance of handling scenarios where JavaScript fails is emphasized. Resilience to such conditions, such as poor or non-existent network connectivity, is crucial for maintaining a functional application.

## Delete Icon Display

The delete icon displayed on the right-hand side of todos is a simple SVG file imported from the local directory. Svelte's resolution mechanism ensures that the SVG is optimized and cached during the application build process, providing a seamless design integration.

For further customization, the SVG file can be placed in the `lib` folder, and the reference updated accordingly.

Feel free to explore these techniques to enhance the robustness and user experience of your SvelteKit applications.