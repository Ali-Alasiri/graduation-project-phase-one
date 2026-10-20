# C2: The AI device and the file lists (first version)

Owner: Mohammed. Status: first version, answers open decisions 7 and 8 in `B1-architecture-v1.md`.
Phase 1 is design only: no code.

## Q1: How is the AI device chosen, and what happens when it is off or replaced?

- **Recommendation: the app suggests, the user decides.** Each device advertises its capabilities (rule 5 in B1). The app ranks the devices that can host the AI role (laptop or PC only, never a phone or tablet) by memory, graphics card, and whether it is usually plugged in and switched on. It suggests the best one, and the user confirms or picks another.
- **Why not fully automatic:** the strongest device is not always the one that stays on (a gaming PC may be off most of the day). The user knows this; the app does not.
- **Only one AI device at a time.** The choice is saved on every device, so all of them know where to send AI requests.
- **When the AI device is off:** hash duplicates, file sharing, clipboard and local search keep working. Similar-file detection and search by meaning show "AI paused: *Laptop-1* is off", with the last results and their date. If another capable device is online, the assistant offers "Move the AI role to *PC-2*?" (it asks, it does not switch by itself).
- **When the AI device is replaced:** the user picks the new device in settings. The new device rebuilds the similarity data from the file lists it already has (see Q2), and the old device deletes its copy. No similarity data is copied between devices. Rebuild time for 100,000 files: not checked (depends on the model chosen in B6).

## Q2: Does every device keep a copy of the other devices' file lists?

- **Recommendation: yes, every device keeps a copy** (this matches the B1 assumption). It is the only way to see a switched-off device's files from anywhere, and it lets a new AI device start without waiting for the others to come online.
- **What is stored per file:** device name, path and file name, size, last modified date, file type, and a fingerprint (hash). No file contents, no thumbnails, and no AI similarity data (that stays on the AI device only).
- **How big it can get:** about 200 bytes per file, so roughly 20 MB for 100,000 files, maybe up to 40 MB with database overhead (my estimate, not checked). For four devices with 100,000 files each, a phone stores about 80 to 160 MB, which is acceptable.
- **How fresh it is:** when two devices connect, they exchange only the changes since the last sync, not the whole list. While both are online, changes are sent as they happen. Every list shows "last updated" with its date, so the user knows when it may be old.
- **Limit:** phones and tablets may only receive updates while the app is open (decision 12, not checked), so their copies can be older than the laptop's.

## Still open
- Minimum hardware for the AI role, and rebuild time: waits for B6 (Ali), decision 4.
- Size estimate: to be checked with C3 and C5 once the storage technology (decision 5) is chosen.
