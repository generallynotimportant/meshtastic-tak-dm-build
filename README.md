# Meshtastic TAK DM fix: GitHub build kit (v0.1)

**BLUF:** GitHub downloads the official Meshtastic Android source, applies the DM fix, builds the app, and hands you an APK to install on each phone. You don't need to install anything on your computer.

## What's in this repo
| File | What it is |
|---|---|
| `.github/workflows/build-apk.yml` | The build script. GitHub runs it when you click **Run workflow**. |
| `meshtastic-tak-directed-dm-v2.8.1-v0.1.patch` | The fix for app version 2.8.1 (the default). |
| `meshtastic-tak-directed-dm-v0.1.patch` | The same fix for the newest app source (`main`). |
| `LOCAL-BUILD-NOTES.md` | For later: building on your own computer instead of GitHub. |

## One-time setup (web browser only)
1. Sign in at github.com, or create a free account.
2. Click **+** (top right), then **New repository**.
   - Name: `meshtastic-tak-dm-build`
   - Visibility: **Public**. Public repos get the 16 GB build machines this app needs, with unlimited free use. Private repos get 8 GB and draw from a monthly minutes allowance. These files hold no passwords, keys or personal info.
   - Leave the "Add a README" box **unchecked**, then click **Create repository**.
3. Add the build script:
   - Click **creating a new file**, or **Add file → Create new file**.
   - In the name box, type exactly `.github/workflows/build-apk.yml`. Each `/` creates a folder; GitHub only runs build scripts that live in `.github/workflows`.
   - Paste the full contents of `build-apk.yml`, then click **Commit changes** twice.
4. Add the patches:
   - Click **Add file → Upload files**.
   - Drag in both `.patch` files and this `README.md`, then click **Commit changes**.

## Build the APK (repeat any time)
1. Open the **Actions** tab. If GitHub asks, click to enable workflows.
2. In the left list, click **Build patched Meshtastic APK**.
3. Click **Run workflow** on the right. Leave the defaults (`v2.8.1` and the 2.8.1 patch), then click the green **Run workflow** button.
4. Refresh the page and click the new run to watch it. Each step turns green as it finishes. The first run takes the longest; later runs reuse downloaded files.
5. When it finishes, scroll to **Artifacts** at the bottom of the run page and download `meshtastic-tak-dm-v2.8.1-fdroid-debug`. It's a zip with the `.apk` inside.

## Install and set up on each phone
1. Copy the `.apk` to the phone (USB cable, Google Drive, and so on) and tap it to install. Android will ask you to allow installs from that app (Files, Drive, etc.); allow it.
2. It installs **next to** the official Meshtastic app, with a separate app ID (`com.geeksville.mesh.fdroid.debug`). Disconnect the official app from the radio and turn off its TAK server. Then connect the radio in the patched app and turn on its TAK server. Both apps use port 8089, so only one can run the TAK server at a time.
3. In the patched app, **export a new TAK data package** and import it into ATAK. Each app install creates its own certificates, so your old zip won't connect.

## Test
Use three phones, all patched, all showing each other on the ATAK map. A DMs B: B should get it and C shouldn't.

## If a build fails
On the run page, click the step with the red ✗, copy the last ~50 lines of its log, and send them to me. If the failure was in **Run TAK module tests**, also download the `test-reports` artifact.

## Building a different version later
On **Run workflow**, change:
- **Meshtastic-Android tag or branch**: for example `v2.8.2`, or `main`.
- **Patch file**: the patch made for that version.

If the patch doesn't fit that version, the run stops at **Apply patch** and says so. Send me the version and I'll port the fix.
