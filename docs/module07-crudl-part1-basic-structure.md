---
title: "Module 7: CRUDL"
layout: default
---

[Module Index](/enhance-workshop)


# Module 7: CRUDL

## Outline

- CRUD Flow (GET, POST, HTML Forms)
- Routes and files
- Dynamic and Catchall Routes

## CRUD Flow
- Create, Read, Update, Delete, List

These 5 operations are so common an entire subset of web apps are know as CRUD apps.
Because the general patterns are so common we will look at how to create CRUD routes for any object.
By the way most people generally say "CRUD", we usually say "CRUDL" adding the "L" for "List".
But it all refers to the same general operations.

Once again we will take an HTML-first approach using plain HTML forms as the basis for each operation.

For this module we will build a Link Tree feature with a page that has a list of links.
We want to be able to do all the CRUDL operations on these links.


## Structure: Routes and Files

- We will structure our CRUDL routes for the `links` object as follows:
- Routes:
  - `/links` - List and Create
    - GET - List and Create form in one page
    - POST - Create Post endpoint
  - `/links/$id` - Read and Update
    - GET - Read and Update form
    - POST - Update Post endpoint
  - `/links/$id/delete` - Delete
    - POST - Deletes object

> Why do we have a POST `/links/$id/delete` route instead of a DELETE `/links/$id` route? It is because browsers only support GET and POST and we want to be able to support non-JavaScript use cases with our forms.


- Files
  - API routes
    - /app/api/links.mjs
    - /app/api/links/$id.mjs
    - /app/api/links/$id/delete.mjs
  - HTML pages
    - /app/pages/links.mjs
    - /app/pages/links/$id.mjs
  - Data Access Layer (see previous module)
    - /app/models/links.mjs
    - /app/models/schema/links.mjs



### Dynamic Routes and Catch All Routes

Enhance has support for dynamic and catchall routes.
The '$' in the above route and path names will match any path part. For example the `$id` will match any object ID at the end of the `/links` route. You can access these path parameters in code like so:

```javascript
const id = req.pathParameters?.id
```

If the file or route is named with `$$` it will match any remaining path with multiple parts. Access to the value matched by `$$` is accessible at:

```javascript
const proxy = req.pathParameters?.proxy
```


## Create

Earlier in the workshop we talked about the fact that Enhance Styles does a pretty hard CSS reset.
As a result if we just build forms with inputs it is difficult to see them while iterating and debugging.
We have some form components pre built that will help with this.
Since most of our CRUDL routes are not public and just for us the styles don't have to visually match our site.

To use them you would:

- First run `npm i @enhance/form-elements`
- Add elements for each of the form elements.
  - Copy and past the code below to `/app/elements/enhance/submit-button.mjs`

```javascript
import { SubmitButton } from "@enhance/form-elements"
export default SubmitButton
```

But to save you lots of copy paste we have done that for you already.

Lets look at page that lists all the links and has a form to create a new link at `/app/pages/links.mjs`:

```javascript
// /app/pages/links.mjs
export default function Links({ html, state }) {
  const { store } = state
  let links = store.links || []

  return html`
<enhance-page-container>
  <main>
    <h1 class="mb1 font-semibold text3">Link Pages</h1>
    ${links.map(link => `<article class="mb2">
      <div class="mb0">
        <p class="pb-2"><strong class="capitalize">Link Text: </strong>${link?.text || ''}</p>
        <p class="pb-2"><strong class="capitalize">Link Url: </strong>${link?.url || ''}</p>
        <p class="pb-2"><strong class="capitalize">Link Published: </strong>${link?.published || ''}</p>
        <p class="pb-2"><strong class="capitalize">Key: </strong>${link?.key || ''}</p>
      </div>
      <p class="mb-1">
        <enhance-link href="/links/${link.key}">Edit this link page</enhance-link>
      </p>
      <form action="/links/${link.key}/delete" method="POST" class="mb-1">
        <enhance-submit-button><span slot="label">Delete this link page</span></enhance-submit-button>
      </form>
      </article>`).join('\n')}
    <details class="mb0" >
      <summary>New link page</summary>
      <enhance-form
        action="/links"
        method="POST">
        <enhance-fieldset legend="Link Page">
          <enhance-text-input label="Link Text" type="text" id="text" name="text"  ></enhance-text-input>
          <enhance-text-input label="Link Url" type="text" id="url" name="url"  ></enhance-text-input>
          <enhance-checkbox label="Published" type="checkbox" id="published" name="published"></enhance-checkbox>
          <enhance-submit-button style="float: right"><span slot="label">Save</span></enhance-submit-button>
        </enhance-fieldset>
      </enhance-form>
    </details>
  </main>
</enhance-page-container>
  `
}
```

- Now that we have a form to create new links we need a place to POST them.
- Next make an API route at `/app/api/links.mjs`

```javascript
// /app/api/links.mjs
import { upsertLink } from '../models/links.mjs'

export async function post (req) {
  await upsertLink(req.body)
  return {
    location: '/links'
  }
}
```

This will:

1. Take the form data received and store it in the database
2. Redirect back to `/links` when done

Lets finish the loop so that we can see the links we created.

## List

We want to view the list the links we've created. Instead of creating a new route. We will provide the UI for viewing and creating new links in the same page.

- Now we need to pass the data for the links to the page to display.
- For that we'll look at the `get` function in `/app/api/links.mjs`


```javascript
// /app/api/links.mjs
import { getLinks, upsertLink, validate } from '../models/links.mjs'

export async function get (req) {
  const links = await getLinks()
  return {
    json: { links }
  }
}

export async function post (req) {
  let { problems, link } = await validate.create(req)

  await upsertLink(link)
  return {
    location: '/links'
  }
}
```

## Update

We have a button to update links from the list view, but we need to add the page and API to support that feature.

First lets start with the update page and form.
This will be similar to the create form except with the addition of a key.
We will also need to pre-populate the form with the previous values so that only the updated values change.

Look at the code in `/app/pages/links/$id.mjs`.

```javascript
export default function UpdateLink({ html, state }) {
  const { store } = state
  const link = store.link || {}

  return html`<enhance-page-container>
  <enhance-form
  action="/links/${link.key}"
  method="POST">
    <enhance-fieldset legend="Link Page">
    <enhance-text-input label="Link Text" type="text" id="text" name="text" value="${link?.text || ''}" ></enhance-text-input>
    <enhance-text-input label="Link Url" type="text" id="url" name="url" value="${link?.url || ''}" ></enhance-text-input>
    <enhance-checkbox label="Published" type="checkbox" id="published" name="published" ${link?.published ? "checked" : ""}></enhance-checkbox>
    <input type="hidden" id="key" name="key" value="${link?.key}" />
    <enhance-submit-button style="float: right"><span slot="label">Save</span></enhance-submit-button>
    </enhance-fieldset>
</enhance-form>
</enhance-page-container>`
}
```

Now lets pass the initial values to the form that will be updated.

Look at the following code in the API route at `/app/api/links/$id.mjs`

```javascript
import { getLink, upsertLink } from '../../models/links.mjs'

export async function get (req) {
  const id = req.pathParameters?.id
  const result = await getLink(id)
  return {
    json: { link: result }
  }
}

export async function post (req) {
  const id = req.pathParameters?.id
  const result = await upsertLink({ ...req.body, key: id })
  return {
    json: { link: result },
    location: '/links'
  }
}

```

Notice the id comes from the path parameter ($id) rather than from the form input.

## Delete

We already added a form in the List view that will POST to delete an object.
We just need to add the API route that handles that POST request.

- Look at the following code in the `/app/api/links/$id/delete.mjs` file.

```javascript
// /app/api/links/$id/delete
import { deleteLink } from '../../../models/links.mjs'

export async function post (req) {
  const id = req.pathParameters?.id
  let link = await deleteLink(id)
  return {
    json: { link },
    location: '/links'
  }
}
```

We now have working CRUDL routes! For a toy app this might be enough, but we are missing some critical pieces. There is no authentication of the user for one thing. Lets fix that.

The thing we need for that is sessions.

That is the next module.



