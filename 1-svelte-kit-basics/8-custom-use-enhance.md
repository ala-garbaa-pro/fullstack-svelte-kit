# Custom `use:enhance` Implementation

## Table of Contents
- [Introduction](#introduction)
- [Simulating Slow Network](#simulating-slow-network)
- [Adding Local State](#adding-local-state)
- [Optimistic UI for Deletions](#optimistic-ui-for-deletions)
- [Conclusion](#conclusion)

## Introduction

The `use:enhance` feature allows us to extend beyond the browser's native behavior by providing a callback, enabling the addition of features such as pending states and optimistic UI.

## Simulating Slow Network

To illustrate the capabilities of `use:enhance`, let's simulate a slow network by adding an artificial delay to two actions in the `page.server.js` file:

```javascript
export const actions = {
    create: async ({ cookies, request }) => {
        await new Promise((fulfil) => setTimeout(fulfil, 1000));
        // ... rest of the code
    },

    delete: async ({ cookies, request }) => {
        await new Promise((fulfil) => setTimeout(fulfil, 1000));
        // ... rest of the code
    }
};
```

## Adding Local State

To enhance user experience, we'll add some local state. In the `page.svelte` file, a variable `creating` is initialized to `false`, representing the current state of creating a new todo. The UI will show a message while data is being saved:

```javascript
<script>
    import { fly, slide } from 'svelte/transition';
    import { enhance } from '$app/forms';

    export let data;
    export let form;

    let creating = false;
    let deleting = [];
</script>
```

```javascript
<form
    method="POST"
    action="?/create"
    use:enhance={() => {
        creating = true;

        return async ({ update }) => {
            await update();
            creating = false;
        };
    }}
>
    <!-- ... rest of the form code -->
</form>

<ul class="todos">
    <!-- ... rest of the code -->
</ul>

{#if creating}
    <span class="saving">saving...</span>
{/if}
```

## Optimistic UI for Deletions

For deletion actions, we can update the UI immediately without waiting for server validation. The `use:enhance` callback includes handling the `deleting` array:

```javascript
<ul class="todos">
    {#each data.todos.filter((todo) => !deleting.includes(todo.id)) as todo (todo.id)}
        <li in:fly={{ y: 20 }} out:slide>
            <form
                method="POST"
                action="?/delete"
                use:enhance={() => {
                    deleting = [...deleting, todo.id];
                    return async ({ update }) => {
                        await update();
                        deleting = deleting.filter((id) => id !== todo.id);
                    };
                }}
            >
                <!-- ... rest of the form code -->
            </form>
        </li>
    {/each}
</ul>
```

## Conclusion

The `use:enhance` feature is highly customizable, offering the ability to cancel submissions, handle redirects, control form resets, and more. Refer to the documentation for comprehensive details.