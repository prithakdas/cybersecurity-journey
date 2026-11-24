# TryHackMe — Love at First Breach

**Room:** Love at First Breach  
**Source:** `Love-at-First-Breach_TryHackMe(2).pdf` (49 pages)

The supplied write-up documents ten web challenges. This summary follows its recorded methods and omits flags and sensitive payload values.

## 1. Valenfind
**Web · Medium · 200 points**
- Register and explore the application.
- Review JavaScript associated with `/prfile/cupid`; the write-up says it hinted at LFI.
- Test the exposed path and read `/opt/valentine/app.py`.
- The source disclosed a key and database path; the write-up then accessed `/api/admin/export_db` using an `X-Valentine-Token` header.
**Concepts:** JavaScript review, LFI, source disclosure, exposed secrets.

## 2. Hidden Deep Into My Heart
**Web · Easy · 100 points**
- Enumerate the application and inspect `/robots.txt`.
- Follow the path disclosed there.
- Use directory fuzzing to discover `/administrator`.
- The write-up uses a credential clue from robots.txt at the administrator login.
**Concepts:** directory enumeration, information disclosure, exposed credentials.

## 3. Signed Message
**Web · Medium · 200 points**
- Register, explore, and inspect the debug functionality.
- The debug output indicates RSA keys are generated deterministically from `{username}_lovenote_2026_valentine`.
- Reconstruct the key with Python and generate an RSA-PSS signature for the known message.
- Submit the signature on the verification page as the admin user.
**Concepts:** deterministic key generation, weak randomness, RSA reconstruction, signature trust.

## 4. Corp Website
**Web · Medium · 200 points**
- Identify the app as Next.js and perform initial enumeration.
- The PDF reports a Nuclei finding related to React2Shell and references CVE-2025-55184, followed by use of a public exploit reference.
- After command execution, inspect the user directory and retrieve the user flag.
- Run `sudo -l`; the notes say Daniel can run Python as root without a password, which was used for privilege escalation.
**Concepts:** vulnerability scanning, command execution, Linux enumeration, sudo misconfiguration.
**Caution:** independently verify the CVE identifier and affected-version details before using this as a real-world advisory.

## 5. CupidBot
**Web · Easy · 100 points**
- Test prompt-injection inputs against the chatbot to retrieve hidden flags.
- When initial attempts fail, ask it to explain further.
- The write-up discovers trigger conditions and uses the `administrator` condition for the final flag.
**Concepts:** prompt injection, instruction-boundary testing, trigger-condition discovery.

## 6. TryHeartMe
**Web · Easy · 100 points**
- Register and inspect browser storage for the JWT.
- Inspect the token and modify the `role` and `credits` claims as described.
- Replace the stored token, refresh, and purchase the newly revealed ValenFlag item.
**Concepts:** JWT trust failures, client-side authorization, business-logic manipulation.

## 7. Speed Chatting
**Web · Easy · 100 points**
- Test the messaging app and its input filtering.
- Upload a profile image, then test other extensions.
- The PDF reports that non-image extensions were accepted.
- It describes uploading a script and receiving a reverse-shell connection with root privileges in the challenge environment.
**Concepts:** unrestricted file upload, server-side execution, reverse shell.

## 8. Cupid’s Matchmaker
**Web · Easy · 100 points**
- Submit the survey and enumerate routes; the write-up finds `/admin`.
- Place a script in the survey Name field and wait for the administrative workflow to process it.
- The author's listener receives a callback.
**Concepts:** stored XSS, administrative-user interaction, client-side data exposure.

## 9. Love Letter Locker
**Web · Easy · 100 points**
- Register and inspect the dashboard; two letters already exist.
- Create another letter and observe the `/letter/3` URL pattern.
- Change the identifier to `/letter/1`; the write-up reports access to another user's letter.
**Concepts:** IDOR / broken object-level authorization, predictable identifiers.

## 10. When Hearts Collide
**Web · Medium · 200 points**
- Inspect the image-matching upload feature.
- Save the provided dog image and use the collision files referenced in the challenge.
- Upload the files one by one; the write-up reports that a match reveals the flag.
**Concepts:** file/hash collision behavior. The PDF does not contain enough textual detail to identify the exact collision construction or underlying implementation weakness.

## Tools & Techniques
Burp Suite · Gobuster · ffuf · Nuclei · Python · JWT inspection · browser developer tools · Linux enumeration.

## Takeaways
- Review JavaScript and debug routes for accidental disclosure.
- Keep cryptographic key generation unpredictable.
- Enforce authorization on the server, not in client-controlled JWT claims.
- Validate uploads by content and safe handling, not only file extension.
- Apply object-level authorization to every user-controlled object identifier.
- Test AI applications for prompt and instruction-boundary weaknesses.

*Summary of the supplied 49-page PDF. Use techniques only in authorized TryHackMe instances and lab environments.*
