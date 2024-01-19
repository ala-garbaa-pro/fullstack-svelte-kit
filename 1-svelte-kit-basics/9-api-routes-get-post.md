# SvelteKit API Routes: GET & POST

Table of Contents:
- [GET Route for Random Number](#get-route-for-random-number)
- [POST Route for Adding Todo](#post-route-for-adding-todo)

## GET Route for Random Number
So far, we've been creating pages in our application, but SvelteKit allows you to create more than just pages. You can also create API routes by adding a `server.js` file instead of a page `server.js` file in a `page.svelte` that exports functions corresponding to HTTP methods like GET, POST, PATCH, PUT, DELETE, and so on.

This app fetches data from a `/roll` API route when you click the button. And at the moment, we haven't implemented that route. So let's do that by adding a `src/routes/roll/+server.js` file. So click the new file icon, `roll/server.js`. From here, we can export functions that correspond to the HTTP Verbs.

```javascript
// src/routes/roll/+server.js
export function GET() {
    const number = Math.floor(Math.random() * 6) + 1;

    return new Response(number, {
        headers: {
            'Content-Type': 'application/json'
        }
    });
}
```

Request handlers must return a `Response` object. Since it's common to return JSON from an API route, SvelteKit provides a convenience function for generating these responses:

```javascript
// src/routes/roll/+server.js
import { json } from '@sveltejs/kit';

export function GET() {
    const number = Math.floor(Math.random() * 6) + 1;

    return json(number);
}
```

Now, if you roll the dice, the response will be the same response object returned from the API route.

## POST Route for Adding Todo
You can also add handlers that mutate data, such as POST. In most cases, you should use form actions instead — you'll end up writing less code, and it'll work without JavaScript, making it more resilient.

Inside the `keydown` event handler of the 'add a todo' `<input>`, let's post some data to the server:

```html
<!-- src/routes/+page.svelte -->
<input
    type="text"
    autocomplete="off"
    on:keydown={async (e) => {
        if (e.key !== 'Enter') return;

        const input = e.currentTarget;
        const description = input.value;

        const response = await fetch('/todo', {
            method: 'POST',
            body: JSON.stringify({ description }),
            headers: {
                'Content-Type': 'application/json'
            }
        });

        input.value = '';
    }}
/>
```

Create the `/todo` route by adding a `src/routes/todo/+server.js` file with a POST handler:

```javascript
// src/routes/todo/+server.js
import { json } from '@sveltejs/kit';
import * as database from '$lib/server/database.js';

export async function POST({ request, cookies }) {
    const { description } = await request.json();

    const userid = cookies.get('userid');
    const { id } = await database.createTodo({ userid, description });

    return json({ id }, { status: 201 });
}
```

In the event handler, you can use the response to update the page:

```html
<!-- src/routes/+page.svelte -->
<input
    type="text"
    autocomplete="off"
    on:keydown={async (e) => {
        if (e.key !== 'Enter') return;

        const input = e.currentTarget;
        const description = input.value;

        const response = await fetch('/todo', {
            method: 'POST',
            body: JSON.stringify({ description }),
            headers: {
                'Content-Type': 'application/json'
            }
        });

        const { id } = await response.json();

        data.todos = [...data.todos, {
            id,
            description
        }];

        input.value = '';
    }}
/>
```

Remember to only mutate data in a way that would yield the same result as reloading the page.