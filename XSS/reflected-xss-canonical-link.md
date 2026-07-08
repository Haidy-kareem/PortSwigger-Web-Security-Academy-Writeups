# Reflected XSS in canonical link tag


## Lab Description

The application reflects user input inside the `href` attribute of the canonical link.

Example:

```
<linkrel="canonical"href='https://LAB-ID.web-security-academy.net/?USER_INPUT'>
```

The goal is to inject an attribute that executes `alert(1)`.

The lab hints that the victim will press:

- ALT + SHIFT + X
- CTRL + ALT + X
- ALT + X

It also mentions that the exploit only works in **Google Chrome**.

---

# Step 1 - Find the Reflection Point

The home page doesn't contain any input fields.

So the first question was:

> **Where is my input reflected?**
> 

Adding any value after the `?` in the URL:

```
/?test
```

and checking **View Page Source** showed:

```
<linkrel="canonical"
href='https://LAB-ID.web-security-academy.net/?test'>
```

This confirmed that the **query string** is reflected inside the canonical link.

---

# Step 2 - Check the Context

Instead of relying on **Inspect**, I checked **View Page Source** because it shows the HTML returned by the server.

The important observation was:

```
href='...'
```

The attribute is enclosed with **single quotes (' )**, not double quotes.

This means I must escape using `'` instead of `"`.

---

# Step 3 - Escape the Attribute

Closing the `href` attribute allows injecting new attributes into the `<link>` element.

The payload injects:

- `accesskey="x"`
- `onclick="alert(1)"`

Resulting HTML becomes:

```
<link
rel="canonical"
href='https://LAB-ID.web-security-academy.net/?'
accesskey='x'
onclick='alert(1)'>
```

At this point, attribute injection is successful.

---

# Step 4 - Why accesskey?

The lab explicitly says the victim presses:

- ALT + SHIFT + X
- CTRL + ALT + X
- ALT + X

This hint indicates that the browser will activate an element with:

```
accesskey="x"
```

---

# Final Payload

```
/?%27accesskey=%27x%27onclick=%27alert(1)
```

---

# Lessons Learned

- Always identify **where** the reflection occurs before crafting a payload.
- **View Page Source** and **Inspect** serve different purposes.
- Never assume attributes use double quotes; always verify the actual HTML.
- Attribute Context XSS requires understanding how the HTML parser interprets injected characters.
- Browser-specific behavior can make an exploit work in one browser but not another.
- The lab hint often reveals the intended exploitation technique (here, the keyboard shortcut hinted at `accesskey`).

---

# Key Takeaways

- Reflection Point → Query String
- Context → HTML Attribute
- Attribute Delimiter → Single Quote (`'`)
- Injection Type → Attribute Injection
- Trigger → `accesskey`
- Executed Event → `onclick`
- Browser Requirement → Google Chrome
