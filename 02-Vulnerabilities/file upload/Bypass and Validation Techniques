# File Upload — Bypass and Validation Techniques

## 1. Introduction

A File Upload Vulnerability occurs when an application fails to validate uploaded files securely.

**Example:** A profile-photo feature is intended to accept JPEG and PNG images, but the server does not properly validate the uploaded content.

**Potential impact:** Stored XSS, unauthorized file access, unsafe file processing, or server-side code execution in a vulnerable configuration.

## 2. Extension Validation

### 2.1 Weak Extension Checks

Some applications check whether a filename contains an allowed extension instead of validating the complete filename.

**Example filename:**

```text
profile.jpg.php
```

**What to check:** Does the application validate the complete extension and ensure the server cannot execute uploaded content?

**Potential impact:** If validation and server handling disagree, an unintended file type might be accepted.

**Fix:** Use an allowlist and store uploads in a non-executable location.

### 2.2 Multiple Extensions

A filename can contain more than one extension.

**Example:**

```text
photo.jpg.php
```

Compare this with:

```text
photo.jpg
```

**What to check:** Whether the application validates the final extension and whether the server uses a consistent file-handling rule.

**Important:** A double extension does not automatically bypass validation.

### 2.3 Blacklist Limitations

A blacklist blocks a list of known extensions but may omit other formats.

**Example:** An application rejects `.php` but has not accounted for other server-interpreted formats supported by its configuration.

**What to check:** Whether the accepted extensions are restricted to the formats the feature actually needs.

**Fix:** Prefer a strict allowlist rather than trying to enumerate every dangerous extension.

### 2.4 Case Sensitivity

An application may normalize extensions incorrectly.

**Example filenames:**

```text
photo.jpg
photo.JPG
photo.JpG
```

**What to check:** Whether all filename variants receive consistent validation.

**Potential impact:** Different components may interpret the same filename differently.

**Fix:** Normalize the extension and validate it consistently on the server.

## 3. MIME-Type Validation

The multipart request includes a `Content-Type` value that describes the claimed file format.

**Example of a normal JPEG upload:**

```http
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg
```

**What to check:** Does the server trust this header alone, or does it independently inspect the uploaded file?

**Potential issue:** A client can submit a claimed media type that does not match the actual content.

**Fix:** Treat client-provided MIME types as untrusted input. Validate the file independently.

## 4. Magic Bytes and File Signatures

Magic bytes are characteristic byte sequences that can help identify a file format.

**Example:** A JPEG file commonly starts with the hexadecimal bytes:

```text
FF D8 FF
```

A PNG file starts with:

```text
89 50 4E 47 0D 0A 1A 0A
```

**What to check:** Does the server inspect the actual content and validate the file structure, rather than relying only on its filename?

**Potential issue:** A matching signature alone does not prove that the entire file is safe.

**Fix:** Combine signature validation with a trusted file parser, format allowlisting, and safe storage.

## 5. Content-Length and File Size

Applications need limits for uploaded files and their processing costs.

**Example:** A profile-photo feature allows images up to 5 MB. Test that an upload exceeding the documented limit is rejected by the server.

**What to check:**
- Maximum file size
- Request-size limits
- Multipart parsing limits
- Image dimensions and processing resources
- Compressed and decompressed file sizes

**Potential impact:** Missing or inconsistent limits can cause excessive resource consumption.

**Fix:** Enforce limits at the application and infrastructure layers.

## 6. Special Characters in Filenames

Different components may handle encoded characters, whitespace, dots, or separators differently.

**Illustrative filenames:**

```text
profile photo.jpg
profile..jpg
profile.jpg.
```

**What to check:** Whether filenames are normalized consistently and stored safely.

**Potential issue:** Inconsistent parsing can result in unexpected names or paths.

**Fix:** Generate server-side filenames instead of trusting user-supplied names.

## 7. Null-Byte and Encoding Issues

Legacy systems have sometimes interpreted null bytes differently across components.

**Illustrative filename representation:**

```text
photo.jpg%00
```

This string represents an encoded null-byte sequence in some contexts; whether it is decoded and how it is handled depends on the application.

**What to check:** Whether decoding and validation occur consistently.

**Important:** Null-byte tricks are not universal and generally do not work against modern, correctly configured stacks.

**Fix:** Use modern libraries, reject invalid filename characters, and avoid unsafe manual decoding.

## 8. Path Traversal

Path traversal occurs when user-controlled filename data influences a filesystem path outside the intended directory.

**Illustrative path:**

```text
../uploads/profile.jpg
```

**What to check:** In an authorized lab, confirm that the application stores files only within its designated upload directory.

**Potential impact:** Depending on filesystem permissions, unsafe path handling may permit unintended file writes or overwrites.

**Fix:** Generate a random server-side filename and use a fixed storage directory. Do not concatenate untrusted paths directly.

## 9. HTTP PUT Method

HTTP PUT can create or replace a resource if the server permits it.

**Illustrative request:**

```http
PUT /uploads/test.txt HTTP/1.1
Host: lab.example
Content-Type: text/plain

Authorized lab test
```

**What to check:** Does the lab server allow this operation without the required authorization?

**Potential impact:** Unauthorized writes or replacement of files, depending on server configuration and permissions.

**Fix:** Disable unnecessary write methods and enforce authentication, authorization, and path restrictions.

## 10. Upload From URL — SSRF Risk

Some applications download a file from a user-supplied URL.

**Illustrative input:**

```text
https://files.example/test-image.png
```

**What to check:** Whether the application fetches only approved external resources.

**Potential issue:** Without destination restrictions, the server might be induced to request resources it should not access.

**Fix:** Restrict destinations, block private and reserved network ranges, validate redirects, and apply outbound network controls.

## 11. Black-Box Testing and Fuzzing

Black-box testing means examining an application without relying on its source code.

**Example:** An authorized lab accepts PNG files but rejects PDF files.

Record the baseline result, then test one property at a time:
- Filename extension
- Declared MIME type
- Actual file format
- File size
- Authentication requirements
- Access to the uploaded file

**What to record:** Request, response, acceptance or rejection, resulting file behavior, and verified security impact.

**Fix:** Ensure validation is consistent and performed server-side.

## 12. Quick Revision

| Technique | Example | Main concern |
|---|---|---|
| Extension validation | `photo.jpg.php` | Inconsistent filename validation |
| Case variation | `photo.JPG` | Inconsistent normalization |
| MIME validation | `Content-Type: image/jpeg` | Trusting client input |
| Magic bytes | JPEG signature | Signature alone is insufficient |
| File size | Upload over the documented limit | Resource exhaustion |
| Special characters | `profile..jpg` | Parsing inconsistencies |
| Null-byte handling | Encoded null-byte sequence | Legacy decoding behavior |
| Path traversal | `../uploads/profile.jpg` | Unsafe filesystem paths |
| HTTP PUT | Create a lab test file | Unauthorized writes |
| URL-based upload | Approved external image URL | SSRF risk |

**Key takeaway:** A bypass is not automatically a vulnerability. Validate the behavior, demonstrate the security impact safely, and recommend a concrete fix.
