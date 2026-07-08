# Lab: Reflected DOM XSS


## Objective

Understand how user input travels from the URL to the JavaScript engine, and why **escaping** is the key concept in this lab.

---

# 1. Source

The user input comes from:

```
window.location.search
```

Example:

```
/?search=test
```

The value of `search` is sent to the server.

---

# 2. Server Response

The server returns the search results as JSON.

Example:

```
{
    "results": [],
    "searchTerm":"test"
}
```

**Important**

JSON is **not executable JavaScript**.

It is only a way to represent data.

---

# 3. Dangerous Sink

The page executes:

```
eval('var searchResultsObj = '+this.responseText);
```

This means:

1. Receive JSON from the server.
2. Concatenate it with:

```
varsearchResultsObj=
```

1. Execute everything as JavaScript.

That is why `eval()` is dangerous.

---

# 4. What is Escaping?

Some characters have a special meaning.

For example:

```
"
```

normally means:

> Start or end of a string.
> 

But sometimes we want a quote **inside** a string.

JavaScript uses the backslash (`\`) to escape special characters.

Example:

```
varname="John\"Doe";
```

The value becomes:

```
John"Doe
```

Notice:

The **backslash is not part of the final value.**

It only tells the parser:

> "Treat the next character differently."
> 

---

# Common Escape Sequences

| Written | Final Value |
| --- | --- |
| `\"` | `"` |
| `\\` | `\` |
| `\n` | New line |
| `\t` | Tab |

---

# How the JavaScript Parser Reads Escapes

The parser always reads **left to right**.

---

## Example 1

```
varx="\"";
```

Parser reads:

```
\
"
```

The backslash tells JavaScript:

> The quote is NOT the end of the string.
> 

Final value:

```
"
```

---

## Example 2

```
varx="\\";
```

Parser reads:

```
\
\
```

The first backslash escapes the second one.

Final value:

```
\
```

Only one backslash remains.

---

## Example 3

```
varx="\\\"";
```

The parser reads:

First:

```
\\
```

↓

becomes

```
\
```

Then it sees

```
\"
```

↓

which becomes

```
"
```

Final value:

```
\"
```

That is:

- one backslash
- one double quote

---

# Representation vs Value

This was the biggest lesson from this lab.

### What Burp shows

Burp shows the **representation** of the data.

Example:

```
{
    "searchTerm":"\\\""
}
```

This is how the data is written inside JSON.

---

### What JavaScript sees

After parsing, JavaScript sees the **actual value**.

So:

```
\\
```

becomes

```
\
```

and

```
\"
```

becomes

```
"
```

The parser removes the escape characters while interpreting them.

---

## Solution

```jsx
\"-alert(1)}//
```

```json
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 45

{"results":[
],
"searchTerm":"\\"-alert(1)
}
//"}
```

---

# Notes

- Always identify the **Source**.
- Always identify the **Sink**.
- Determine the execution **context** (HTML, JavaScript, Attribute, URL, etc.).
- Check whether the application performs **encoding** or **escaping**.
- Remember that **Burp shows the serialized representation**, not necessarily the exact value that JavaScript will interpret.
- The JavaScript parser processes escape sequences **from left to right**, and each backslash only affects the **next character**.
