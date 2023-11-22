---
title: "Module 4: API Design"
layout: default
---

[Module Index](/enhance-workshop)



# Module 4: API Design

## Finish index.html

Before we move on lets put some finishing touches on our home page.
We have all the pieces in place to write HTML pages using web components.

Here is some content for the developer portfolio that you can customize to make it your.
Copy and paste the markup below into your `index.html`.

```html
<nav-bar class='pb4 sticky inset-bs-0 z1'></nav-bar>
  <site-container>
    <h1 class='mb6 text5 font-light text-center tracking-2'>
      Hi — I’m <span class='font-semibold'>Axol Lotl</span><br /> </h1>

    <text-container class='leading3 mi-auto mbe6'>
      <p>
        I’m a web developer currently working at <a href='https://begin.com'>Begin</a>, where my friends and I maintain the excellent HTML first framework, <a href='https://enhance.dev'>Enhance</a>.
      </p>
      <p>
        I truly believe that the internet is the most powerful tool for bringing people together and raising our collective understanding. I have dedicated my career to building tools to enable people to share their dreams with the world. I've worked on and with the Web Platform for 25 years and plan on continuing to make web development more approachable for creatives and developers alike.
      </p>
      <p>
        To learn more about my experience, you can <a href='/resume'>check out my Résumé</a>.
      </p>
    </text-container>
  </site-container>
<site-footer></site-footer>
```

There are two new components here we have not built yet.
Lets add `<site-footer>` and `<text-container>` below to the `/app/elements` folder.

```javascript
// /app/elements/site-footer.mjs
export default function SiteFooter({ html }) {
  return html`
    <style>
      text-container {
        border-color: #ccc;
      }
    </style>
    <footer class='pbe6'>
      <site-container>
        <text-container class='mi-auto text-center border-solid border-bs1 pbs3'>
          <p class='text-1'>
            &copy; 2023 Axol Lotl</span>
          </p>
          <p class='text-1'>
            Built with <a href='https://enhance.dev'>Enhance</a>
          </p>
        </text-container>
      </site-container>
    </footer>
  `
}
```

```javascript
// /app/elements/text-container.mjs
export default function TextContainer({ html }) {
  return html`
    <style>
      :host {
        display: block;
        max-inline-size: 68ch;
      }

      p + p {
        margin-block-start: 1rem;
      }
    </style>
    <slot></slot>
  `
}
```

Yay! We now have a nice looking landing page.
It is plain HTML with static content hardcoded.
There are a lot of sites on the web that should be built that way instead of as SPA's.

Next we want to build out our Résumé page.
For that we want to pull data in rather than hardcoding the markup.
That requires some other tools.


## Enhance Project Structure

In the Enhance project structure so far we have seen the following:


- `app/pages` - HTML pages
- `app/elements` - Custom Elements as components

There are a couple more parts of the project structure we will need to handle other kinds of data.

- `app/head.mjs` - Customize the document `<head>`
- `app/api` - API routes to return data (JSON, XML, etc.), or to pass data to a corresponding `app/pages` route to make the page dynamic
- `public` - Static assets like images. Includes fingerprinting. Accessed at `/_public/file/path`

## Data Store

In recent history Static Site Generators and JAMStack was all the rage.
The idea was that nothing could be as fast as having pregenerated HTML or Javascript ready to send to the client on any request.
There is a grain of truth to this but this thinking lead to Gatsby sites that took 30 minutes to rebuild if you found a typo in the footer.
It also spread the scourge of SPA's, because if your site is pregenerated markup all dynamic behavior must be done on the client.


## Adding API Routes
API routes are a way to combine data in response to an HTTP request.

They can respond with data (JSON, XML, etc).
Or the data can be passed to a page route make dynamic HTML markup based on the data.

Add the following code to a new API route at `/app/api/resume.mjs`:


```javascript
// /app/api/resume.mjs
export async function get(req){

  const experience = [
    {
      company: 'Begin',
      role: 'Web Developer',
      start: 2015,
      end: null,
      description: 'Currently leading development of Enhance, an HTML first full stack framework leveraging web components and making it easier to work with existing web platform technologies.'
    },
    {
      company: 'Loxa Design Studio',
      role: 'Designer',
      start: 2007,
      end: 2015,
      description: 'I lead the design team at Loxa for nearly 8 years. Together, we crafted end to end design solutions for clients, touching on information architecture, interface design, and user experience.'
    },
    {
      company: 'Axol Web Widgets Co.',
      role: 'Founder',
      start: 2001,
      end: 2007,
      description: 'Axol Web Widgets Co. was a design and development studio. I founded the company and was its lead developer, specializing in creating Macromedia Flash intros, WordPress themes, and WinAmp skins.'
    },
  ]

  return { json: {experience} }
}
```

Here we create an array of Jobs for our résumé.

## API Routes
The API route data can be hard coded (as it is here), read from a database, fetched from other third party API's, or imported from anywhere.
This is where the backend work happens to respond to requests.

**Notice that this function recieves the request object as an argument (show as `req`) which can be used for preparing the response.**

The exported function from this api file is named for the method that it will handle.
In the above example it is name `get` and will respond only to GET requests.
It can be post, patch, delete, or any other method.
We will generally stick to `get` and `post` because these are the only two methods that the browser understands.
We can also use `any` to indicate this method should respond to any method request.


When the response is ready you `return` the response object.

To pass data on to a corresponding HTML page put the data in the `json` property in the response.
If there is a matching `/app/pages` route defined this data will be passed to that page to use in preparing the HTML.
- JSON vs. HTML
    - If both an API and Pages route exist then the response is determined by the requests `accepts` header. If it is set to `accept: application/json` it will return the JSON directly from the API route.
    - If it is set to  `accept: text/html` it will pass the JSON data on to the pages route for rendering.

The response can include other properties including:
  - json: JSON data response or pass data to HTML page
  - status: HTTP status code
  - location: Redirect location headers
  - headers: Set arbitrary headers
  - session: Set session cookies
Note: you may return any type of data from an API request. See [Responses](https://arc.codes/docs/en/reference/runtime-helpers/node.js#responses) for more details.


## Dynamic HTML pages

Now that we have résumé data passed to our `/app/pages/resume.mjs` how do we make use of it?
Lets create an element to show our résumé data.

We can access it in the data store.
Look at the code below.
Notice the second argument to the function called `state`.
This contains the data passed from the API route in a property called `store`.

```javascript
// /app/elements/resume/experience.mjs
export default function Experience ({ html, state }) {
  const { store } = state
  const { experience = [] } = store
  const jobs = experience.map(job => {
    const { company, role, start, end, description } = job
    return `
      <article class='grid-lg gap4-lg col-2 mbe4 leading3'>
        <h2 class='mbe0'>
          <span class='uppercase tracking1 font-semibold'>${company}</span><br />
          <span class='uppercase tracking1 text-1'>${start}–${end ? end : 'present'}</span>
        </h2>

        <dl>
          <dt>Role</dt>
          <dd>${role}</dd>

          <dt>Description</dt>
          <dd>${description}</dd>
        </dl>
      </article>
    `
  }).join('')

  return html`
    <section class='mi-auto'>
      ${jobs}
    </section>
  `
}
```
Copy and paste this code into `/app/elements/resume/experience.mjs`.

This will create a `<resume-experience>` tag to display our jobs.

Lets drop it in our resume page and watch the data flow.


```html
<!-- /app/pages/resume.html--->
<nav-bar class='pb4 sticky inset-bs-0 z1'></nav-bar>
<site-container>
    <h1 class='mb6 text5 font-light text-center tracking-2'>
      Résumé
    </h1>
    <resume-experience></resume-experience>
</site-container>
```


Now we have the data we want lets finish this résumé with some styles.
The data list needs a little work.
Lets make a component `<data-list>` to apply the styles to:

```javascript
// /app/elements/data-list.mjs
export default function DataList({ html }) {
  return html`
    <style>
      dt {
        color: var(--grey-400);
        font-weight: 600;
      }

      dd + dt {
        margin-block-start: var(--space-0);
      }
    </style>
    <dl>
      <slot></slot>
    </dl>
  `
}
```

Now lets update `<resume-experience>` to use that.
Replace the `<dl>` with our `<data-list>`.

```javascript
// /app/elements/resume/experience.mjs
export default function Experience ({ html, state }) {
  const { store } = state
  const { experience = [] } = store
  const jobs = experience.map(job => {
    const { company, role, start, end, description } = job
    return `
      <article class='grid-lg gap4-lg col-2 mbe4 leading3'>
        <h2 class='mbe0'>
          <span class='uppercase tracking1 font-semibold'>${company}</span><br />
          <span class='uppercase tracking1 text-1'>${start}–${end ? end : 'present'}</span>
        </h2>

        <data-list class=''>
          <dt>Role</dt>
          <dd>${role}</dd>

          <dt>Description</dt>
          <dd>${description}</dd>
        </data-list>
      </article>
    `
  }).join('')

  return html`
    <section class='mi-auto'>
      ${jobs}
    </section>
  `
}
```

Now create the résumé container in the elements folder.

```javascript
// /app/elements/resume/container.mjs
export default function ResumeContainer({ html }) {
  return html`
    <style>
      :host {
        display: block;
        max-inline-size: 84ch;
      }
    </style>
    <slot></slot>
  `
}
```

Now back to `/app/pages/resume.html`.
Lets add a footer and a résumé container.

```html
<nav-bar class='pb4 sticky inset-bs-0 z1'></nav-bar>
<site-container>
    <h1 class='mb6 text5 font-light text-center tracking-2'>
      Résumé
    </h1>
    <resume-container class='mi-auto'>
      <resume-experience></resume-experience>
    </resume-container>
</site-container>
<site-footer></site-footer>
```

We have a decent looking Résumé page along with tools to add data to create pages that are dynamic.

But what if we wanted to pass some data to all the pages instead of just a single page.

## Passing Data Using `head.mjs`

We briefly mentioned the `head.mjs` in the custom elements module.
It is where we put a link tag for styles if we want to pull in custom stylesheets.
Your project already has one, let's update it so that looks like this:

```javascript
// /app/head.mjs

import { getStyles }  from '@enhance/arc-plugin-styles'

const { linkTag } = getStyles

export default function Head() {

  return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <meta name="og:type" content="website" />
      <link rel="icon" href="/_public/favicon.svg">
      <title>Axol Lotl</title>
      <meta name="description" content="Portfolio for Axol Lotl, Senior Developer" />
      ${linkTag()}


      <style>
        body { background: white; }
        @font-face {
          font-family: "HK Grotesk";
          font-weight: 300;
          src: url("/_public/fonts/HKGrotesk-Light.woff2") format("woff2")
        }

        @font-face {
          font-family: "HK Grotesk";
          font-weight: 400;
          src: url("/_public/fonts/HKGrotesk-Regular.woff2") format("woff2")
        }

        @font-face {
          font-family: "HK Grotesk";
          font-weight: 600;
          src: url("/_public/fonts/HKGrotesk-SemiBold.woff2") format("woff2")
        }

        body {
          color: var(--dark);
          text-rendering: optimizeLegibility;
        }

        a {
          text-decoration: underline;
          text-decoration-thickness: 0.0625em;
          text-underline-offset: 0.0625em;
        }
      </style>
    </head>
    <body class='font-sans'>
`
}
```

The only thing dynamic in this is the `linkTag()` which adds a link to enhance styles.
Otherwise this file generates the string for the html, head, and body tags.

It is the first element processed by enhance to create the page.
It is a special element because the head cannot be a custom element.
But when it is processed we have access to the request and the store.

This gives us an opportunity to inject data into the store here that will be available to every other element processed.

For instance we have a nav bar and footer where the author info is hard coded.

Lets make that dynamic and pull it from the store.

The first argument of `head.mjs` is an object that contains the request and the store.
We could read the store here to put something in the head, but instead we will inject our author data.

Just in case we want to override this for a specific route we will check to see if an author is already passed and leave it there if it is.

```javascript

import { getStyles }  from '@enhance/arc-plugin-styles'

const { linkTag } = getStyles

export default function Head(state) {
  const { req, store } = state

  if (store.author === undefined) {
    store.author = {
      name: 'Axol Lotl',
      title: 'Web Developer',
      githubUsername: 'enhance-dev',
    }
  }

  return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      /* shortened */
    </head>
    <body class='font-sans'>
`
}

```

That data is now available.
Lets update the nav bar to use it.
We will add the author name, title, and add an avatar as an image from the github profile.
You can either make the changes below or copy and paste this into `/app/elements/nav-bar.mjs`


```javascript
export default function NavBar({ html, state }) {
  const { store } = state
  const { author } = store
  return html`
    <style>
      :host {
        display: block;
        position: relative;
      }

      .backdrop {
        backdrop-filter: blur(2px);
        background: hsla(0deg 0% 100% / 0.9);
        --mask-image: linear-gradient(to bottom, black 50%, transparent);
        mask-image: var(--mask-image);
        -webkit-mask-image: var(--mask-image);
        inset-block-end: -20%;
      }

      img {
        border-radius: 0.25em;
        height: 2.25em;
        width: auto;
      }
    </style>
    <site-container>
      <nav class='flex align-items-center gap0 leading1'>
        <a href='/' class='no-underline flex align-items-center gap0'>
          <img src='https://github.com/${author.githubUsername}.png' alt='Avatar for Axol Lotl' />
          <h1 class='font-semibold tracking-1'>
            ${author.name}<br />
            <span class='font-normal'>${author.title}</span>
          </h1>
        </a>
        <ul class='mis-auto flex gap0 list-none text-1 uppercase tracking1 font-semibold'>
          <li><a href='/'>Home</a></li>
          <li><a href='/resume'>Résumé</a></li>
        </ul>
      </nav>
      <div class='backdrop absolute inset-0 z-1'></div>
    </site-container>
  `
}
```

Now the same with the site-footer.
Lets also make the data dynamic while we are at it.


```javascript
export default function SiteFooter({ html, state }) {
  const { store } = state
  const { author } = store
  return html`
    <style>
      text-container {
        border-color: #ccc;
      }
    </style>
    <footer class='pbe6'>
      <site-container>
        <text-container class='mi-auto text-center border-solid border-bs1 pbs3'>
          <p class='text-1'>
            &copy; ${new Date().getFullYear()} ${author.name}</span>
          </p>
          <p class='text-1'>
            Built with <a href='https://enhance.dev'>Enhance</a>
          </p>
        </text-container>
      </site-container>
    </footer>
  `
}

```

Since we are updating the head.mjs lets make the title metadata dynamic.
Lets make an object with titles for different paths and put that in the `/app/lib/titles-by-path.mjs`.

```javascript
// /app/lib/titles-by-path.mjs
export default {
  '/': 'Senior Developer',
  '/resume': 'Résumé',
  '/linktree': 'Links',
}
```

We can use the `state.req.path` in the `head.mjs` to find the current path that we are on and put that in the `<title>`.

We also add the path to the data store so that any other elements can access it if needed (i.e. highlighting current location in nav bar).


```javascript
import titlesByPath from './lib/titles-by-path.mjs'
import { getStyles }  from '@enhance/arc-plugin-styles'

const { linkTag } = getStyles

export default function Head(state) {
  const { req, store } = state
  const { path, session } = req

  if (store.path === undefined) {
    store.path = path
  }
  if (store.author === undefined) {
    store.author = {
      name: 'Axol Lotl',
      title: 'Web Developer',
      githubUsername: 'enhance-dev',
    }
  }

  const title = titlesByPath[path] || ''

  return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <meta name="og:type" content="website" />
      <link rel="icon" href="/_public/favicon.svg">
      <title>Axol Lotl: ${title}</title>
      <meta name="description" content="Portfolio for Axol Lotl, Senior Developer" />
      ${linkTag()}

      <style>
        @font-face {
          font-family: "HK Grotesk";
          font-weight: 300;
          src: url("/_public/fonts/HKGrotesk-Light.woff2") format("woff2")
        }

        @font-face {
          font-family: "HK Grotesk";
          font-weight: 400;
          src: url("/_public/fonts/HKGrotesk-Regular.woff2") format("woff2")
        }

        @font-face {
          font-family: "HK Grotesk";
          font-weight: 600;
          src: url("/_public/fonts/HKGrotesk-SemiBold.woff2") format("woff2")
        }

        body {
          color: var(--dark);
          text-rendering: optimizeLegibility;
        }

        a {
          text-decoration: underline;
          text-decoration-thickness: 0.0625em;
          text-underline-offset: 0.0625em;
        }
      </style>
    </head>
    <body class='font-sans'>
`
}

```


Now we have data flowing throughout our site.
At this point the source of the data is mostly static, but the same tools can be used to pass any kind of data around.



