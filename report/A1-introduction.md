# A1: Chapter 1, Introduction (reviewed text)

Owner of the review: Ali. Status: review of the Chapter 1 draft in the integrated report (`Phase_1_and_2.pdf`), 8 Nov 2026. Checked against `B6-technology-final.md`, `C4-duplicate-detection.md`, `C7-password-management.md`, `C11-security-privacy.md`, `A7-problem-statement.md` and the approved proposal (`docs/proposal/Cross_Device_AI_Proposal_AR.pdf`). The draft was right in most places, so it was corrected, not rewritten. The corrected text is below; the list of changes is at the end.

---

## Chapter One: Introduction

### 1.1 Introduction
This is the Phase 1 report of the graduation project Cross-Device AI Assistant. The same files and photos are often stored on a user's phone, laptop and tablet and use storage twice, and the existing solutions upload the files to an outside cloud. The project proposes a tool that works only on the user's local network: it finds identical files by their fingerprint (hash) and similar photos with small AI models that run on one of the user's own computers, searches across all the devices, and lets the devices share files directly. Phase 1 is analysis and design: no code is written.

### 1.2 Project Overview
The project is an AI assistant that brings a user's devices together in one network. It has no final name yet; this report calls it the Cross-Device AI Assistant. Every device (a Windows, macOS or Linux computer, an Android phone or tablet, an iPhone or iPad) runs the same app. The apps find each other on the local network, trust each other only after the user has paired them, and talk directly. One laptop or PC, chosen by the user, also takes the AI role; phones and tablets never do. Every feature is off until the user turns it on. The system has eight features:
1. Duplicate file detection: find identical files, and similar photos and videos, on more than one device.
2. Unified search: find a file on another device from any device, by name or by meaning.
3. Password management: one encrypted password vault, shared only between the user's own devices.
4. Clipboard sync: copy on one device, paste on another.
5. Screen sharing and remote control: view and control one device from another.
6. Extra screen: use a tablet or another device as a second monitor.
7. Direct file sharing: send files between devices over the local network.
8. Offline records: see a device's files while it is switched off, from its last known list.

### 1.3 Project Scope
- **Phase 1 is analysis and design only.** Its results are documents: analysis, architecture, decisions and diagrams.
- **Features.** All eight features are designed. Password management, screen sharing and remote control, and the extra screen stay in scope as design only.
- **Local network only.** No internet, no cloud server and no user account; data goes only to the user's own paired devices.
- **Platforms.** Windows, macOS, Linux, Android and iPhone/iPad (minimum versions in Table 3.1). iPhone and iPad have the most limits: they see only the files the user allows, cannot be controlled from another device and never take the AI role.
- **Out of scope.** Syncing over the internet, sharing with other people, a browser extension and a chat assistant.
- **Language.** The report is in English.

### 1.4 Aim and Objectives
**Aim.** To design an AI assistant that connects a user's devices directly over the local network, with no internet and no cloud server, and understands the files across all of them.

**Objectives.**
- Design the architecture of the system and choose its technology for all five systems.
- Design how devices find each other, pair and trust each other, and what a trusted device may do.
- Design each feature: duplicate file detection, unified search, file sharing, passwords, clipboard sync, screen sharing and remote control, extra screen.
- Keep a record of each device's files so that they can be seen while the device is off.
- Protect the user's data: encryption in transit and at rest, and every feature off until the user turns it on.
- Draw the diagrams, list the requirements and plan the evaluation for Phase 2.

### 1.5 Motivation
People now own several devices, and the same photos and documents end up on more than one of them. This wastes storage without the user knowing, makes it hard to remember which device holds a file, and splits everyday tasks (moving a file, copying text, using another screen) across different apps and cables. Cloud services solve part of this, but only by uploading personal files to an outside server and needing a constant internet connection. This project keeps everything on the user's own devices and local network.

### 1.6 Project Plan and Schedule
Phase 1 is done in the five steps of Table 1.1. The next phase is the implementation and testing of the system.

*Table 1.1: The steps of Phase 1.* (unchanged)

Table 1.2 lists what each round of work has delivered.

| Round | What was delivered |
|---|---|
| Round 1 | System architecture (first version); technology choices (first version); the AI device and file-list answers; a study of three existing applications; clipboard sync notes. |
| Round 2 | System architecture, version 2; the design of device discovery, pairing and trust; the final technology choices; the duplicate-file design; the file indexing and offline-records design with the updated AI device design; the problem statement; a study of six existing applications; the clipboard sync design; the first version of the screen sharing and remote control design. |
| Round 3 | Password management and security and privacy designs (first versions); this review of Chapter 1. *[To complete when the round closes, with the other members' deliveries.]* |

*Table 1.2: What each round delivered.*

### 1.7 Outline of the Report
Chapter 1 introduces the project, its scope and its aim. Chapter 2 gives the background and studies existing systems. Chapter 3 states the problem, lists the functional and non-functional requirements, and gives the technology choices and the evaluation plan. Chapter 4 presents the system design: the architecture, the data-flow, sequence and use case diagrams, the data model, the feature designs and the user interface. The references are at the end.

### 1.8 Summary
This chapter introduced the Cross-Device AI Assistant, its eight features, its scope, its aim and its motivation. The next chapter gives the background and studies existing systems.

---

## List of changes and why

| Section | Change | Why |
|---|---|---|
| 1.1 | "finds identical and similar files ... with machine learning" → identical files by hash, similar photos with small AI models on one of the user's computers; added search | Exact copies use no AI (B6 choice 4, C4); search is one of the main features |
| 1.2 | Added: trust only after pairing; the AI role on a laptop or PC chosen by the user, never a phone; every feature off until turned on | B1 v2, C2 Q1, C11 section 3 |
| 1.2 item 1 | "identical or similar files" → identical files, and similar photos and videos | C4 compares photos and videos only for "similar" |
| 1.2 item 2 | Added "by name or by meaning" | C2, C5 |
| 1.2 item 3 | "all accounts and passwords of all devices in one place" → one encrypted vault shared between the user's own devices | C7: accounts are typed or imported, not collected from every device |
| 1.3 | Added "Local network only", iPhone/iPad limits and "Out of scope" | B1 rule 1, B6 choice 1, C7 section 7, C9 |
| 1.4 | Pairing objective now includes "what a trusted device may do"; new objective on data protection; evaluation plan "for Phase 2" | C1 and C11 split the trust work; C11 is a design of its own |
| 1.5 | Rewritten shorter; it repeated 1.1 almost word for word | Uses the three problems of A7 and adds the privacy reason |
| 1.6 | Table 1.2 renamed "What each round delivered" and given a Round 3 row (to complete at the end of the round) | The draft said "the first two rounds" and "next round" |
| 1.7 | Chapter 3 also has the requirements and evaluation plan; Chapter 4 also has the diagrams and data model | Matches the report's table of contents |

**Outside Chapter 1, for Mahmoud to decide:**
- The cover says "Phase 1 and 2 Report", while 1.1, the abstract and the PDF title say "Phase 1". One of them should change.
- Table 3.1, row "Storage": if C11 is accepted, the main database is also encrypted (C11 section 4), and the vault key is random and wrapped by the master password (C7 section 3). Both are flagged as changes to B6 in those files.
- Section 4.4.4: the preview size (224 × 224) and the three frames per video could be added in one line (C4, proposed to Mohammed on 8 Nov).
