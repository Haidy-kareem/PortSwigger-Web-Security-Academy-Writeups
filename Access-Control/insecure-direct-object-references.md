## Objective

Find the password belonging to the user Carlos and use it to log into his account.

---

### Reconnaissance

I started by exploring the application normally instead of immediately modifying requests.

I opened the **Live chat** functionality, sent a message, and then selected **View transcript**.

This allowed me to observe how the application retrieves and displays chat transcripts.

When inspecting the resulting request and URL, I noticed that the transcript was being retrieved using a `.txt` filename containing a number.

For example:

```
GET /download-transcript/2.txt
```

The filename appeared to be predictable, as the transcripts were assigned incrementing numbers.

This was interesting because the filename itself was being used as a direct reference to a server-side object.

---

### Testing the Object Reference

Since the transcript filename was predictable, I tested whether I could access another transcript simply by modifying the filename.

I changed:

```
GET /download-transcript/2.txt
```

to:

```
GET /download-transcript/1.txt
```

The server responded with a different chat transcript.

This confirmed that I could manipulate the object reference and retrieve a different file without going through any authorization process.

While reviewing the returned transcript, I found information containing **Carlos's password**.

---

### Exploitation

After obtaining the password, I returned to the main lab page and went to the login page.

I used the discovered credentials to authenticate as Carlos.

The login was successful, and the lab was solved.

---

### Why This Is an IDOR

The vulnerability exists because the application trusts a user-controlled reference:

```
/download-transcript/1.txt
```

and directly uses it to retrieve a server-side file.

The application does not properly check whether the authenticated user is authorized to access that particular transcript.



---

### Security Impact

An IDOR vulnerability can expose sensitive information or allow unauthorized actions depending on what the referenced object represents.

For example, an attacker might be able to access:

- Another user's private documents
- Invoices or orders
- Chat transcripts
- Personal information
- Account details
- Files stored on the server

If the vulnerable object supports modification or deletion, the impact can be even more severe because an attacker may be able to modify or delete another user's data.

---

### Notes

IDOR is not simply about changing a user ID.

The core idea is:

> **Can I manipulate a user-controlled reference and access an object that I am not authorized to access?**
> 

In this lab, the object was a **chat transcript file**, and the vulnerable reference was its **filename**.
