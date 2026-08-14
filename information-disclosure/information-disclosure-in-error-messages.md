## Reconnaissance

I started by interacting with the application normally and observing the requests and responses through Burp Suite.

I identified a parameter that appeared to expect a numeric value. Instead of providing a valid number, I supplied a string value:

```
hehe
```

This caused the application to return a detailed server-side error instead of a generic error message.

---

## Information Disclosure

The response contained a full Java stack trace, including:

```
java.lang.NumberFormatException
```

and:

```
java.lang.Integer.parseInt
```

This immediately revealed that the supplied input was being processed as an integer by the backend.

More importantly, the response disclosed the technology and exact framework version at the end of the stack trace:

```
Apache Struts 2 2.3.31
```

Therefore, the application was exposing:

**Framework:** Apache Struts 2

**Version:** 2.3.31

---

## Why This Is Information Disclosure

The application should not expose internal stack traces or framework details to users.

A safer response would be something generic such as:

```
Internal Server Error
```

Instead, the application disclosed:

- The Java exception type
- The internal parsing method
- Stack trace information
- Backend implementation details
- The framework name
- The exact framework version

This demonstrates how a seemingly simple input error can reveal valuable technical information.

---

## Security Relevance

After identifying:

```
Apache Struts 2.3.31
```

I researched the framework version and confirmed that it is associated with known security vulnerabilities, including remote code execution issues.

This demonstrates why version disclosure can be more significant than simply revealing a technology name.

The attack chain can be summarized as:

```
Unexpected input
        ↓
Verbose error message
        ↓
Technology and version disclosure
        ↓
Identify known vulnerabilities affecting the version
        ↓
Potential further exploitation
```

---

## Lab Result

The objective of the lab was only to obtain the vulnerable framework version.

After identifying:

```
2.3.31
```

I submitted the version number and successfully solved the lab.

---

## Real-World Consideration

In a real-world security assessment, identifying a vulnerable framework version would usually be considered **evidence of an information disclosure**, but the version alone would not necessarily prove that the underlying vulnerability is exploitable.

For a stronger real-world finding, I would normally need to demonstrate the actual security impact of the disclosed vulnerability in an authorized testing environment.

For example, if the identified version is affected by a remote code execution vulnerability, a proper assessment would typically require safely demonstrating code execution, such as executing a harmless command like:

```
whoami
```

and providing the resulting output as evidence.

The important distinction is:

**Lab:** identifying the vulnerable version is sufficient.

**Real-world assessment:** the disclosed version should ideally be followed by validated evidence of exploitability and impact, where explicitly authorized and safe to do so.

---

## Impact

The immediate impact is the disclosure of sensitive technical information about the application's backend.

This information can help an attacker:

- Fingerprint the technology stack
- Identify outdated or vulnerable components
- Search for known vulnerabilities
- Study publicly available framework source code
- Develop or select appropriate exploitation techniques

The final severity therefore depends on what an attacker can achieve using the disclosed information.
