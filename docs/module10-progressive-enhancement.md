---
title: "Module 10: Progressive Enhancement"
layout: default
---


[Module Index](/enhance-workshop)

# Module 10: Progressive Enhancement

Up until this point in the workshop we have only written server side code. It just goes to show you how great the platform has gotten in the past few years. Now we are going to focus on progressive enhancement.

> The word progressive in progressive enhancement means creating a design that achieves a simpler-but-still-usable experience for users of older browsers and devices with limited capabilities, while at the same time being a design that progresses the user experience up to a more-compelling, fully-featured experience for users of newer browsers and devices with richer capabilities.
> From MDN

## Progressively Enhancing Components

Let's make the user experience for deleting links from our database a bit nicer. Currently, when you delete one of the links it requires a page reload. That's fine but we can make that user experience a bit smoother.

- Create a new file called `app/components/delete-button.mjs`. In that file copy the following contents.

```javascript
/* globals customElements, document */
import CustomElement from '@enhance/custom-element'

export default class DeleteButton extends CustomElement {
  static observedAttributes = ['key']

  keyChanged(value) {
    this.querySelector('form').setAttribute('action', `/links/${value}/delete`)
  }

  connectedCallback() {
    this.button = this.querySelector('button')
    this.button.addEventListener('click', this.#handleClick);
  }

  disconnectedCallback() {
    this.button.removeEventListener('click', this.#handleClick);
  }

  #handleClick = async event => {
    event.preventDefault()
    let element = document.getElementById(this.getAttribute('key'))
    let display = element.style.display
    element.style.display = 'none'
    let { action, method } = event.target.closest('form')
    try {
      await fetch(action, {
        method: method,
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json"
        },
      })
      element.remove()
    } catch(error) {
      console.error("Whoops!", error)
      element.style.display = display
    }
  }

  render({ html, state  }) {
    const { attrs } = state
    const { key } = attrs
    return html`
      <form action="/links/${key}/delete" method="POST" class="mb-1">
          <enhance-submit-button><span slot="label">Delete this link</span></enhance-submit-button>
      </form>
      `
  }
}

customElements.define('delete-button', DeleteButton)
```

- Those 40 lines of JavaScript are all we need to make our button more interactive. Let's go over a few of the salient points.

- The class now extends `CustomElement`.
- `attributeChangedCallback` is gone and it has been replaced  with the `keyChanged` method.
- If any attributes in the `observedAttributes` array are modified, `CustomElement` will check to see if there is a corresponding `attributeChanged` method. If yes, it will execute that method only if the value  of the attribute has changed.
- As well there is now a `render` method which is called to re-render the component on any attribute changes. This is a great way of sharing a render method between the client and the server. Isomorphic JavaScript that works for once!

> A note about the `#handleClick` function. You might be wondering why the method name starts with a `#`? We are using a private method in this case as we don't want any JavaScript outside of the web component to be able to delete our links. Deleting is a destructive action so we want to limit it to user initiated actions.
>
> Also, you may have noticed we are using an arrow function. This is purely a preference of mine but it saves you from having to bind the value of `this` to the class.

- We'll replace the existing delete button with this component in just a moment but first we need to talk about attribute handling in Enhance.

### Attributes

- Attributes are another way you can pass data into your Enhance components. For example:

```html
<delete-button key="link1"></delete-button>
```
- When server side rendered your attributes are available as key value pairs in `state.attrs`.
- Remember we are talking about HTML attributes here. This means that the value of every attribute that is passed into your web component is treated like a string. So if you pass in a number you need to convert it to an number first.
- When it comes to booleans you should check for the existence of the attribute. If the attribute exists then it's true, otherwise it's false.
- Naming attributes can be challenging as well. We strongly recommend using all lowercase attribute names like all HTML tags. Improper naming can make them hard to use with JavaScript or prevent the `attributeChangedCallback` from firing. For more info [read](https://enhance.dev/docs/learn/concepts/state/attributes#naming-attributes).

## Back to PE

- Let's make sure our new delete button is working properly.
- Create a new file `app/browser/links.mjs` and the contents should look like:

```javascript
import DeleteButton from '../components/delete-button.mjs'

export { DeleteButton }
```

- Open `app/pages/links.mjs` and replace:

```html
<form action="/links/${item.key}/delete" method="POST" class="mb-1">
  <enhance-submit-button><span slot="label">Delete this link</span></enhance-submit-button>
</form>
```

with

```html
<delete-button key="${item.key}"></delete-button>
```

- And add `<script type="module" src="/_public/browser/links.mjs"/>` to the end of that page.

- In order for this to work we need to make one more change to `app/pages/links.mjs`. When we map through all the links we will add an `id` to the `article` tag so we can do some DOM surgery.
- Navigate to `https://localhost:3333/links` and delete a few links. You'll notice, no more page refresh.
