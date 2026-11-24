# c4ptur3-th3-fl4g — TryHackMe

**Source:** `c4ptur3-th3-fl4g_TryHackMe.pdf` (8 pages)

## Overview
This room practices common encodings, audio spectrogram inspection, and image steganography.

## Task 1 — Translation & Shifting
The PDF documents these decoding methods:
- Leetspeak substitution: `4→a`, `1→l`, `0→o`, `2→r`, `7→t`, `3→e`, `9→g`.
- Binary using CyberChef **From Binary**.
- Base32 using **From Base32**.
- Base64 using **From Base64**.
- Hexadecimal using **From Hex**.
- ROT13 and ROT47.
- Morse code using **From Morse Code**.
- Decimal values using **From Decimal**.
- A multi-stage chain: Base64 → Morse → Binary → ROT47 → Decimal.

**Tool:** CyberChef.  
**Lesson:** identify the representation first and preserve the exact order of transformations in layered encodings.

## Task 2 — Spectrogram
1. Open the supplied audio in Sonic Visualiser.
2. Select **Layer → Add Spectrogram**.
3. Read the hidden text displayed in the spectrogram and submit it.

## Task 3 — Image Steganography
The documented workflow uses `steghide`:
```bash
steghide --info <image>
steghide extract -sf <image>
```
The PDF says to press Enter when prompted for the passphrase, then inspect the extracted text file.

## Task 4 — Security Through Obscurity
Run `strings <image>` and review the output, including its final lines, for embedded text.

## Tools & Concepts
CyberChef · Sonic Visualiser · steghide · strings · binary/Base32/Base64/hex · ROT13/ROT47 · Morse · decimal encoding.

*Summary of the supplied write-up. Flags are intentionally omitted.*
