# Forms

## Table of Contents
- [Introduction](#introduction)
- [Creating a Todo App](#creating-a-todo-app)
- [Handling Form Submission](#handling-form-submission)
- [Adding Delete Functionality](#adding-delete-functionality)

### Introduction

In this tutorial, we will explore the use of forms in web development and how they can be utilized to submit data from the browser to the server.

### Creating a Todo App

To demonstrate the concept, let's build a simple todo app. We have an in-memory database set up in `src/lib/server/database.js` and a `load` function in `src/routes/+page.server.js` to fetch user-specific todos.

```html
src/routes/+page.svelte

<h1>todos</h1>

<form method="POST">
	<label>
		add a todo:
		<input
			name="description"
			autocomplete="off"
		/>
	</label>
</form>

<ul class="todos">
	<!-- ... -->
</ul>
```

### Handling Form Submission

When a user types something into the `<input>` field and hits Enter, a POST request is triggered to the current page. To handle this POST request, we create a server-side action in `src/routes/+page.server.js`.

```javascript
src/routes/+page.server.js

import * as db from '$lib/server/database.js';

export function load({ cookies }) {
	// ...
}

export const actions = {
	default: async ({ cookies, request }) => {
		const data = await request.formData();
		db.createTodo(cookies.get('userid'), data.get('description'));
	}
};
```

### Adding Delete Functionality

In most cases, a page will have multiple actions. For our todo app, we want to add a delete action to remove completed todos.

```javascript
src/routes/+page.server.js

export const actions = {
	create: async ({ cookies, request }) => {
		const data = await request.formData();
		db.createTodo(cookies.get('userid'), data.get('description'));
	},

	delete: async ({ cookies, request }) => {
		const data = await request.formData();
		db.deleteTodo(cookies.get('userid'), data.get('id'));
	}
};
```

To invoke the create action, update the form in `src/routes/+page.svelte`.

```html
src/routes/+page.svelte

<form method="POST" action="?/create">
	<label>
		add a todo:
		<input
			name="description"
			autocomplete="off"
		/>
	</label>
</form>
```

Additionally, create a form for each todo to enable deletion.

```html
src/routes/+page.svelte

<ul class="todos">
	{#each data.todos as todo (todo.id)}
		<li>
			<form method="POST" action="?/delete">
				<input type="hidden" name="id" value={todo.id} />
				<span>{todo.description}</span>
				<button aria-label="Mark as complete" />
			</form>
		</li>
	{/each}
</ul>
```

Now, the todo app allows both the creation and deletion of todos, demonstrating the bidirectional data flow between the browser and the server using forms.