# Host Header Bypass and Manipulation Techniques

## 1. What Is Host Header Manipulation?

The `Host` header identifies the hostname requested by an HTTP client. If a server trusts an unvalidated value, it may use an attacker-controlled hostname to generate URLs, redirects, or other application responses.

**Important:** Changing the `Host` header alone does not prove a vulnerability. You must verify an unintended security impact on an authorized target.

## 2. Common Techniques

### A. Direct Host Manipulation

**Concept:** Replace the expected hostname with another hostname.

Normal request:

```http
GET / HTTP/1.1
Host: example.com
```

Modified request:

```http
GET / HTTP/1.1
Host: untrusted.example
```

**Potential weakness:** The application uses the supplied hostname without validating it.

**Check:** Does the response generate an unexpected absolute URL or redirect?

### B. Alternative Host Headers

Some applications or proxy configurations trust forwarding headers.

Common examples:

- `X-Forwarded-Host`
- `X-HTTP-Host-Override`
- `Forwarded`
- `X-Host`
- `X-Forwarded-Server`

Example:

```http
GET / HTTP/1.1
Host: example.com
X-Forwarded-Host: untrusted.example
```

**Potential weakness:** The frontend routes using `Host`, but the backend uses an untrusted forwarding header when constructing URLs.

**Check:** In an authorized test, compare the response with and without the additional header. A changed response is only a lead, not proof of impact.

### C. Duplicate Host Headers

**Concept:** Send repeated `Host` headers to investigate inconsistent parsing between proxies and application servers.

```http
GET / HTTP/1.1
Host: example.com
Host: untrusted.example
```

Different components may reject, select, or interpret duplicate headers differently.

**Potential impact:** Incorrect routing or hostname-dependent URL generation.

**Safe check:** Record whether the server rejects the request, returns an error, or behaves inconsistently. Do not assume that duplicate headers are automatically exploitable.

### D. Leading Whitespace and Header Parsing

Some malformed or ambiguous header formatting can be interpreted differently by different HTTP components.

**Security concern:** A proxy and backend may disagree about how a request should be parsed.

Modern HTTP implementations may reject malformed requests. Do not assume whitespace creates a bypass.

**Safe check:** Use a controlled lab to compare parser behaviour and document the exact response.

### E. Absolute-Form Request Target

An HTTP request can contain an absolute URL in its request line in certain request contexts.

Example:

```http
GET https://example.com/ HTTP/1.1
Host: untrusted.example
```

**Concept:** Investigate whether the proxy, request target, and backend disagree about the destination hostname.

**Potential weakness:** Different components use different hostname sources.

### F. Weak Host Validation

Incomplete validation may allow unexpected hostnames.

**1. Incorrect suffix matching**

A check that merely tests whether a string ends with an approved domain may accept a hostname that was never intended to be trusted.

Correct validation should compare parsed hostnames against an explicit allowlist, using proper hostname boundaries.

**2. Unintended subdomains**

An application may trust every subdomain when only a small set should be permitted.

A subdomain should be trusted only when explicitly authorized and appropriately controlled.

**3. Port and hostname parsing**

Hostname and port parsing should follow a proper URL/authority parser. Avoid ad hoc string splitting or substring checks.

## 3. Safe Testing Workflow

1. Confirm that the target is explicitly in scope.
2. Capture the original request and response.
3. Change one header or request property at a time.
4. Compare status codes, redirects, response bodies and generated URLs.
5. Check whether the difference creates a real security impact.
6. Stop if testing could affect other users, accounts, email delivery or shared caches.
7. Save redacted evidence for the report.

## 4. Common False Positives

- The server accepts a custom `Host` header but does not use it dangerously.
- A proxy returns an error for malformed or duplicate headers.
- A response changes only for the tester's request.
- A redirect points to a configured, trusted domain.
- A forwarding header is ignored by the backend.

## 5. Prevention

- Validate hostnames against an explicit allowlist.
- Configure trusted proxy headers at the infrastructure layer.
- Reject ambiguous or invalid requests consistently.
- Avoid hostname checks based on simple prefixes or suffixes.
- Use a fixed, configured canonical domain for sensitive absolute URLs.
- Keep proxy and backend parsing rules consistent.

## Quick Revision

**Root cause:** Untrusted hostname data or inconsistent parsing.

**Main areas to inspect:** Host validation, forwarding headers, duplicate headers, request-target parsing and subdomain trust.

**Remember:** A bypass is a weakness in a control; a reportable vulnerability requires a demonstrated security impact.
