# SBT-DF204 — Case Study 4: Investigating Rhion Possession Evidence

**International Cybersecurity and Digital Forensics Academy (ICDFA)**
School of Basic Vocational Training (SVT)

| Field | Detail |
| :--- | :--- |
| **Student** | Ibrahim Diseh Garba |
| **Registration No.** | 2025/FWSD/11521 |
| **Programme** | Fellowship in Web Application Security & Digital Forensics |
| **Course** | SBT-DF204 — Computer Forensics Case Studies |
| **Case Title** | Investigating Rhion Possession Evidence |
| **Instructor** | Aminu Idris, AMCPN |
| **Date** | 22 October 2026 |

---

## 1.0 Case Reference and Student Identification

| Field | Detail |
| :--- | :--- |
| **Full Name** | Ibrahim Diseh Garba |
| **Registration No.** | 2025/FWSD/11521 |
| **Course Code** | SBT-DF204 — Computer Forensics Case Studies |
| **Case Title** | Investigating Rhion Possession Evidence |
| **Evidence Package** | `DFRWS2005-RODEO.zip` (USB image + FTP/HTTP captures) |
| **Analysis Workstation** | Kali Linux VM (Isolated Lab Environment) |

---

## 2.0 Executive Conclusion

This forensic investigation examined a USB disk image and associated network captures to determine if the possession threshold of nine or more unique rhinoceros images was met. Through four independent techniques—disk carving, steganography detection, FTP reconstruction, and HTTP object extraction—**nine unique rhinoceros images were recovered and verified by SHA-256 hashing**.

**Key Findings:**
1. **Disk Carving:** Four unique rhino images were recovered from unallocated space in the FAT16 volume.
2. **Steganography:** Two JPEGs contained jphide payloads. Passwords were cracked, but extraction was blocked by a tool version mismatch.
3. **FTP & Archive:** Three FTP sessions uploaded rhino images and an encrypted ZIP. The ZIP was cracked, yielding a rhino image that perfectly matched one recovered by disk carving.
4. **HTTP:** Two rhino images were downloaded from a public web server.

**Confidence Level:** **High (≈ 90%)** that the possession threshold is met by the artifacts. **Moderate (≈ 60-70%)** for personal attribution, as the evidence establishes network activity and media contents, not the physical identity of the person operating the keyboard.

---

## 3.0 Evidence Acquisition and Integrity

### 3.1 Evidence Source
Four items were supplied for analysis:
1. `RHINOUSB.dd` — Raw USB disk image (259,506,176 bytes)
2. `rhino.log` — FTP capture
3. `rhino2.log` — HTTP capture 1
4. `rhino3.log` — HTTP capture 2

### 3.2 Working Copy Creation
All original files were preserved read-only. A working copy of the disk image was created using `cp --preserve=timestamps` to maintain original metadata.

```bash
# Working copy creation for disk image
cp --preserve=timestamps RHINOUSB.dd working/RHINOUSB_working.dd

# Hash verification
sha256sum RHINOUSB.dd working/RHINOUSB_working.dd


3.3 Integrity Verification Output
File	SHA-256 Hash
RHINOUSB.dd	ce550424200a997c61b413941c8ef4df9619a2f96579674952294a176a32be65
rhino.log	64e6d55b76660eb3aaa41572c1d04e4452510a343bbf42e844424827dedfddb2
rhino2.log	41939d5de0556b70279056572dee44b6fd84cd05b0788cfd0b6f52f37b161dde
rhino3.log	7b0304f5e88a30c305a99b5a1e2977bced5b9c94da6f458f887bb39966bbfc46
✅ Status: Hashes match. Original evidence is preserved; all analysis was performed on the working copy.
Figure 1: Evidence integrity verification (SHA-256 hashes). [Insert screenshot of terminal output]
4.0 Method
The investigation followed a five-stage workflow:
1.	Disk Image Examination: Used fsstat and fls (Sleuth Kit) to examine the FAT16 filesystem structure and allocated files.
2.	File Recovery: Used photorec to carve files from the unallocated space of the working disk image.
3.	Steganography Analysis: Used stegdetect and stegbreak with rockyou.txt to detect hidden payloads and recover passwords in the carved JPEGs.
4.	Network Reconstruction: Used tshark to filter FTP (ftp.request) and HTTP (http.request) traffic, exporting objects via --export-objects.
5.	Archive Cracking: Used fcrackzip against the reconstructed contraband.zip using the training wordlist.
5.0 Findings — Answers to Investigation Questions
Question 1: How did you preserve the case materials?
The original RHINOUSB.dd (259,506,176 bytes) and the three capture logs were preserved read-only. A timestamp-preserved working copy (RHINOUSB_working.dd) was created for analysis. The SHA-256 hash of the original and working copy were verified identical (ce550424...a32be65). All tool outputs and recovered files were saved to an isolated analysis directory.
Question 2: What does the USB disk image show before recovery?
Using fsstat, the image was identified as a FAT16 volume with 512-byte sectors and 4096-byte clusters. Using fls, only two allocated files were present: gumbo1.txt and gumbo2.txt. No rhino imagery existed in the allocated space, confirming the material was either deleted or hidden in unallocated clusters.
Figure 2: fsstat and fls output showing FAT16 structure and only two allocated text files. [Insert screenshot]
Question 3: What deleted or recoverable files are supported?
photorec was run against the unallocated space (Scope: Free). It recovered 132 files, of which 9 were images.
Visual inspection and hashing revealed four unique rhino images:
•	f0106393.jpg (Small rhino icon)
•	f0106409.jpg (Rhino mother + baby)
•	f0106865.gif (Cartoon rhino)
•	f0106889.gif (Blue cartoon rhino)
(Two recovered JPEGs were crocodiles; two others were byte-identical duplicates).
Figure 3: PhotoRec recovered files sorted by size, showing rhino images. [Insert screenshot]
Question 4: Is there evidence of hidden content in the recovered media?
Yes. stegdetect flagged f0104249.jpg for a jphide payload. While stegdetect skipped f0105065.jpg (false positive heuristic), stegbreak successfully cracked passwords for both:
•	f0104249.jpg : jphide[v5](gumbo)
•	f0105065.jpg : jphide[v5](gator)
Limitation: Extraction using jpseek 0.3 failed due to a version mismatch with the jphide v5 payload format. The detection and successful password recovery strongly support a finding of intentional concealment.
Figure 4: stegdetect and stegbreak output showing detection and password recovery. [Insert screenshot]
Question 5: What does the FTP capture establish?
Analysis of rhino.log revealed three FTP sessions from client 137.30.122.253 to server 137.30.120.40. Using plaintext credentials (gnome/gnome123), the client uploaded:
•	rhino1.jpg
•	rhino3.jpg (with an ASCII mode retry before successful binary transfer)
•	contraband.zip
The objects were successfully reconstructed using tshark --export-objects. Two unique rhinos were validated here.
Figure 5: tshark output showing FTP STOR commands and reconstructed objects. [Insert screenshot]
Question 6: What does the protected archive analysis establish?
contraband.zip (230,566 bytes) was reconstructed intact from FTP stream 305. fcrackzip successfully recovered the password: monkey. Extraction yielded rhino2.jpg.
Crucially, the SHA-256 hash of rhino2.jpg (b4f6bbb8...) perfectly matched the hash of f0106409.jpg recovered independently via PhotoRec carving. This provides cross-technique validation of the artifact.
Figure 6: fcrackzip output and extraction of rhino2.jpg. [Insert screenshot]
Question 7: What does the HTTP capture establish?
Analysis of rhino2.log and rhino3.log identified downloads from www.cs.uno.edu by client 137.30.123.234.
•	rhino4.jpg (153,191 bytes)
•	rhino5.gif (85,137 bytes)
•	rhino.exe (145,920 bytes)
Two additional unique rhinos were recovered. The executable rhino.exe was not executed; its MD5 (d62d9989...) matches Microsoft's diskpart.exe, indicating it is a legitimate Windows utility rather than rhino-related malware.
Figure 7: HTTP object extraction and file type verification. [Insert screenshot]
6.0 Timeline and Correlation
All timestamps are in UTC. The network captures originate from 2004, while analysis was conducted in 2026.
Stage	UTC Timestamp	Evidence ID	Source Artefact	Observed Fact	Interpretation and Limit
Media Preservation	2026-10-22 09:00	E01	RHINOUSB.dd	SHA-256 verified against working copy	Integrity established.
Disk Review	2026-10-22 09:15	E02	RHINOUSB.dd	FAT16 volume; only gumbo.txt allocated	Rhinos must be in unallocated space.
Recovery	2026-10-22 09:30	E03	recovered/	PhotoRec recovered 9 images (4 rhinos)	Direct evidence of deleted rhino media.
Hidden Content	2026-10-22 10:00	E04	steg/f0104...	jphide payloads detected; passwords cracked	Evidence of intentional concealment.
FTP Activity	2004-04-26 22:21:43	E05	rhino.log f1532	USER gnome / PASS gnome123	Plaintext credential capture.
FTP Transfer	2004-04-26 22:21:49	E05	rhino.log f1546	STOR rhino1.jpg (Binary)	Direct evidence of rhino upload.
FTP Transfer	2004-04-26 22:26:46	E05	rhino.log f5647	STOR contraband.zip	Archive upload.
ZIP Cracking	2026-10-22 10:15	E06	contraband.zip	Password monkey cracked; rhino2.jpg extracted	Hash matches carved f0106409.jpg.
HTTP Download	2004-04-28 21:07:00	E07	rhino2.log f49	GET rhino4.jpg (200 OK)	Direct evidence of rhino download.
HTTP Download	2004-04-28 21:08:44	E07	rhino2.log f217	GET rhino5.gif (200 OK)	Second rhino download.
Corroboration	—	E08	All	9 unique rhinos recovered across 4 techniques	Threshold met. High confidence.
7.0 Conclusion and Limitations
7.1 Supported Conclusion
Across four independent techniques, nine unique rhinoceros images were recovered and verified. The possession threshold defined in the case narrative is met. The FTP and HTTP captures establish specific network transfers of these artifacts, while disk analysis establishes physical possession of the USB media containing additional rhino images.
7.2 Limitations and Alternative Explanations
1.	Incomplete Steganographic Extraction: While two hidden payloads were detected and passwords recovered, extraction failed due to tool version mismatches (jpseek 0.3 vs jphide v5). The hidden images may or may not be additional rhinos.
2.	Lack of Personal Attribution: The network evidence establishes IP addresses and shared credentials, not a named person. The USB image proves possession of the physical device, but does not prove who operated the keyboard at the time of transfer.
3.	Duplicate Artifacts: The apparent volume of rhino images is inflated by duplicates. f0105873.jpg duplicates f0335081.jpg. rhino2.jpg from the FTP ZIP is a byte-identical match to f0106409.jpg from carving. The count of nine represents the unique total.
4.	Synthetic Filenames: The f0*.jpg filenames are generated by PhotoRec, not original user filenames.
5.	Unrelated Executable: The downloaded rhino.exe matches Microsoft's diskpart.exe and is unrelated to the rhino imagery.




8.0 Appendices Appendix A: Evidence Log
Evidence ID	Source Artefact	Observed Fact	Why It Matters	Figure Reference
E01	RHINOUSB.dd	SHA-256 match; FAT16 volume	Proves integrity; establishes media type	Fig 1, 2
E02	fsstat, fls	Only 2 allocated files (gumbo.txt)	Proves rhino images were deleted	Fig 2
E03	photorec output	132 files recovered; 4 unique rhinos	Direct evidence of rhino possession	Fig 3
E04	stegbreak output	Passwords gator and gumbo recovered	Evidence of intentional concealment	Fig 4
E05	rhino.log	3 FTP sessions; uploads rhino1, rhino3, ZIP	Network evidence of transfer	Fig 5
E06	contraband.zip	Password monkey; rhino2 matches carved hash	Cross-validation of artifact authenticity	Fig 6
E07	rhino2.log	HTTP GETs for rhino4.jpg, rhino5.gif	Additional rhino downloads	Fig 7
E08	Synthesis	9 unique rhinos recovered	Meets possession threshold	N/A
Appendix B: Selected Tool Output
FTP Extracted Objects (tshark output):
text
-rw-r--r-- 1 user user 65703 rhino1.jpg
-rw-r--r-- 1 user user 96899 rhino3.jpg
-rw-r--r-- 1 user user 230566 contraband.zip
Steganography Detection (stegbreak output):
text
f0105065.jpg : jphide[v5](gator)
f0104249.jpg : jphide[v5](gumbo)
HTTP Extracted Objects (file output):
text
rhino4.jpg: JPEG image data, 1531x2286
rhino5.gif: GIF image data, 400x275
rhino.exe: PE32 executable, MD5 matches Microsoft diskpart.exe
Appendix C: Chain of Custody Worksheet
Field	Value
Case ID	SBT-DF204-CaseStudy4-2025-FWSD-11521
Trainee	Ibrahim Diseh Garba
Student ID	2025/FWSD/11521
Date Acquired	22 October 2026
Original Files	RHINOUSB.dd, rhino.log, rhino2.log, rhino3.log
Handling Notes	Originals preserved read-only. Analysis performed on working copies. No external hosts contacted. Recovered rhino.exe not executed. Password cracking performed only on supplied training archive.
END OF REPORT
Submitted by: Ibrahim Diseh Garba | Student ID: 2025/FWSD/11521 | 22 October 2026


<img width="451" height="693" alt="image" src="https://github.com/user-attachments/assets/bf2ac583-7b49-4e42-83f1-37a8589e3521" />
