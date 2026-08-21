### Lab Overview

This lab contained a **blind OS command injection vulnerability** in the feedback submission functionality.

The application executed system commands using user-controlled input, but the output of the executed command was not directly displayed in the HTTP response. Because of this, I could not simply use a command such as `whoami` and read its output.

Instead, I used a **time-based technique** to determine whether my injected command was being executed.

### Identifying the Vulnerable Parameter

I intercepted the feedback submission request and tested the different parameters to identify which one could influence the underlying system command.

The request contained parameters such as:

- `name`
- `email`
- `subject`
- `message`

The vulnerable parameter was the **`email`** parameter.

### Testing for Blind Command Injection

Since the command output was not visible in the response, I used a command that would intentionally introduce a delay.

I injected the following payload into the `email` parameter:

```
;sleep 10 ;
```

The `sleep 10` command should make the server wait for approximately **10 seconds** if the injected command is executed successfully.

### Measuring the Response Time

I first sent a normal request and observed that the response was returned very quickly, taking only a small number of milliseconds.

After injecting:

```
;sleep 10 ;
```

the response took approximately **10 seconds** to return.

This significant increase in response time indicated that the `sleep` command had been executed by the server.

### Why This Works

The semicolon `;` can be used in shell syntax to separate commands.

The payload:

```
;sleep 10 ;
```

attempts to terminate the existing command context and execute `sleep 10` as an additional command.

If the application places the user-controlled `email` value into an OS command without properly sanitizing shell metacharacters, the injected command can be interpreted by the shell.

Because the output of the command is not returned to me, I cannot directly see its result. Instead, the intentional delay provides a **side channel** that allows me to determine whether the command was executed.

### Result

The response time increased from milliseconds to approximately **10 seconds** after using:

```
;sleep 10 ;
```

This confirmed that the `email` parameter was vulnerable to **blind OS command injection** and that the injected OS command was successfully executed.

### Notes

- **Blind OS command injection** means that command execution is possible, but the command's output is not directly visible.
- A time delay can be used as a **side channel** to confirm command execution.
- The `sleep 10` command was useful because its effect could be measured through the HTTP response time.
- The important part of this lab was not seeing command output, but recognizing that a measurable change in response time can prove that the injected command was executed.
