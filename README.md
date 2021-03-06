# HTML syntax guidelines

This is a draft, WIP document aiming to collect the various de facto design decisions that permeate the syntax of modern HTML elements and their APIs.
It is primarily aimed at custom element authors, but hopefully some of these may be more broadly useful.

## Related Work

Other (generally higher level) guidelines for writing custom elements:

- [Gold Standard](https://github.com/webcomponents/gold-standard/wiki)
- [W3C TAG Web Components Design Guidelines](https://w3ctag.github.io/webcomponents-design-guidelines/#native-html-elements)
- For naming: [W3C TAG naming principles](https://w3ctag.github.io/design-principles/#naming-is-hard)
- [Google WC Best Practices](https://developers.google.com/web/fundamentals/web-components/best-practices)
- [Helix UI Best Practices](https://github.com/HelixDesignSystem/helix-ui/wiki/Custom-Elements)
- [Domenic comment on light DOM vs shadow DOM for children](https://github.com/w3c/csswg-drafts/issues/3431#issuecomment-453130662)

## Attributes & Properties

- Do not set attributes on the light DOM in the constructor. Use `ElementInternals` for specifying default ARIA roles instead of directly setting ARIA attributes on the element.
- Reflect all primitive attributes as properties.

### Attribute, child element, JS property, or CSS custom property?

- As a general rule of thumb, if the text is going to be presented in the UI, opt towards a child element. Child elements can have HTML, as well as i18n attributes (`dir`, `lang` etc).
	- Like: `<caption>`, `<figcaption>`, `<title>` in SVG, `<desc>` in SVG
	- Unlike: `img[src]`, `[title]`, `input[placeholder]`
- Avoid mixing languages. No JSON or JS in attributes.
	- Instead of JS in attributes, use a JS property that accepts a function.
	- Exception: [Event handler attributes](https://w3ctag.github.io/design-principles/#always-add-event-handlers)
	- Instead of JSON in attributes:
		- Link or nest a subtree that provides the same complex structure 
			- Like: `<datalist>`, `<select>`, `<table>`
		- Use a JS property to read/write the data as a plain JS object
			- This seems reasonable, but precedent?
		- TBD: If you do both, should they be synced up?
			- Huge performance penalty if they are
			- Confusing if they aren't
- Avoid HTML in attributes. HTML should be element content, not attribute content.
	- Unlike: `iframe[srcdoc]`
	- Would formatting be useful? Use a child element of a specific type instead of an attribute
		- Like: `figure > figcaption`, `table > caption`, `fieldset > legend`, `details > summary`
- Does the attribute affect presentation? If so, consider using a custom property
	- Problem: No way to react to style changes currently

### Naming

- Use all lowercase in docs. 
- Prefer nouns
- Prefer single words. Two words are more rare but there is precedent. Avoid names of three words or more.
	- If using multiple words, concatenate. No camelCase, no hyphens, no underscores.
		- Examples: `maxlength`, `tabindex`, `datetime`
- Avoid abbreviations to only save a few characters. 
	- E.g. it's `<video>`, not `<vid>`, `<source>` not `<src>`
	- Many older element names are abbreviated. At the time, saving characters was very important, but these days readability is a bigger focus
	- TBD: Are inline elements an exception to this? Is it because keeping them short interferes less with text readability?
		- Like: `<em>`, `<sub>`, `<ins>`, `<b>`, `<i>` etc
- For the property reflecting the attribute, use camelCase
	- E.g. the `tabindex` attribute is reflected by the `tabIndex` property
- Re-use existing names for the same concepts when possible.
	- E.g. in an accordion component, use an `open` attribute/property instead of e.g. `expanded`

### By data type

- Boolean:
	- Attribute presence is true (regardless of value), attribute omission is false.
		- Like: `checked`, `selected`, `disabled`, `required`, `ol[reversed]`, `script[async]`, `details[open]`
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
			- Like: `input[list]`, `label[for]`
			- That is painful, especially in repeated structures where it forces tooling for id generation.
			- Perhaps the `<label>` model is best: id ref works, but nesting of a certain element type also works. However, this is not always feasible.
		- SVG uses `#id`, which at least is extensible to selectors in general
		- No precedent for relative selectors (e.g. "the element with the class .foo that is after the current element"). Perhaps [`:scope`](https://drafts.csswg.org/selectors-4/#the-scope-pseudo) or [`&`](https://drafts.csswg.org/css-nesting-1/#direct) could be used (without `@nest`).
	- Avoid references to other elements when not necessary. E.g. it's best for an element to manage its own state than a parent element pointing to it. 
		- Like: `input[type=radio][checked]`, `select > option[selected]`
		- Unlike: [`<generic-tabs selected="1">`](https://genericcomponents.netlify.app/generic-tabs/demo/index.html)
			- Never reference by child index. This is flimsy (reference changes as children are added/removed), and not idiomatic to HTML.
- Color
	- Are you absolutely sure this is not presentational? Most HTML attributes that take colors as values are deprecated.
	- The only non-deprecated precedent in HTML is `input[type=color][value]` which accepts hex colors, but there are discussions to expand this, as it limits it to sRGB.
	- Accepting any CSS `<color>` value seems the way to go.
- URL
	- If the attribute is central to the element, `href` or `src` with URL as its value
		- Unlike: `object[data]`
		- Historically, `href` is for linking and `src` for embedding, though the difference can be murky
	- Relative URLs should be relative to the current document
	- The properties reflecting these attributes should return a `URL` object
		- Precedent: `<a>` includes properties that effectively mirror those of `URL`, but `URL` did not exist at the time.
		- Other HTML elements (e.g. `<iframe>`) just return the absolute URL as a string from the property, but returning a `URL` does that anyway when coerced to a string, and is far more flexible.
- Dates, times, durations
	- Use the [same format as `time[datetime]`](https://html.spec.whatwg.org/multipage/text-level-semantics.html#the-time-element)
	- TBD: Should the property return a `Date` or reflect the string? 
		- Precedent: `<input type=date>` has separate `valueAsDate` and `valueAsNumber` properties. `<time>` seems to have nothing, it just reflects the attribute as a string.

## Elements
 
- Naming
	- Prefer single words (after the hyphen in custom elements)
	- Prefer nouns or adjectives, avoid verbs
		- Built-in nouns: `<label>`, `<table>`, `<source>`, `<picture>`, `<template>`, `<input>`, `<output>`, `<option>`, `<article>`, `<img>`, `<dfn>`, `<code>`
		- Bult-in adjectives: `<small>`, `<big>`
		- Built-in verbs: `<embed>`, `<select>`
- Use specific element types for specifying specific pieces of data (think attributes with structure/formatting), otherwise handle any child 
	- Like: `figure > figcaption`, `table > caption`, `details > summary`
	- If that piece of data is required and the child is not present, generate a sensible default
		- Like `details > summary`: if `<summary>` is missing, one is generated
	- Do not require `slot` attributes if they are obvious from the element type
- Parent needs to be able to communicate state summary without having to traverse children
		- Like: `HTMLSelectElement#value`
- Update the DOM when properties change, to enable attribute selectors and easier debugging
	- Like: `details[open]`
	- Unlike: `input[value]`, `option[selected]`
- List of all built-in element names: https://codepen.io/leaverou/pen/abZvbor?editors=0110

## Events

- Naming
	- One or two words, concatenated 
		- Like: `input`, `change`, `blur`, `error`, `beforeunload`, `hashchange`
		- Unlike: `DOMContentLoaded`, `SVGScroll`
	- Avoid abbreviations
		- Unlike: `DOMAttrModified`
	- General format: `[ before | after ]? <noun>? <verb> [start | end]?`
	- Use present tense for verbs
		- Like: `change`, `input`, `open`, `load`
		- Unlike: `appinstalled`, `connected`
- Use standard events where applicable, e.g. `blur` instead of `slBlur`
- See also [W3C TAG Design Principles on Event Design](https://w3ctag.github.io/design-principles/#event-design)
- List of all built-in events: https://codepen.io/leaverou/pen/yLJNmxJ?editors=0110

## Methods

- Use standard methods when possible, e.g. `element.focus()` instead of `element.setFocus()`
