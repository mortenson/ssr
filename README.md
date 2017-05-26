# Server-side rendering web components

This is currently a WIP of how one would server-side render web components.

## Why

- Lightweight rehydration of shadow content.
- Web crawlers can index both light and shadow DOM.
- Selectors work through shadow roots (possible Selenium integration), though they won't be the same on the server as on the client.

## Caveats

- While the page can be rendered without JavaScript, it won't be pretty because there is no style emulation being done on the server.

## Notes

- Uses inline an `<script>` method for rehydration. This [seems](https://discourse.wicg.io/t/declarative-shadow-dom/1904/8) to be more performant and simplifies the usage for the consumer because there's no client code. This creates more weight to send to the client, but it doesn't make any assumptions on how the page is being rendered, other than you don't mess with its output. If a shared function were created, it would make an assumption on how and where you're using the rendered content, which at this point seems like we shouldn't have an opinion on.
- Inline `<script>` tags use relative DOM accessors like `document.currentScript`, `previousElementSibling` and `firstElementChild`. Any HTML post-processing could affect the mileage of it, so beware.
- Could use a `<shadow-root>` element, but that would mean:
  - Probable performance hit (see above).
  - Requires client to include a script (friction).
  - Pollutes the custom element namespace, or requires the consumer to manually register (more friction).
- Shadow root content, prior to being hydrated, is *not* inert. Putting it inside of a `<template>` tag means that it's not participating in the document. It can't be found by a `querySelector` and likely won't be crawled.

## Controversial opinion

Require JavaScript for your users and use this for things that don't care how it looks.

## How

*For all you haters, this example is using vanilla custom elements and shadow DOM in order to show that it can work with any web component library.*

On the server (`example.js`):

```js
const { render } = require('./index');
const { customElements, HTMLElement } = window;

class Hello extends HTMLElement {
  connectedCallback () {
    const shadowRoot = this.attachShadow({ mode: 'open' });
    shadowRoot.appendChild(document.createTextNode('Hello, '));
    shadowRoot.appendChild(document.createElement('slot'));
    shadowRoot.appendChild(document.createTextNode('!'));
  }
}

customElements.define('x-hello', Hello);

const hello = new Hello();
hello.appendChild(document.createTextNode('World'));

render(hello).then(console.log);
```

And then just `node` your server code:

```
$ node example.js
<x-hello><shadow-root>Hello, <slot></slot>!</shadow-root>World</x-hello><script>var a=document.currentScript.previousElementSibling,b=a.firstElementChild;a.removeChild(b);for(var c=a.attachShadow({mode:"open"});b.hasChildNodes();)c.appendChild(b.firstChild);</script>
```

On the client, just inline your server-rendered string:

```js
<x-hello><shadow-root>Hello, <slot></slot>!</shadow-root>World</x-hello><script>var a=document.currentScript.previousElementSibling,b=a.firstElementChild;a.removeChild(b);for(var c=a.attachShadow({mode:"open"});b.hasChildNodes();)c.appendChild(b.firstChild);</script>
```

[See it in action!](https://www.webpackbin.com/bins/-Kl27vKrFK82_BDrv6h4)
