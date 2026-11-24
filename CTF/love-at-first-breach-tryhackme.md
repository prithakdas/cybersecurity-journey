# TryHackMe: Love at First Breach — Web Challenge Write-ups

**Author:** Prithak Das  
**Platform:** TryHackMe  
**Room:** Love at First Breach 2026  
**Category covered:** Web  
**Challenges documented:** 10  
**Source:** `Love-at-First-Breach_TryHackMe(3).pdf` (49 pages)

This write-up expands the steps and screenshots in the supplied PDF into a readable walkthrough. The activities below refer to the intentionally vulnerable TryHackMe challenge machines. Replace `Target_MachineIP` and `Kali_IP` with the values assigned to your own lab instance. Do not apply these techniques to systems without explicit authorization.

---

## Contents

1. Valenfind — Local File Inclusion (LFI) and exposed application secrets
2. Hidden Deep Into my Heart — robots.txt discovery and exposed credentials
3. Signed Message — deterministic RSA key generation and signature forgery
4. Corp Website — Next.js exploitation and Linux privilege escalation
5. CupidBot — prompt injection
6. TryHeartMe — JWT trust and client-side authorization
7. Speed Chatting — unrestricted file upload and server-side code execution
8. Cupid's Matchmaker — stored XSS and session exposure
9. Love Letter Locker — Insecure Direct Object Reference (IDOR)
10. When Hearts Collide — MD5 hash collision

---

## 1. Valenfind

<text>**Difficulty:** Medium · **Points:** 200 · **Category:** Web</text>

### Objective

The challenge introduces Valenfind, a newly released dating application. The goal is to investigate the web application and uncover the hidden flag.

### Walkthrough

1. Start the target machine and browse to the application at the assigned target IP and port.
2. Register a new account through the sign-up page.
3. After registration, the application redirects to the dashboard.
4. Explore the application and inspect its client-side JavaScript. The PDF identifies a JavaScript resource under `/prfile/cupid` (as written in the notes) that reveals a file path and suggests a Local File Inclusion (LFI) issue.
5. Send a request to the discovered path and inspect the response.
6. Test whether path traversal can read a known system file. The notes use the common Linux file `/etc/passwd` with traversal sequences.
7. Use the disclosed application path to inspect `/opt/valentine/app.py`.
8. The Python source exposes a key and a database path.
9. The notes then identify an administrative database-export endpoint at `/api/admin/export_db`.
10. Send a request to that endpoint with the `X-Valentine-Token` header set to the key recovered from the application source.
11. The endpoint returns the challenge flag.

### Security observations

- Client-side JavaScript can disclose internal routes and implementation details.
- An LFI flaw can expose application source code and secrets stored on disk.
- An administrative export endpoint protected only by a static token is vulnerable if that token is exposed elsewhere in the application.

### Remediation

- Avoid building filesystem paths directly from untrusted input; canonicalize paths and enforce an allowlist.
- Keep secrets out of source files and client-accessible resources. Use managed secret storage or protected environment configuration.
- Require strong server-side authorization for administrative export functions.
- Rotate credentials and tokens if source disclosure is suspected.

**Evidence in source PDF:** pages 1–4.

---

## 2. Hidden Deep Into my Heart

**Difficulty:** Easy · **Points:** 100 · **Category:** Web

### Objective

Cupid's Vault is described as a system that should protect hidden information. The task is to find the information left exposed by the application.

### Walkthrough

1. Start the target machine and access the web application.
2. Run basic web enumeration against the target.
3. The notes show a Gobuster scan discovering `/console` and `/robots.txt`.
4. Check `/console`. In the recorded attempt, it does not provide useful access.
5. Read `/robots.txt`. It contains a disallowed path and a comment-like value that resembles a password.
6. Browse to the disclosed path, recorded in the PDF as `/cupids_secret_vault/`.
7. The page contains a short message but no immediately visible flag.
8. Use the discovered password value on the administrator login page at `/administrator`.
9. Use `admin` as the username, as shown in the notes.
10. Successful authentication reveals the flag.

### Security observations

- `robots.txt` is public and must never be used to store passwords, API keys, or other secrets.
- Disallowed paths are not access controls. They merely provide crawler guidance.
- Exposed credentials can turn a seemingly harmless information disclosure into account compromise.

### Remediation

- Remove secrets and sensitive path hints from public files.
- Protect administrative routes with authentication and authorization controls.
- Rotate any credential disclosed through a public resource.
- Monitor access to administrative endpoints.

**Evidence in source PDF:** pages 5–10.

---

## 3. Signed Message

**Difficulty:** Medium · **Points:** 200 · **Category:** Web / Cryptography

### Objective

LoveNote claims that messages are protected by RSA-2048 digital signatures. The challenge is to investigate whether the signing system can be trusted and obtain a valid administrator signature.

### Walkthrough

1. Start the machine and open the LoveNote application.
2. Create an account and explore the available pages.
3. The registration flow generates a public/private key pair. The PDF shows the public key and private key sections on the account-created screen.
4. Enumerate application paths. The notes use directory discovery with `ffuf`.
5. Inspect the discovered `/debug` page.
6. The debug information reveals that RSA key material is generated deterministically rather than randomly.
7. The seed format recorded in the notes is:

   ```text
   {username}_lovenote_2026_valentine
   ```

8. The same username therefore produces the same derived RSA key material in this challenge implementation.
9. The notes infer that identifying the administrator username makes it possible to regenerate the corresponding key.
10. A Python script is then used to reconstruct the RSA key for the username `admin` and generate an RSA-PSS signature for the application's known welcome message.
11. Submit the generated signature on the Verify page using the `admin` username.
12. The application reports that the signature is valid and displays the flag.

### Key-generation logic documented in the PDF

The supplied script uses:
- SHA-256 to derive a digest from the seed.
- SymPy's `nextprime` to derive prime values.
- A second seed formed by appending `pki` to the original seed.
- RSA public exponent `65537`.
- PyCryptodome's RSA and PSS modules.
- SHA-256 as the message digest for the signature.

The key construction shown in the source follows this pattern:

```python
seed = f"{username}_lovenote_2026_valentine".encode()
p = derive_prime(seed)
q = derive_prime(seed + b"pki")

n = p * q
phi = (p - 1) * (q - 1)
d = pow(65537, -1, phi)
```

The original PDF contains the full script across pages 15–17. The essential flaw is deterministic, predictable key generation based on a guessable username-derived seed.

### Security observations

- Cryptographic keys must be generated from a cryptographically secure random source.
- Debug endpoints must not reveal key-generation internals or sensitive implementation details.
- A strong algorithm such as RSA-PSS cannot compensate for predictable private-key generation.

### Remediation

- Generate private keys with a cryptographically secure random number generator.
- Never derive private-key material from usernames or other predictable identifiers.
- Remove debug endpoints from production and protect diagnostic information.
- Protect private keys and restrict signing operations to authorized server-side code.

**Evidence in source PDF:** pages 11–18.

---

## 4. Corp Website

**Difficulty:** Medium · **Points:** 200 · **Category:** Web / Linux

### Objective

The challenge asks the participant to retrace an attack against the Romance & Co. website and identify how the compromise occurred.

### Walkthrough

1. Start the target machine and access the web application.
2. Inspect the application and identify that the site appears to use Next.js.
3. Run directory enumeration. The notes report that Gobuster did not reveal much useful information.
4. Run Nuclei to check for known issues.
5. The scan output is described in the PDF as indicating a React2Shell-related vulnerability, identified in the notes as **CVE-2025-55184**.
6. The write-up says a public GitHub exploit repository named `Chocapikk/CVE-2025-55182` was downloaded and used. The CVE identifier in the scan notes and the identifier in the repository name differ; preserve this distinction when reproducing the lab rather than assuming they are interchangeable.
7. In the authorized challenge environment, the exploit was used to obtain command execution.
8. The notes then inspect the `daniel` user's home directory and read `/home/daniel/user.txt`, obtaining the first flag.
9. Continue local enumeration with `sudo -l`.
10. The output shows that user `daniel` can run Python as root without a password.
11. The notes use this sudo permission to escalate privileges and access the root account.
12. Inspect the root directory and retrieve the final flag.

### Security observations

- Vulnerable server-side rendering or server-action components can expose application execution paths.
- A web compromise can become a full host compromise when local privilege boundaries are weak.
- Passwordless sudo access to an interpreter such as Python is especially dangerous because it can provide arbitrary code execution with elevated privileges.

### Remediation

- Upgrade affected Next.js/React components to vendor-fixed versions and verify the exact affected package and CVE against the official advisory.
- Restrict server-side functionality and validate untrusted inputs.
- Apply least privilege to service and application accounts.
- Avoid granting unrestricted sudo access to general-purpose interpreters.
- Review sudoers entries and monitor privileged command execution.

**Evidence in source PDF:** pages 19–21.

---

## 5. CupidBot

**Difficulty:** Easy · **Points:** 100 · **Category:** Web / LLM Security

### Objective

CupidBot is an AI chatbot that writes Valentine's messages. The challenge states that three flags are hidden in its system and asks the participant to retrieve them using prompt injection.

### Walkthrough

1. Open the CupidBot challenge and interact with the chatbot.
2. Ask directly for the prompt-injection flag, using the question shown in the PDF.
3. The chatbot returns a flag.
4. Ask for the system flag using a similar direct question.
5. The chatbot returns the second flag.
6. For the final flag, the initial direct prompts do not work consistently.
7. Ask the chatbot to explain more about the final flag and its conditions.
8. The response reveals trigger conditions associated with the different flags.
9. The notes identify an administrator-related trigger. Use the term `administrator` in the relevant prompt.
10. The chatbot returns the third flag.

### Security observations

- System prompts are not a security boundary.
- Hidden instructions and secrets placed in model context may be disclosed if the application trusts user-controlled conversational input.
- Trigger-based flag logic can be exposed by asking the model to explain its own conditions.

### Remediation

- Do not place secrets or authorization tokens in prompts or model context.
- Enforce access control in application code, not through natural-language instructions.
- Treat model output as untrusted and separate user-facing responses from privileged data.
- Test prompt injection, indirect prompt injection, and data-exfiltration paths.

**Evidence in source PDF:** pages 22–24.

---

## 6. TryHeartMe

**Difficulty:** Easy · **Points:** 100 · **Category:** Web / JWT

### Objective

The TryHeartMe shop contains a hidden item called `ValenFlag`. The challenge is to access and purchase it.

### Walkthrough

1. Start the target machine and open the shop.
2. Register a user account.
3. Open the product page for **Rose Bouquet**.
4. Open browser developer tools and navigate to the Storage area.
5. Locate the token stored by the application and copy it for inspection.
6. Decode the JWT payload using a JWT inspection tool.
7. The PDF shows payload fields including `email`, `role`, `credits`, and `theme`.
8. Modify the `role` and `credits` values as shown in the challenge notes, then use the resulting token in the application in place of the original token.
9. Refresh the product page and observe the changed application state.
10. Purchase the Rose Bouquet and return to the shop.
11. The hidden `ValenFlag` item becomes visible.
12. Purchase the hidden item to obtain the flag.

### Security observations

The challenge demonstrates the risk of trusting client-controlled token claims for authorization or account balance. Merely encoding a JWT payload does not make it trustworthy. The server must validate the token's signature and enforce authorization and balance checks independently.

### Remediation

- Verify JWT signatures using a secure, explicit algorithm allowlist.
- Reject unsigned or incorrectly signed tokens.
- Never trust client-supplied role or balance claims without server-side validation.
- Keep purchase eligibility and account balances in authoritative server-side storage.
- Rotate signing keys if compromise is suspected.

**Evidence in source PDF:** pages 25–30.

---

## 7. Speed Chatting

**Difficulty:** Easy · **Points:** 100 · **Category:** Web / File Upload

### Objective

Speed Chatting is a messaging application released without sufficient security testing. The challenge investigates weaknesses in its chat and profile-photo functionality.

### Walkthrough

1. Start the target machine and open the Speed Chatting application.
2. Test the chat input with a basic XSS probe. The PDF shows `<script>alert(1);</script>`; the application appears to apply a filter to the message.
3. Explore the profile-photo upload feature.
4. Upload a normal PNG image and observe the success message.
5. Test a file with a different extension. The notes report that the application accepts extensions such as `.py` and `.txt`, indicating inadequate file-type validation.
6. The PDF then demonstrates creating a Python file that opens a connection back to the participant's listener.
7. Start a listener on the configured port in the controlled TryHackMe environment.
8. Upload the Python file through the profile-photo upload function.
9. The screenshot sequence shows a reverse-shell connection and a shell running with root privileges.
10. Enumerate the accessible files and retrieve the flag.

### Security observations

- File upload validation based only on the filename extension is insufficient.
- Uploaded files should never be executed by the web server.
- Upload directories should be isolated from application code and served as static content.
- A file-upload flaw can have severe impact when the application executes uploaded content with excessive privileges.

### Remediation

- Validate file content using safe parsing and allowlisted formats; do not rely only on extensions or client-supplied MIME types.
- Store uploads outside executable web roots or on isolated object storage.
- Generate server-side filenames and enforce size limits.
- Run the application with a dedicated low-privilege account.
- Apply strict filesystem permissions and prevent execution in upload directories.

**Evidence in source PDF:** pages 31–37.

---

## 8. Cupid's Matchmaker

**Difficulty:** Easy · **Points:** 100 · **Category:** Web / Stored XSS

### Objective

Cupid's Matchmaker collects personality survey submissions for manual review. The challenge investigates how submitted survey data is handled by the application.

### Walkthrough

1. Start the target machine and open the matchmaking application.
2. Submit the survey and observe the confirmation message.
3. Enumerate application paths. The notes report discovering an `/admin` route.
4. Inspect how survey fields are rendered and identify the Name field as the injection point used in the challenge.
5. The PDF shows a JavaScript payload designed to send the browser's cookie to a listener controlled by the participant.
6. Start a local HTTP listener before submitting the survey. The notes use Python's built-in HTTP server on port `8002`.
7. Submit the crafted survey entry in the lab.
8. Wait for the application workflow to process the submission.
9. The listener receives a request containing the encoded cookie data.
10. Decode the returned URL-encoded value as shown in the notes to recover the flag.

### Security observations

- Untrusted survey fields must be safely encoded before being rendered in an administrative interface.
- Stored XSS can affect users who view submitted content later, including administrators.
- Session cookies exposed to JavaScript can be stolen if they are not protected with appropriate cookie attributes.

### Remediation

- Apply context-aware output encoding to all user-controlled content.
- Use a restrictive Content Security Policy as defense in depth.
- Set session cookies with `HttpOnly`, `Secure`, and an appropriate `SameSite` policy.
- Avoid rendering submitted HTML or JavaScript in administrative dashboards.
- Review and sanitize stored content before displaying it.

**Evidence in source PDF:** pages 38–41.

---

## 9. Love Letter Locker

**Difficulty:** Easy · **Points:** 100 · **Category:** Web / IDOR

### Objective

Love Letter Locker presents itself as a private letter-sharing application. The challenge is to access another user's letter.

### Walkthrough

1. Start the target machine and open the website.
2. Register a new account using a username and password.
3. Open the account dashboard.
4. Notice that the dashboard already displays two letters, even though the newly registered user has not created any.
5. Create a new letter. The notes show the archive count increasing to three.
6. Open the newly created letter and inspect its URL. The example shown is `/letter/3`.
7. Change the identifier in the URL to `/letter/1`.
8. The application displays another user's letter, identified in the notes as Gonz0's letter, and the flag is present in its contents.

### Security observations

The application appears to rely on an object identifier in the URL without properly checking whether the current user owns or is authorized to view the requested letter. This is an Insecure Direct Object Reference (IDOR), also described as broken object-level authorization.

### Remediation

- Perform an authorization check on every request for a letter.
- Scope database queries to the authenticated user's permitted objects.
- Do not rely on unpredictable IDs as a replacement for access control.
- Add tests that attempt to access another user's objects by changing identifiers.

**Evidence in source PDF:** pages 42–44.

---

## 10. When Hearts Collide

**Difficulty:** Medium · **Points:** 200 · **Category:** Web / Cryptography

### Objective

The challenge is titled **When Hearts Collide** and asks whether the participant can find an MD5 match. The application is a matchmaking page that accepts an uploaded image and compares it with a randomly selected dog image.

### Walkthrough

1. Start the target machine and browse to the matchmaking application.
2. The page provides an image-upload area and a random-match feature.
3. Use the random-match feature to display a dog image.
4. Right-click the displayed image and save it locally as `dog.png`.
5. Open a terminal in the directory where the image was saved.
6. The PDF shows using a collision-generation utility to create two different files with the same MD5 digest, based on the downloaded image as the starting point.
7. The output consists of two files, shown in the screenshots as `collision1.jpg` and `collision2.jpg`.
8. Upload the two generated files to the website one at a time.
9. The application processes the uploaded image and the dog image through its checksum-based matching flow.
10. When the MD5 collision condition is satisfied, the page displays the successful match and the flag.

### Security observations

- MD5 is cryptographically broken and should not be used where collision resistance is required.
- Two different files can be constructed with the same MD5 digest.
- A system that treats an MD5 match as proof that two files are identical can be deceived.

### Remediation

- Do not use MD5 for security-sensitive integrity or identity decisions.
- Use a modern collision-resistant hash such as SHA-256 when a cryptographic digest is required.
- If the application needs to determine whether two images are visually or semantically similar, use a suitable image-comparison method rather than treating a cryptographic hash as a similarity score.
- Validate uploaded files and keep the comparison logic on the server.

**Evidence in source PDF:** pages 45–49. The final screenshot on page 49 shows the successful match and the challenge flag.

---

## Challenge Summary

| # | Challenge | Difficulty | Points | Main concept |
|---:|---|---|---:|---|
| 1 | Valenfind | Medium | 200 | LFI, source disclosure, exposed token |
| 2 | Hidden Deep Into my Heart | Easy | 100 | robots.txt, credential exposure |
| 3 | Signed Message | Medium | 200 | Predictable RSA key generation, signature forgery |
| 4 | Corp Website | Medium | 200 | Next.js exploitation, sudo misconfiguration |
| 5 | CupidBot | Easy | 100 | Prompt injection |
| 6 | TryHeartMe | Easy | 100 | JWT trust, authorization logic |
| 7 | Speed Chatting | Easy | 100 | Unrestricted file upload, code execution |
| 8 | Cupid's Matchmaker | Easy | 100 | Stored XSS, session exposure |
| 9 | Love Letter Locker | Easy | 100 | IDOR / broken object-level authorization |
| 10 | When Hearts Collide | Medium | 200 | MD5 collision |

## Skills and Tools Practiced

- Web application exploration and directory enumeration
- Gobuster, ffuf, and Nuclei
- Browser developer tools and JavaScript inspection
- Local File Inclusion and path traversal analysis
- Linux file and privilege enumeration
- Python scripting and RSA-PSS
- JWT inspection and server-side authorization analysis
- Prompt injection testing
- File-upload security testing
- Stored XSS and cookie security
- IDOR testing
- MD5 collision analysis

## Overall Lessons

1. Treat client-side code, public metadata files, and debug endpoints as potential information-disclosure sources.
2. Authentication is not authorization: every sensitive operation must verify the current user's permissions.
3. Cryptographic security depends on correct key generation and safe algorithms, not merely on using RSA or a digital signature library.
4. Never trust client-controlled JWT claims or balances without server-side validation.
5. File uploads must be validated, isolated, and prevented from executing.
6. Stored user content must be safely encoded before it is displayed to other users.
7. AI systems must not use hidden prompts as their only security control.
8. MD5 collisions demonstrate why legacy hashes are unsuitable for security-sensitive identity checks.

## Source and Reproducibility Notes

This document is based on the supplied 49-page PDF, including screenshots. Some details in the original notes are abbreviated or visible only in screenshots:
- **Valenfind:** the source path is transcribed as `/prfile/cupid` in the PDF; confirm the exact spelling from the live challenge if reproducing.
- **Corp Website:** the notes mention CVE-2025-55184 while linking an exploit repository whose name references CVE-2025-55182. These identifiers are intentionally recorded as shown and should be checked against the room's current instructions and vendor advisories.
- **Speed Chatting:** the PDF shows the file-upload-to-shell workflow but does not include a complete textual transcript of every command.
- **When Hearts Collide:** the screenshots show collision files being generated and uploaded, but do not preserve a complete command-line transcript or all utility parameters.

The exact challenge flags are not reproduced in this public-facing walkthrough. All techniques described are for the intentionally vulnerable TryHackMe room and should only be used in systems where testing is authorized.
