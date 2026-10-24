# PeerLink — serverless messenger, calls & file transfer for Android

PeerLink is an Android app (Capacitor + vanilla JS) built on **PeerJS / WebRTC**.
Messages, voice/video calls and files travel **directly between devices**; the PeerJS
signaling server is only used to introduce two peers.

## Features

**Messaging** — 1:1 chats, mesh group chats (admin, rename, add/remove, leave), delivery + read receipts,
typing indicators, replies, reactions, edit, delete for me / everyone, forward, search, link detection,
disappearing messages (5 min – 7 days), drafts, pinned & muted chats, message requests from unknown people,
block list, offline **outbox** (queued messages are delivered the next time both peers are online).

**Files** — any file up to 4 GB, photos (thumbnails), videos, **voice notes** with waveform + speed control.
64 KB chunks with CRC32 per chunk, sliding-window flow control + back-pressure, pause / **auto-resume from the
last stored 1 MB block**, accept / decline, auto-accept rules, progress / speed / ETA, save to
`Documents/PeerLink` or share sheet.

**Calls** — 1:1 voice and video over PeerJS media connections, mute, camera on/off, flip camera, audio-only answer,
ringtone, call history in chat, unknown callers rejected by default.

**Security** — every link runs a signed challenge-response (ECDSA P-256, non-extractable key). Contacts pin
each other's key on first use; a changed key shows a warning, and a **safety number** can be compared in person.
WebRTC (DTLS) encrypts all traffic. Optional PIN app lock. Relay-only mode hides your IP.

**Connectivity** — PeerJS Cloud by default; custom PeerJS server, custom STUN/TURN, auto-reconnect with back-off,
duplicate-connection resolution, RTT / route (direct vs relayed) display, diagnostics screen with logs.

**UX** — Material-3 inspired light/dark theme with accent colours, QR / `peerlink://` invites, edge-to-edge,
Android back-button handling, backup export / import.

## Get an APK with GitHub Actions (no local tooling)

1. Create a new GitHub repository.
2. Upload **both** of these to the repository (drag & drop on the web UI works):
   * `PeerLink-Android.zip` (do **not** unzip it), and
   * `.github/workflows/build-apk.yml` (the same file is also inside the zip; if you upload the *unzipped* project,
     the workflow is already in place).
3. Open the **Actions** tab → *Build Android APK* (it starts automatically on push; or press *Run workflow*).
4. When it finishes (~6–10 min) download **PeerLink-APK** from the run's *Artifacts*:
   `PeerLink-<version>-release.apk` (signed) and `…-debug.apk`. Copy to your phone and install
   (allow "install unknown apps").
5. To publish a Release with the APKs: push a tag like `v1.0.0`, or run the workflow manually with *release* ticked.

**Stable signing (recommended):** add repository secrets `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`,
`ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`. Without them a throw-away key is generated per run, so a newer build
cannot be installed over an older one (uninstall first).

## Local development

```bash
npm install
npm run dev            # browser preview (open two profiles: /?p=a and /?p=b)
npm test               # logic tests
npm run build
npx cap add android    # first time
npm run android:sync   # build + sync + patch manifest
npm run android:open   # Android Studio
```
Open `/?p=name&debug` to run several identities in one browser and expose `window.__pl` for debugging.

### Self-hosting the signaling server
`npx peerjs --port 9000 --path /` → Settings → Network → Custom server (TLS is required for internet hosts because the
app is served from `https://localhost`). For calls and transfers across strict NATs, add a TURN server (coturn).

## Project layout
```
src/net.js        PeerJS lifecycle, signed handshake, links, back-pressure, stats
src/chat.js       contacts, DMs, groups, receipts, reactions, outbox, TTL
src/transfer.js   file transfer engine (chunking, resume, integrity, save/share)
src/calls.js      voice/video calls          src/crypto.js   keys, safety number, PIN
src/db.js         IndexedDB                  src/ui/*        screens
scripts/patch-android.mjs   manifest permissions, deep links, versioning
test/             mock PeerJS + Playwright end-to-end harness (see test/e2e.mjs)
```

## Honest limitations
* **No push notifications** (no server): messages arrive while the app is running; the OS may suspend a backgrounded
  WebView after a while. Offline messages are delivered when both peers are next online.
* Group chats are full-mesh (fine for small groups); group calls and screen sharing are not included.
* Android WebView needs a current *Android System WebView* (Chrome 80+).
* PeerJS Cloud is a free community service; host your own for reliability.
