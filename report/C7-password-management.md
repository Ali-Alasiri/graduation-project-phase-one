# C7: Password management (first version)

Owner: Ali. Status: first version, 8 Nov 2026. Phase 1 is design only: no code. The professor agreed that C7 stays in the report as a design. Names follow `B1-architecture-v2.md`; technology from `B6-technology-final.md`; trust from C1 v2 (draft). **[PROPOSAL]** = a design choice or number to settle in Phase 2. **[NOT CHECKED]** = not confirmed in a primary source or by a test.

## 1. How it works in one paragraph
Each device that the user turns the feature on for keeps the same password vault, encrypted, inside the device app. The vault is opened with one master password (or a fingerprint or face on devices that have it). Changes travel only between trusted devices on the local network, already encrypted with the vault key, so no server and no internet are involved. The user copies a password, or, on Android, iPhone/iPad and Mac, lets the system's AutoFill fill it in other apps.

## 2. What is stored, and where the accounts come from
- **Stored [PROPOSAL]:** logins (name, website or app, user name, password, note) and secure notes. Not in v1: payment cards, passkeys, one-time codes.
- **Typed in** by the user, with a password generator.
- **Imported from a file (recommended):** a CSV file exported by a browser or another password manager. Chrome exports and imports passwords as `.csv` with the columns `url`, `username`, `password`, and warns that anyone who uses the device can open such a file [6]. So after import the app offers to delete the file.
- **Read from browsers directly: not recommended.** On Windows, Chrome 127 started "App-Bound Encryption", which ties stored secrets to Chrome itself, so decryption by another app fails; it started with cookies and Google plans it for passwords [7]. On phones the browsers' stores are inside their own app sandbox [NOT CHECKED]. Reading them would also need code per browser and per system.

## 3. The vault
- **File:** a second SQLite file encrypted with SQLCipher (B6 choice 5).
- **Keys [PROPOSAL]:** at first setup the app creates a random 256-bit **vault key** and gives it to SQLCipher as a raw key, which makes SQLCipher skip its own key derivation [3]. The master password is turned into a **wrapping key** with **Argon2id, 64 MiB of memory, 3 passes, 1 thread, 16-byte random salt**. This is above OWASP's minimum (19 MiB, 2 passes, 1 thread [1]); Bitwarden's default is 32 MiB, 6 passes, 4 threads [2]. The wrapping key encrypts the vault key (AES-256-GCM). Each device stores only the **wrapped** vault key, the salt and the settings. The master password, the wrapping key and the plain vault key are never stored, as B6 requires. Changing the master password only re-wraps the vault key; the vault is not re-encrypted. Argon2id time on our phones (target: under 1 s) and the Dart package for it: [NOT CHECKED].
- **Locks [PROPOSAL]:** when the app closes, when the screen locks, after 5 minutes without use (desktop setting up to 30), and by a "Lock" button. Locking removes the vault key from memory; whether Dart really clears memory is [NOT CHECKED].
- **Fingerprint or face (optional, per device):** the vault key is also kept in the system key store behind biometrics. On iPhone/iPad/Mac, a Keychain item with `biometryCurrentSet`, which Apple invalidates when fingers are added or removed or Face ID is set up again [4]. On Android, a Keystore key that needs strong biometrics (strong-only is enforced from Android 11, per `flutter_secure_storage` [5]). Windows Hello and Linux: master password only in v1 [NOT CHECKED]. The master password is asked again after a restart and every 14 days [PROPOSAL].
- **Forgotten master password:** there is no server and no reset. (1) If any device still opens the vault with a fingerprint or face, the user sets a new master password there, and the newly wrapped key goes to the other devices. (2) At setup the app shows a **recovery code** (random, 128 bits) to print or write down; it also wraps the vault key [PROPOSAL]. (3) Without either, the vault is lost. The setup screen says this plainly.

## 4. Syncing between devices
- **Off until the user turns it on** on each device (B1 rule). Only devices that are trusted (C1) **and** where the user has typed the master password once take part: a new device receives the wrapped vault key and the salt from a trusted device and opens it with the master password. A device that is trusted for other features but never joined the vault can read nothing.
- **What travels:** each changed entry, encrypted with the vault key (AES-256-GCM), inside the trusted TLS connection [PROPOSAL]. So a device that only relays (C3's "passing copies on") cannot read it, and the vault never leaves a device unencrypted (B1 decision 11).
- **Each entry carries** a random id, a version number, the time and device of the last change; a deleted entry becomes a tombstone kept 90 days [PROPOSAL].
- **One rule for two edits of the same entry [PROPOSAL]:** if both devices changed the same entry since they last synced, the later edit (by its time) becomes the current entry and the other is kept in the entry's **history**, with a notice "Edited on two devices; previous version saved". Nothing is lost. (Clocks that differ between devices can pick the "wrong" winner, but the other version is still in the history.)
- **A device that was switched off:** it asks for the changes after the last version it has (as C3 does for file lists). Phones and tablets sync only while the app is open (B6 choice 6). A device that is locked stores received changes still encrypted and applies them when the user unlocks.

## 5. Using passwords in other apps and sites (autofill)

| System | What an app can do | v1 plan |
|---|---|---|
| Android 8+ | Be an **autofill service**: the user picks it in system settings, then it fills logins in other apps; Chrome uses it only when the user chooses "Autofill using another service" in Chrome [8] | Autofill service (native plugin) |
| iPhone/iPad 12+, macOS 11+ | A **Credential Provider extension** for AutoFill (passwords, passkeys, one-time codes); needs Apple's AutoFill entitlement [9] | Extension, native code [NOT CHECKED with Flutter] |
| Windows | Since Nov 2025, third-party managers can plug in for **passkeys**; for passwords in any app nothing like this was found [10] [NOT CHECKED] | Copy and auto-clear |
| Linux | No system autofill API found [NOT CHECKED] | Copy and auto-clear |

**Copy:** the copied password is marked secret with the formats C8 lists, so C8 never syncs it, and the app clears the clipboard after 30 s [PROPOSAL]. A browser extension is not in v1.

## 6. Risks and protection

| Risk | Protection | What remains |
|---|---|---|
| Lost device, locked | Vault encrypted; vault key stored only wrapped (Argon2id); remove the device (C1, C11) | Offline guessing of a weak master password |
| Stolen device, unlocked | Auto-lock after 5 min and on screen lock; showing or copying a password asks for fingerprint/face again [PROPOSAL] | What is on screen before the lock |
| Trusted device that misbehaves | Only devices that joined the vault can read; every change keeps a history; tombstones are visible | A device that joined can read every password; remove it and change important passwords |
| Malware on a device | Vault locked most of the time; key store with biometrics on phones | Malware running as the user can read memory while the vault is open, or record the master password; on Windows the key store only protects from other users (B6 point 6) |
| Copied vault file | Useless without the master password or recovery code | Offline guessing (slowed by Argon2id) |
| Weak master password | At least 12 characters and a strength meter [PROPOSAL]; Argon2id | A guessable password stays guessable |
| Password leak through the clipboard | Secret mark, cleared after 30 s, never synced (C8) | Apps on Windows/Linux that read the clipboard before it is cleared |

## 7. Compared with Bitwarden (A5)
Like Bitwarden, the vault is encrypted on the device with a key that comes from the master password (Bitwarden: Argon2id or PBKDF2 [2]), and the master password is never sent. Unlike Bitwarden (A5 [5]), there is **no server**: devices sync directly on the local network, so nothing goes to a cloud. **What we give up:** no web vault and no browser extension, no account recovery or emergency access by a server, no sharing with other people, and a new device can join only while a trusted device that has the vault is online.

## 8. Step-by-step: add a login on the phone, use it on the laptop
Editable source: `diagrams/C7-vault-sequence.drawio`; image: `diagrams/C7-vault-sequence.png`.

![C7 sequence diagram: add a login on the phone, use it on the laptop](../diagrams/C7-vault-sequence.png)

## 9. Requirements (for A8 and A9)
- FR-C7-1 The app must keep the vault in a separate SQLCipher file whose key is random and stored only wrapped by a key derived from the master password with Argon2id.
- FR-C7-2 The app must never store or send the master password, the wrapping key or the plain vault key.
- FR-C7-3 The app must lock the vault on app close, on screen lock and after 5 minutes without use.
- FR-C7-4 The app must allow fingerprint or face unlock only through the system key store, and ask for the master password after a restart.
- FR-C7-5 The app must import logins from a CSV file and offer to delete the file afterwards.
- FR-C7-6 The app must sync the vault only with trusted devices that joined the vault, and only after the user turned the feature on.
- FR-C7-7 The app must send vault changes only encrypted with the vault key, inside the trusted connection.
- FR-C7-8 The app must keep both versions when the same entry was changed on two devices.
- FR-C7-9 The app must mark copied passwords as secret, never sync them through C8, and clear the clipboard after 30 seconds.
- FR-C7-10 The app must show the recovery code once at setup and say that a forgotten master password cannot be reset.
- NFR-C7-1 Opening the vault with the master password must take at most about 1 second on the minimum phone [to test].
- NFR-C7-2 No vault data may leave the local network.

## Open points and who settles them
Removing a device: change the vault key or not (Ali, C11); the per-feature permission for passwords (C11); the vault screens (D1, Mahmoud). **Phase 2 tests:** Argon2id time on phones and the Dart package; memory clearing in Dart; AutoFill extensions with Flutter; Windows Hello; whether phones' browser stores can be read.

## Sources (opened 8 Nov 2026)
1. OWASP, Password Storage Cheat Sheet (Argon2id minimum configurations): https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
2. Bitwarden, KDF algorithms (PBKDF2 600,000; Argon2id 32 MiB, 6 iterations, 4 threads): https://bitwarden.com/help/kdf-algorithms/
3. Zetetic, "Using random values as SQLCipher keys" (raw key bypasses the KDF): https://www.zetetic.net/blog/2019/06/07/technical-guidance-using-random-values-as-sqlcipher-keys/
4. Apple, `SecAccessControlCreateFlags.biometryCurrentSet`: https://developer.apple.com/documentation/security/secaccesscontrolcreateflags/biometrycurrentset
5. Team files: `B6-technology-final.md` (choices 5 and 6, point 6) and `B6-technology-v0.md` source 29 (`flutter_secure_storage`); `C8-clipboard-sync.md` section 4; `C3-file-indexing.md` section 2; `research/A5-existing-systems.md` section 4 (Bitwarden).
6. Google, "Import or export passwords with Chrome": https://support.google.com/chrome/answer/13068232
7. Google Security Blog, "Improving the security of Chrome cookies on Windows" (30 Jul 2024): https://security.googleblog.com/2024/07/improving-security-of-chrome-cookies-on.html
8. Android, "Build autofill services": https://developer.android.com/identity/autofill/autofill-services
9. Apple, `ASCredentialProviderViewController`: https://developer.apple.com/documentation/authenticationservices/ascredentialproviderviewcontroller
10. CyberInsider, "Windows 11 integrates native support for Bitwarden and 1Password" (12 Nov 2025): https://cyberinsider.com/windows-11-integrates-native-support-for-bitwarden-and-1password/
