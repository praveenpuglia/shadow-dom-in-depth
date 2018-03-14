# Shadow DOM

> Heads Up! It's all about the V1 Spec.

## What's New?

* Examples. I am putting examples that'll help everyone understand it better, step by step. [Check them out.](./examples)

Let me know if a "Everything you need to know about Custom Elements" document like this one would help you. If so, I'll put one up 👨‍💻.

## Browser Support

* Chrome : Works
* Firefox : In development & behind feature flag.
* Opera : Works
* Safari : Works but few things are buggy.
* Edge : Under Consideration.

Comprehensive browser support info here - https://caniuse.com/#feat=shadowdomv1.

## Introduction

In a nutshell, Shadow DOM enables local scoping for HTML & CSS.

> Shadow DOM fixes CSS and DOM. It introduces scoped styles to the web platform. Without tools or naming conventions, you can bundle CSS with markup, hide implementation details, and author self-contained components in vanilla JavaScript. - https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom

It's like it's own little world which hardly affects or gets affected by the outside world.

It's what you write as a **component author** to abstract away the implementation details of your component. It can also decide what to do with the user-provided **light DOM**.

## Terminologies

**- DOM :** What we get over the wire (or wireless :|) is a string of text. To render something on the screen, the browsers have to parse that string of text and convert it into a data model so it can understand things better. It also preserves the hierarchy from the original string by creating putting those parsed objects in a tree structure.

We need to do that to make the machines understand our documents better. This tree like data model of our document is called Document Object Model.

**- Component Author :** The person who creates a component and defines how it works. Generally the person who writes a lot of shadow DOM code. Example - Browser vendors create the `input` element.

**- Component User :** Well, they use components built by authors. They can pass in light DOM and set attributes and properties on your component. They can even extend the internals of a component if they want. Example - We the user who use the `input` element.

**- Shadow Root:** It's what gets attached to an element to give that element it's shadow DOM. Technically it's a non-element node, a special kind of _Document Fragment_.

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ISOLATION
    #shadow-root
        ...
    ____________________________________________ DOCUMENT FRAGMENT

    <!--LIGHT DOM-->
</custom-picture>
```

Throughout the document, I have put shadow root inside those weird ASCII boundaries. This will put more emphasis on thinking how they are actually document fragments and they have a wall around them.

**- Shadow Host:** The element to which the shadow root gets attached. A host can access it's shadow root via a property on itself. `.shadowRoot`.

**- Shadow Tree :** All the elements that go into the Shadow Root, which is scoped from outside world, is called Shadow Tree.

> The elements in a shadow tree are **not** descendants of the shadow host in general (including for the purposes of Selectors like the descendant combinator) - Spec

**- Light DOM:** - The set of DOM elements we can sandwich between the opening and closing tags. - The DOM that lives outside shadow DOM. - The DOM, the _user_ of your element writes. - The DOM that is the actual children of your element.

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
    ___________________________

    <!--Light DOM-->
    <img src="https://path.to/a-kitten.png">
    <cite>A Nice Kitten!</cite>
    <!--Light DOM ends-->
</custom-picture>
```

**- DocumentFragment:**

> The DocumentFragment interface represents a minimal document object that has no parent. It is used as a light-weight version of Document to store a segment of a document structure comprised of nodes just like a standard document. The key difference is that because the document fragment isn't part of the actual DOM's structure, changes made to the fragment don't affect the document, cause reflow, or incur any performance impact that can occur when changes are made. - [MDN](https://developer.mozilla.org/en/docs/Web/API/DocumentFragment)

## How to create Shadow DOM?

```html
<div class="dom"></div>
```

```js
let el = document.querySelector('.dom');
el.attachShadow({ mode: 'open' });
// Just like prototype & constructor bi-directional references, we have...
el.shadowRoot; // the shadow root.
el.shadowRoot.host; // the element itself.

// put something in shadow DOM
el.shadowRoot.innerHTML = 'Hi I am shadowed!';

// Like any other normal DOM operation.
let hello = document.createElement('span');
hello.textContent = 'Hi I am shadowed but wrapped in span';
el.shadowRoot.appendChild(hello);
```

### Off Topic Question - Could we use `append()` instead of `appendChild()` ?

Yes! But here are the differences from MDN.

* `ParentNode.append()` allows you to also append DOMString object, whereas `Node.appendChild()` only accepts Node objects.
* `ParentNode.append()` has no return value, whereas `Node.appendChild()` returns the appended Node object.
* `ParentNode.append()` can append several nodes and strings, whereas `Node.appendChild()` can only append one node.

### What happens if we use an `input` element instead of the div to attach the shadow DOM?

Well, it doesn't work. Because the browser already hosts it's own shadow DOM for those elements. Bunch of red colored english alphabets will be thrown at console's face. 😰

## Notes & Tips

* Shadow DOM, cannot be removed once created. It can only be replaced with a new one.
* If you are creating a custom element, you should be creating the shadowRoot in it's constructor. It can also be probably called in `connectedCallback()` but I am not sure if that introduces performance problems or any other problems. 🤷‍♂️
* To see how browsers implement shadow DOM for elements like `input` or `textarea`, Go to `DevTools > Settings > Elements > [x] Show user agent shadow DOM`.

## Shadow DOM Modes

### Open

You saw the `{mode: "open"}` in the `attachShadow()` method right? Yeah! That's it. What open mode does is that it provides a way for us to access the shadow DOM of an element in JS via the element. It also lets us access the host element from within the shadow DOM.

This is done by the two implicit properties created when we call `attachShadow()` in `open` mode.

1.  The element gets a property called `shadowRoot` which points to the shadow DOM being attached to it.
2.  The `shadowRoot` gets a property called `host` pointing to the element itself.

```js
// From the "How to create shadow DOM" example
el.attachShadow({ mode: 'open' });
// Just like prototype & constructor bi-directional references, we have...
el.shadowRoot; // the shadow root.
el.shadowRoot.host; // the el itself.
```

### Closed

Pass `{mode: "closed"}` to `attachShadow()` to create a closed shadow DOM. It makes the shadow DOM inaccessible from JS.

```js
el.shadowRoot; // null
```

### So which one should I use?

Almost always use `open` mode shadow DOMs because they make it possible for both the component author and user to change things how they want.

Remember we did `el.shadowRoot` stuff up there? Yeah! That won't work with `closed` mode. The element doesn't get any reference to it's shadow DOM. This is especially a problem when creating custom elements and want to access the shadow DOM for manipulation.

```js
class CustomPicture extends HTMLElement {
  constructor() {
    this.attachShadow({ mode: 'open' }); // this.shadowRoot exists. add or remove stuff in there using this ref.
    this.attachShadow({ mode: 'closed' }); // this.shadowRoot returns null. Bummer!
  }
}
// You could always do the following in your constructor.
// but it totally defies the purpose of using closed mode.
this._shadowRoot = this.attachShadow({ mode: 'closed' });
```

Also, closed mode isn't a security mechanism. It just gives a fake sense of security. Nobody can stop someone from modifying how `Element.prototype.attachShadow()` works.

## Styles inside Shadow DOM.

* They are scoped.
* They don't leak out.
* They can have simple names.
* They are cool, be like them. 😎

```html
<custom-picture>
    #shadow-root
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        <style>
            /*Applies only to spans inside shadow DOM. Doesn't leak out.*/
            span {
                color: red;
            }
        </style>
        <span>Hello!</span>
        __________________________________________________________________
</custom-picture>
```

### Oh! oh! So can we also include a stylesheet?

Yeeeeaaaah.. but not in all browsers. 😕

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <!--All styles coming from custom-picture.css will be scoped inside this shadow root-->
        <link rel="stylesheet" href="custom-picture.css">
        <span>Hello!</span>
    _____________________________________________________
```

### Does it get affected by global CSS?

Yeah. In some ways. Only the properties that are inherited will make their way through the shadow DOM boundary. Example -

* color
* background
* font-family etc.

The `*` selector also affects things because `*` means all elements and that includes the element to which you are attaching the shadow root too, the host element. Things which get applied to the host and can be inheried, will pass the shadow DOM boundary to apply to inner elements.

## Styles Terminologies

* :host: targets the host element. BUT!

```css
/* winner */
custom-picture {
    background: red;
}
/* loser */
#shadow-root
    <style>
        :host {
            background: green;
        }
    </style>
```

* :host(`<selector>`): Does the component **host** match the **selector** ? Basically to target different states of the same host. Examples

```css
:host([disabled]) {
  ...;
}

:host(:focus) {
  ...;
}

:host(:focus) span {
  /*change all spans inside the element when the host has focus*/
}
```

* :host-context(`<selector>`): Is the **host** a descendant of **selector** ? Lets us change component's styles based on how the parent looks like. General application could be in theming.

```css
:host-context(.light-theme) {
  background: lightgray;
}

:host-context(.dark-theme) {
  background: darkgray;
}

/*You can also do...*/
:host-context(.aqua-theme) > * {
  color: aqua; /* lame */
}
```

### A note about :host() and :host-context()

Both of these functional pseudo classes can only take `<compound-selector>` but not a combinator like space or ">" or "+" etc.
In case of `:host()` that means you can only choose the attributes and other aspects of the host element.

In case of `:host-context()` that means you can choose the attributes and other aspects of one particular element which is an ancestor of the `:host`. Only one!

### Do the modes in `attachShadow()` affect how styles get applied/cascaded?

Nope! Only affects how we do things in JS.

### How does the user-agent styles get applied to even the shadow DOM elements?

Based on how shadow root or in general DocumentFragment works, user-agent styles (global) shouldn't have applied to all the elements inside a shadow root. So how do they work?

From the spec...

> `Window`s gain a private slot `[[defaultElementStylesMap]]` which is a map of local names to stylesheets. This makes it possible to write elements inside shadow root and still get the default browser styles applied to them.

### Which styles to put in shadow DOM ?

The purpose of having styles inside a shadow DOM is to just have default styles and provide hooks via CSS custom properties so component users can make changes to those default styles via, CSS custom properties (a.k.a CSS variables)

```html
<business-card>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <h1 class="card-title">Hardcoded Title - </h1>
    ---------------------------------------------------
</business-card>
```

```css
/*inside shadow DOM*/
.card-title {
  color: var(--card-title-color, #000);
}

/* Component users can then override this color as*/
business-card {
  --card-title-color: magenta;
}
```

### Notes from the Specs

* A shadow host is outside of the shadow tree it hosts, and so would ordinarily be untargettable by any selectors evaluated in the context of the shadow tree (as selectors are limited to a single tree), but it is sometimes useful to be able to style it from inside the shadow tree context.
* To work around that problem, the shadow host is treated as replacing the shadow root node.
* When considered within its own shadow trees, the shadow host is featureless. Only the `:host`, `:host()`, and `:host-context()` pseudo classes are allowed to match it.

## Events in Shadow DOM.

Events work towards maintaining the encapsulation provided by the shadow DOM. Essentially, if an event occurs somewhere in shadow DOM, to outside world it'll look as if that event has triggered from the host element itself and not a secific part of the shadow DOM. This is called re-targeting of the event.

Inside the shadow DOM however the events aren't retargered and we can find out which specific element was an event associated with.

So say if our flattened DOM tree looks like this..

```html
<body>
    <custom-picture>
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #shadow-root
            <button> Hello </button>
        ---------------------------------
    </custom-picture>
</body>
```

On click of the button, to body, or anywhere outside custom-picture, the `event.target` will point to the `<custom-picture>` itself.

> If the shadow tree is open, calling `event.composedPath()` will return an array of nodes that the event traveled through.

Inside `<custom-picture>` however, the event target will be the button which was really clicked.

Most events go out of shadow DOM boundary and when they do, they get re-targered. Some events aren't allowed to pass that boundary. Precisely these...

* abort
* error
* select
* change
* load
* reset
* resize
* scroll
* selectstart

## Slots

Slots are a pretty big thing in Shadow DOM.

> Slots are placeholders inside your component that users can fill with their own markup. - https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#slots

When creating custom components, we want to be able to provide only the necessary markup that goes into a particular component and use/group/style that as we want to as component authors.

The DOM that a component user provides is called light DOM and slots are the way we can arrange that markup however we want. Style based on differnt situations and group them.

There are two aspects of a slot.

* **Light DOM Elements :** They say which slot they wanna go into.

```html
<custom-picture>
    <!--Light DOM <img> saying it should be put into the "profile-picture" slot-->
    <img src="assets/user.svg" slot="profile-picture">
</custom-picture>
```

* **The Actual Slot :** The `<slot>` element, residing somewhere in shadow DOM with a name for itself to be found by light DOM.

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <slot name="profile-picture">
            <!--The <img> from the light DOM gets rendered here!-->
        </slot>
    _________________________________________
```

### A very important note about how light DOM and slots play together.

> Elements are allowed to "cross" the shadow DOM boundary when a <slot> invites them in. These elements are called distributed nodes. Conceptually, distributed nodes can seem a bit bizarre. Slots don't physically move DOM; they render it at another location inside the shadow DOM. - https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#slots

### What if I don't provide the `slot` attribute in the `<img>` in `<custom-picture>` ?

You'll see nothing rendered. Here's why.

1.  A host element that has shadow DOM, only renders stuff that goes inside it's shadow DOM.
2.  In order to get light DOM elements rendered, they need to be part of the shadow DOM.
3.  The way we make them part of the shadow DOM is by putting them in slots.
4.  In our example above, there's no element in light DOM that wants to go into a slot named `profile-picture`.
5.  Since there's no one, the `<img>` from light DOM is not rendered.
6.  Takeaway - Named slots accomodate only those light DOM elements which specify they wanna go into that particular slot.

### What if I want to render all elements that don't say where they should go ?

This will require us to have a general purpose slot in our shadow DOM. A slot without a name.
Example -

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <!--General purpose slot, render every element from light DOM that doesn't mention a slot name, here.-->
        <slot>
            <!--The <img> from the light DOM gets rendered here!-->
        </slot>
    _________________________________________
    <!--Light DOM-->
    <img src="assets/user.svg">
</custom-picture>
```

### What if I add two unnamed slots?

Woah! 😲
Actually, we can have multiple unnamed slots or even named slots but they are essentially useless.
The light DOM elements will go into the first slots they match. So in the case above...

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <slot>
            <!--The <img> from the light DOM gets rendered here! Winner!-->
        </slot>
        <slot>
            <!--Doesn't come here!-->
        </slot>
    _________________________________________
    <!--Light DOM-->
    <img src="assets/user.svg">
</custom-picture>
```

### What if there's a slot but no light DOM elements wants to go there?

Nothing will be rendered, unless there's a fallback content the slot itself provides and how to do that?

```html
#shadow-root
    <slot name="nobody-comes-here">
        <h1> I'll show up when noone does!</h1>
    </slot>

    <style>
        /* And that fallback can be styled from within the shadow DOM just like we do styles*/
        slot[name="nobody-comes-here"] h1{
            color: #bada55;
        }
    </style>
```

### So what are slotted elements and how can we style them?

Light DOM elements that go into a slot are called slotted elements. As mentioned above, these are also called distributed elements which cross the shadow DOM boundary.

These slotted elements can be styled using the `::slotted()` functional pseudo element. The syntax is as follows.

```css
::slotted(<compound-selector >) {
  /* styles */
}
```

Example -

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <slot>
            <!--The <img> from the light DOM gets rendered here!-->
        </slot>

        <style>
            /* find the slotted image and set their width and height */
            ::slotted(img) {
                width: 256px;
                height: 256px;
            }
        </style>
    _________________________________________
    <!--Light DOM-->
    <img src="assets/user.svg">
</custom-picture>
```

Here's how the spec formally defines it.

> The ::slotted() pseudo-element represents the elements assigned, after flattening, to a slot. This pseudo-element only exists on slots.

Flattened trees are [here](https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#lightdom).

Important thing to remember is that only direct children of the host element can be assigned to a slot. For example

```html
<custom-picture>
    <div class="picture-wrapper">
        <!--This won't work! Slots can't pick descendants out of the host element's light DOM tree and put them in.-->
        <img src="assets/user.svg" slot="assign-me" />
    </div>
</custom-picture>
```

I, however, don't know why that's not possible and what are the reasons behind it. So I created [this bug](https://github.com/w3c/csswg-drafts/issues/1530);

### How can I pass a light DOM element multiple levels down?

It may sound like we don't need to think about this scenario but it's often required.

```html
<parent-element>
    <!--parent-element uses child-element in it's shadow DOM and we want this span to render inside that child-element's shadow DOM-->
    <span slot="parent-slot">Finally</span>
</parent-element>
```

Here's what our custom elements look like.

```js
class ParentElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
        <!--We specify a slot property on the slot itself. Which specifies where it goes in the child-element's shadow DOM-->
        <child-element>
            <slot name="parent-slot" slot="child-slot"></slot>
        </child-element>
    `;
  }
}

class ChildElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <slot name="child-slot">
        <!--The span with the thext "Finally" gets rendered here!-->
      </slot>
    `;
  }
}

window.customElements.define('parent-element', ParentElement);
window.customElements.define('child-element', ChildElement);
```

### Slots have JavaScript API

* To find the elements that went into a slot - [`slot.assignedNodes()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSlotElement/assignedNodes);
* To find out which slot an light DOM element is assigned to - [`element.assignedSlot`](https://developer.mozilla.org/en-US/docs/Web/API/Element/assignedSlot);

## Resources

* http://robdodson.me/shadow-dom-css-cheat-sheet/
* https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom
* https://drafts.csswg.org/css-scoping/

## More Queries & Bugs

* https://stackoverflow.com/questions/44564366/inheritance-inside-a-shadow-dom-slot
* https://github.com/w3c/csswg-drafts/issues/1535
* https://github.com/w3c/csswg-drafts/issues/1530
* https://twitter.com/praveenpuglia/status/876677801146896384
