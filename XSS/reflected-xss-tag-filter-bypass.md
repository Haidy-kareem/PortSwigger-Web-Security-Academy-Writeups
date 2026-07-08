# Lab: Reflected XSS into HTML context with most tags and attributes blocked


## Overview

This lab contains a search feature that reflects user input directly into the page's HTML. A Web Application Firewall (WAF) blocks most HTML tags and event attributes, but the filtering is based on a blocklist rather than proper context-aware sanitization. This makes it possible to bypass the filter using a tag and event handler the WAF does not block.

**Goal:** Trigger `print()` in the victim's browser without any user interaction.

---

## Find the Injection Point

Searching for a term shows it reflected inside an `<h1>` element:

```html
<h1>0 search results for 'new'</h1>
```

The input lands inside a text node, inside existing HTML. This means before injecting any new tag, the current context has to be broken out of first.

---

## Fuzz the WAF with Burp Suite

Instead of guessing payloads one by one, send the search request to **Burp Intruder** and fuzz the input with a wordlist of common HTML tags and event handlers and PortSwigger provides one for this exact lab(cheat sheet).

**Results:**

- Almost every tag is blocked (`<script>`, `<svg>`, `<iframe>`, `<img>`, etc.)
- Almost every event handler is blocked (`onload`, `onerror`, `onclick`, etc.)
- The fuzzing process revealed that some tags were not blocked by the WAF. While custom tags like `<xss>` were reflected successfully, they offered no exploitable behavior. The `<body>` tag, on the other hand, allowed attaching an event handler that could be triggered during exploitation.
- `onresize` is one of the few event handlers that survives the filter

<img width="1912" height="1020" alt="image" src="https://github.com/user-attachments/assets/969a1e33-1a1e-41d3-9cd5-7d5f3ec90652" />


<img width="1911" height="996" alt="image" src="https://github.com/user-attachments/assets/51aced21-2010-4b39-9cc9-7171c2a12cb5" />


<img width="1911" height="815" alt="image" src="https://github.com/user-attachments/assets/85095e36-a863-44bd-aeb3-4a14093f84fa" />


This tells us the filter is a **blocklist**, not real sanitization, anything not on its list gets through.

---

## Break Out of the Current Context

Since the input lands inside existing HTML (`<h1>0 search results for 'INPUT'</h1>`), a raw `<body onresize=print(1)>` payload would just get printed as text, not parsed as a tag.

To make the browser treat it as real HTML, the surrounding quote and tag need to be closed first:

```
"><body onresize=print(1)>
```

This turns the reflected output into:

```html
<h1>0 search results for '"><body onresize=print()></h1>
```

Because `<body>` isn't allowed to legally sit inside `<h1>`, the browser's HTML parser auto-corrects the markup and hoists the `<body>` tag (with its `onresize` attribute) up to merge with the page's actual `<body>` element. You can confirm this in **DevTools → Elements** - the `onresize` attribute now sits on the real `<body>` tag of the page.

---

## Solve the "No User Interaction" Problem

`onresize` only fires when the browser window is resized, it won't fire on page load, and we can't ask the victim to resize their window manually.

So load the vulnerable page inside an `<iframe>`, then resize the *iframe* itself via CSS right after it loads. Resizing the iframe fires a resize event inside the page it contains, which triggers `onresize` on `<body>`  no click, no real window resize needed.

---

## Build the Delivery Payload

On the PortSwigger **Exploit Server**, host this HTML:

```html
<iframe
  src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E"
  onload="this.style.width='100px'">
</iframe>
```

**What each part does:**

| Part | Purpose |
| --- | --- |
| `src=...?search="><body onresize=print()>` | The vulnerable URL, with our context-breaking payload URL-encoded in the `search` parameter |
| `onload="this.style.width='100px'"` | Fires as soon as the iframe finishes loading, shrinking its width |
| Shrinking the iframe | Fires a `resize` event *inside* the loaded page → triggers `onresize` on `<body>` → runs `print()` |

---

## Deliver and Confirm

Click **Store** then **Deliver to victim** on the exploit server. The victim's browser loads the iframe, the iframe resizes itself immediately, `onresize` fires, and `print()` executes lab solved.

<img width="1912" height="1080" alt="image" src="https://github.com/user-attachments/assets/1a3c762c-7186-46c6-936b-14fe2e02b991" />


---

## Root Cause

- The WAF relies on **blocklisting** tags/attributes instead of proper output encoding.
- The application doesn't account for **HTML parser auto-correction**, which lets a "misplaced" tag get merged into valid DOM structure.
- Not every dangerous event needs user interaction, some can be triggered programmatically (like `onresize` via an iframe).

## Notes

- Blocklists are inherently incomplete, always assume attackers will find the gap.
- Context matters: escaping out of the current HTML context is often step one, not the payload itself.
- Browsers "fixing" broken HTML for you can become a vulnerability.
- XSS exploitation frequently requires **chaining** multiple small tricks (context escape + allowed tag + non-standard trigger) rather than one single payload.
