# Pulling to iCloud / local storage via iOS Shortcuts

Apple has no public API for writing files to an iCloud destination from a server, so this
piece runs as an on-device automation instead. This shortcut matches the "pull, not
push" model of the rest of this pipeline. The credential lives only in the
Shortcut on your device, not in a cloud secret, making this easy for anyone to implement.

## Setup

1. Generate a fine-grained GitHub token (github.com/settings/tokens?type=beta)
   scoped to **only** your source resume repo, with **Contents: Read-only**
   permission.

2. In the Shortcuts app: **Automation** tab → **+** → **Personal
   Automation** → **Time of Day** (pick a daily time) → **Next**.

3. Add these actions in order:

   **Get Contents of URL**
   - URL: `https://api.github.com/repos/YOUR_USERNAME/YOUR_REPO/contents/main.pdf`
     (use `main.pdf` for the full/private version, or `main-public.pdf` for
     the redacted one, depending on which you want on this device)
   - Method: GET
   - Headers:
     - `Authorization`: `Bearer YOUR_TOKEN`
     - `Accept`: `application/vnd.github.raw`

   **Rename File**
   - Input: Contents of URL
   - New name: whatever filename you want, e.g. `Resume.pdf`

   **Save File**
   - Input: **the output of Rename File** — not "Contents of URL". Tap the
     input chip and explicitly pick the renamed file; this is a common
     mistake since Save File's default input suggestion often points past
     the Rename step.
   - Destination: iCloud Drive
   - Ask Where to Save: **Off**
   - Subpath: e.g. `Resume` (optional)
   - Overwrite if File Exists: **On**

4. **Turn off "Ask Before Running"** for the automation, so it runs silently
   in the background.

## Known iOS Shortcuts quirks

- The first run may prompt for permission to access files in the
  background (`com.apple.WorkflowKit.BackgroundShortcutRunner`) — this is
  normal, so you may allow it once.
- `Save File` can throw a **"No bookmark found for file"** error the first
  time it targets a new folder. If this happens, temporarily turn "Ask
  Where to Save" back on, run once manually and create/select the
  destination folder interactively (using the picker's own "New Folder"
  button if creating one), then switch "Ask Where to Save" back off. This
  establishes a working bookmark for subsequent silent runs.
- If it still fails, try dropping the Subpath and saving straight to the
  iCloud Drive root. That configuration is the most consistently reliable
  for background automations.
