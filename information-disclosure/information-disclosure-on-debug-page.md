## 1. Finding the Debug Page Through the Source Code

I started by inspecting the application's source code to look for hidden functionality and developer comments.

While reviewing the HTML source, I found a commented-out link:

```html
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```

Although the link was commented out and not visible in the normal application interface, it revealed the location of the debug page:

```
/cgi-bin/phpinfo.php
```

This is an example of information disclosure through **developer comments**, where internal functionality is accidentally left in the application's source code.

<img width="1918" height="872" alt="image" src="https://github.com/user-attachments/assets/755d1a3b-c802-446a-9fdc-23d04c11aa55" />


Developer comment revealing the hidden debug endpoint.
---

## 2. Accessing the Debug Page

I then accessed the discovered endpoint directly:

```
/cgi-bin/phpinfo.php
```

The page loaded successfully and displayed detailed information about the PHP environment and the application server.

The page exposed a large amount of internal information, including:

- PHP configuration
- Server information
- Environment variables
- HTTP request information
- File paths
- Loaded modules
- Application environment details

One of the exposed environment variables was:

```
SECRET_KEY
```

The page also revealed other information such as:

```
REMOTE_HOST
USER
HOME
SCRIPT_FILENAME
HTTP_HOST
SERVER_SOFTWARE
PATH
```

<img width="1917" height="877" alt="image" src="https://github.com/user-attachments/assets/eba7192e-6697-4bab-a319-09f29055bab6" />


Exposed PHP debug information revealing environment variables, including the SECRET_KEY.

---

## 3. What is `phpinfo.php`?

The `phpinfo()` function in PHP is mainly used for debugging and displaying information about the PHP environment.

A page containing something like:

```php
<?php phpinfo(); ?>
```

can expose information such as:

- PHP version
- Server software
- Loaded modules
- Configuration settings
- Environment variables
- Internal file paths
- HTTP headers
- CGI information

This information can be useful during debugging, but exposing it publicly in a production application can lead to information disclosure.

The main issue in this lab is not that `phpinfo()` exists, but that the debug page is **publicly accessible** and exposes sensitive application information.

---

## 4. Finding the Page Using Content Discovery

Another way to discover the debug page is through **content discovery**.

Instead of inspecting the source code manually, tools such as **Dirb** and **Dirsearch** can be used to enumerate hidden directories and files.

For example, Dirsearch can test many possible paths using a wordlist and identify resources that exist on the server but are not linked from the visible application.

I tested the application using content discovery tools and was able to identify the `/cgi-bin/` directory and the exposed `phpinfo.php` file.

<img width="1915" height="1022" alt="image" src="https://github.com/user-attachments/assets/bec48c1c-c761-4948-b939-90824534aec4" />


<img width="1917" height="846" alt="image" src="https://github.com/user-attachments/assets/aa70697e-3327-4927-b807-545f4453f431" />


Content discovery identifying the exposed CGI directory and debug functionality.

---

## 5. Discovering `SECRET_KEY`

After accessing the debug page, I inspected the exposed environment variables and searched for:

```
SECRET_KEY
```

The page displayed the value associated with this environment variable.

<img width="1916" height="873" alt="image" src="https://github.com/user-attachments/assets/1a40d556-0e73-490a-907c-ef99b24ee3a9" />


<img width="1918" height="821" alt="image" src="https://github.com/user-attachments/assets/c6006f6e-8023-4569-a150-35d4c2fd27d8" />


---

## 6. Why This Is a Vulnerability

The vulnerability exists because a debugging page containing internal application information was left accessible to users.

The page exposes information that should normally only be available to developers or administrators.

The attack path was:

```
Developer comment
        ↓
Hidden debug endpoint
        ↓
/cgi-bin/phpinfo.php
        ↓
PHP environment information
        ↓
SECRET_KEY disclosed
```

This is an example of **Information Disclosure due to insecure configuration and exposed debugging functionality**.

---

## 7. Impact

The impact depends on what information is exposed through the debug page.

In this lab, the page exposed the `SECRET_KEY`, which is much more sensitive than simply revealing a PHP or server version.

A publicly accessible debug page may also expose:

- Environment variables
- Application configuration
- Internal file paths
- Server information
- PHP modules
- Framework information
- Request data
- Potentially credentials or other secrets

An attacker can use this information to better understand the application and potentially use exposed secrets or configuration as part of further attacks.

---


## Final Result

The hidden debug page was discovered through the application's source code and could also be found through content discovery.

The debug page exposed sensitive PHP environment information, including the `SECRET_KEY`.

The lab was solved by obtaining the value of the `SECRET_KEY` and submitting it as required.
