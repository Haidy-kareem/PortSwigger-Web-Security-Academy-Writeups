## Lab Overview

This lab contained a **path traversal vulnerability** where the application or web server stripped directory traversal sequences before processing the user input.

The goal was to retrieve the contents of:

```
/etc/passwd
```

## Identifying the Vulnerable Parameter

I intercepted the request responsible for loading product images using **Burp Suite** and identified the `filename` parameter as the user-controlled input.

The request used a structure similar to:

```
/image?filename=...
```

I first tested a normal traversal sequence:

```
../../../../etc/passwd
```

However, the traversal sequence was blocked or sanitized by the application.

## Bypassing the Sanitization

I then used **URL encoding** to represent the traversal characters differently.

A URL-encoded traversal sequence is:

```
%2e%2e%2f
```

I also tested **double URL encoding**:

```
%252e%252e%252f
```

For the complete traversal path, I used:

```
%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252fetc/passwd
```

After the decoding stages, the encoded sequences could be converted back into:

```
../../../../etc/passwd
```

## Why This Worked

The important part was the difference between **filtering and decoding**.

The encoded payload did not initially appear as the normal `../` sequence to the filter. After being decoded by the application or another component, it became a valid traversal sequence.

With double encoding, the input could pass through multiple decoding stages before becoming:

```
../
```

This allowed the traversal to bypass the sanitization.

## Result

The server returned the contents of:

```
/etc/passwd
```

confirming that the encoded path traversal was successful.

### Notes

- `../` can be represented as `%2e%2e%2f`.
- Double URL encoding represents it as `%252e%252e%252f`.
- The effectiveness of double encoding depends on how many times the application or web server decodes the input.
- The main concept is to understand **when decoding and filtering happen** in the request-processing chain.
