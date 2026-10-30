# Meshtastic TAK directed-DM fix: v0.1 build notes

**BLUF:** Build the Meshtastic Android app with this patch and install it on each phone. DMs are then sent only to the recipient's radio. Radio firmware stays stock.

## Which patch
| File | Apply to |
|---|---|
| `meshtastic-tak-directed-dm-v2.8.1-v0.1.patch` | `v2.8.1` tag (matches your current app line) |
| `meshtastic-tak-directed-dm-v0.1.patch` | `main` (2.8.4 base, as of 2026-10-26) |

## Build (PC with JDK 25 + Android SDK / Android Studio, 16 GB+ RAM; not the Pi)
```bash
git clone https://github.com/meshtastic/Meshtastic-Android.git
cd Meshtastic-Android
git checkout v2.8.1
git apply /path/to/meshtastic-tak-directed-dm-v2.8.1-v0.1.patch
./gradlew :core:takserver:allTests          # runs the new TakDirectedRouterTest
./gradlew :androidApp:assembleFdroidDebug
# APK: androidApp/build/outputs/apk/fdroid/debug/
```

## Install notes
- A debug build installs **next to** the store app, as `com.geeksville.mesh.fdroid.debug`. Connect only one of the two apps to the radio, and enable the TAK server in only one, because both want port 8089.
- Each app install creates its own TAK certificates. **Export a new TAK data package from the patched app and import it into ATAK**, or ATAK won't connect.

## What changed
- **Send:** a DM goes to the recipient's node when the app has heard that person's PLI or chat over the mesh in the last 30 minutes. Otherwise it's broadcast as before. Packets over 210 bytes also stay broadcast, because firmware PKI encryption adds 12 bytes and the radio rejects oversized packets.
- **Receive:** a DM addressed to someone else is dropped before it reaches ATAK. All Chat, PLI, markers, and unknown rooms are unchanged.
- Only the sender needs the patch for the unicast path. Receivers on the stock app are protected too, because the radio never hands them the packet.

## Not covered
- Legacy v1 (firmware 2.7.x, port 72) traffic is unchanged.
- Nobody on the mesh has heard the recipient yet: the DM is still broadcast to all, as today.
