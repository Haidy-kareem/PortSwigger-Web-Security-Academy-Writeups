# Lab: Reflected XSS with event handlers and href attributes blocked


## Lab Overview

This lab contains a **Reflected Cross-Site Scripting (XSS)** vulnerability. The application filters common XSS vectors by blocking:

- JavaScript event handlers (`onclick`, `onload`, etc.).
- The `href` attribute on anchor (`<a>`) elements.

The objective is to inject a payload that executes `alert()` when the victim clicks a link labeled **"Click me"**.

---

## Finding Allowed Tags and Attributes

To understand the filtering behavior, I first needed to identify which HTML tags and attributes were allowed by the application.

I used **Burp Suite** to intercept the request containing my input and sent it to **Intruder** to test different XSS vectors.

I also used the **PortSwigger XSS Cheat Sheet** as a reference for different payloads and HTML/SVG elements that could potentially bypass the filter.

By testing different tags and attributes and comparing the responses, I noticed that common XSS vectors such as:

- `<script>` tags
- Event handler attributes (`onclick`, `onload`, etc.)
- Direct JavaScript URLs in `href`

were blocked or removed.

However, some SVG-related elements were still accepted by the application, including:

```html
<svg>
<a>
<animate>
<text>
```

## Understanding the Vulnerability

My first thought was to use one of the common XSS techniques:

```html
<a href="javascript:alert(1)">Click me</a>
```

or

```html
<button onclick="alert(1)">Click me</button>
```

Neither approach works because the application blocks both `href` values containing JavaScript and all event handler attributes.

At this point, the challenge becomes:

> **How can JavaScript be executed without using `onclick` or directly writing a `href` attribute?**
> 

---

## The JavaScript Concept Behind the Exploit

The important concept in this lab is understanding that **JavaScript is not executed because we type `javascript:` directly into the HTML**.

Instead, JavaScript executes because the browser eventually creates a clickable link whose `href` becomes:

```
javascript:alert(1)
```

The browser treats `javascript:` as a JavaScript URL. When the user clicks the link, the browser evaluates the JavaScript code contained in that URL.

Normally, we would create this by writing:

```html
<a href="javascript:alert(1)">
```

Since the filter blocks that, we need another way to make the browser assign the `href` value for us.

---

## Why `<animate>` Works

SVG provides an element called `<animate>` whose purpose is to modify an element's attributes.

Instead of writing:

```html
<a href="javascript:alert(1)">
```

we create an empty anchor:

```html
<a>
```

Then we tell the browser:

> "Animate the `href` attribute and set its value to `javascript:alert(1)`."
> 

The key part is:

```html
<animate attributeName="href"
         values="javascript:alert(1)" />
```

Here:

- `attributeName="href"` tells the browser which attribute should be modified.
- `values="javascript:alert(1)"` provides the new value.

The application blocks manually writing a `href` attribute, but it does **not** block the browser from creating or updating that attribute through SVG animation.

After the browser processes the SVG, the anchor effectively behaves as if it were:

```html
<a href="javascript:alert(1)">
```

When the victim clicks **"Click me"**, the browser follows the JavaScript URL and executes:

```jsx
alert(1)
```

---

## Final Payload

```html
<svg>
    <a>
        <animate attributeName="href"
                 values="javascript:alert(1)" />
        <text x="20" y="20">Click me</text>
    </a>
</svg>
```

---

## Key Takeaway

This lab demonstrates an important XSS principle:

Security filters often block **specific HTML syntax**, but browsers provide multiple mechanisms that can produce the same final DOM.

The vulnerability exists because the filter prevents developers from directly writing a dangerous `href`, yet allows an SVG feature (`<animate>`) that causes the browser to generate the exact same dangerous attribute after parsing.

The exploit is therefore not about bypassing JavaScript itself—it is about leveraging browser behavior to create executable JavaScript through an alternative path that the filter failed to consider.
