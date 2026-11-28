# L3m0n CTF 2026 (Quals) — Chapter 0

**Author:** Prithak Das  
**Platform:** L3m0nCTF Beginner Labs  
**Chapter:** 0  
**Challenge count:** 15 (3 per category)  
**Points:** 50 TRP each, as shown in the challenge board  
**Categories:** Cryptography · Pwn · Web · Reverse Engineering · Digital Forensics

> This write-up is based on the challenge board and the solving conversation. Where an exact ciphertext, command transcript, or raw output is not available in the retained discussion, that limitation is stated rather than fabricating it.

## Challenge Board

| Category | Challenge | Points |
|---|---|---:|
| Cryptography | Secret Decoder Ring | 50 TRP |
| Cryptography | Layers of the Onion | 50 TRP |
| Cryptography | XOR Hero | 50 TRP |
| Pwn | Overflowing Ooze | 50 TRP |
| Pwn | Magic Number | 50 TRP |
| Pwn | Candy Shop Overflow | 50 TRP |
| Web | Behind the Curtain | 50 TRP |
| Web | Cookie Monster's Jar | 50 TRP |
| Web | Villain's Login | 50 TRP |
| Reverse Engineering | Strings Attached | 50 TRP |
| Reverse Engineering | Snake Charmer | 50 TRP |
| Reverse Engineering | Gadget's Crackme | 50 TRP |
| Digital Forensics | Snapshot Secrets | 50 TRP |
| Digital Forensics | Identity Crisis | 50 TRP |
| Digital Forensics | Wiretap Whiskers | 50 TRP |

---

# 1. Cryptography

## 1.1 Secret Decoder Ring

**Technique:** Caesar cipher  
**Shift:** 7 positions backwards

Each alphabetic character was shifted by a fixed number of positions. Testing the Caesar shifts and moving the text **7 positions backwards** revealed the readable content and flag.

**Procedure**
1. Identify the text as a Caesar-style substitution.
2. Test the possible shifts or use a Caesar decoder.
3. Apply a shift of 7 backwards.
4. Read the decoded flag.

The exact ciphertext and decoder command are not retained in the available transcript.

**Flag**
```text
L3m0n{3b56db3a159b883d2371437952be1635}
```

**Lesson:** Caesar ciphers have a small key space; checking all 25 nontrivial shifts is practical.

## 1.2 Layers of the Onion

**Technique:** Multi-layer encoding  
**Decoding order:** Hexadecimal → Base64 → Base32

The data was encoded in multiple layers. The solve removed the layers in sequence:
1. Decode the outer hexadecimal layer.
2. Decode the resulting Base64 data.
3. Decode the next Base32 layer.
4. Continue examining decoded output until the flag is revealed.

The original encoded input and exact command transcript are not retained.

**Flag**
```text
L3m0n{022a9be46872055f0d6617b210e3aa88}
```

**Lesson:** Encoding is not encryption. Output character sets, padding, and recognizable data formats help identify each layer.

## 1.3 XOR Hero

**Technique:** Single-byte XOR  
**Key:** `0x5a`

The challenge data was XOR-encoded with a single-byte key. XORing the data with the same key recovered the plaintext.

**Procedure**
1. Recognize the data as XOR-encoded.
2. Determine the key `0x5a`.
3. XOR each byte with that key.
4. Inspect the recovered plaintext for the flag.

The original ciphertext and exact command/script are not retained.

**Flag**
```text
L3m0n{1cd8fe934260b44140ef396a04dfadb2}
```

**Lesson:** XOR is reversible; applying the same key twice restores the original data.

---

# 2. Pwn

## 2.1 Overflowing Ooze

**Technique:** Buffer overflow / adjacent-variable overwrite

The local reproduction demonstrated that writing past the input buffer could modify the adjacent authorization-related `is_admin` value.

**Local command shown**
```bash
python3 -c 'print("A"*16 + "B")' | ./ooze
```

The first 16 `A` characters filled the buffer. The following `B` byte has decimal value 66 and reached the adjacent variable. The local program displayed `is_admin = 66` and entered the admin branch.

The local flag file was unavailable. The flag recorded during the challenge solve was:

```text
L3m0n{99729af6855230fcd26e57976327d406}
```

**Lesson:** Buffer overflows can corrupt adjacent local data, not only saved return addresses. A local demonstration of control flow or variable corruption is distinct from retrieving a hosted flag.

## 2.2 Magic Number

**Technique:** Buffer overflow / structure-field overwrite

The provided program placed a 32-byte character buffer immediately before an unsigned integer field:

```c
struct {
    char spell[32];
    unsigned int secret;
} book;

book.secret = 0;
gets(book.spell);

if (book.secret == 0x6e306d33) {
    // print flag
}
```

The target value was `0x6e306d33`. In little-endian byte order, its bytes are `33 6d 30 6e`, which represent ASCII `3m0n`.

**Payload:** 32 `A` characters followed by `3m0n`.

**Local command shown**
```bash
python3 -c 'print("A"*32 + "3m0n")' | ./magic
```

The hosted lab confirmed `secret = 0x6e306d33` and entered the success branch. The same overwrite was reproduced locally, but the local flag file was absent.

**Flag**
```text
L3m0n{99211fb421de7faa767937a2451338ee}
```

**Lesson:** Unsafe input can overwrite adjacent structure members. Buffer size, memory layout, and endianness are important when targeting a numeric field.

## 2.3 Candy Shop Overflow

**Technique:** Signed integer overflow / business-logic bypass

The Golden Flag Bar cost 1,000,000 coins. The program multiplied price by quantity into a signed 32-bit `int`, then checked whether the total exceeded the wallet:

```c
int total = menu[choice - 1].price * qty;

if (total > wallet) {
    // insufficient funds
}

wallet -= total;

if (choice == 4) {
    // print flag
}
```

**Successful hosted input**
```text
Menu choice: 4
Quantity: 2148
```

The mathematical product was:

```text
1,000,000 × 2,148 = 2,148,000,000
```

This exceeds the maximum signed 32-bit integer, 2,147,483,647. The challenge arithmetic wrapped the value to:

```text
-2,146,967,296
```

The negative total passed the `total > wallet` check. Subtracting a negative total increased the wallet. The hosted lab showed the wallet balance as `2,146,967,396`.

**Flag**
```text
L3m0n{ce9a10a23c2e29a7a001d321d177f668}
```

**Lesson:** Validate quantity limits, use checked arithmetic or wider intermediate types, and never base payment or authorization decisions on an overflowed value.

---

# 3. Web

## 3.1 Behind the Curtain

**Technique:** Source inspection / hidden information discovery

The flag was split across several public resources:
- A fragment in the HTML source.
- Another fragment in the linked CSS.
- A hidden path disclosed by `robots.txt`.
- The final fragment in the vault at `/secret-cartoon-vault/`.

**Procedure**
1. Inspect the page source and find the HTML fragment.
2. Inspect the linked CSS file and find another fragment.
3. Check `robots.txt`.
4. Visit `/secret-cartoon-vault/`.
5. Join the fragments in their original order.

**Fragments recovered**
```text
HTML:        L3m0n{654233b0b8a
CSS:         740dec49e0f
Hidden vault: 0eb3275ebe}
```

**Flag**
```text
L3m0n{654233b0b8a740dec49e0f0eb3275ebe}
```

**Lesson:** Client-side source, stylesheets, and discovery files can expose information not visible in the rendered page.

## 3.2 Cookie Monster's Jar

**Technique:** Cookie manipulation / Base64-encoded client-side role data

The application used a cookie named `jar_session`. Its value was Base64-encoded JSON containing a role initially set to `guest`.

**Procedure**
1. Inspect browser cookies and locate `jar_session`.
2. Base64-decode the cookie value.
3. Inspect the decoded JSON.
4. Change the role from `guest` to `admin`.
5. Base64-encode the modified JSON.
6. Replace the cookie value and refresh the page.

**Finding:** Encoding is not a security control. If authorization relies on a user-editable cookie, the role can be tampered with. Privilege decisions must be enforced server-side and protected with integrity controls.

**Flag**
```text
L3m0n{ea00ae29b665dcc28b7168e55291c133}
```

## 3.3 Villain's Login

**Technique:** SQL injection in authentication

**Payload shown**
```sql
' OR '1'='1' --
```

The injected condition `'1'='1'` is always true. The trailing SQL comment marker was used to ignore the remainder of the original query in the challenge's SQL context. The payload was entered in the username field with an arbitrary password, bypassing the vulnerable login logic.

**Flag**
```text
L3m0n{6decfe706de49799146a988b599e476a}
```

**Lesson:** Parameterized queries prevent untrusted input from being interpreted as SQL syntax.

---

# 4. Reverse Engineering

## 4.1 Strings Attached

**Technique:** Static inspection with `file` and `strings`

**Commands shown**
```bash
file strings_attached
strings strings_attached | grep "L3m0n"
```

The first command identified the file as an ELF 64-bit binary. The second searched printable strings for the challenge flag prefix.

**Output**
```text
L3m0n{a659f74d32b844aa086c18b1ecc30a71}
```

**Lesson:** Basic static inspection can reveal hard-coded strings without executing a binary.

## 4.2 Snake Charmer

**Technique:** Reversing a custom string transformation

The transformation applied by the challenge was:
1. Reverse the string.
2. Add 3 to each character's ASCII value.
3. XOR each resulting value with `0x21`.

To reverse it, apply the inverse operations in reverse order:
1. XOR with `0x21`.
2. Subtract 3 from each character.
3. Reverse the string.

```text
Encoded value
    ↓ XOR 0x21
    ↓ subtract 3 from each character
    ↓ reverse
Original plaintext
```

The precise original encoded input and full script are not retained in this context.

**Flag**
```text
L3m0n{2242060639c0e971ff0ffe2ad1917ff5}
```

**Lesson:** Invert transformations in reverse order. Reversal, arithmetic shifts, and XOR are reversible when their parameters are known.

## 4.3 Gadget's Crackme

**Technique:** Static analysis / XOR-obfuscated string

**Commands shown**
```bash
file gadgets_crackme
strings gadgets_crackme
```

The binary was identified as an ELF 64-bit x86-64 executable and was not stripped. The strings output exposed symbols such as `lock`, `check`, success/failure messages, and an encoded key.

**Encoded key**
```text
I6h5k~dg7161gg<d03c0d27`3<20`17ff6641fx
```

Using the known flag prefix, the first encoded character `I` and plaintext character `L` gave the XOR key:

```text
'I' XOR 'L' = 0x05
```

**Python decoding logic**
```python
encoded = "I6h5k~dg7161gg<d03c0d27`3<20`17ff6641fx"
key = 0x05

decoded = ''.join(chr(ord(c) ^ key) for c in encoded)
print(decoded)
```

**Flag**
```text
L3m0n{ab2434bb9a56f5a72e6975e42cc3314c}
```

**Lesson:** Binary metadata and printable strings can expose useful clues. Known plaintext can help derive a simple XOR key.

---

# 5. Digital Forensics

## 5.1 Snapshot Secrets

**Technique:** Image metadata analysis

**Command shown**
```bash
exiftool postcard.jpg
```

The metadata output was inspected for unusual fields. The flag was located in the image's **User Comment** metadata field.

**Flag**
```text
L3m0n{d4a8428dbfbbc956e0203a634f14644f}
```

**Lesson:** Metadata can contain hidden information even when an image looks ordinary.

## 5.2 Identity Crisis

**Technique:** File-format validation / damaged PNG signature

The image was not recognized because its PNG signature had been damaged. The standard PNG signature is:

```text
89 50 4E 47 0D 0A 1A 0A
```

**Procedure**
1. Inspect the file's initial bytes.
2. Compare them with the expected PNG signature.
3. Restore the correct signature bytes.
4. Reopen the repaired image and inspect its contents.

**Flag**
```text
L3m0n{f11018bdfd3753a668238892d9fe236a}
```

**Lesson:** File identification relies on magic bytes, not just filenames or extensions. A damaged header can prevent a file from opening even when its remaining data is intact.

## 5.3 Wiretap Whiskers

**Technique:** Packet-capture analysis / plaintext HTTP inspection

The packet capture contained HTTP traffic. The analysis focused on locating the login request and inspecting its contents.

**Procedure**
1. Open the packet capture in a packet-analysis tool.
2. Identify HTTP traffic.
3. Locate the login request.
4. Inspect its method, path, headers, and body.
5. The plaintext POST request to `/login` exposed the challenge information and flag.

The exact packet filter and complete packet transcript are not retained.

**Flag**
```text
L3m0n{1708b56cb516b2c88abafc4d9b3b37e5}
```

**Lesson:** Plaintext HTTP can expose sensitive information in transit. Real applications should use HTTPS and avoid transmitting credentials or secrets in cleartext.

---

# Flag Summary

## Cryptography
| Challenge | Flag |
|---|---|
| Secret Decoder Ring | `L3m0n{3b56db3a159b883d2371437952be1635}` |
| Layers of the Onion | `L3m0n{022a9be46872055f0d6617b210e3aa88}` |
| XOR Hero | `L3m0n{1cd8fe934260b44140ef396a04dfadb2}` |

## Pwn
| Challenge | Flag |
|---|---|
| Overflowing Ooze | `L3m0n{99729af6855230fcd26e57976327d406}` |
| Magic Number | `L3m0n{99211fb421de7faa767937a2451338ee}` |
| Candy Shop Overflow | `L3m0n{ce9a10a23c2e29a7a001d321d177f668}` |

## Web
| Challenge | Flag |
|---|---|
| Behind the Curtain | `L3m0n{654233b0b8a740dec49e0f0eb3275ebe}` |
| Cookie Monster's Jar | `L3m0n{ea00ae29b665dcc28b7168e55291c133}` |
| Villain's Login | `L3m0n{6decfe706de49799146a988b599e476a}` |

## Reverse Engineering
| Challenge | Flag |
|---|---|
| Strings Attached | `L3m0n{a659f74d32b844aa086c18b1ecc30a71}` |
| Snake Charmer | `L3m0n{2242060639c0e971ff0ffe2ad1917ff5}` |
| Gadget's Crackme | `L3m0n{ab2434bb9a56f5a72e6975e42cc3314c}` |

## Digital Forensics
| Challenge | Flag |
|---|---|
| Snapshot Secrets | `L3m0n{d4a8428dbfbbc956e0203a634f14644f}` |
| Identity Crisis | `L3m0n{f11018bdfd3753a668238892d9fe236a}` |
| Wiretap Whiskers | `L3m0n{1708b56cb516b2c88abafc4d9b3b37e5}` |

---

## Conclusion

Chapter 0 comprised 15 beginner challenges, with three challenges in each of five categories. The collection covered classical ciphers and encoding, memory corruption and integer overflow, web source discovery and authentication flaws, static reverse engineering, and file/packet forensics.

The central lesson was to inspect the evidence, understand how the underlying mechanism works, and verify results rather than relying only on a challenge's visible description.
