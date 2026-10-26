### Case Study 4: Investigating Rhion Possession Evidence

**International Cybersecurity and Digital Forensics Academy (ICDFA)**
School of Basic Vocational Training (SVT)

---

### Author Information

| Field | Details |
| :--- | :--- |
| **Student Name** | Ibrahim Diseh Garba |
| **Registration No.** | 2025/FWSD/11521 |
| **Programme** | Fellowship in Web Application Security & Digital Forensics |
| **Course** | SBT-DF204 — Computer Forensics Case Studies |
| **Instructor** | Aminu Idris, AMCPN |
| **Date** | 22 October 2026 |

---

### Executive Summary

This repository contains the forensic artefacts, methodology, and final report for an offline digital forensic investigation into a suspected case of illegal possession and distribution of prohibited imagery (rhinoceroses). 

The investigation analysed a seized USB disk image (`RHINOUSB.dd`) alongside three authorised network traffic captures (`rhino.log`, `rhino2.log`, and `rhino3.log`). Using four independent forensic techniques, **nine unique rhinoceros images** were recovered and validated by SHA-256 hashing, meeting the possession threshold described in the case narrative.

> **Primary Conclusion:** The technical evidence establishes a clear timeline of file deletion, concealment, and transfer. However, the logs alone cannot definitively prove human identity, highlighting the distinction between machine attribution and person-level attribution.

---

### Key Findings

The possession threshold of nine unique rhino images was met through the following recoveries:

| Source / Technique | Rhinos Recovered | Details |
| :--- | :---: | :--- |
| **Technique 1: PhotoRec Carving** | **4** | Recovered from unallocated space on `RHINOUSB.dd`. |
| **Technique 2: Steganography** | **2** | `jphide` payloads detected. Passwords (`gator`, `gumbo`) cracked. *(Extraction documented as a tool limitation).* |
| **Technique 3: FTP & ZIP** | **3** | Uploads intercepted; `contraband.zip` cracked (`monkey`) to reveal `rhino2.jpg`. |
| **Technique 4: HTTP Extraction** | **2** | Direct downloads of `rhino4.jpg` and `rhino5.gif` from a local web server. |
| **TOTAL UNIQUE EXTRACTED** | **9** | **Meets the 9+ threshold.** |

### 🔗 Cross-Technique Correlation
The strongest piece of corroborating evidence was a byte-for-byte match between two independent techniques:
*   `rhino2.jpg` (extracted from the cracked FTP ZIP archive) possessed the exact same SHA-256 hash (`b4f6bbb8...`) as `f0106409.jpg` (recovered directly from the USB image via PhotoRec).

---

### Methodology & Tools Used

The investigation followed the Lab Guide Phases 0-7. The environment was standardised using a Kali Linux VM with networking disabled. 

### Phase 1: Evidence Acquisition & Integrity
*   **Commands:** `sha256sum`, `md5sum`, `cp --preserve=timestamps`
*   **Action:** Original evidence was hashed and preserved read-only. A working copy was created and verified byte-for-byte identical.

### Phase 2: Disk Image Analysis & File Carving
*   **Tools:** Sleuth Kit (`mmls`, `fsstat`, `fls`, `icat`), PhotoRec 7.2
*   **Action:** Identified a FAT16 filesystem. Carved 132 files from unallocated space, isolating 9 image files.

### Phase 3: Steganography Detection
*   **Tools:** `stegdetect`, `stegbreak`, `jpseek`
*   **Action:** Detected `jphide(*)` payloads in two JPEGs. Successfully cracked passwords using `rockyou.txt`. 
*   *Note:* Payload extraction failed due to a version mismatch (`jpseek 0.3` vs `jphide v5`) and 32-bit Wine dependency conflicts in Kali.

### Phase 4: Network Traffic Analysis (FTP & HTTP)
*   **Tools:** Wireshark, `tshark`
*   **Action:** Followed TCP streams (UTC timezone). Extracted objects from FTP-DATA and HTTP. 
*   **Credentials:** Identified plaintext FTP credentials (`gnome`/`gnome123`) which were documented but not reused.

### Phase 5: Protected Archive Analysis
*   **Tools:** `fcrackzip`, `unzip`
*   **Action:** Cracked `contraband.zip` via dictionary attack, extracting `rhino2.jpg`.

### Phase 6: Static Executable Analysis
*   **Tools:** `file`, `md5sum`, `exiftool`, `binwalk`
*   **Action:** Identified `rhino.exe` (MD5: `d62d9989...`) as Microsoft's legitimate `diskpart.exe`. **The file was NOT executed.**

---

### Challenges & Limitations

1.  **Tool Compatibility:** Legacy steganography tools (`jpseek 0.3`) required 32-bit Wine dependencies that conflicted with the modern Kali XFCE desktop environment. Thus, while detection and password recovery succeeded, payload extraction was blocked.
2.  **Password Cracking Limitations:** The ZIP archive extraction relied on a dictionary attack (`rockyou.txt`). A complex password would have rendered the archive inaccessible.
3.  **Data Duplication:** The PhotoRec carving process recovered numerous duplicate and non-rhino images (e.g., crocodiles). Meticulous hash comparison was required to isolate the 9 unique rhinos.
4.  **Attribution:** While the network captures establish source IPs (`137.30.12x.x`) and the user account (`gnome`), they do not definitively prove the physical identity of the person at the keyboard

---

# 📄 License

This repository and the accompanying forensic scripts/documentation are licensed under the **MIT License**. 

You are free to use, copy, modify, merge, publish, and distribute this work for educational and research purposes. 

See the [LICENSE](LICENSE) file in the root of this repository for the full legal text.




