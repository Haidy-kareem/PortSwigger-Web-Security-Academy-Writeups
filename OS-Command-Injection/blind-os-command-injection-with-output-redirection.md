## Objective

The goal of this lab was to exploit a **blind OS command injection** vulnerability.

In this type of vulnerability, the application executes operating system commands based on user-controlled input, but the output of the command is not directly displayed in the application's response.

The challenge was therefore to execute a command and find a way to retrieve its output.

---

## 1. Identifying the Vulnerability

The application contains a feedback form where user input is sent to the server.

While testing the application, I found that one of the parameters could be used to inject an operating system command.

Because this was a blind command injection vulnerability, simply executing:

```bash
whoami
```

would not display the result directly in the response.

I therefore needed another way to retrieve the command output.

---

## 2. Using Output Redirection

I used the Linux output redirection operator `>` to save the result of the command into a file.

The payload I used was:

```bash
whoami > /var/www/images/images.txt
```

The `whoami` command returns the username of the user running the command.

Normally, its output would be displayed in the terminal. However, using `>` redirects the output into a file instead.

In this case, the output was saved to:

```
/var/www/images/images.txt
```

This was useful because `/var/www/images/` was the directory used by the application to store its images.

---

## 3. Understanding the File Path

At this point, the command had created the following file:

```
/var/www/images/images.txt
```

It is important to distinguish between a **filesystem path** and a **URL**.

The following is a filesystem path on the server:

```
/var/www/images/images.txt
```

It is not necessarily the URL that should be entered into the browser.

I initially tried to access the filesystem path directly through the browser, but this resulted in a `Not Found` response.

This happened because the application does not expose `/var/www/images/` directly as part of the URL.

---

## 4. Finding the Image URL Structure

To understand how the application accesses images, I inspected an existing image.

I found an image URL similar to:

```
/image?filename=30.jpg
```

This showed that the application uses an `/image` endpoint and passes the image name through the `filename` parameter.

The important structure was:

```
/image?filename=FILENAME
```

Therefore, I could use the same endpoint to request the file that I had created.

---

## 5. Accessing the Generated File

The file I created was named:

```
images.txt
```

So instead of trying to access:

```
/var/www/images/images.txt
```

I requested the file through the application's image endpoint:

```
/image?filename=images.txt
```

The application returned the contents of the file.

The result was:

```
peter-dh4t1R
```

This was the output of the `whoami` command.

---

## 6. Why This Worked

The complete process was based on combining two different pieces of information.

First, I used command injection to execute:

```bash
whoami > /var/www/images/images.txt
```

This caused the server to execute `whoami` and save its output in the image directory.

Second, I analyzed how the application normally retrieves images and found that it uses:

```
/image?filename=FILENAME
```

Since my file was called `images.txt`, I could retrieve it using:

```
/image?filename=images.txt
```

The response contained:

```
peter-dh4t1R
```

which confirmed that the command had executed successfully.

---

## 7. Final Payload

The command injection payload used during the lab was:

```bash
whoami > /var/www/images/images.txt
```

The generated file was:

```
/var/www/images/images.txt
```

The URL used to retrieve the file was:

```
/image?filename=images.txt
```

The returned command output was:

```
peter-dh4t1R
```

---

## 8. Notes

### Blind Command Injection

Blind command injection occurs when an application executes an injected command but does not directly return its output to the user.

### Output Redirection

The `>` operator can be used to redirect command output into a file:

```bash
command > file
```

For example:

```bash
whoami > /var/www/images/images.txt
```

### Filesystem Path vs URL

A filesystem path such as:

```
/var/www/images/images.txt
```

is different from the URL used by the application to retrieve the file:

```
/image?filename=images.txt
```

The application determines how the URL maps to files on the server.

### Application Functionality Can Help Exploit Blind Vulnerabilities

Instead of guessing the URL, I inspected how the application normally loads its images.

The existing image endpoint revealed the required URL structure, which allowed me to retrieve the output of the injected command.
