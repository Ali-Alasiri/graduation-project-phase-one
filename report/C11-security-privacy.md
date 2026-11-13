# Security and Privacy

## Introduction

The app moves file lists, files, passwords and screen images between the user's devices, so the user must be able to trust that this data reaches only their own devices and nobody else. Security and Privacy decides what a paired device may do, how the data stored on each device is protected, and what happens when a device is lost. For the user this means three things. Every feature is off until the user turns it on, and the user decides which device may do what. The data stored by the app is encrypted, and some of it is never stored at all. A lost device can be removed from any other device. This section builds on Device Discovery, Pairing and Trust, which decides how devices meet and become paired.

## 1. What this design decides

- **Device Discovery, Pairing and Trust decides** what identifies a device, what a device announces on the network, how a new device is paired (by QR code or by comparing a code), how every connection is checked, how a removal is made and passed on (including the exchange of removal lists at every new connection), and that an unknown device receives no feature data.
- **Security and Privacy decides** what a paired device may do (permissions for each feature, section 3), how stored data is protected (section 4), what happens to a lost device beyond its removal (section 5), the use of the operating system's key store on each system, and backups.
- **The check between them.** Before any feature data goes to a device, the core services answer two questions: is the device paired, and is this feature turned on for this device? Both answers must be yes.

## 2. Threats and protection

The final technology choices already provide the base: TLS 1.3 in which the connecting device checks the other device's certificate and then proves its own key with a signed challenge; SQLCipher for stored data; device keys in the operating system's key store; and no internet.

| Threat | Protection | What remains |
|---|---|---|
| **Someone on the same Wi-Fi** | The person can see announcements and can complete TLS on the normal port, but is refused at the signed challenge. The pairing port answers only while the user has a pairing window open, and pairing needs a code or QR code that a person checks. All traffic uses TLS 1.3 | The person learns that the app is running and sees the device ids, and can disturb discovery |
| **A stolen phone that is unlocked** | Every feature is off until the user turns it on. The vault locks after 5 minutes and when the screen locks. Deleting files, showing passwords and allowing remote control ask for the fingerprint, face or PIN again (sections 3 and 5) | Whatever is open on the screen. The thief can use the features already allowed until the user removes the phone from another device |
| **A paired device that misbehaves** | It receives only the features turned on for it. A delete request needs the owner device's permission or its user's confirmation, and is recorded in the deletion log. A device accepts a file list only if the owner device's signature is valid and its change number is higher than the one it already holds, so a device that passes a list on cannot change it | It can read what it was allowed to read, send false removal messages, and hold back or delay a list that it passes on |
| **Malware on one device** | Keys are in the operating system's key store; the vault is locked most of the time; the main database is encrypted (section 4) | Malware running as the user can read memory while the app is open. On Windows, by inference, it can also read the key store. No app can fully prevent this |
| **A lost device that was switched off** | The user removes it from any other device, and the removal is passed on | A device that was off still trusts the lost device until it meets another of the user's devices (section 5) |

## 3. Permissions for each feature

Every feature is off by default on each device and for each pair of devices. "Who approves" names the person who must agree.

| Feature | Off by default | Who approves | Once or every time |
|---|---|---|---|
| File Indexing and Offline-Device Records (sharing the file list) | Yes | The owner turns on "Share my list" for each device | Once |
| Photo previews for the AI device | Yes | The owner of the photos | Once, with a switch for each device |
| Duplicate File Detection: viewing groups | Uses the file lists | Not needed | Not needed |
| Duplicate File Detection: deleting on another device | Yes | The owner turns on "Allow my other devices to delete duplicates here", or the user confirms each request on the owner device | Once, or for each request. Each deletion is still confirmed by the user, checked again, and recorded in a deletion log kept for 30 days on the owner device. On Android, photos and videos also show the system's own prompt; other Android files show the app's own confirmation |
| Unified Search | Uses the file lists and the AI device | Not needed | Not needed |
| Direct File Sharing | Yes | The receiving device | As Direct File Sharing describes |
| Password Management | Yes | The user types the master password on each device that joins | Once for each device; the vault is unlocked in each session |
| Clipboard Synchronisation | Yes | Each device; sending and receiving are turned on separately for each device | Once; secret items are never sent |
| Screen Sharing | Yes | The user on the shared device | Every time |
| Remote Control | Yes | The user on the controlled device, separately from viewing | Every time; "Stop" always works |
| Extra Screen | Yes | The user on both devices | Every time |

Changing a permission requires the device to be unlocked. Changing the permission to delete, the password settings or the permission to control a device also requires the system's fingerprint, face or PIN check.

## 4. Data stored on each device

**The three security rules of the system.** The main database is encrypted with a random key that is kept in the operating system's key store. The vault key is random and is stored only wrapped by a key made from the master password, so the master password can be changed. A device accepts a file list only if the owner device's signature is valid and its change number is higher than the one it already holds.

| Data | Where | Protection |
|---|---|---|
| Password vault | Every device that has joined the vault | SQLCipher with a random vault key, stored only wrapped by a key made from the master password with Argon2id (Password Management) |
| Main database: the device's own file list, the copies of the other devices' lists, the list of paired devices, settings, the list of ignored duplicate groups and the deletion log | Every device | SQLCipher with a random key kept in the operating system's key store. SQLCipher uses AES-256 and checks a message authentication code on every page, so any change to the file is detected. This protects a copied file and other user accounts; it does not protect against malware running as the user |
| Similarity data (vectors) | The AI device only | Kept in the main database, so it is encrypted in the same way |
| The device's private key | Every device | The operating system's key store: Keychain on Apple systems, Android Keystore, libsecret on Linux, and a protected file on Windows |
| Backups | Not applicable | The app's data is left out of cloud backups. On Android this is also necessary because the key in the key store cannot be restored from a backup |

**Never stored:** the files of other devices (except a file that the user received with Direct File Sharing); photo previews (deleted once the vector is made); clipboard items; screen images; the master password and the key made from it; and the plain vault key, except the optional fingerprint or face copy inside the operating system's key store.

## 5. A lost device

**The rule.** From any other device, the user chooses *Devices → Lost device → Remove*. Four things then happen.
1. The removal is recorded and passed on as Device Discovery, Pairing and Trust describes.
2. If the lost device had the AI role, the role moves to another device.
3. The lost device's file list is deleted by the other devices.
4. If the lost device had joined the vault, the app offers "Change the vault key", which is recommended. An unlocked device makes a new vault key, encrypts the vault again and sends the new key, wrapped with the master password, only to the remaining devices. Each remaining device asks for the master password once more, and its fingerprint or face copy of the key is replaced. The app shows a new recovery code, because the old code opens only the old key. This protects the entries added from then on.

The lost device keeps its old copy, protected only by the old master password, and nothing can erase that copy without a server. The app therefore advises the user to change the passwords of their important accounts.

**What the user sees:** "*Phone-1* was removed. Your devices that are on now will refuse it. Devices that are switched off will refuse it after they next meet one of your devices; until then they may still accept it. If you turn one of them on away from your other devices, remove *Phone-1* there too. The phone still holds the data it had: change the passwords of your important accounts."

**The remaining gap.** A device that was off still trusts the lost device until it hears of the removal. Device Discovery, Pairing and Trust exchanges removal lists at every connection before anything else. That design also proposes that, at start-up, a device waits up to 5 seconds to reach the user's other devices before it accepts connections, if any of them are known to be online. The gap remains only if the lost device is the first device it meets. Without a server this gap cannot be closed, and the message above tells the user what to do.

## 6. File channel

These answers settle the security questions of Direct File Sharing.

- **A separate channel, in chunks, that can resume: yes.** Files travel on their own file channel, apart from the control channel. The file is sent in chunks of 1 MiB, and the receiving device keeps its chunks when the connection drops. The file channel uses Dart's own TLS sockets, as the control channel does, so no extra library is needed. Reason: a large file must not hold up the control messages, and a dropped connection then costs at most one chunk.
- **A fingerprint for each chunk and for the whole file: both.** Each chunk carries its own SHA-256 fingerprint, and the finished file is checked with its full SHA-256 fingerprint. Reason: after a drop, the chunk fingerprints show which kept chunks are intact and belong to the same file, so only missing or damaged chunks are sent again; the full fingerprint confirms the finished file.
- **Encrypted like the control channel: yes.** The file channel uses TLS 1.3 with the same certificate check and signed challenge. It is opened only to a paired device for which Direct File Sharing is turned on. Reason: the contents of files are the most private data that the app moves.
- **The duplicate-detection fingerprint reused for "You already have this file": yes.** The receiving device compares the file's size and full SHA-256 fingerprint with its own file list. If its file of the same size has no full fingerprint yet, it computes one first. Reason: both features then use one fingerprint, and no file is read twice.

## 7. Privacy

Data leaves a device only to the user's own paired devices, and only for features that the user has turned on. This data is: file lists (names, sizes, dates and fingerprints, never the contents), photo previews for the AI device (not kept), files that the user sends, clipboard text that the user allows, and screen images during a session that the user has accepted. Nothing leaves the local network: there is no server, no cloud and no account. The app has no analytics, no tracking and no advertising, and it sends no usage data to the developers.

## 8. Requirements

- FR-SEC-1 The app must send feature data to a device only if the device is paired and that feature is turned on for it.
- FR-SEC-2 The app must keep every feature off by default on each device.
- FR-SEC-3 The app must ask the user on the target device, every time, before Screen Sharing or Remote Control starts, and the user on both devices, every time, before Extra Screen starts.
- FR-SEC-4 The app must let another device delete files only if the owner device has turned on "Allow my other devices to delete duplicates here" or the user confirms the request on the owner device.
- FR-SEC-5 The app must check a file again before deleting it and must record every deletion in a log kept for 30 days.
- FR-SEC-6 The app must encrypt the main database with a random key that is kept in the operating system's key store, and the vault as Password Management requires.
- FR-SEC-7 The app must never store the photo previews, clipboard items or screen images of other devices.
- FR-SEC-8 The app must accept a file list only if the owner device's signature is valid and its change number is higher than the one it already holds.
- FR-SEC-9 The app must let the user remove a lost device from any other device and must offer to change the vault key.
- FR-SEC-10 The app must tell the user, when a device is removed, that switched-off devices learn of the removal only at their next contact (section 5).
- FR-SEC-11 The app must ask for the system's fingerprint, face or PIN check before the permission to delete, the password settings or the permission to control a device is changed.
- FR-SEC-12 The app must send no data outside the local network and must collect no usage data.

## 9. Limitations

The following has not yet been verified: that the app's data can be left out of cloud backups on each system, and how much the encryption of the main database slows search and synchronisation. The evaluation plan measures the cost of the encryption on Windows, Android and Ubuntu 24.04. macOS, iPhone and iPad cannot be built or tested by the team, because no member has a Mac; their key store and backup behaviour are checked against Apple's documentation only.

## 10. Summary

Security and Privacy keeps the user in control. Every feature starts off, each device is allowed only what the user turns on, and actions that cannot be undone, such as deletion and remote control, ask again. Stored data is encrypted with keys held by the operating system or made from the master password, and photo previews, clipboard items and screen images are never stored. A lost device can be removed from any other device, and the app states plainly what the removal cannot undo.

## References (opened 8 and 13 November 2026)

1. Zetetic, SQLCipher design (AES-256 and a message authentication code on every page): https://www.zetetic.net/sqlcipher/design/
2. Team documents: Device Discovery, Pairing and Trust; Technology Choices (final); File Indexing and Offline-Device Records; Duplicate File Detection; Direct File Sharing; Password Management; Clipboard Synchronisation; Screen Sharing and Remote Control; Extra Screen; Implementation and Evaluation Plan.
