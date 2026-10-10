# SSTI — Server-Side Template Injection

## 1. What Is SSTI?

**Simple Definition:** SSTI tab hota hai jab website user ke input ko plain text ke bajaye template syntax ki tarah process karti hai.

**Root Cause:** User input ko unsafely template source mein add karna.

**Flow:**
```text
User Input → Web Application → Template Engine → Output
```

## 2. Common Template Engines

| Engine | Language / Framework |
|---|---|
| Jinja2 | Python |
| Django Templates | Python / Django |
| Twig | PHP |
| FreeMarker | Java |
| Thymeleaf | Java / Spring |
| Handlebars | JavaScript |
| Apache Velocity | Java |

## 3. SSTI Detection Examples

Sirf authorized targets ya practice labs mein harmless probes use karo.

### A. Arithmetic Evaluation

Kuch template engines arithmetic expressions evaluate karte hain.

```text
{{7*7}}
```

**Possible result:** `49`

Alternative syntax examples:

```text
${7*7}
#{7*7}
<%= 7*7 %>
```

**Note:** Ye syntax engine-dependent hai. In sabka evaluate hona expected nahi hai.

### B. Variable / Expression Check

```text
{{ name }}
${name}
```

**Observe:** Kya output change hota hai, value resolve hoti hai, ya expression literal text ke roop mein dikhta hai?

### C. Plain-Text Control Test

```text
ssti-test-123
```

Pehle ordinary text bhejo aur response note karo. Phir harmless expression bhejkar compare karo.

### D. Error Behavior

Kisi harmless, incomplete expression ko sirf controlled lab mein test karke dekho ki response mein template-related error aata hai ya nahi.

**Important:** Error message, `49`, ya changed response alone SSTI confirm nahi karta. Multiple observations aur application behavior verify karo.

## 4. Engine Identification Guide

| Observation | Possible Direction |
|---|---|
| `{{7*7}}` evaluates | Curly-brace expression syntax support karne wala engine investigate karo |
| `${7*7}` evaluates | Dollar-brace syntax support karne wala engine investigate karo |
| Expression literal show hota hai | Engine unsupported syntax, escaping, ya plain-text handling possible hai |
| Template-related error | Engine ya parsing behavior ka clue mil sakta hai |

**Remember:** Ek syntax kai engines mein similar ho sakta hai. Output se engine ka naam directly confirm mat karo.

## 5. Possible Impact

- Information disclosure
- Sensitive data exposure
- Some configurations mein server-side code execution (RCE)

Har SSTI se RCE nahi hota. Severity verified impact par depend karti hai.

## 6. Prevention

- User input ko template source mein concatenate mat karo.
- Predefined templates use karo.
- User input ko data ke roop mein pass karo.
- Secure configuration aur updated dependencies maintain karo.

## 7. Quick Testing Checklist

- [ ] Authorized target ya lab.
- [ ] Baseline response recorded.
- [ ] Harmless probes tested.
- [ ] Response differences compared.
- [ ] False positives checked.
- [ ] Actual impact verified.
- [ ] Evidence documented.

## 8. References

- [PortSwigger SSTI Guide](https://portswigger.net/web-security/server-side-template-injection)
- [CWE-1336](https://cwe.mitre.org/data/definitions/1336.html)
- [Jinja Documentation](https://jinja.palletsprojects.com/)

**Key Takeaway:** SSTI tab confirm hota hai jab evidence dikhaye ki untrusted input server-side template syntax ke roop mein evaluate ho raha hai — sirf unusual response se nahi.
