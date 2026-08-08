## Lab Description

This lab demonstrates a **Broken Access Control** vulnerability where authorization checks are incorrectly enforced based on the **HTTP request method**.

The application contains an admin endpoint that manages user roles. Although the endpoint should only be accessible to administrators, the front-end only protects one HTTP method, while the back-end unintentionally accepts another.

Our goal is to **promote the low-privileged user (`wiener`) to administrator**.

---

## Objective

Promote the user **wiener** to an administrator by exploiting flawed method-based access controls.

---

### Step 1 – Explore the Application as Administrator

The lab provides administrator credentials:

```
administrator:admin
```

After logging in, navigate to:

```
/admin
```

The admin panel contains functionality for managing user roles.

Using Burp Suite, intercept the request generated when clicking the **Upgrade** button.

Captured request:

```
POST /admin-roles HTTP/2

username=carlos&action=upgrade
```

From this request we learn:

- Endpoint: `/admin-roles`
- Parameters:
    - `username`
    - `action`
- Default HTTP Method: `POST`

At this stage, we are **only collecting information** about how the application works.

---

### Step 2 – Switch to a Normal User

Logout from the administrator account.

Login using:

```
wiener:peter
```

Capture your new session cookie.

Example:

```
Cookie: session=tACfLolJERJYEsU2vxgE0uVvasGC9I78
```

This cookie identifies every future request as **wiener**.

---

### Step 3 – Replay the Admin Request

Send the administrator request to Burp Repeater.

Replace:

- Administrator session cookie
- Target username

Modified request:

```
POST /admin-roles HTTP/2

Cookie: session=<wiener_session>

username=wiener
action=upgrade
```

The server correctly blocks this request because the current user is not an administrator.

---

### Step 4 – Analyze the Lab Title

The lab title is:

> Method-Based Access Control Can Be Circumvented
> 

This immediately suggests that the vulnerability is **related to the HTTP method**, not the endpoint or parameters.

Instead of changing the URL or parameters, focus on changing only the request method.

---

### Step 5 – Change the HTTP Method

Convert the request from **POST** to **GET**.

Final request:

```
GET /admin-roles?username=wiener&action=upgrade HTTP/2

Cookie: session=<wiener_session>
```

Notice that:

- Parameters are moved from the request body to the URL.
- The POST body is removed.
- The session cookie remains the one belonging to **wiener**.

---

### Result

The server processes the request successfully and upgrades **wiener** to administrator.

The authorization mechanism only protected **POST** requests, while the back-end still accepted **GET** requests for the same privileged action.

---

### Why This Works

The application performs authorization checks depending on the HTTP method.

```
POST  ---> Authorization Check ✔
GET   ---> Missing Authorization Check ✘
```

Since the back-end accepts GET requests without enforcing the same authorization rules, a low-privileged user can execute administrative functionality simply by changing the request method.

This is a classic example of **Broken Access Control** caused by inconsistent authorization logic.

---

### Root Cause

The application assumes that sensitive functionality will always be accessed using POST requests.

However, the back-end also accepts GET requests without applying identical authorization checks.

As a result:

- POST is protected.
- GET is not.

This inconsistency allows privilege escalation.

---

### Security Impact

An attacker can:

- Escalate privileges
- Modify user roles
- Access administrative functionality
- Compromise the authorization model
