## Lab Goal

The lab has a protected admin panel that's vulnerable to an information disclosure issue, which reveals details about how its authentication actually works. The goal is to use that leaked information to bypass authentication, access the admin panel, and delete the user `carlos`.

Provided low-privilege credentials: `wiener:peter`

---

## Step One: Recon

I started with a **Dirsearch** scan against the target to see what was hiding:

```
dirsearch -u "https://TARGET.web-security-academy.net/"
```

Result: `/admin` returned **401 Unauthorized**  so the admin path exists, but it's gated behind some kind of authorization.

<img width="1507" height="812" alt="image" src="https://github.com/user-attachments/assets/1c4744a8-6512-4390-810d-7b9f9da61caf" />


---

## What I actually tried (and got wrong) before finding the fix

I didn't follow the official walkthrough from the start  I was poking around on my own, and that led to a bunch of dead ends before I landed on the real solution:

### Attempt 1: Changing the `id` parameter

I tried hitting `/my-account?id=admin` instead of `id=wiener`, hoping it would just hand me admin data directly.

**Result:** Failed the server responded with `302 Found`, redirecting me to `/login`. So that parameter alone wasn't the way in.

### Attempt 2: Discovering the hidden header

Back in **Burp Repeater**, I sent a request using an unusual method (`TRACE`) to `/my-account`. This revealed something important: the server was automatically attaching a header to every request:

```
X-Custom-IP-Authorization: <my IP>
```

So there's a hidden header silently carrying my IP with every request, and it looked like the backend was using it to decide who's allowed into the admin panel.

### Attempt 3: Misreading the IP I found during recon

Back when I first spotted the header via that `TRACE` request, the response had it populated with an IP address. I assumed this was some kind of special "admin" IP like the server was revealing which address counted as authorized so I manually added the header using that same value (`X-Custom-IP-Authorization: 156.218.253.226`) and tried `id=admin` again alongside it.

**Result:** Still failed another `302` redirect back to `/login`. Turns out that IP wasn't special at all; it was just my own IP being reflected back at me, since the header gets populated per-request with whoever's making the call. So adding the header wasn't the problem the value I assumed was correct was wrong.

### Attempt 4: Match & Replace with the same wrong value

I went to **Proxy > Match and Replace** and set up a rule to auto-inject that header into every request but I was still using my own public IP. Tested it against the `POST /login` request and intercepted traffic, and it still made no difference.

<img width="1568" height="764" alt="image" src="https://github.com/user-attachments/assets/142e3117-acda-48e3-ac6b-335fbf090ea1" />


---

## The turning point

After stepping back, This header is being auto-populated with my IP... what is it actually being used to verify?

It's checking whether the request is coming **from the server itself (localhost)**. The front-end/load balancer stamps this header with the visitor's real IP, and the back-end trusts any request that appears to come from a local IP treating it as an internal, privileged source (like an internal monitoring tool).

So the fix had nothing to do with any IP I'd seen during recon it was to **spoof** the header using the loopback address of my own machine (the one running Burp), `127.0.0.1`, so the request looks like it's coming from the server itself.

---

## The Actual Solution

1. Went to **Proxy > Match and Replace** and created a rule:
    - **Type:** Request header
    - **Match:** left blank (so it adds the header if it doesn't already exist)
    - **Replace:**
        
        ```
        X-Custom-IP-Authorization: 127.0.0.1
        ```
        
2. Enabled the intercept — now every outgoing request automatically carried that header set to `127.0.0.1`.
3. Browsed to `/my-account?id=wiener` in the browser  loaded fine (lab still not marked solved, since this is just the normal user page).
4. Navigated directly to `/admin`  **and the admin panel loaded!** 
5. From inside the admin panel, deleted the target user, and got the confirmation:
    
    ```
    User deleted successfully!
    ```
    
    <img width="1541" height="784" alt="image" src="https://github.com/user-attachments/assets/3b2e62f1-cb6b-44a6-9ff9-09e4a3135be9" />

    

   <img width="1566" height="784" alt="image" src="https://github.com/user-attachments/assets/9b899fcc-773e-4bf1-a381-ff1356d2e628" />

## Notes

- The `X-Custom-IP-Authorization` header was exposed simply by sending a `TRACE` request — a classic example of how enabling unusual HTTP methods can leak internal implementation details.
- My main mistake was sending **my own IP** instead of a **localhost IP**. Backends often implicitly trust `127.0.0.1`/`localhost` as an "internal, trusted" source, not just any IP that looks legitimate.
- Manipulating a parameter like `id=admin` alone wasn't enough — there was a second layer of verification (the header check), and I needed to defeat both, not just the first thing I noticed.
- **Match and Replace** in Burp is a great tool when you need to consistently inject a static header into every request without editing each one by hand.
