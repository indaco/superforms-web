<script lang="ts">
  import Head from '$lib/Head.svelte'
  import Next from '$lib/Next.svelte'
  import { concepts } from '$lib/navigation/sections'

	export let data;
</script>

# Progressive enhancement

<Head title="Progressive enhancement with use:enhance" />

By using `enhance` returned from `superForm`, we'll get the client-side enhancement expected from a modern website:

```svelte
<script lang="ts">
  const { form, enhance } = superForm(data.form);
  //            ^^^^^^^
</script>

<form method="POST" use:enhance>
```

Now all form submissions will happen on the client, and we unlock lots of client-side features like events, timers, auto error focus, etc.

The `use:enhance` action takes no arguments; instead, events are used to hook into the default SvelteKit use:enhance parameters and more. Check out the [events page](/concepts/events) for details.

> Without `use:enhance`, the form will be static. The only things that will work are [constraints](/concepts/client-validation#constraints) and [resetForm](/concepts/enhance#resetform). Also note that SvelteKits own `use:enhance` cannot be used; you must use the one returned from `superForm`.

## Differences from SvelteKit's use:enhance

There are three notable differences between the Superforms and SvelteKit `enhance`.

### 1. ActionResult errors are changed to failure

Any [ActionResult](https://kit.svelte.dev/docs/types#public-types-actionresult) with status `error` is transformed into `failure` to avoid form data loss. The SvelteKit default is to render the nearest `+error.svelte` page, which will wipe out the form and all data that was just entered. Returning `fail` with a [status message](/concepts/messages) or using the [onError event](/concepts/events#onerror) is a more user-friendly way of handling server errors.

### 2. The form isn't resetted by default

Resetting the form is disabled as default to avoid accidental data loss. It's not always wanted as well, for example in backend interfaces, where the form data should be kept after updating. 

It's easy to enable though, read further down at the `resetForm` option for details.

### 3. A tainted form warning is added

[Tainted fields](/concepts/tainted) is a feature which shows a dialog to the user when navigating away from a modified ("tainted" or "dirty") form. This is enabled by default, again to avoid data loss for the user.

## Usage

If you want to modify the basic `use:enhance` behavior, here are the options available along with the default values; you don't need to add them unless you want to change a value.

```ts
const { form, enhance, reset } = superForm(data.form, {
  applyAction: true,
  invalidateAll: true,
  resetForm: false
});
```

## When to change the defaults?

Quite rarely! If you have a single form on the page and nothing else is causing the page to invalidate, you'll probably be fine as it is. For multiple forms on the same page, you have to experiment with these three options. Read more on the [multiple forms](/concepts/multiple-forms) page.

`applyAction` and `invalidateAll` are the most technical ones; if you're dealing with a single form per page, you can most likely skip them.

### applyAction

When `applyAction` is `true`, the form will have the default SvelteKit behavior of both updating and reacting on `$page.form` and `$page.status`, and also redirecting automatically.

Turning this behavior off can be useful when you want to isolate the form from other sources updating the page, for example, Supabase events, a known source of confusing form behavior. Read more about `applyAction` [in the SvelteKit docs](https://kit.svelte.dev/docs/form-actions#progressive-enhancement-applyaction).

### invalidateAll

When `invalidateAll` is `true` (the default) and a successful validation result is returned from the server, the page will be invalidated and the load functions will run again. A login form takes advantage of this to update user information on the page.

### resetForm

When `true`, reset the form upon a successful validation result.

Note however, that since we're using `bind:value` on the input fields, a HTML form reset (clearing all fields in the DOM) won't have any effect. So in Superforms, **resetting means going back to the initial state of the form data**, effectively setting `$form` to what was initially sent to `superForm`. Usually, this is what is returned in the load function, [default values](/default-values) if only the schema was passed to `superValidate`.

For a custom reset, you can instead modify the `data` field returned from `superValidate` on the server, or use the [events](/concepts/events) together with the [reset](/api#superform-return-type) function on the client.

## Making the form behave like the SvelteKit default

You can remove the differences described above by setting the following options. Use with care, since the purpose of the changes is to protect the user from data loss.

```ts
const { form, enhance } = superForm(data.form, {
  // Reset the form upon a successful result
  resetForm: true,
  // On ActionResult error, render the nearest +error.svelte page
  onError: 'apply',
  // No message when navigating away from a modified form
  taintedMessage: null
});
```

<Next section={concepts} />
