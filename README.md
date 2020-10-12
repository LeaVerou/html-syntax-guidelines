# HTML syntax guidelines

This is a draft, WIP document aiming to collect the various de facto design decisions that permeate the syntax of modern HTML elements and their APIs.

## Related Work

Other (generally higher level) guidelines for writing custom elements:

- https://github.com/webcomponents/gold-standard/wiki
- https://w3ctag.github.io/webcomponents-design-guidelines/#native-html-elements
- https://developers.google.com/web/fundamentals/web-components/best-practices
- https://github.com/HelixDesignSystem/helix-ui/wiki/Custom-Elements

## Attributes & Properties

### Naming
- Use all lowercase in docs. 
- No camelCase, no hyphens, no underscores.
- Prefer nouns
- Prefer single words (after the hyphen in custom elements). Two words are suboptimal but ok. Avoid names of three words or more.
- Avoid abbreviations to save only a few characters. 
	- E.g. it's `<video>`, not `<vid>`, `<source>` not `<src>`
	- Many older element names are abbreviated. At the time, saving characters was very important, but these days readability is a bigger focus
- See also [W3C TAG naming principles](https://w3ctag.github.io/design-principles/#naming-is-hard)

### Values

- Avoid mixing languages. No JSON or JS in attributes.
- Avoid HTML in attributes. HTML should be element content, not attribute content.
	- Unlike: `iframe[srcdoc]`
	- Would formatting be useful? Use a child element of a specific type
		- Like: `<caption>` inside `<table>`, `<figcaption>` inside `<figure>`

### By type

- Boolean:
	- Attribute presence is true (regardless of value), attribute omission is false.
		- Like: `checked`, `selected`, `disabled`, `required`, `ol[reversed]`, `script[async]`
		- Like `<audio>`/`<video>` `loop`, `controls`, `autoplay`
		- Unlike: ARIA boolean attributes
	- As a side effect, attributes need to be designed such that the default is always false.
	- Additional data can be provided via an optional attribute value
		- Like: `a[download]`
	- Mirror the attribute in a boolean property with the same name, which returns `true` or `false`
- Arrays
	- Small, unformatted lists of values can be space-separated attributes
		- Like: `class`, `iframe[sandbox]`
	- For larger lists, reference to other subtree
		- Like: `input[list]`
- Key-value pairs
	- Contents or reference to subtree if many
		- Like: `<select>`, `<datalist>`
	- For small lists, comma-separated pairs where the key is an unquoted ident, and the value is a quoted string
		- Like: [`iframe[allow]`](https://wiki.developer.mozilla.org/en-US/docs/Web/HTTP/Feature_Policy/Using_Feature_Policy#The_iframe_allow_attribute)
- Reference to other element
	- Many different patterns here:
		- HTML tends to use ID refs 
			- Like: input[list], label[for]
			- That is painful, especially in repeated structures where it forces tooling for id generation.
			- Perhaps the <label> model is best: id ref works, but nesting of a certain element type also works. However, this is not always feasible.
		- SVG uses `#id`, which at least is extensible to selectors in general
		- No precedent for relative selectors (e.g. "the element with the class .foo that is after the current element"). Perhaps [`:scope`](https://drafts.csswg.org/selectors-4/#the-scope-pseudo) or [`&`](https://drafts.csswg.org/css-nesting-1/#direct) could be used (without `@nest`).
	- Avoid references to other elements when not necessary. E.g. it's best for an element to manage its own state than a parent element pointing to it. 
		- Like: `input[type=radio][checked]`, `select > option[selected]`
		- Unlike: [`<generic-tabs selected="1">`](https://genericcomponents.netlify.app/generic-tabs/demo/index.html)
			- Never reference by child index. This is flimsy (reference changes as children are added/removed), and not idiomatic to HTML.
- Color
	- Are you absolutely sure this is not presentational? Most HTML attributes that take colors as values are deprecated.
	- The only non-deprecated precedent in HTML is `input[type=color][value]` which accepts hex colors, but there are discussions to expand this, as it limits it to sRGB.
	- Accepting any CSS <color> value seems the way to go.
- URL
	- `href` or `src` attribute with URL as its value
		- Unlike: `object[data]`
- Dates, times, durations
	- Use the [same format as `time[datetime]`](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-time-element)

## Elements
 
- Naming
	- Prefer nouns, avoid verbs
	- See also [W3C TAG naming principles](https://w3ctag.github.io/design-principles/#naming-is-hard)
- Only use specific element types for specifying specific pieces of data, otherwise handle any child 
	- Like: `<figure>` and `<figcaption>`
	- Parent needs to be able to communicate state summary without having to traverse children
		- Like: `HTMLSelectElement#value`
- Update the DOM when properties change, to enable attribute selectors
	- Like: `<details>`
	- Unlike: `<input>`, `<select>`

## Events

- One or two words, joined 
	- Like: input, change, blur, error, beforeunload, hashchange
	- Unlike: DOMContentLoaded, SVGScroll
- No abbreviations
	- Unlike: DOMAttrModified
- Format `[ before | after ]? <noun>? <verb> [start | end]?`
- Use present tense for verbs
	- Like: `change`, `input`, `open`, `load`
	- Unlike: `appinstalled`, `connected`
- Use standard events where applicable, e.g. `blur` instead of `slBlur`
- See also [W3C TAG Design Principles on Event Design](https://w3ctag.github.io/design-principles/#event-design)

## Methods

- Use standard methods when possible, e.g. `element.focus()` instead of `element.setFocus()`
