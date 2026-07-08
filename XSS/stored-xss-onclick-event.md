# Lab: Stored XSS into onclick Event with Angle Brackets and Double Quotes HTML-Encoded and Single Quotes and Backslash Escaped


## Lab Description

The application contains a comment feature where the **Website** field is reflected inside an `onclick` event handler on the comment author's name.

The rendered HTML looks like this:

```html
<a id="author"
   href="https://google.com"
   onclick="var tracker={track(){}};tracker.track('https://google.com');">
    Haidy
</a>
```

The application applies the following protections:

- `<` and `>` are HTML-encoded.
- `"` is HTML-encoded.
- Literal `'` and `\` characters are backslash-escaped.

The objective is to execute `alert(1)` when the comment author's name is clicked.

---

## Find the Reflection Point

After submitting a comment with the following Website value:

```
https://test.com
```

and checking **View Page Source**, the input appeared here:

```html
onclick="var tracker={track(){}};tracker.track('https://test.com');"
```

This immediately reveals an important detail:

The input is **not** reflected as normal HTML.

Instead, it is reflected:

- inside an HTML attribute (`onclick`)
- inside a JavaScript string (`'...'`)

This creates a **nested context**:

```
HTML Attribute
    └── JavaScript
            └── JavaScript String
```

Understanding the exact reflection context is always the first step before attempting any payload.

---

## Understand the Context

The rendered attribute is:

```html
onclick="var tracker={track(){}};tracker.track('WEBSITE_VALUE');"
```

Here:

- The `onclick` attribute contains JavaScript code.
- The Website value is enclosed inside single quotes.
- Those quotes define a JavaScript string.

Normally, breaking out of this string would require injecting another single quote (`'`).

---

## Testing the Filter

Submitting a literal single quote produced:

```
'
```

which became:

```
\'
```

This indicates that the application escapes literal single quotes before they reach the browser.

As a result, a normal quote can no longer terminate the JavaScript string. At this point, a traditional string-breaking payload is no longer possible.

---

## Looking for Another Way

Instead of submitting a literal quote, the payload used the HTML entity:

```
&#x27;
```

Resulting payload value:

```
https://google.com&#x27;-alert(1)-&#x27;
```

At first glance, this looks strange because `&#x27;` represents exactly the same character: `'`. So why wasn't it escaped?

From the application's behavior, we can infer that the server escapes **literal** single quote characters, but it does **not** appear to decode HTML entities before applying that escaping. To the server, this input is simply plain text: `&`, `#`, `x`, `2`, `7`, `;`. Since there is no actual `'` character at this stage, nothing is escaped.

---

## Why the Browser Treats It as a Real Quote

This is the key idea behind the entire lab.

The browser does **not** execute the raw HTML immediately. Instead, it first parses the HTML document. During HTML parsing, character references inside attribute values are automatically decoded. For example, `&#x27;` becomes `'` - before the JavaScript contained inside the `onclick` attribute is ever executed.

Notice something important: the server already finished its filtering **before** the browser decoded `&#x27;`. The server has already completed its processing. The browser performs HTML entity decoding locally while parsing the response. By the time JavaScript starts executing, the entity has already become a real single quote.

This is why the bypass works because the filtering and the decoding happen at different stages of processing.

---

## What the Browser Actually Executes

Before HTML entity decoding, the attribute looks like this:

```html
onclick="var tracker={track(){}};tracker.track('https://google.com&#x27;-alert(1)-&#x27;');"
```

While parsing the HTML, the browser automatically decodes `&#x27;` into a real single quote (`'`). So the JavaScript that is eventually executed becomes:

```jsx
var tracker={track(){}};
tracker.track('https://google.com'-alert(1)-'');
```

At first glance, this expression looks invalid. However, JavaScript evaluates the operands of an expression before applying the operator. When evaluating:

```jsx
'string' - alert(1) - ''
```

the engine first evaluates `'string'`, then `alert(1)`, then `''`. Only after evaluating those operands does it attempt the subtraction. Although the subtraction itself produces `NaN`, `alert(1)` has already executed as a side effect during expression evaluation.

The goal of the lab is simply to execute JavaScript, so triggering `alert(1)` is enough.

---

## Why This Bypass Works

The important idea is **not** HTML encoding itself. The important idea is that **the server and the browser process the input at different stages**.

Think of the input passing through two different readers.

**Reader 1 - The Server**
The application filters the submitted input. Based on the observed behaviour, it escapes **literal** single quotes (`'`). However, it does not appear to decode HTML entities before applying that escaping. As a result, `'` is escaped, while `&#x27;` passes through unchanged.

**Reader 2 - The Browser**
When the browser receives the HTML, it parses the document. As part of normal HTML parsing, character references inside attribute values are decoded automatically. So `&#x27;` becomes `'`. At this point, the server has already completed its filtering. The browser simply continues building the DOM and later executes the JavaScript inside the `onclick` handler.

The quote effectively appears **after** the security check has already happened. This difference in processing stages is what makes the bypass possible.

---

## Why Doesn't This Work Everywhere?

This technique only works when there is a mismatch between how different stages process the same input. It **does not** work in every situation. For example:

- If the application decodes HTML entities **before** applying its filter, the decoded single quote will be escaped normally.
- If the input is inserted directly into JavaScript source code instead of an HTML attribute, there is no HTML entity decoding step, so `&#x27;` remains plain text.
- If another protection mechanism (such as CSP) prevents JavaScript execution, this bypass alone is not enough.

The key takeaway is that the success of this technique depends on **where** the input is reflected and **how** it is processed before execution.

---

## Final Payload

```
https://google.com&#x27;-alert(1)-&#x27;
```

Submitted as the **Website** field value when posting a comment. Clicking the comment author's name afterward triggers `alert(1)`.

---

## Notes

- Always identify the exact reflection context before building a payload.
- Reflection inside an HTML attribute is different from reflection inside JavaScript or HTML text.
- HTML entity encoding is **not** a universal XSS bypass.
- This technique works only because HTML entity decoding happens after the application's filtering.
- Always inspect the rendered source to understand exactly how the application transformed your input.
- Focus on the parsing context rather than the specific JavaScript function. The function name is usually irrelevant - the context is what matters.
