# SvelteKit Environment Variables

## Table of Contents
- [Introduction](#introduction)
- [Adding Environment Variables](#adding-environment-variables)
- [Accessing Environment Variables in SvelteKit](#accessing-environment-variables-in-sveltekit)
- [Keeping Secrets](#keeping-secrets)
- [Static vs Dynamic Environment Variables](#static-vs-dynamic)
- [Dynamic Environment Variables](#dynamic-environment-variables)
- [Exposing Environment Variables to the Browser](#exposing-environment-variables-to-the-browser)
- [Conclusion](#conclusion)

## Introduction

Environment variables play a crucial role in web development, containing sensitive information such as API keys and database credentials. This guide explores the use of environment variables in SvelteKit, a framework for building web applications.

## Adding Environment Variables

Environment variables can be stored in a .env file, ensuring they are available to the application. Consider using .env.local or .env.[mode] files as per Vite documentation, and always add sensitive files to .gitignore.

```env
.env
PASSPHRASE="open sesame"
```

## Accessing Environment Variables in SvelteKit

In the exercise, the goal is to allow user access to the website based on a correct passphrase stored in an environment variable. The following code snippet demonstrates how to implement this in SvelteKit:

```javascript
// src/routes/+page.server.js
import { redirect, fail } from '@sveltejs/kit';
import { PASSPHRASE } from '$env/static/private';

export function load({ cookies }) {
    if (cookies.get('allowed')) {
        throw redirect(307, '/welcome');
    }
}

export const actions = {
    default: async ({ request, cookies }) => {
        const data = await request.formData();

        if (data.get('passphrase') === PASSPHRASE) {
            cookies.set('allowed', 'true', {
                path: '/'
            });

            throw redirect(303, '/welcome');
        }

        return fail(403, {
            incorrect: true
        });
    }
};
```

## Keeping Secrets

To prevent sensitive data from being exposed to the browser, SvelteKit restricts the import of certain modules, such as $env/static/private, to server-side code only.

```javascript
// src/routes/+page.svelte
<script>
    // This import will cause an error
    import { PASSPHRASE } from '$env/static/private';
    export let form;
</script>
```

## Static vs Dynamic Environment Variables

Static environment variables (known at build time) are recommended for performance optimizations. Dynamic environment variables (known at runtime) can be used when values are not determined until the app runs.

```javascript
// src/routes/+page.server.js
import { redirect, fail } from '@sveltejs/kit';
import { env } from '$env/dynamic/private';

export function load({ cookies }) {
    if (cookies.get('allowed')) {
        throw redirect(307, '/welcome');
    }
}

export const actions = {
    default: async ({ request, cookies }) => {
        const data = await request.formData();

        if (data.get('passphrase') === env.PASSPHRASE) {
            cookies.set('allowed', 'true', {
                path: '/'
            });

            throw redirect(303, '/welcome');
        }

        return fail(403, {
            incorrect: true
        });
    }
};
```

## Exposing Environment Variables to the Browser

Public environment variables, distinguished by a PUBLIC_ prefix, can be safely exposed to the browser. Values can be added to the .env file and imported into Svelte components.

```env
.env
PUBLIC_THEME_BACKGROUND="steelblue"
PUBLIC_THEME_FOREGROUND="bisque"
```

```javascript
// src/routes/+page.svelte
<script>
    import {
        PUBLIC_THEME_BACKGROUND,
        PUBLIC_THEME_FOREGROUND
    } from '$env/static/public';
</script>
```

For dynamic public environment variables, use the dynamic module.

```javascript
// src/routes/+page.svelte
<script>
    import { env } from '$env/dynamic/public';
</script>

<main
    style:background={env.PUBLIC_THEME_BACKGROUND}
    style:color={env.PUBLIC_THEME_FOREGROUND}
>
    {env.PUBLIC_THEME_FOREGROUND} on {env.PUBLIC_THEME_BACKGROUND}
</main>
```

## Conclusion

Congratulations on completing the tutorial! You are now equipped with the knowledge to build applications using Svelte and SvelteKit. As these technologies evolve, stay updated by periodically checking the documentation and participating in the Svelte community through Discord and Twitter. Happy coding!