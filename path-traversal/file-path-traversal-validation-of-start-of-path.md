## Lab Overview

This lab contained a **path traversal vulnerability** in the product image functionality.

The application transmitted the full file path through a request parameter and attempted to prevent path traversal by validating that the supplied path started with the expected image directory.

The goal was to retrieve the contents of `/etc/passwd`.

## Identifying the Vulnerable Parameter

I inspected the request used to load a product image and found that the `filename` parameter contained the full path to the image.

The application expected the path to start with:

```
/var/www/images/
```

This suggested that the application was checking whether the user-supplied path started with the expected directory.

## Bypassing the Validation

Instead of removing the expected directory from the path, I kept it at the beginning and added path traversal sequences after it.

I used:

```
/var/www/images/../../../../../../etc/passwd
```

The path still started with:

```
/var/www/images/
```

so it passed the application's validation.

However, when the filesystem resolved the `../` sequences, the path traversed out of the images directory and eventually reached:

```
/etc/passwd
```

## Why This Worked

The application validated only the **start of the supplied path**, rather than checking the final resolved path.

Conceptually, the validation was checking:

```
/var/www/images/../../../../../../etc/passwd
        ↓
Starts with /var/www/images/ ?
        ↓
Yes ✓
```

But after resolving the traversal sequences, the actual file being accessed was outside the allowed directory:

```
/var/www/images/../../../../../../etc/passwd
        ↓
/etc/passwd
```

Therefore, the validation could be bypassed.

## Result

The application successfully returned the contents of `/etc/passwd`, confirming that the path traversal vulnerability could be exploited despite the validation of the path prefix.

### Notes

- Checking only whether a path starts with an allowed directory is not sufficient.
- The application should validate the **canonical/resolved path**, not just the raw input.
- The important concept in this lab is the difference between the **supplied path** and the **final path after resolving `../` sequences**.
