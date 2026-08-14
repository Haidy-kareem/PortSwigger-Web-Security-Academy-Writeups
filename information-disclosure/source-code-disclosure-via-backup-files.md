## 1. Discovering the Backup Directory

I started the reconnaissance by checking the application's `robots.txt` file:

```
/robots.txt
```

The file contained:

```
User-agent: *
Disallow: /backup
```

This revealed the existence of a `/backup` directory that was not linked from the main application.

Although `robots.txt` is intended to provide instructions to web crawlers, it is **not an access control mechanism**. The directory can still be requested directly.

<img width="1917" height="876" alt="image" src="https://github.com/user-attachments/assets/c8bc8fee-9586-4ab6-9a23-b1dc1020e875" />


---

## 2. Accessing the Backup Directory

I then accessed the discovered directory:

```
/backup
```

The server returned a directory listing containing a backup file:

```
ProductTemplate.java.bak
```

The `.bak` extension indicated that this was a backup copy of a Java source file.

This was particularly interesting because backup files may contain the original source code rather than the processed output normally returned by the application.

<img width="1918" height="883" alt="image" src="https://github.com/user-attachments/assets/680fc91e-fcc6-4bed-90f3-2803aae4bcc8" />


---

## 3. Accessing the Backup File

I opened the exposed backup file:

```
ProductTemplate.java.bak
```

Instead of receiving a normal application response, the server returned the source code directly.

The leaked Java source code revealed internal implementation details, including the database connection configuration.

The source code contained a database connection with hard-coded credentials.

<img width="1916" height="872" alt="image" src="https://github.com/user-attachments/assets/c14f1795-75b6-4e61-9ead-b7401afe914e" />


---

## 4. Identifying the Sensitive Information

The exposed source code showed that the application connects to a PostgreSQL database.

More importantly, the database password was hard-coded directly in the source code.

This is the sensitive information required by the lab.


<img width="1917" height="877" alt="image" src="https://github.com/user-attachments/assets/9d2acbf5-0fb4-4f17-9d8a-894c2e2c811c" />

---

## 5. Alternative Discovery Method: Content Discovery

The `/backup` directory did not have to be discovered using `robots.txt`.

Another approach would be to use **content discovery tools**, such as:

- Dirb
- Dirsearch

These tools enumerate directories and files using wordlists and can identify resources that are not linked from the visible application.

In this lab, Dirb or Dirsearch could also be used to discover the hidden backup directory.

---

## 6. Why This Is a Vulnerability

The application should not expose backup copies of source-code files through the web server.

An attacker does not need to compromise the server first. The sensitive source code is directly available through the web application.

---

## 7. Impact

Source code disclosure can reveal valuable information about the internal implementation of an application.

In this lab, the impact is more serious because the leaked source code contains hard-coded database credentials.

Potential impact includes:

- Disclosure of database credentials
- Exposure of internal application logic
- Disclosure of database structure and technology
- Identification of vulnerable code
- Potential unauthorized database access
- Further attacks using information obtained from the source code

The actual severity depends on the privileges associated with the disclosed credentials and what access they provide.
