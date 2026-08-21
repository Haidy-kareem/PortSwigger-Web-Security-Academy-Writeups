## Lab Overview

This lab contained a **path traversal vulnerability** in the product image functionality.

The application attempted to prevent path traversal by stripping traversal sequences such as `../` from the user-supplied filename. However, the filtering was performed **non-recursively**, which made it possible to bypass the protection using nested traversal sequences.

The goal was to retrieve the contents of the `/etc/passwd` file.

## Identifying the Vulnerable Parameter

I intercepted the request responsible for loading a product image using **Burp Suite** and identified the `filename` parameter as the user-controlled input.

The request was structured like:

```
/image?filename=...
```

This indicated that the value of `filename` was being used to determine which file the application would read.

## Testing Normal Path Traversal

I first tried a standard path traversal payload:

```
../../../etc/passwd
```

However, this did not work because the application stripped the `../` traversal sequences from the input before processing the filename.

For example, conceptually:

```
../../../etc/passwd
        ↓
etc/passwd
```

As a result, the application no longer received a valid traversal path.

## Understanding the Filter

The important detail was that the application stripped the traversal sequence **non-recursively**.

This means that the application removed the sequence once, but it did not repeatedly inspect the resulting string for new traversal sequences.

This created an opportunity to use a **nested traversal sequence**.

For example:

```
....//
```

After the inner `../` sequence is removed, the remaining characters form:

```
../
```

Therefore, a sequence that initially did not look like a normal traversal sequence could become one after the application's filtering process.

## Bypassing the Filter

I used nested traversal sequences in the `filename` parameter:

```
....//....//....//etc/passwd
```

The application stripped the inner traversal sequences, effectively transforming the input into:

```
../../../etc/passwd
```

Because the filtering was not performed recursively, the newly formed `../` sequences were not removed again.

The resulting path allowed the application to traverse outside the intended image directory and access:

```
/etc/passwd
```

The server returned the contents of the file, confirming that the bypass was successful.

## Why the Bypass Worked

The vulnerability was not simply caused by the existence of a path traversal flaw. The important part was the **incorrect filtering logic**.

The application assumed that removing `../` once would be enough to prevent traversal.

However:

```
....//
```

could be transformed into:

```
../
```

after the filtering step.

Since the application did not perform another filtering pass, the newly created traversal sequence remained in the final path.
