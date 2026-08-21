# Path Traversal

This folder contains my write-ups for Path Traversal labs from PortSwigger Web Security Academy. The included write-ups cover how applications that read/write files based on user-supplied filenames can be tricked into accessing files outside the intended directory, and the different filter/validation bypass techniques attackers use to defeat weak sanitization.

## Covered Concepts

* Traversal Sequences (`../`)
* Non-Recursive Stripping Bypass
* Superfluous URL-Decoding Bypass
* Start-of-Path Validation Bypass
* File Extension Validation Bypass (Null Byte)

## Write-ups

* [File path traversal, traversal sequences stripped non-recursively](https://github.com/Haidy-kareem/PortSwigger-Web-Security-Academy-Writeups/blob/main/path-traversal/file-path-traversal-traversal-sequences-stripped-non-recursively.md)
* [File path traversal, traversal sequences stripped with superfluous URL-decode](https://github.com/Haidy-kareem/PortSwigger-Web-Security-Academy-Writeups/blob/main/path-traversal/file-path-traversal-traversal-sequences-stripped-with-superfluous-url-decode.md)
* [File path traversal, validation of start of path](https://github.com/Haidy-kareem/PortSwigger-Web-Security-Academy-Writeups/blob/main/path-traversal/file-path-traversal-validation-of-start-of-path.md)
* [File path traversal, validation of file extension with null byte bypass](https://github.com/Haidy-kareem/PortSwigger-Web-Security-Academy-Writeups/blob/main/path-traversal/file-path-traversal-validation-of-file-extension-with-null-byte-bypass.md)
