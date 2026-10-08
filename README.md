# Penetration Testing CTF Writeups

A collection of practical penetration-testing and web-security challenge
writeups completed in a controlled CTF/lab environment.

> **Disclaimer:** These techniques are intended for authorized labs,
> CTFs, and systems where you have explicit permission to test. Do not
> use them against real systems without authorization.

## Challenges

  \#   Challenge           Vulnerability / Technique
  ---- ------------------- ---------------------------------------------
  1    Auth Bypass         OTP parameter manipulation
  2    Terminal Override   Command injection / hidden-file enumeration
  3    IDOR Rush           Insecure Direct Object Reference (IDOR)
  4    X-MY-SCRIPT         Cross-Site Scripting (XSS)
  5    Roommate Finder     SQL Injection
  6    Trustfall           Server-Side Request Forgery (SSRF)
  7    Eval Me             Server-Side Template Injection (SSTI)
  8    Inside Out          Path Traversal

------------------------------------------------------------------------

## 1. Auth Bypass

### Objective

Use authentication-bypass techniques to retrieve the challenge flag.

### Tools

-   Burp Suite
-   Browser
-   FoxyProxy / browser proxy configuration

### Procedure

1.  Open Burp Suite and turn **Intercept ON**.
2.  Configure the browser proxy to Burp, normally `127.0.0.1:8080`.
3.  Trigger the OTP/login verification request in the target lab.
4.  Intercept the request in Burp.
5.  Locate the `otp` parameter. It may appear in:
    -   POST form data
    -   GET query parameters
    -   JSON data
6.  Remove the complete `otp` parameter from the request.
7.  Forward the modified request.
8.  Inspect the server response for a `CTF{...}` flag.

### Key Learning

Authentication logic should never rely only on the presence or absence
of a client-controlled parameter. Server-side verification must be
enforced.

------------------------------------------------------------------------

## 2. Terminal Override

### Objective

Use command-injection skills to locate a hidden file containing the
flag.

### Procedure

1.  Open the challenge and locate the command-execution input.
2.  Run:

``` bash
dir
```

3.  Enumerate hidden files:

``` bash
dir -a
```

4.  Identify the hidden file, shown in the lab as:

``` text
.file.sh
```

5.  Read its human-readable content:

``` bash
strings .file.sh
```

6.  Search the output for the `CTF{...}` flag and submit it.

### Key Learning

Command execution vulnerabilities can allow unintended operating-system
commands to be executed. Proper input validation and command
allowlisting are important defenses.

------------------------------------------------------------------------

## 3. IDOR Rush

### Objective

Exploit an insecure direct object reference to obtain elevated access
and retrieve the flag.

### Procedure

1.  Open Burp Suite and enable **Intercept**.
2.  Configure the browser to use Burp.
3.  Start the account-registration process in the lab.
4.  Submit the form so Burp captures the request.
5.  Find a parameter such as:

``` text
user_id=123
```

6.  Change the value to:

``` text
user_id=0
```

7.  Forward the modified request.
8.  Check the response in Burp HTTP history.
9.  If the lab is vulnerable, the response shows elevated access and/or
    the flag.

### Key Learning

Object identifiers must be authorized server-side. Changing an ID should
never allow a user to access another user's resources or privileges.

------------------------------------------------------------------------

## 4. X-MY-SCRIPT

### Objective

Use an XSS vulnerability to trigger the required JavaScript action and
retrieve the flag.

### Procedure

1.  Open the X-MY-SCRIPT challenge.
2.  The lab provides a hint to log:

``` text
VULN_TO_XSS!
```

3.  Use an XSS payload such as:

``` html
<img src=a onerror=console.log("VULN_TO_XSS!")>
```

4.  Place the payload in the vulnerable URL parameter.
5.  Submit the crafted URL through the challenge interface.
6.  Check the result/console.
7.  The lab returns the flag when the payload executes successfully.

### Key Learning

Untrusted input reflected into HTML or JavaScript contexts can lead to
XSS. Output encoding and contextual input handling are key defenses.

------------------------------------------------------------------------

## 5. Roommate Finder

### Objective

Use SQL injection to retrieve the flag from the challenge database.

### Procedure

1.  Open the Roommate Finder challenge.
2.  Fill the form with sample values.
3.  Submit the form and intercept the request with Burp Suite.
4.  Send the request to **Repeater**.
5.  Modify the vulnerable parameters using the SQL injection technique
    demonstrated in the lab.

Example payload from the writeup:

``` text
name='&guests=na&neatness=na&sleep='UNION+SELECT+1,flag,null,null,null,null+FROM+flag&awake='UNION+SELECT+1,flag,null,null,null,null+FROM+flag
```

6.  Send the request.
7.  Inspect the successful response for the `CTF{...}` flag.

### Key Learning

SQL injection occurs when user input is incorporated into SQL queries
without safe parameterization. Prepared statements/parameterized queries
are the primary defense.

------------------------------------------------------------------------

## 6. Trustfall

### Objective

Exploit an SSRF vulnerability to access the flag through the vulnerable
endpoint.

### Procedure

1.  Open the Trustfall challenge.
2.  Test the endpoint root path.
3.  Capture the request using Burp Suite.
4.  Send the request to **Repeater**.
5.  Modify the `path` parameter according to the lab instructions.
6.  The writeup demonstrates:

``` text
trustfall?path=1/flag
```

7.  Send the request and inspect the response.
8.  Retrieve the `CTF{...}` flag from the response.

### Key Learning

SSRF can allow a server to make requests to unintended internal or
protected resources. Servers should strictly validate and allowlist
outbound destinations and paths.

------------------------------------------------------------------------

## 7. Eval Me

### Objective

Exploit the calculator application's server-side evaluation logic to
retrieve the flag.

### Procedure

1.  Open the calculator web application.
2.  Perform a normal calculation such as:

``` text
7*7
```

3.  Capture the request using Burp Suite.
4.  Send the request to **Repeater**.
5.  Modify the `calc` parameter using the SSTI payload demonstrated in
    the lab:

``` text
{{config.__class__.from_envvar["__globals__"]["__builtins__"]["__import__"]("os").popen("cat+flag.txt").read()}}
```

6.  Send the modified request.
7.  Inspect the response for the flag.

### Key Learning

Server-side template injection can occur when user-controlled data is
interpreted as template code. Templates should not evaluate untrusted
input as executable expressions.

------------------------------------------------------------------------

## 8. Inside Out

### Objective

Use path traversal to access a file containing the flag.

### Procedure

1.  Open the challenge URL.
2.  Observe the `path` parameter, for example:

``` text
/?path=axe.txt
```

3.  Test path traversal against the lab.
4.  If direct traversal is filtered, URL-encode the traversal sequence.
5.  The writeup demonstrates the encoded form:

``` text
%252E%252E
```

6.  Use the traversal to enumerate the accessible directory.
7.  Locate the flag file.
8.  Modify the URL to target the flag:

``` text
/?path=%252E%252E/flag.txt
```

9.  Inspect the response and copy the `CTF{...}` flag.

### Key Learning

Path traversal can expose files outside an application's intended
directory. Applications should canonicalize paths and enforce strict
directory boundaries.

------------------------------------------------------------------------

## Skills Practiced

-   Burp Suite Proxy & Repeater
-   HTTP request/response analysis
-   Authentication bypass
-   Command injection
-   IDOR
-   Cross-Site Scripting (XSS)
-   SQL Injection
-   Server-Side Request Forgery (SSRF)
-   Server-Side Template Injection (SSTI)
-   Path Traversal
-   Basic Linux command-line enumeration
-   Web application security testing

## Tools Used

-   Kali Linux
-   Burp Suite
-   Web Browser
-   FoxyProxy
-   CyberChef
-   Linux command-line utilities

## Lab Workflow

``` text
Recon / Understand the Application
            ↓
Identify the Input or Request
            ↓
Capture Traffic with Burp Suite
            ↓
Modify the Request
            ↓
Send via Proxy / Repeater
            ↓
Analyze the Response
            ↓
Locate CTF Flag
            ↓
Document the Finding
```

## References

-   Source material: `PT writeup.pdf`
-   Challenge environment: Authorized CTF / penetration-testing lab

## Notes

This README summarizes the procedures in the provided practical writeup.
The challenge names, terminology, workflow, and example techniques are
based on the supplied document.
