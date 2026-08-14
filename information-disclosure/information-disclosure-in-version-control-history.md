## Overview

Version control systems such as Git keep a history of changes made to an application's source code.

If a `.git` directory is accidentally exposed through a web application, an attacker may be able to discover and download the repository, inspect previous commits, and recover sensitive information that is no longer present in the current version of the application.

In this lab, the exposed Git history contained a previously hard-coded administrator password.

---

## 1. Discovering the Exposed Git Repository

I started with **content discovery** using Dirsearch to identify hidden files and directories that were not linked from the main application.

The scan revealed that the `.git` directory was publicly accessible:

```
/.git
```

This was an important finding because a publicly exposed Git repository may contain the application's version-control history, previous source-code versions, and sensitive information that has been removed from the current application.

<img width="1917" height="1009" alt="image" src="https://github.com/user-attachments/assets/a3511bc0-7be6-4629-8bf9-367692ecc263" />


---

## 2. Downloading the Exposed Repository

After discovering the exposed `.git` directory, I downloaded the repository data locally using `wget`:

```bash
wget -r https://YOUR-LAB-ID.web-security-academy.net/.git/
```

This allowed me to inspect the Git repository locally using the standard Git tools instead of manually requesting individual `.git` files.

The recovered repository contained the application files as well as the `.git` directory.

---

## 3. Inspecting the Git History

I then explored the downloaded repository using Git.

The commit history revealed two commits:

```
91988f7 Remove admin password from config
146b547 Add skeleton admin panel
```

The commit message:

```
Remove admin password from config
```

was particularly interesting because it indicated that an administrator password had previously existed in the configuration and had later been removed.

---

## 4. Analyzing the Commit Difference

I compared the initial commit with the latest commit:

```bash
git diff 146b547 91988f7
```

The diff showed that the hard-coded administrator password had been replaced with an environment variable:

```diff
-ADMIN_PASSWORD=<old-password>
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```

This revealed that the original administrator password was still stored in the Git history even though it had been removed from the current configuration.

<img width="937" height="162" alt="image" src="https://github.com/user-attachments/assets/afe88948-1b67-4ea8-8b7d-bcc16fa25675" />


---

## 5. Using the Leaked Credentials

After recovering the administrator password from the previous version of `admin.conf`, I returned to the lab and used the credentials to log in as the administrator.

I then accessed the administrator interface and deleted the user:

```
carlos
```

The lab was successfully solved.

---

## 7. Impact

Exposing a version-control repository can reveal sensitive information from both current and historical versions of an application.

Potentially exposed information includes:

- Passwords
- API keys
- Database credentials
- Secret keys
- Internal endpoints
- Deleted files
- Previous application logic
- Vulnerable source code

In this lab, the exposed history contained a valid administrator password, resulting in direct administrator account compromise.
