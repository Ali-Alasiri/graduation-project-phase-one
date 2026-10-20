# A5 – Existing Systems (Part 1): KDE Connect, Syncthing, LocalSend

## 1. KDE Connect

KDE Connect links a phone and a computer on the same local network so the user can share clipboard text, send files, receive notifications, control media, and use the phone as a remote control.  
Devices find each other with UDP broadcast and mDNS, then the user confirms pairing on both sides and the connection is encrypted with TLS.  
It does **not** detect duplicate files across devices, provide unified search, manage passwords, keep a useful offline-device history, or offer any local AI features.  
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
It does **not** offer one-time file sending, clipboard sharing, notification mirroring, remote control, duplicate-file detection, unified search or password management.  
It runs on Windows, macOS, Linux, FreeBSD, OpenBSD and community clients for Android and iOS.  
Internet is optional (works fully offline on a local network).

**Sources:**  
- https://docs.syncthing.net/users/security.html  
- https://docs.syncthing.net/dev/device-ids.html  
- https://syncthing.net/

---

## 3. LocalSend

LocalSend lets the user send files and short messages to nearby devices over the local network with no account or internet required.  
Devices discover each other mainly by multicast UDP (port 53317); if that fails they fall back to HTTP discovery. Transfers are encrypted with HTTPS.  
It does **not** keep folders in continuous sync, share the clipboard, show notifications, provide remote control, detect duplicates, offer unified search or manage passwords.  
It runs on Windows, macOS, Linux, Android, iOS and Fire OS.  
It does not need internet.

**Sources:**  
- https://github.com/localsend/localsend/blob/main/README.md  
- https://github.com/localsend/protocol/blob/main/README.md

---


## Comparison Table

| App         | Main purpose                     | Connection method             | File transfer             | Internet | Platforms                     |
|-------------|----------------------------------|-------------------------------|---------------------------|----------|-------------------------------|
| KDE Connect | Phone ↔ PC integration           | UDP + mDNS + TLS pairing      | One-time share            | No       | Linux, Android, Windows, macOS, iOS |
| Syncthing   | Continuous folder sync           | Device ID + local discovery   | Continuous sync           | Optional | Windows, macOS, Linux, Android/iOS |
| LocalSend   | Quick file & message transfer    | Multicast UDP + HTTP          | One-time transfer         | No       | Windows, macOS, Linux, Android, iOS |