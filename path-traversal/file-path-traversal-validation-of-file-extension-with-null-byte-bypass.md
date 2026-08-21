## Lab Overview

This lab contained a **path traversal vulnerability** in the product image functionality.

The application validated that the supplied filename ended with the expected file extension, such as `.png`.

The goal was to retrieve the contents of `/etc/passwd`.

## Identifying the Vulnerable Parameter

I inspected the request used to load a product image and identified the `filename` parameter as the user-controlled input.

The application expected the filename to end with `.png`. Therefore, a normal path traversal payload such as:

```
../../../etc/passwd
```

would not pass the validation because it did not have the required `.png` extension.

## Bypassing the File Extension Validation

I used a **URL-encoded null byte**, represented as:

```
%00
```

I then appended `.png` after the null byte:

```
../../../etc/passwd%00.png
```

This allowed the input to satisfy the application's extension check because the supplied filename technically ended with `.png`.

However, in contexts where the null byte is interpreted as a string terminator, the file-handling component can treat the path as ending at `%00`. This means that `.png` is not considered part of the actual file path.

As a result, the application can effectively process the filename as:

```
/etc/passwd
```

instead of trying to access a file named `/etc/passwd.png`.

## Why This Worked

The bypass worked because different parts of the application could interpret the same input differently.

The validation logic only checked that the filename ended with `.png`, while the file-handling component could interpret `%00` as the end of the filename.

Therefore, the following payload could satisfy the extension validation while still targeting `/etc/passwd`:

```
../../../etc/passwd%00.png
```

## Result

The application returned the contents of `/etc/passwd`, confirming that the path traversal vulnerability could be exploited despite the `.png` extension validation.

### Notes

- `%00` represents a URL-encoded null byte.
- The technique depends on how the application and underlying libraries handle null bytes.
- The important concept is that the validation and file-processing stages may interpret the same input differently.
- Simply checking that a filename ends with an expected extension is not sufficient protection against path traversal.
