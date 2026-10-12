# Host Header Injection — Overview

## 1. What Is It?

**Simple definition:** Host Header Injection tab hota hai jab web application HTTP request ke `Host` header ya related headers par galat tarike se trust karti hai.

The `Host` header identifies the hostname requested by the client.

### Normal Request

```http
GET / HTTP/1.1
Host: example.com
```

Application ko sirf trusted hostnames accept karne chahiye.

### Potentially Unsafe Behavior

```http
GET / HTTP/1.1
Host: untrusted.example
```

Agar application is untrusted value ka use URLs, redirects ya cache-related responses banane mein karti hai, toh security issue ho sakta hai.

*Example domain illustrative hai; ye kisi real vulnerability ka proof nahi hai.*

## 2. Root Cause

- Hostname validation missing hona.
- Request headers par excessive trust.
- Password-reset URLs ko request hostname se generate karna.
- Reverse proxy aur backend ke beech inconsistent header handling.
- Cache configuration mein hostname ko properly account na karna.

## 3. Important Headers

| Header | Purpose |
|---|---|
| `Host` | Requested hostname |
| `X-Forwarded-Host` | Proxy ke through original host batane ke liye commonly used |
| `Forwarded` | Proxy-related request information |
| `X-Host` | Kuch applications mein custom header |

**Note:** Forwarding headers ko automatically trusted nahi maana chahiye.

## 4. How to Detect

Authorized target ya lab mein:

1. Normal request ko Burp Suite mein capture karo.
2. Original hostname aur response record karo.
3. Controlled test hostname ke saath behavior compare karo.
4. Check karo ki response, redirect ya generated URL mein hostname reflect hota hai ya nahi.
5. Confirm karo ki is behavior se actual security impact hota hai.

### Example

Normal response mein:

```text
https://example.com/account
```

Controlled testing ke baad response mein unexpected hostname appear hota hai.

Ye **potential indicator** hai, confirmed vulnerability nahi. Application ka intended behavior aur impact verify karna zaroori hai.

## 5. Possible Impact

- **Password-reset link manipulation:** Incorrect domain ke saath reset link generate ho sakta hai.
- **Open redirect:** Unsafe URL construction se redirect behavior affect ho sakta hai.
- **Web cache poisoning:** Incorrectly cached response doosre visitors ko serve ho sakta hai.
- **Security control weakness:** Hostname ko authentication ya access-control decision mein galat tarike se use kiya ja sakta hai.

Har finding ka impact alag hota hai. Sirf header modify hone se vulnerability ya high severity prove nahi hoti.

## 6. Prevention

- Allowed hostnames ki strict allowlist use karo.
- Sensitive URLs ko configured canonical domain se generate karo.
- Forwarding headers sirf trusted proxies se accept karo.
- Duplicate ya conflicting Host headers ko reject karo.
- Cache keys aur redirect logic review karo.
- Authentication ko Host header par depend mat karo.

## 7. Quick Checklist

- [ ] Authorized target ya lab.
- [ ] Baseline request and response saved.
- [ ] Host-related behavior compared.
- [ ] False positives checked.
- [ ] Security impact verified.
- [ ] Evidence documented.
- [ ] Remediation recommended.

## 8. References

- [PortSwigger — HTTP Host Header Attacks](https://portswigger.net/web-security/host-header)
- [OWASP](https://owasp.org/)

**Key takeaway:** Host Header Injection ka main issue sirf header change hona nahi, balki application ka untrusted hostname ko security-sensitive operations mein unsafe tarike se use karna hai.
