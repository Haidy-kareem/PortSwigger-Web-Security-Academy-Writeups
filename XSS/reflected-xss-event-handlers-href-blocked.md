# Lab: Reflected XSS with event handlers and href attributes blocked


## Lab Overview

This lab contains a reflected XSS vulnerability. The application blocks the two most common injection techniques:

- All JavaScript **event handler attributes** (`onclick`, `onload`, `onerror`, etc.)
- The **`href` attribute** on `<a>` elements when it contains a `javascript:` value

However, the filter still allows several SVG tags, including `<svg>`, `<a>`, `<animate>`, and `<text>`.

**Goal:** get the browser to execute `alert(1)` when the victim clicks a link labeled "Click me".

## Recon / Approach

Before jumping to a specific payload, I used **Burp Suite** to intercept the request to the vulnerable `search` parameter and confirm exactly what was being reflected and what the filter was stripping out. Sending the same input back through Repeater made it easy to see, character by character, which attributes and keywords got blocked.

In parallel, I went through the **PortSwigger XSS cheat sheet** and tried a batch of common tags/vectors against the filter to map out what was and wasn't allowed:

- Classic `<a href="javascript:alert(1)">` → blocked (href with `javascript:` stripped/rejected)
- `<button onclick="alert(1)">` and other event-handler based vectors → blocked (all `on*` attributes filtered)
- `<svg>`, `<a>`, `<animate>`, `<text>` → passed through untouched

Once I confirmed `<animate>` wasn't being filtered, the direction became clear: instead of writing the dangerous attribute myself, I could get the *browser* to write it for me via SVG animation.

## The Core Idea

The important realization here isn't "here's a magic payload" — it's understanding **why** `javascript:` URLs execute at all.

The browser doesn't care whether `href="javascript:alert(1)"` was typed directly into the HTML or produced some other way. If, after parsing, an `<a>` element ends up with that href, clicking it will run the JavaScript.

The filter blocks us from writing that `href` by hand. But SVG's `<animate>` element exists specifically to modify an element's attributes over time — and it isn't on the blocklist. So instead of:

```html
<ahref="javascript:alert(1)">Click me</a>
```

(blocked), we write:

```html
<a><animateattributeName="href"values="javascript:alert(1)"/>    Click me</a>
```

and let `<animate>` assign the `href` attribute for us, after the filter has already done its job.

## Final Payload

Inject the payload into the URL search parameter

```html
<svg> <a> <animate attributeName="href"values="javascript:alert(1)"/>      <text x="20"y="20">Click me </text> </a> </svg>
```

### Breaking it down

| Part | Purpose |
| --- | --- |
| `<svg>` | Wraps the content — required, since `<animate>` and `<text>` are SVG elements |
| `<a>` (empty) | Anchor element with no `href` written, so the filter has nothing to catch |
| `<animate attributeName="href" ...>` | Tells the browser: "set the `href` attribute of the parent element" |
| `values="javascript:alert(1)"` | The value the browser assigns to `href` |
| `<text x="20" y="20">Click me</text>` | Visible, clickable label inside the SVG |

After the browser parses the SVG, the DOM effectively becomes:

```html
<a href="javascript:alert(1)">Click me</a>
```

Clicking the link triggers the JavaScript URL, and `alert(1)` fires.

## Notes

Filters tend to block specific **syntax** (`onclick=`, `href="javascript:...`), not the final **DOM state** an attacker is trying to reach. SVG's `<animate>` element lets the browser construct that same dangerous `href` attribute *after* the filter has already run, which is why this bypass works.
