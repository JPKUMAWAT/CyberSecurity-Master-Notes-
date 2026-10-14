# File Upload Vulnerability — Summary

A **File Upload Vulnerability** occurs when a web application fails to properly validate or restrict files uploaded by users. Weak validation, unsafe file handling, or incorrect server configuration may allow unauthorized or potentially dangerous files to be uploaded.

## Potential Impact

- **Remote Code Execution (RCE):** In vulnerable configurations, an uploaded file may execute on the server.
- **Stored Cross-Site Scripting (XSS):** Uploaded HTML or SVG content may execute scripts when handled unsafely.
- **Path Traversal:** Improper file path handling may allow files to be written outside the intended directory.
- **Sensitive Information Disclosure:** Improperly protected uploaded files may expose private data.
- **Denial of Service (DoS):** Oversized or specially crafted uploads may consume excessive resources.

## Common Root Causes

- Insufficient file type and content validation.
- Trusting the filename or client-supplied `Content-Type`.
- Unsafe storage locations or predictable file URLs.
- Missing file size and upload limits.
- Server misconfiguration that permits uploaded files to execute.

## What to Check During Authorized Testing

- Whether only permitted file types are accepted.
- Whether file content is validated independently of its extension.
- Whether uploaded files are stored safely and access-controlled.
- Whether uploaded content is served with appropriate headers.
- Whether size limits and server-side processing controls are enforced.

## Prevention

- Use server-side allowlists for permitted file types.
- Validate file signatures and content where appropriate.
- Generate safe filenames instead of trusting user input.
- Store uploads outside executable web directories when possible.
- Apply file size limits and access controls.
- Disable execution in upload directories.
- Scan files when appropriate and serve untrusted content safely.

**Key Takeaway:** File upload security depends on validating file content, storing files safely, and preventing uploaded content from executing or exposing sensitive information.
