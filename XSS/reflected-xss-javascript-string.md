# Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded



## 1. Introduction

The vulnerability occurs when user input is reflected inside a JavaScript string without proper sanitization, allowing an attacker to break out of the string and execute arbitrary JavaScript.

---

## 2. Vulnerability Explanation

Cross-Site Scripting (XSS) is a client-side vulnerability that allows an attacker to inject malicious JavaScript into a web application.

When the injected code is executed in another user's browser, the attacker may be able to:

- Steal session tokens
- Perform actions on behalf of the victim
- Read sensitive information
- Modify page content

This lab demonstrates **Reflected XSS**, where the payload is supplied in an HTTP request and immediately reflected back in the server's response.

Unlike Stored XSS, the payload is not saved by the application and must be delivered to the victim through a crafted request or URL.

---

## 3. Lab Objective

The objective of the lab is to execute JavaScript in the victim's browser by triggering the `alert()` function.

---

## 4. Reconnaissance / Analysis

I started by testing the search functionality and inspecting the page source.

I observed that my input was reflected inside the following JavaScript code:

```jsx
var searchTerms = 'USER_INPUT';
```

This indicated that the reflection occurred inside a **JavaScript string context**.

At first, I tested simple inputs and noticed that the application automatically appended:

```jsx
';
```

after my input.

This meant that simply injecting JavaScript would likely break the script due to unmatched quotes.

To exploit the vulnerability successfully, I needed to:

1. Escape the original string.
2. Execute my JavaScript code.
3. Ensure that the remaining quote added by the application would not cause a syntax error.

---

## 5. Exploitation

### Initial Attempt

I first tried:

```
';'alert(1)

```

!image.png

which produced:

```jsx
var searchTerms = '';
'alert(1)';
```

The script is syntactically valid, but alert(1) is treated as a string literal

rather than a function call, so it never executes.

### Successful Payload

After analyzing the generated script, I crafted the following payload:

```
';alert(1);'haidy
```

The resulting JavaScript became:

```jsx
var searchTerms = '';
alert(1);
'haidy';
```

### Why It Works

- The first `'` closes the original string.
- `alert(1);` executes JavaScript.
- `'haidy'` creates a valid string literal.
- The application's trailing quote completes the second string.

As a result, the final script remains syntactically valid and the alert executes successfully.

### Official PortSwigger Solution

PortSwigger used:

```
'-alert(1)-'
```

Resulting in:

```jsx
var searchTerms = ''-alert(1)-'';
```

which JavaScript interprets as:

```jsx
'' - alert(1) - ''
```

This is also syntactically valid and triggers `alert(1)` during expression evaluation.

Although the payload differs from mine, both approaches rely on the same concept:

- Escaping the original string.
- Executing JavaScript.
- Keeping the final script valid.

---

## 6. Result

The payload successfully executed JavaScript and triggered the alert dialog, completing the lab objective.

!image.png

---

!image.png

## 7. Mitigation / Fix

The root fix is JavaScript-context output encoding. Before inserting user-controlled
input into a JavaScript string, the application must escape special characters such
as `'`, `"`, and `\`. This prevents an attacker from breaking out of the string and
injecting arbitrary JavaScript.

A safer design is to avoid embedding user input directly into JavaScript altogether.
Instead, user-controlled data can be stored in HTML `data-*` attributes and accessed
through JavaScript using APIs such as:

`document.getElementById(...).dataset.value`

This separates data from executable code and eliminates the injection point entirely.

A Content Security Policy can reduce the impact of a successful injection, but it is
not a substitute for fixing the injection point itself.

---

## Notes

The most important lesson from this lab was that finding a payload is less important than understanding the context.

The exploitation process followed these steps:

1. Identify where the input is reflected.
2. Determine the context.
3. Escape the context safely.
4. Execute JavaScript.
5. Ensure the resulting code remains syntactically valid.

Understanding the context allows multiple payloads to be created, rather than relying on memorized solutions.
