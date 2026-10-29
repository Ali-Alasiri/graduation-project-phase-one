# A5 – Existing Systems Study

This document studies six existing applications that solve parts of the cross-device problem.  
The goal is to show what already exists and clearly identify the gaps that our project aims to fill.

---

## 1. KDE Connect

KDE Connect links a phone and a computer on the same local network so the user can share clipboard text, send files, receive notifications, control media, and use the phone as a remote control.  
Devices find each other with UDP broadcast and mDNS, then the user confirms pairing on both sides and the connection is encrypted with TLS.  
It does **not** detect duplicate files across devices, provide unified search, manage passwords, keep a useful offline-device history, or offer any local AI features (not listed in the official feature pages).  
It runs on Linux, Android, Windows, macOS and iOS (limited features on iOS).  
It does not need internet.

**Sources:**  
- https://userbase.kde.org/KDEConnect  
- https://invent.kde.org/network/kdeconnect-meta/-/raw/master/protocol.md  
- https://kdeconnect.kde.org/

---

## 2. Syncthing

Syncthing continuously synchronises selected folders between devices in a pure peer-to-peer way without any cloud.  
Each device has a unique Device ID (from its certificate). Devices discover each other by local broadcast/multicast and optionally through global discovery or relays; only approved devices can connect and all traffic is TLS-encrypted.  
It does **not** offer one-time file sending, clipboard sharing, notification mirroring, remote control, duplicate-file detection beyond normal sync, unified search or password management (not listed in official documentation).  
It runs on Windows, macOS, Linux and FreeBSD/OpenBSD. Android support is provided by the community app **Syncthing-Fork**. iOS support is provided by the third-party app **Möbius Sync**.  
Internet is optional (works fully offline on a local network).

**Sources:**  
- https://docs.syncthing.net/users/security.html  
- https://docs.syncthing.net/dev/device-ids.html  
- https://syncthing.net/  
- Android: https://github.com/researchxxl/syncthing-android  
- iOS: https://mobiussync.com/

---

## 3. LocalSend

LocalSend lets the user send files and short messages to nearby devices over the local network with no account or internet required.  
Devices discover each other mainly by multicast UDP (port 53317); if that fails they fall back to HTTP discovery. Transfers are encrypted with HTTPS.  
It does **not** keep folders in continuous sync, share the clipboard, show notifications, provide remote control, detect duplicates, offer unified search or manage passwords (not listed in the official README).  
It runs on Windows, macOS, Linux, Android, iOS and Fire OS.  
It does not need internet.

**Sources:**  
- https://github.com/localsend/localsend/blob/main/README.md  
- https://github.com/localsend/protocol/blob/main/README.md

---

## 4. Bitwarden

Bitwarden is an open-source password manager that stores logins, notes, cards and other secrets in an end-to-end encrypted vault.  
The vault is encrypted on the user’s device before it is sent to Bitwarden’s cloud (or a self-hosted server). Devices sync the encrypted vault after the user logs in with the master password.  
It does **not** detect duplicate files, provide unified search across device files, share the clipboard beyond password autofill, offer screen sharing, or act as a general cross-device file assistant.  
It runs on Windows, macOS, Linux, iOS, Android and all major browsers.  
It normally needs internet for synchronisation (self-hosting is possible).

**Sources:**  
- https://bitwarden.com/  
- https://bitwarden.com/help/data-storage/

---

## 5. Spacedesk

Spacedesk turns another device into an extra monitor for a Windows PC. The primary machine runs the Spacedesk driver; the secondary machine runs a viewer app.  
Devices must be on the same local network (Wi-Fi, Ethernet or USB). The viewer discovers the primary machine automatically and connects over TCP/IP. No account is required.  
It does **not** manage files, detect duplicates, provide unified search, handle passwords, or offer any AI features.  
Primary machine: Windows only. Secondary machines: Windows, Android, iOS and any device with an HTML5 browser.  
It does not need internet.

**Sources:**  
- https://www.spacedesk.net/  
- https://manual.spacedesk.net/

---

## 6. Apple Continuity

Apple Continuity is a set of built-in features that let Apple devices work together: Handoff (continue a task on another device), Universal Clipboard, AirDrop, Sidecar (use iPad as a second display), and more.  
Devices must be signed in with the same Apple ID and have Wi-Fi and Bluetooth enabled. Discovery and connection happen automatically when the devices are near each other.  
It does **not** work with non-Apple devices, does not detect duplicate files across devices, does not provide a unified search of all local files, and does not offer a general local AI assistant.  
It runs only on Apple devices (Mac, iPhone, iPad, Apple Watch, etc.).  
It needs Wi-Fi and Bluetooth; some features also use iCloud.

**Sources:**  
- https://support.apple.com/guide/mac-help/work-across-devices-using-continuity-mchl1d734309/mac  
- https://www.apple.com/macos/continuity/

---

## Comparison Table

| App            | What it does                                      | How devices connect                          | File transfer / main feature          | Needs internet | Systems                                      | What is missing compared to our project                  |
|----------------|---------------------------------------------------|----------------------------------------------|---------------------------------------|----------------|----------------------------------------------|----------------------------------------------------------|
| KDE Connect    | Phone–PC integration (notifications, clipboard, remote control) | UDP + mDNS → user pairing + TLS             | One-time share + remote control       | No             | Linux, Android, Windows, macOS, iOS (limited)| No duplicate detection, no unified search, no password vault, no local AI |
| Syncthing      | Continuous folder synchronisation                 | Device ID + local discovery (+ optional global) | Continuous bidirectional sync        | Optional       | Windows, macOS, Linux, *BSD + community Android/iOS | No one-time send UI, no clipboard, no search, no AI, no offline history view |
| LocalSend      | Quick file & message transfer                     | Multicast UDP + HTTP fallback                | One-time encrypted transfer           | No             | Windows, macOS, Linux, Android, iOS, Fire OS | No continuous sync, no search, no duplicates, no passwords, no AI |
| Bitwarden      | End-to-end encrypted password manager             | Account login → cloud or self-hosted sync    | Vault synchronisation                 | Yes (normally) | Windows, macOS, Linux, iOS, Android, browsers| No file management, no duplicate detection, no screen sharing, no general AI |
| Spacedesk      | Use another device as an extra Windows monitor    | Local network discovery (LAN/Wi-Fi/USB)      | Screen extension                      | No             | Primary: Windows; Secondary: Windows/Android/iOS/HTML5 | No file features, no search, no passwords, no AI, Windows-centric |
| Apple Continuity | Seamless features between Apple devices (Handoff, Clipboard, AirDrop, Sidecar) | Same Apple ID + Wi-Fi + Bluetooth           | Task handoff, clipboard, file share, second display | Wi-Fi/Bluetooth | Apple devices only                          | Apple-only, no cross-platform support, no duplicate detection, no local AI |

**Sources opened on 29 Oct 2026**