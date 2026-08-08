## 1. Lab Description

This lab contains an admin panel with a multi-step process for changing a user's role.

The objective is to log in as the low-privileged user:

```
wiener:peter
```

and exploit a broken access control mechanism to promote the account to administrator.

The lab also provides:

```
administrator:admin
```

These credentials can be used to understand the intended administrative workflow.

---

## 2. Understanding the Multi-Step Process

Before attempting to exploit the vulnerability, I first logged in using the provided administrator credentials:

```
Username: administrator
Password: admin
```

I accessed the admin panel and performed a normal role-change operation.

The goal was to understand how the application handles the role upgrade and, more importantly, to observe the individual HTTP requests generated during the process.

I kept Burp Suite running and captured the complete workflow.

The process consisted of several steps, including:

```
Access role management - Select the target user - Request the role upgrade -Request the role upgrade

```

I saved the generated requests in Burp Suite for further analysis.

---

## 3. Identifying the Final Request

After completing the legitimate workflow as an administrator, I reviewed the captured requests.

I noticed that the final step contained the actual role-upgrade action.

The relevant request contained parameters similar to:

```
action=upgrade&confirmed=true&username=carlos
```

This was interesting because the final request appeared to perform the privileged action directly.

At this point, I wanted to determine whether the final endpoint independently verified that the current user was an administrator.

---

## 4. Switching to the Low-Privileged User

I then logged out of the administrator account and logged in using:

```
Username: wiener
Password: peter
```

I captured Wiener's requests in Burp Suite.

One of the requests in HTTP history showed Wiener's authenticated session:

```
GET /my-account?id=wiener HTTP/2
Cookie: session=<wiener-session>
```
<img width="1918" height="915" alt="image" src="https://github.com/user-attachments/assets/0de7c3c6-07fe-4ba2-bcd9-08218ae5f18d" />


> **Figure 1 — HTTP request showing Wiener's authenticated session**
> 

---

## 5. Replaying the Final Step

Instead of trying to access the administrator panel directly as Wiener, I went back to the final confirmation request that I had previously captured while using the administrator account.

I sent this request to Burp Repeater.

The important part was that I replaced the administrator's session cookie with **Wiener's session cookie**.

I also changed the target username from `carlos` to `wiener`, because the goal was to promote my own account.

The resulting request was:

```
POST /admin-roles HTTP/2

Cookie: session=<wiener-session>

...

action=upgrade&confirmed=true&username=wiener
```

The request was accepted by the server and returned a redirect response:

```
HTTP/2 302 Found
Location: /login
```

<img width="1917" height="913" alt="image" src="https://github.com/user-attachments/assets/be9be880-6e97-4315-975b-d2ef1dea0f76" />


> **Figure 2 — Replaying the final confirmation request using Wiener's session**
> 

---

## 6. Why the Attack Worked

The application correctly restricted access to the earlier administrative steps.

However, the final confirmation request did not properly verify whether the authenticated user had administrator privileges.

As a result, I was able to take a request that was normally intended for an administrator and replay it using Wiener's low-privileged session.

The server processed:

```
action=upgrade
confirmed=true
username=wiener
```

without rejecting the request based on Wiener's privileges.

This allowed Wiener to promote his own account.

---

## 7. Verifying the Privilege Escalation

After sending the request, I returned to the application using Wiener's account.

I accessed the admin functionality and confirmed that Wiener now had administrator privileges.

I also tested the functionality by using the admin panel to upgrade another user, confirming that the privilege escalation had actually succeeded.

This demonstrated that the vulnerability was not simply a successful HTTP request, but a real **vertical privilege escalation**.

---

## 8. Notes

The main lesson from this lab is:

> **Every step that performs a privileged action must independently enforce authorization.**
> 

A multi-step workflow is not secure simply because the first steps are protected.

As penetration testers, we should therefore:

1. Map the complete workflow.
2. Capture every request.
3. Identify which request actually performs the sensitive action.
4. Replay individual requests using a lower-privileged session.
5. Check whether the server independently enforces authorization.
