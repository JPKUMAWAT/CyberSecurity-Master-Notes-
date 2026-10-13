# Exposed Source Code — Overview and Detection

## 1. Introduction

Exposed Source Code is a security issue in which application source code or version-control data intended to remain private becomes accessible to unauthorized users.

Exposed files may reveal application logic, configuration details, internal paths, and potentially sensitive information such as database credentials or secret keys.

**Root Cause:** Misconfigured web servers, exposed version-control directories, or development files accidentally deployed to production.

## 2. Security Impact

- **Source Code Disclosure:** An attacker may learn how the application works.
- **Credential Exposure:** Source files may accidentally contain passwords, API keys, or tokens.
- **Configuration Disclosure:** Internal settings and service details may become visible.
- **Attack Surface Discovery:** Repository contents may reveal endpoints and application components.
- **Historical Data Exposure:** Repository history may contain information removed from the latest version.

The actual impact depends on the files exposed and whether sensitive information is present.

## 3. Where to Find It

Common version-control paths include:

| Version Control System | Common Path |
|---|---|
| Git | `/.git/` |
| Subversion (SVN) | `/.svn/` |
| Mercurial (Hg) | `/.hg/` |
| Bazaar (Bzr) | `/.bzr/` |
| Darcs | `/_darcs/` |
| BitKeeper | `/Bitkeeper/` |

These are potential paths to check during authorized testing, not proof that a vulnerability exists.

## 4. Detection Methodology

### Step 1 — Confirm Scope

Verify that the website and the proposed testing methods are permitted by the owner or bug bounty program.

### Step 2 — Check the Suspected Path

Request the relevant path on the authorized target.

Example:
```http
GET /.git/ HTTP/1.1
Host: target.example
```

Use a reserved example hostname in documentation; replace it only when testing an explicitly authorized asset.

### Step 3 — Inspect the Response

Look for:
- Directory listings
- Version-control metadata
- Accessible repository files
- Responses revealing source code or configuration

### Step 4 — Validate the Finding

A response status such as `200 OK` is not sufficient by itself. Confirm that meaningful repository information or source code is actually exposed.

### Step 5 — Document Evidence

Record the affected path, relevant response, observed disclosure, and verified impact. Collect only the minimum evidence necessary.

## 5. False Positives

- The server returns `404 Not Found`.
- Access is blocked by a `403 Forbidden` response.
- A custom error page returns `200 OK`.
- The path exists but contains no accessible repository information.
- A directory name is visible without meaningful source-code disclosure.

A blocked path alone does not prove that all related files are protected, but further testing must remain within the approved scope.

## 6. Prevention

- Block public access to version-control metadata directories.
- Exclude development and repository files from production deployments.
- Keep secrets out of source code.
- Use a secret manager for sensitive credentials.
- Rotate any credentials confirmed to have been exposed.
- Add deployment checks to detect accidentally included repository files.

## 7. References

- Git Dumper: https://github.com/arthaud/git-dumper
- SVN Extractor: https://github.com/anantshri/svn-extractor
- Hg Dumper: https://github.com/arthaud/hg-dumper
- Source reference supplied in the study material: https://www.nakanosec.com/2020/02/exposed-source-code-pada-website.html

## Quick Revision

**Exposed Source Code** means private source code or version-control information is accessible when it should not be.

**Remember:** Identify the exposed resource, validate meaningful disclosure, assess the real impact, and recommend a fix.
