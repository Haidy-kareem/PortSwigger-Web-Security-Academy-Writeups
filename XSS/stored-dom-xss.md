# Lab: Stored DOM 


### Lab Goal

Understand why writing a custom HTML escaping function is dangerous and how incomplete escaping can still lead to DOM XSS.

---

# Vulnerability Type

**Stored DOM XSS**

The payload is stored on the server, returned as JSON, parsed by JavaScript, and then inserted into the DOM.

---

# Source

### Finding the vulnerable JavaScript

Open the blog post.

Right-click on View Page Source.

Locate the JavaScript file for the comment section.

Open it.

The application retrieves comments from the server.

```
xhr.open("GET",postCommentPath+window.location.search);
```

The response is received as:

```
this.responseText
```

and converted into JavaScript objects using:

```
JSON.parse(this.responseText)
```

**Important**

`JSON.parse()` is **not** the vulnerability.

It simply converts JSON strings into JavaScript objects.

---

# Sink

The comment body is inserted using:

```
commentBodyPElement.innerHTML=escapeHTML(comment.body);
```

The dangerous sink is:

```
innerHTML
```

because it parses HTML.

---

# The Protection

The developer attempted to prevent XSS using:

```
functionescapeHTML(html) {
returnhtml.replace('<','&lt;')
.replace('>','&gt;');
}
```

At first glance, this looks correct.

However, there is a critical mistake.

---

# Root Cause

`String.replace()` without a global regular expression replaces **only the first occurrence**.

For example:

Input:

```
<<script>>
```

After escaping:

```
&lt;<script>&gt;
```

Only:

- the first `<`
- the first `>`

were escaped.

The remaining HTML characters stayed untouched.

---

# Discovery Process

### Test 1

Input:

```
<h1>Haidy</h1>
```

Result:

The HTML did **not** execute.

Reason:

The first opening tag was escaped.

---

### Test 2

Input:

```
<h1><h1>Haidy</h1>
```

Result:

The second `<h1>` was interpreted as HTML.

This proved that only the first angle brackets were escaped.

<img width="1611" height="616" alt="image" src="https://github.com/user-attachments/assets/41dccaf3-6514-4b35-8168-973a27e693f4" />

---

After confirming this behavior, I realized that any payload relying on a **second HTML tag** could bypass the custom escaping function.

so i tried :

```jsx
<img><img src=x onerror="alert(1)"></img>
```

<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/9d0c8e42-f7b9-464a-8eb1-b5aa142750f6" />


and it worked out 

# Notes

### 1. Never trust custom escaping functions.

A function named `escapeHTML()` is not necessarily secure.

Always inspect its implementation.

---

### 2. Understand how `replace()` works.

```
replace('<','&lt;')
```

replaces only the **first occurrence**.

To replace every occurrence:

```
replace(/</g,'&lt;')
```

or use a trusted escaping library.

---

### 3. `innerHTML` is only dangerous when unsafe data reaches it.

`innerHTML` itself is not the vulnerability.

The vulnerability is caused by **insufficient sanitization** before writing to `innerHTML`.

---

### 4. `JSON.parse()` is unrelated to the XSS.

It simply converts JSON into JavaScript objects.

The vulnerability happens **after** parsing, when the
