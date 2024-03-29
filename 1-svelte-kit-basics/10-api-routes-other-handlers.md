# API Routes: Other Handlers

## Table of Contents
- [PUT Handler](#put-handler)
- [DELETE Handler](#delete-handler)
- [Integration in Event Handlers](#integration-in-event-handlers)

Similarly, we can add handlers for other HTTP verbs. Add a `/todo/[id]` route by creating a `src/routes/todo/[id]/+server.js` file with PUT and DELETE handlers for toggling and removing todos, using the `toggleTodo` and `deleteTodo` functions in `src/lib/server/database.js`:

```javascript
// src/routes/todo/[id]/+server.js
import * as database from '$lib/server/database.js';

export async function PUT({ params, request, cookies }) {
    const { done } = await request.json();
    const userid = cookies.get('userid');

    await database.toggleTodo({ userid, id: params.id, done });
    return new Response(null, { status: 204 });
}

export async function DELETE({ params, cookies }) {
    const userid = cookies.get('userid');

    await database.deleteTodo({ userid, id: params.id });
    return new Response(null, { status: 204 });
}
```

Since we don't need to return any actual data to the browser, we're returning an empty `Response` with a `204 No Content` status.

### PUT Handler
```javascript
// src/routes/todo/[id]/+server.js
export async function PUT({ params, request, cookies }) {
    // ...
}
```

### DELETE Handler
```javascript
// src/routes/todo/[id]/+server.js
export async function DELETE({ params, cookies }) {
    // ...
}
```

## Integration in Event Handlers

```svelte
// src/routes/+page.svelte
<label>
    <input
        type="checkbox"
        checked={todo.done}
        on:change={async (e) => {
            const done = e.currentTarget.checked;

            await fetch(`/todo/${todo.id}`, {
                method: 'PUT',
                body: JSON.stringify({ done }),
                headers: {
                    'Content-Type': 'application/json'
                }
            });
        }}
    />
    <span>{todo.description}</span>
    <button
        aria-label="Mark as complete"
        on:click={async (e) => {
            await fetch(`/todo/${todo.id}`, {
                method: 'DELETE'
            });

            data.todos = data.todos.filter((t) => t !== todo);
        }}
    />
</label>
```