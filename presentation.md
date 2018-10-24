---
theme : "black"
transition: "slide"
highlightTheme: "dracula"
controls: false
transitionSpeed: fast
progress: false
---

# OWASP Top 10 - 2017

The Ten Most Critical Web Application Security Risk

---

## About

--

The **Open Web Application Security Project** (OWASP) is an open community dedicated to enabling organizations to develop, purchase, and maintain applications and APIs that can be trusted.

note:

* non-profit organization since 2001
* 8 employees and 42k+ volunteers

--

The **OWASP Top 10** is an awareness document for web application security. It represents a broad consensus about the most critical security risks to web applications.

Adopting the OWASP Top 10 is perhaps the most effective first step towards changing the software development culture within your organization into one that produces secure code.

---

## Injection

--

### SQL Injection

```csharp
var result = "SELECT * FROM users WHERE name = '" + userName + "';"
```

--

![XKCD - Little Bobby Tables](./exploits_of_a_mom.png)

--

### OS Commands

```csharp
void RunExport(string userName)
{
    var p = new Process();
    p.StartInfo.FileName = "export.exe";
    p.StartInfo.Arguments = " -user " + userName;
    p.Start();
}
```

--

### CSV Injection

```csv
UserID,UserName,ValidFrom
1,Fero,2018-1-1
2,Jozo,2018-2-1
3,=cmd|'/C ping -t 8.8.8.8'!A1,2018-4-1
```

--

### RegEx injection

```csharp
var testPassword = new Regex(userName);
var match = testPassword.Match(password);
if (match.Success) ...
```

note:

* It's called ReDoS
* Google for Evil Regex



--

### ...and other

* LDAP injection
* XPath injection
* NoSQL queries injection
* etc ...

--

### Prevention

* Use safe API (e.g. parameterized SQL queries, timeoutable Regex, ...)
* Use whitelist on server-side validation
* Properly sanitize (escape) user input
* Conduct a secure code review

---

## Broken Authentication

note: Two main souces

* Problems with user names and paswords
* Broken session Management

--

### Authentication

* Credential stuffing
* Brute force attacks
* Weak passwords
* Broken forgot-password process
* Missing 2FA

--

### Session Management

* Low session ID entropy
* Content inside session ID
* Not using HttpOnly, SameSite and Secure cookies
* Insufficient session expiration
* Session fixation

--

### Prevention

* Implement 2FA
* No default credentials
* Check for weak passwords or password reuse
* Implement NIST 800-63 guidelines for password policy
* Limit or increasingly delay failed login attempt
* Implement attack detection mechanisms

---

## Sensitive Data Exposure

--

### Things to protect

* Passwords
* Credit card numbers
* Personal information
* Business secrets
* Everything under GDPR

note: Usually common sense works here quite good, but we are failing to explicitly state data protection requirements.

--

### How to protect sensitve data

* Do not transfer unecrypted (FTP, HTTP, SMTP)
* Identify and classify processed data
* Ensure proper key management
* Do not store what you don't need
* Limit exposure time
* Encrypt w/ state of the art crypto
* Perform independent security audit

---

## XML External Entities (XXE)

--

### Billion laughs attack

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
 <!ENTITY lol "lol">
 <!ELEMENT lolz (#PCDATA)>
 <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
 <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
 <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
 <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
 <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
 <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
 <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
 <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
 <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

<!-- .element: class="stretch" -->

--

### Cannonical example

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo
```

--

### Alternative to probe private network

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "https://192.168.1.1/private" >]>
<foo>&xxe;</foo
```

--

### Alternative to DoS server

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///dev/random" >]>
<foo>&xxe;</foo
```

--

### Prevention

* Configure your XML parser
  * Disable DTD
  * Disable external entities
* Validate incoming XML using XSD
* Add timeout user input parsing
* Use JSON 😝

---

## Broken Access Control

note: Access control, sometimes called authorization

Broken authentication is failture of recognizing who you are
Broken authorization is failure of recognizing what you are allowed to do

--

### It's your fault

* Bypassing access control checks by modifying the URL
* Missing access controls on API accessed with POST, PUT or DELETE

--

### Prevention

* Test, test & test
* Properly implement business logic
  * Deny by default
* Log access control failures, alert admins when appropriate
* Rate limit API and controller access to minimize the harm from automated attack tooling

---

## Security Misconfiguration

Note: In traditional companies, you can blame IT, in DevOps companies, it's again your fault.

--

### Examples

* Missing appropriate security hardening across any part of the application stack
* Unnecessary features are enabled or installed
* Default accounts and their passwords still enabled and unchanged.
* Directory listing enabled

Note:

* 40k of mongoDBs found with default username and password in 2017.
* Amazon S3 bucket misconfigurations

--

### Prevention

* Talk to IT guys
* OS hardening, ideally fully automated
* Minimal platform - use only what you really need

---

## Cross-Site Scripting (XSS)

--

### 3 types of XSS

* Reflected XSS
* Stored XSS
* DOM XSS

--

### Reflected XSS

The application or API includes unvalidated and unescaped user input as part of HTML output.

```html
https://example.com/page?var=<script>alert('xss')</script>
```

--

### Stored XSS

 The application or API stores unsanitized user input that is viewed at a later time by another user or an administrator.

 ```basic
print "<html>"
print "Latest comment:"
print database.latestComment
print "</html>"
 ```

 Note: Stored XSS is often considered a high risk.

--

### DOM XSS

Legitimate JavaScript frameworks, single-page applications, and APIs that dynamically include attacker-controllable data to a page are vulnerable to DOM XSS.

Note: Difference is subtle. In DOM-based XSS, the malicious JavaScript is executed after the page has loaded, as a result of the page's legitimate JavaScript treating user input in an unsafe way.

--

### Prevention

* Using frameworks that automatically escape XSS
* Escaping untrusted data based on the usage context (body, attribute, JavaScript, CSS, or URL)
* Use Content Security Policy (CSP) to mitigate impact of XSS
* Test - lots of automated tools exits

Note: Escape rules are hard, for further details see OWASP Cheat Sheets

---

## Insecure Deserialization

Note: This one is really technical one and deserves dedicated talks (both .NET & Java)

--

### Definition

Applications is vulnerable if deserialize hostile or tampered objects supplied by an attacker. This can result in two primary types of attacks:

* Remote Code Execution
* Data tampering attacks

--

### Typical targets

Serialization may be used in applications for:

* Remote- and inter-process communication (RPC/IPC)
* Wire protocols, web services, message brokers
* Caching/Persistence
* Databases, cache servers, file systems
* HTTP cookies, HTML form parameters, API authentication tokens

--

### Prevention

* Signature or encryption on serialized objects
* Monitor deserialization exceptions
* Use serialization medium that only allow primitive data types
* ... at the end it's easy. Don't trust anything comming from user

---

## Using Components with Known Vulnerabilities

Note: Well this one is self describing.

--

### Your application is vulnerable

Note: accept it

--

### Prevention

* Remove unnecessary dependencies
* Use tools to detect vulnerable components
* Have a plan for monitoring, triaging, and updating

---

## Insufficient Logging & Monitoring

Note: This is not the fist time I am mentioning logging today

--

### Why you should care

* Most successful attacks start with vulnerability probing.
* In 2016, identifying a breach took an average of 191 days

--

### What to monitor

* All login
* Access control failures
* Server-side input validation failures
* Cryptography and deserializatino exceptions

--

### Recommendation

* Use centralized log management solutions
* Consider append-only database tables for audit trails
* Establish monitoring and alerting for suspicious activities
* Test your response plan

---

## Where to go from here

* [OWASP Cheat Sheets Series](https://www.owasp.org/index.php/OWASP_Cheat_Sheet_Series)
* [OWASP Testing Guide](https://www.owasp.org/index.php/OWASP_Testing_Guide_v4_Table_of_Contents)
* [OWASP Proactive Controls ](https://www.owasp.org/index.php/OWASP_Proactive_Controls)

---

## References

* [OWASP Top 10 - 2017](https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf)
* [Excess XSS - A comprehensive tutorial on cross-site scripting](https://excess-xss.com/)