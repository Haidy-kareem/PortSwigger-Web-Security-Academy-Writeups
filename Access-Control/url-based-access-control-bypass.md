## Lab Overview

This lab demonstrates a **Broken Access Control** vulnerability caused by a **platform misconfiguration**.

The application contains an administrator panel located at:

```
/admin
```

Although the endpoint is protected, the protection is implemented by the **front-end proxy** rather than the backend application.

The backend framework supports the `X-Original-URL` header and uses its value to determine the actual request path. Because the proxy ignores this header, an attacker can bypass the access control mechanism.

---

## Objective

Access the administrator panel and delete the user **carlos**.

---

### Step 1 – Discover the Restricted Endpoint

Browsing directly to:

```
GET /admin HTTP/2
```

returns:

```
403 Forbidden
```

This confirms that the endpoint exists but access is denied.

At this point, the response suggests that the request is being blocked **before reaching the application**, which often indicates a reverse proxy or web server restriction.

---

### Step 2 – Test URL Override Headers

Since some backend frameworks support URL rewriting headers, I tested the `X-Original-URL` header.

Instead of requesting `/admin` directly, I sent:

```
GET / HTTP/2
Host: target

X-Original-URL: /admin
```

### Why does this work?

The front-end proxy only validates the requested URL:

```
/
```

and allows the request.

However, the backend reads:

```
X-Original-URL: /admin
```

and processes the request as if it were sent directly to `/admin`.

As a result, the administrator panel becomes accessible.

---

### Step 3 – Identify the Delete Function

Inside the administrator panel, the delete action uses the endpoint:

```
/admin/delete
```

with the parameter:

```
username
```

Normally, requesting this endpoint directly is also blocked.

---

### Step 4 – Bypass the Protection Again

To reach the delete functionality, I applied the same technique.

Request:

```
GET /?username=carlos HTTP/2
Host: target

X-Original-URL: /admin/delete
```

The proxy sees:

```
/?username=carlos
```

while the backend processes:

```
/admin/delete?username=carlos
```

The request is accepted and the user **carlos** is successfully deleted.

---

### Root Cause

The application relies on the front-end proxy to enforce access control.

However, the backend trusts the `X-Original-URL` header without verifying whether the original request was authorized.

Because the proxy and backend interpret the request differently, an attacker can bypass the protection.

---

### Security Impact

An attacker can:

- Access hidden administrative endpoints.
- Bypass platform-level access controls.
- Execute privileged administrative actions.
- Compromise the application's authorization model.
