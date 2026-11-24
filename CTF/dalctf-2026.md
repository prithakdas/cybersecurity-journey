# DALCTF 2026 — Detailed Challenge Write-ups

**Author:** Prithak Das  
**Source:** Personal notes, `DalCTF2026_writeup(2).pdf` (17 pages)  
**Challenges documented:** 8

This walkthrough follows the supplied notes and screenshots. Where the PDF does not contain the exact code, cryptographic parameters, or derivation, that limitation is stated rather than guessed.

---

## 1. Baby Web

**Points shown:** 50 · **Area:** Web / source inspection

### Walkthrough
1. Open the challenge instance.
2. Right-click the page and choose **View page source** (PDF page 2).
3. In the source view, press **Ctrl + F** and search for `{`.
4. Inspect the matching source text. The flag is embedded in the page source even though it is not displayed in the normal page view.
5. Submit the discovered value.

### Takeaway
Anything delivered to the browser—HTML, JavaScript, comments, or source-map data—can be inspected by the visitor. Client-side hiding is not access control; sensitive values must remain server-side.

**Evidence:** PDF pages 1–3.

---

## 2. Warmer Up

**Points shown:** 50 · **Area:** PDF / document inspection

### Walkthrough
1. Download the supplied `rules1.pdf` file (page 3).
2. Open the document and inspect its visible contents.
3. Press **Ctrl + A** to select the document text.
4. Copy the selection and paste it into a plain-text editor; the original notes use Mousepad.
5. Review the extracted text. The content that was difficult to notice in the normal PDF view becomes readable and reveals the challenge answer.

### Why this works
A PDF's visual presentation and its underlying text layer are different things. Text can remain selectable or extractable even when it is visually obscured, small, or otherwise difficult to notice.

### Takeaway
Before publishing documents, inspect their text layer, metadata, embedded objects, and copy/extraction behavior. Visual concealment does not make information confidential.

**Evidence:** PDF pages 3–4.

---

## 3. Warmup

**Points shown:** 50 · **Area:** Web / contextual clue discovery

### Clue
The challenge caption asks: **“Did you read the rules?”** This points away from the challenge instance and toward the event's Rules page.

### Walkthrough
1. Read the challenge title and caption carefully.
2. Navigate to the DALCTF website's **Rules** section.
3. Inspect the page content, including the text near the end.
4. The relevant answer is present on the Rules page; submit it to complete the challenge.

The screenshot on PDF page 5 shows the Rules page and the relevant text.

### Takeaway
Challenge descriptions, captions, event pages, announcements, and downloadable resources can all contain intentional clues. Review the surrounding context before jumping to automated enumeration.

**Evidence:** PDF page 5.

---

## 4. Angry Shamir

**Points shown:** 50 · **Area:** Cryptography

### Walkthrough recorded in the source
1. Download the attached ciphertext/parameter file.
2. Inspect the file and identify the cryptographic values provided.
3. Analyze the RSA/Shamir-related parameters.
4. Use a Python script to recover the plaintext.
5. Save and execute the script.
6. Use the recovered plaintext to identify and submit the answer.

### Reproducibility limitation
The PDF confirms that parameter analysis and a Python script were used successfully, but it does not provide a readable complete mathematical derivation, exact parameter values, or the full solver source. It therefore does not support naming a specific factorization weakness or reconstructing the exact algorithm.

To make this section fully reproducible, add the original challenge attachment, public parameters/ciphertext, mathematical relationship exploited, complete Python solver, and an explanation of how the recovered plaintext was validated.

**Takeaway:** A cryptography write-up should preserve the actual inputs, derivation, code, and validation—not just the statement that a script worked.

**Evidence:** PDF pages 6–7.

---

## 5. Weird Movements

**Points shown:** 388 · **Area:** Digital forensics / HID analysis

### Objective
The supplied capture file contains data that can be extracted as `hid.txt`. The notes interpret the records as Human Interface Device (HID) mouse events and reconstruct the cursor movement to reveal an image.

### Step 1 — Inspect and extract
The original notes recommend basic file inspection with tools such as:

```bash
strings <capture-file>
binwalk <capture-file>
```

Review the output and extract the embedded `hid.txt` data.

### Step 2 — Interpret the records
The supplied script processes lines containing 8 hexadecimal characters (four bytes). It uses:
- Byte 0 for mouse button state.
- Byte 1 for X movement delta.
- Byte 2 for Y movement delta.

The X and Y values are initially unsigned bytes. Values greater than 127 are converted to signed 8-bit deltas by subtracting 256. The script accumulates these deltas to reconstruct the cursor's position.

### Step 3 — Reconstruct the strokes
Bit 0 of the button byte (`buttons & 0x01`) indicates whether the left mouse button is pressed. While pressed, the current coordinate is added to the active stroke. When released, the stroke is saved and a new one begins. The Y coordinate is negated for plotting orientation.

### Python reconstruction

This is a cleaned-up, formatted version of the logic shown on PDF pages 8–9:

```python
import matplotlib.pyplot as plt

x = 0
y = 0
segments = []
current = []

with open("hid.txt", "r", encoding="utf-8") as handle:
    for line in handle:
        line = line.strip()

        if len(line) != 8:
            continue

        data = bytes.fromhex(line)

        buttons = data[0]
        dx = data[1]
        dy = data[2]

        # Convert unsigned bytes to signed 8-bit movement deltas.
        if dx > 127:
            dx -= 256
        if dy > 127:
            dy -= 256

        # Reconstruct cumulative cursor coordinates.
        x += dx
        y += dy

        # Bit 0 represents the left mouse button.
        left_pressed = buttons & 0x01

        if left_pressed:
            current.append((x, -y))
        else:
            if len(current) > 1:
                segments.append(current)
            current = []

if len(current) > 1:
    segments.append(current)

for segment in segments:
    xs = [point[0] for point in segment]
    ys = [point[1] for point in segment]
    plt.plot(xs, ys)

plt.axis("equal")
plt.savefig("flag.png")
```

### Step 4 — Decode the drawing
1. Run the script in the directory containing `hid.txt`.
2. Open the generated `flag.png`.
3. Read and transcribe the letters drawn by the cursor.
4. The PDF records the intermediate transcription as `qnypgs{u3u3_v_f2_c41ag}`.
5. Apply **ROT13** in CyberChef.
6. Submit the decoded result.

### Takeaway
HID telemetry can expose user interaction. Correct signed-byte conversion, cumulative coordinate tracking, button-state interpretation, and stroke segmentation are all necessary to reconstruct the drawing accurately.

**Tools:** `strings`, `binwalk`, Python, Matplotlib, CyberChef.  
**Evidence:** PDF pages 7–9.

---

## 6. Secure the login

**Points shown:** 295 · **Area:** Secure coding / remediation

### Scenario
The challenge describes NovaCorp's internal employee portal as a simple username-and-password login that was shipped quickly. The task is to identify and correct the weakness without breaking legitimate access.

### Walkthrough recorded
1. Open the defense environment linked from the challenge.
2. Inspect the login implementation in the provided code editor.
3. Identify the vulnerability in the authentication code.
4. Correct the vulnerable code.
5. Select **Run All Attacks** in the attack panel.
6. Wait for the validation process to finish.
7. Confirm that the attack checks are neutralized and the completion screen appears.

PDF page 11 shows the code editing and attack-validation process; page 12 shows the successful completion state.

### What the evidence supports
The completion screenshot includes a result label referencing SQL injection. This supports identifying SQL injection as a central challenge issue. The PDF does not preserve enough readable source code to document the exact vulnerable query or the exact patch applied.

### General remediation guidance
For login code that builds SQL queries from user input:
- Use parameterized queries or prepared statements.
- Do not concatenate raw username or password values into SQL statements.
- Keep authentication and authorization checks on the server.
- Return safe, consistent authentication errors.

The points above are general secure-development guidance; the precise challenge patch is not legible in the supplied source.

**Evidence:** PDF pages 10–12.

---

## 7. Render and Plunder

**Points shown:** 318 · **Area:** Application security / code remediation

### Scenario
The challenge describes a user-profile service with a renderer. The task is to inspect the implementation and patch vulnerabilities while keeping the application functional.

### Walkthrough recorded
1. Open the challenge's defense environment.
2. Review the supplied application code.
3. Identify the vulnerable behavior in the profile/renderer functionality.
4. Apply fixes to the issues found.
5. Run the application and start the provided attack-validation process.
6. Wait for the checks to complete.
7. Confirm that the attack panel reports successful neutralization and that the completion screen is displayed.

PDF pages 13–14 show the challenge description, code review, running attack panel, and completion screen.

### What the evidence supports
The notes state that multiple vulnerabilities were found and fixed. The completion screen visibly references IDOR- and SSTI-related terms, but the PDF does not include enough readable code or explanation to map each term to a specific vulnerable function or patch.

### General defensive lessons
- Enforce object-level authorization whenever a user accesses or modifies a profile or another resource.
- Treat user-controlled template content as data, not executable template syntax.
- Avoid passing untrusted input into template engines without safe handling.
- Re-run security tests after patching to check exploit resistance and normal application behavior.

These are general defensive recommendations; the exact challenge code changes are not documented clearly enough in the source PDF.

**Evidence:** PDF pages 13–14.

---

## 8. Snow And Water

**Points shown:** 62 · **Area:** OSINT / geolocation

### Objective
The challenge presents an interactive 360-degree scene. The task is to identify the real-world location from visual clues.

### Walkthrough
1. Open the **Snow And Water** challenge and load the panorama.
2. Rotate the camera through the available 360-degree view.
3. Examine the surroundings rather than relying on a single frame.
4. Note the clues recorded in the PDF:
   - Houses close to a body of water.
   - A snowy or snow-covered mountain in the background.
   - The style and arrangement of nearby buildings.
5. Use Google Lens to search for visually similar locations.
6. Compare possible matches with the landscape and settlement in the panorama.
7. The notes identify the area as near **Kapisillit, Greenland**.
8. Place the marker on Kapisillit in the map interface and submit.
9. The final screenshot on PDF page 17 confirms successful completion.

### Takeaway
Geolocation is more reliable when multiple independent clues agree. Compare building styles, terrain, shoreline geometry, vegetation, mountain profiles, and settlement layout instead of trusting a single reverse-image result.

**Tool:** Google Lens.  
**Evidence:** PDF pages 15–17.

---

## Challenge Summary

| Challenge | Points shown | Main skill |
|---|---:|---|
| Baby Web | 50 | Page-source inspection |
| Warmer Up | 50 | PDF text extraction |
| Warmup | 50 | Contextual clue discovery |
| Angry Shamir | 50 | Cryptographic analysis |
| Weird Movements | 388 | HID forensics and reconstruction |
| Secure the login | 295 | Secure coding and attack validation |
| Render and Plunder | 318 | Application security remediation |
| Snow And Water | 62 | Visual OSINT and geolocation |

## Tools and Techniques

- **View Page Source / browser tools:** inspect client-delivered content.
- **PDF viewer and Mousepad:** expose selectable text in a document.
- **Python:** cryptographic analysis and HID record processing.
- **`strings` and `binwalk`:** inspect files and identify embedded content.
- **Matplotlib:** plot reconstructed mouse movement.
- **CyberChef:** apply ROT13.
- **Google Lens:** identify visual location clues.

## Key Takeaways

1. Inspect page source and surrounding event content before assuming a challenge requires complex exploitation.
2. A document's visual layout does not guarantee that its underlying text is hidden.
3. Cryptography notes should preserve the actual parameters, derivation, solver, and validation.
4. HID records can be reconstructed into a visual trace when byte interpretation and cursor state are handled correctly.
5. Defensive challenges require fixing the root cause and validating the patch against the supplied attack suite.
6. Geolocation should be based on multiple corroborating visual clues.

## Reproducibility Notes

The walkthrough is based on the supplied 17-page PDF, including its screenshots. Three sections have source limitations:
- **Angry Shamir:** exact cryptographic values, derivation, and complete solver are not included in readable form.
- **Secure the login:** the exact patch is not sufficiently readable; the completion screenshot points to SQL injection.
- **Render and Plunder:** multiple issues are mentioned, and the completion screen references IDOR/SSTI terms, but exact vulnerable lines and patches are not documented.

To make those sections fully reproducible, add the original challenge files, readable code screenshots, and the solver or patch details.

*Flags and complete answer strings are intentionally omitted from this public-facing write-up. All testing described here was performed in the event's challenge environments.*
