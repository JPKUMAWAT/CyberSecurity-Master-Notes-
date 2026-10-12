# Host Header Injection — Bypass and Manipulation Techniques

## 1. Overview

Host Header manipulation occurs when an application or server accepts an unexpected hostname from an HTTP request.

If the application does not properly validate or normalize the hostname, different components—such as a reverse proxy, load balancer, and backend server—may interpret the request differently.

**Important:** Changing the `Host` header does not automatically mean a vulnerability exists. A security issue requires a demonstrable impact.

## 2. Lack of Host Validation

**Concept:** The application accepts a client-provided hostname without restricting it to approved domains.

**Normal request:**

```http
GET / HTTP/1.1
Host: example.com
```

**Modified request:**

```http
GET / HTTP/1.1
Host: untrusted.example
```

**Potential weakness:** The application uses the supplied hostname to generate absolute URLs, redirects, or password-reset links.

**What to observe:**
- Does the response contain an unexpected hostname?
- Does a generated link use an untrusted domain?
- Is there any security impact beyond changing the response?

## 3. Special Header Manipulation

Applications behind proxies may use forwarding headers to determine the original hostname.

Common headers include:

- `X-Forwarded-Host`
- `X-HTTP-Host-Override`
- `Forwarded`
- `X-Host`
- `X-Forwarded-Server`

**Example:**

```http
GET / HTTP/1.1
Host: example.com
X-Forwarded-Host: untrusted.example
```

**Potential weakness:** The frontend routes the request using `Host`, but the backend trusts an unvalidated forwarding header when generating URLs.

**What to observe:**
- Whether the backend uses a different hostname.
- Whether redirects or generated links change.
- Whether the header is ignored or rejected.

**Key point:** These headers do not have the same meaning in every application. Their effect depends on the server and proxy configuration.

## 4. Duplicate Host Headers

**Concept:** A request contains more than one `Host` header. Different HTTP components may reject the request or interpret the duplicate fields differently.

**Example:**

```http
GET / HTTP/1.1
Host: example.com
Host: untrusted.example
```

**Potential weakness:** The frontend and backend disagree about which hostname should be used.

**What to observe:**
- Whether the server rejects the request.
- Whether a proxy and backend interpret it consistently.
- Whether the hostname affects security-sensitive output.

**Important:** Duplicate `Host` headers can be invalid or rejected. Do not assume that the first or last header will always be selected.

## 5. Leading Whitespace and Header Parsing

**Concept:** Malformed header formatting may be interpreted differently by different HTTP components.

Example of unusual formatting:

```http
GET / HTTP/1.1
   Host: untrusted.example
Host: example.com
```

**Potential weakness:** One component may normalize or reject the malformed line while another handles it differently.

**What to observe:**
- Whether the request is rejected.
- Whether the frontend and backend parse the request consistently.
- Whether any hostname-dependent behaviour changes.

**Note:** Modern servers may reject malformed headers. This example is for understanding parser differences, not a guaranteed bypass.

## 6. Absolute-Form Request Target

**Concept:** An HTTP request can use an absolute URL in the request line in certain proxy/request contexts.

**Example:**

```http
GET https://example.com/ HTTP/1.1
Host: untrusted.example
```

The absolute URL and the `Host` header specify different hostnames.

**Potential weakness:** Different components may rely on different hostname sources.

**What to observe:**
- Which destination the proxy selects.
- Which hostname the backend uses.
- Whether the application generates inconsistent URLs or redirects.

The outcome depends on the HTTP implementation and deployment configuration.

## 7. Weak Host Validation

### 7.1. Port and Hostname Parsing

**Concept:** An application incorrectly parses the hostname and port because it relies on custom string processing.

Illustrative input:

```http
GET / HTTP/1.1
Host: example.com:8080
```

**What to check:** Does the application parse the hostname and optional port correctly, according to the expected URL and HTTP syntax?

Do not assume that a colon-containing hostname is automatically a bypass.

### 7.2. Prefix and Suffix Validation Errors

**Concept:** The application checks whether a hostname contains or ends with an approved domain instead of comparing it against an explicit allowlist.

Suppose the intended domain is `example.com`. A naive suffix check may incorrectly accept a hostname such as:

```http
Host: vaadataexample.com
```

**Root cause:** The application uses a string suffix check without verifying the hostname boundary.

**Correct approach:** Parse the hostname and compare it with an explicit list of approved hostnames.

### 7.3. Unintended Subdomain Trust

**Concept:** The application trusts every subdomain even though only specific hosts should be authorized.

Illustrative hostname:

```http
Host: compromised-subdomain.example.com
```

**Potential weakness:** The application treats an unapproved subdomain as trusted.

**Important:** This scenario assumes the subdomain is under an attacker's control or is otherwise compromised. The hostname alone does not prove that condition.

**Correct approach:** Explicitly allow only the required hostnames and ensure subdomains are securely managed.

## 8. Additional Validation Checks

During authorized testing, consider these general questions:

- Is the hostname checked against an explicit allowlist?
- Are uppercase/lowercase and trailing-dot handling consistent?
- Is the port parsed and validated correctly?
- Are unexpected or malformed headers rejected?
- Do the reverse proxy and backend agree on the hostname?
- Are forwarding headers trusted only from configured proxies?
- Does the application use the hostname for redirects, generated URLs, or sensitive links?

These are validation questions, not proof that every variation is exploitable.

## 9. Safe Testing Workflow

1. Confirm that the target and testing methods are in scope.
2. Record the original request and response.
3. Change one header or request property at a time.
4. Compare status codes, redirects, response bodies, and generated URLs.
5. Determine whether the behaviour creates a genuine security impact.
6. Use accounts and resources you control.
7. Avoid testing that could affect other users, password-reset emails, or shared caches.
8. Save redacted evidence for the report.

## 10. Prevention

- Validate the hostname against an explicit allowlist.
- Use a standards-compliant HTTP and URL parser.
- Reject invalid or ambiguous requests consistently.
- Configure trusted proxy headers correctly.
- Do not trust client-supplied forwarding headers by default.
- Avoid simplistic prefix, suffix, or substring validation.
- Trust only explicitly approved subdomains.
- Use a fixed canonical domain for sensitive URLs.
- Keep reverse-proxy and backend behaviour consistent.

## 11. Quick Revision Table

| Technique | Main weakness to investigate |
|---|---|
| Lack of Host validation | Untrusted hostname accepted |
| Special headers | Untrusted forwarding-header trust |
| Duplicate Host | Inconsistent header parsing |
| Leading whitespace | Inconsistent normalization |
| Absolute-form request | Conflicting hostname sources |
| Port manipulation | Incorrect hostname/port parsing |
| Prefix/suffix bypass | Weak string-based validation |
| Subdomain trust | Overly broad hostname allowlist |

## Final Takeaway

**Root cause:** Untrusted hostname input, weak validation, or inconsistent parsing between HTTP components.

**Testing principle:** Identify the behaviour, verify the security impact, and document the evidence. A modified header alone is not a valid vulnerability finding.

**Recommended references:**
- PortSwigger Web Security Academy — HTTP Host Header Attacks
- CWE-20 — Improper Input Validation
- OWASP — Input Validation Cheat Sheet
