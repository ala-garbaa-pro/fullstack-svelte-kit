# Form Validation

## Table of Contents
1. [Introduction](#introduction)
2. [Browser-Based Form Validation](#browser-based-form-validation)
3. [Server-Side Validation](#server-side-validation)
4. [Handling Errors](#handling-errors)
5. [Conclusion](#conclusion)

## Introduction

As a web developer, it's crucial to implement robust form validation to ensure that users cannot submit nonsensical or malicious data. This post outlines the importance of form validation and provides insights into both browser-based and server-side validation techniques.

## Browser-Based Form Validation

The first line of defense against mischievous user input is the browser's built-in form validation. This can be easily implemented by adding attributes to form elements. For example, marking an `<input>` as required can be achieved as follows:

```html
<form method="POST" action="?/create">
	<label>
		add a todo
		<input
			name="description"
			autocomplete="off"
			required
		/>
	</label>
</form>
```

## Server-Side Validation

While browser-based validation is helpful, it has limitations, especially for rules like uniqueness that cannot be expressed using HTML attributes. Additionally, users with advanced knowledge might manipulate the validation using browser devtools. Therefore, server-side validation is crucial.

In the `src/lib/server/database.js` file, server-side validation is implemented when creating a todo:

```javascript
export function createTodo(userid, description) {
	if (description === '') {
		throw new Error('todo must have a description');
	}

	const todos = db.get(userid);

	if (todos.find((todo) => todo.description === description)) {
		throw new Error('todos must be unique');
	}

	todos.push({
		id: crypto.randomUUID(),
		description,
		done: false
	});
}
```

## Handling Errors

To improve the user experience, it's essential to handle errors gracefully. In the `src/routes/+page.server.js` file, the `fail` function from SvelteKit is utilized to return data along with an appropriate HTTP status code:

```javascript
import { fail } from '@sveltejs/kit';
import * as db from '$lib/server/database.js';

export function load({ cookies }) {...}

export const actions = {
	create: async ({ cookies, request }) => {
		const data = await request.formData();

		try {
			db.createTodo(cookies.get('userid'), data.get('description'));
		} catch (error) {
			return fail(422, {
				description: data.get('description'),
				error: error.message
			});
		}
	}
}
```

In the `src/routes/+page.svelte` file, the returned data is accessed via the `form` prop:

```html
<script>
	export let data;
	export let form;
</script>

<div class="centered">
	<h1>todos</h1>
	
	{#if form?.error}
		<p class="error">{form.error}</p>
	{/if}
	
	<form method="POST" action="?/create">
		<label>
			add a todo:
			<input
				name="description"
				value={form?.description ?? ''}
				autocomplete="off"
				required
			/>
		</label>
	</form>
</div>
```

## Conclusion

Implementing both browser-based and server-side form validation is essential to ensure data integrity and enhance the user experience. Handling errors gracefully and providing clear feedback to users contributes to a more user-friendly application.