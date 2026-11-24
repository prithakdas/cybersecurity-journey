# DALCTF 2026 — Write-up Notes

**Source:** `DalCTF2026_writeup(1).pdf` (17 pages)

## Challenges documented

### 1. Baby Web
Inspect the page source in the browser and search for brace-delimited text. The PDF reports that the flag was present in the source.
**Concepts:** source inspection, client-side information disclosure.

### 2. Warmer Up
Download `rules1.pdf`, select/copy its contents, and paste into a text editor to expose text not readily visible in the normal PDF view.
**Concepts:** PDF text extraction, hidden document content.

### 3. Warmup
Treat the challenge caption as a hint to inspect the event's Rules page. The write-up reports the flag there.
**Concepts:** contextual clues, website content inspection.

### 4. Angry Shamir
Download and inspect the supplied cryptographic data, analyze the RSA/Shamir-related parameters, and use a Python script to recover plaintext. The PDF reports success but does not provide a complete derivation in its text.
**Concepts:** RSA parameter analysis, Python-assisted cryptanalysis.

### 5. Weird Movements
- Inspect the capture file with tools such as `strings` and `binwalk`.
- Extract `hid.txt`.
- Parse 8-byte hexadecimal HID records, extracting button state and signed X/Y movement deltas.
- Accumulate cursor coordinates while the left mouse button is pressed and plot the strokes with Python/Matplotlib.
- Transcribe the resulting image and apply ROT13 as documented.

**Concepts:** file analysis, HID forensics, signed-byte conversion, coordinate reconstruction, plotting, ROT13.

### 6. Secure the login
The write-up describes identifying and correcting an issue in the supplied code, then running **Run All Attacks**. The exact vulnerable code and remediation are not sufficiently described in the PDF text to name the issue reliably.
**Concepts:** secure coding, remediation, regression validation.

### 7. Render and Plunder
The write-up reports multiple issues were corrected before running the application and completing the challenge. It does not describe the individual vulnerabilities in enough detail to identify them.
**Concepts:** code review, multi-issue remediation.

### 8. Snow And Water
Inspect the 360-degree scene for geographical clues around the water and snowy mountain. Use Google Lens to correlate the visual evidence. The write-up identifies the location as near **Kapisillit, Greenland**, then submits the map marker.
**Concepts:** visual OSINT, geolocation.

## Tools & Techniques
Browser source inspection · PDF text selection · Python · `strings` · `binwalk` · Matplotlib · ROT13 · Google Lens.

*Summary of the supplied 17-page write-up. Flags are omitted; undocumented details are not inferred.*
