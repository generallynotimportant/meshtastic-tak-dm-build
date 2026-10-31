# Meshtastic TAK DM fix: build kit (v0.4)

**BLUF:** This repo builds patched Meshtastic apps, for Android phones (ATAK) and Windows (WinTAK), straight from Meshtastic's official source. With the patch, TAK direct messages go only to the person they're for, and mesh radios show on the map as blue friendly symbols instead of green team dots. You can also pick which radios appear on the map, and give any radio a breadcrumb trail of dots (e.g. a dog's tracker).

## What the patch changes
| Version | Change |
|---|---|
| v0.2 | A GeoChat DM is sent only to the recipient's radio when it's known, encrypted to that radio by firmware 2.5+. A DM that still arrives as a broadcast is dropped unless it's for this device's TAK user. All Chat is unchanged. |
| v0.4 | Adds a **Mesh to CoT radios** card (TAK Server settings, shown when Mesh to CoT is on): **Only show selected radios**, a **Choose radios** list with **Show** / **Trail** checkboxes, and trail settings (dot interval, keep time, colour). Hidden radios are removed from the map. Trail dots are recorded even while ATAK is disconnected and replayed when it reconnects; they never go over the radio. |
| v0.3 | Adds: mesh radio markers ("Mesh to CoT") no longer carry a TAK team, so ATAK/WinTAK draws the MIL-STD-2525 friendly symbol (blue rectangle) instead of a Green team dot. |

## Files
| File | What it is |
|---|---|
| `.github/workflows/build-apk.yml` | Builds the Android app (APK). |
| `.github/workflows/build-desktop-windows.yml` | Builds Meshtastic Desktop for Windows (MSI + EXE), used with WinTAK. |
| `meshtastic-tak-directed-dm-v2.8.1-v0.4.patch` | Current patch for app version 2.8.1 (the default). |
| `meshtastic-tak-directed-dm-v2.8.1-v0.3.patch` | DM fix + blue radio icons, without the radio picker or trails. Known good. |
| `meshtastic-tak-directed-dm-v2.8.1-v0.2.patch` | DM fix only. Known good. |
| `meshtastic-tak-directed-dm-v0.3.patch` | v0.3 for the newest app source (`main`). |
| `*-v0.1.patch` | Old; fails Meshtastic's iOS test compile. Kept for history. |
| `LOCAL-BUILD-NOTES.md` | Building on your own computer instead of GitHub. |

## Build
1. Go to **Actions**, click the workflow in the left list (**Build patched Meshtastic APK** or **Build patched Meshtastic Desktop (Windows)**), then **Run workflow**. Keep the defaults and click the green **Run workflow**.
2. When the run finishes, download the file under **Artifacts** at the bottom of the run page. GitHub keeps it for 14 days; rebuild any time.

## Install: Android phones (ATAK)
1. The APK zip holds three files. Install the one with **`universal`** in its name. Play Protect will scan it; if it warns, choose **More details → Install anyway**.
2. It installs alongside the official Meshtastic app. Disconnect the official app from the radio and turn off its TAK server. Then connect the radio in the patched app and turn on its TAK server.
3. In the patched app, export the TAK data package and import it into ATAK. Each install makes its own certificates, so an old package won't connect.
4. Each new build is signed with a different debug key, so uninstall the old patched app before installing a newer one.

## Install: Windows PC (WinTAK)
1. Install the `.msi` from the Windows build zip. It's unsigned, so if Windows warns: **More info → Run anyway**. It replaces any official Meshtastic Desktop install.
2. Plug in the PC's radio by USB, connect to it in Meshtastic Desktop, and turn on the TAK server.
3. Export the TAK data package and unzip it. WinTAK doesn't import this package directly, so set it up by hand:
   - In WinTAK, add a new server connection.
   - **Address:** `127.0.0.1`, **Port:** `8089`, **SSL/TLS**
   - Import `truststore.p12` (CA) and `client.p12` (client certificate), both with password `meshtastic`.
   - The connection indicator turns green when it's connected.

## Test (three stations, each with its own radio)
1. Wait until every station shows on every other station's map. That's how each app learns which radio everyone is on.
2. A DMs B: B gets it, C doesn't. C (WinTAK) DMs A: A gets it, B doesn't. All Chat still reaches everyone.
3. Mesh radios (with **Mesh to CoT** on) should show as blue friendly symbols, and TAK users as their team-colored dots.

## Breadcrumb trail (e.g. tracking a dog)
1. On the tracker radio, set its position broadcast interval to about the trail interval you want (Meshtastic app → the radio's **Position** settings). The trail can't be finer than how often the radio reports.
2. In the patched app: **TAK Server → Mesh to CoT** on → **Mesh to CoT radios → Choose radios**, then tick **Trail** for the tracker.
3. Dots appear only when the radio has moved and the interval has passed, labelled with its short name and the time. They expire after the keep time. Unticking **Trail** removes them.
4. Trail dots only exist for positions your device actually heard. Out of range means gaps.

## Don't press
**TAK Mesh Test (Debug)** in the app's TAK settings transmits 41 sample TAK messages over the mesh for real, including fake markers and emergency alerts. It only appears because these are debug builds.

## If a build fails
Click the step with the red ✗ and copy the last ~50 lines of its log, or use **⋯ → Download log archive**. If the failure was in **Run TAK module tests**, also download the `test-reports` artifact.

## Building a different version later
On **Run workflow**, change **Meshtastic-Android tag or branch** (for example `v2.8.2`) and **Patch file**. If the patch doesn't fit, the run stops at **Apply patch** and says so; the patch then needs porting to that version.
