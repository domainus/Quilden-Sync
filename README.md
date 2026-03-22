# Quilden Sync

**Quilden Sync** is a free Obsidian plugin that syncs your vault to a GitHub repository with optional zero-knowledge end-to-end encryption. Combined with [Quilden](https://quilden.com) — a web-based Obsidian-style editor — your notes become accessible from any device or network, without installing anything.

Requires Obsidian `1.11.4` or newer.

---

> **IT IS HIGHLY RECOMMENDED TO BACKUP YOUR VAULT BEFORE USING THIS PLUGIN FOR THE FIRST TIME.**

## Features

- **Automatic GitHub sync** — push and pull your vault on save, on a schedule, or manually
- **End-to-end encryption** — AES-256-GCM with PBKDF2 key derivation (600,000 iterations). Your password never leaves your device; GitHub only ever sees ciphertext
- **Conflict resolution** — choose between local-wins, remote-wins, or newer-wins strategies
- **Sync history & restore** — a full timestamped log of every sync event. Browse per-file commit history and restore to any previous version from inside Obsidian
- **Selective encryption** — encrypt only Markdown files, media files, or everything
- **Cross-platform** — works on iOS, iPadOS, Android, macOS, Windows, and Linux (wherever Obsidian runs)
- **No third-party services** — sync goes directly to GitHub. No subscription, no intermediary cloud

---

## Installation via BRAT (Recommended)

[BRAT](https://github.com/TfTHacker/obsidian42-brat) (Beta Reviewers Auto-update Tool) lets you install community plugins not yet in the official directory.

### Step 1 — Install BRAT

1. Open Obsidian → **Settings** → **Community plugins**
2. Click **Browse**, search for **BRAT**, and install it
3. Enable BRAT

### Step 2 — Add Quilden Sync via BRAT

1. Open **Settings** → **BRAT**
2. Click **Add Beta plugin**
3. Paste the repository URL:
   ```
   https://github.com/AA1labs/quilden-sync
   ```
4. Click **Add Plugin**
5. BRAT installs the plugin and keeps it updated automatically

### Step 3 — Enable the plugin

1. Go to **Settings** → **Community plugins**
2. Toggle **Quilden Sync** on

---

## Manual Installation

1. Download the latest release from the [Releases](https://github.com/AA1labs/quilden-sync/releases) page
2. Extract `main.js`, `manifest.json`, and `styles.css` (if present) into your vault at:
   ```
   <vault>/.obsidian/plugins/quilden-sync/
   ```
3. Restart Obsidian and enable the plugin under **Settings → Community plugins**

---

## Configuration

Open **Settings → Quilden Sync** after enabling the plugin.

Authenticate with either:

1. **Connect with GitHub** to sign in through Quilden and store the returned token in Obsidian's built-in secrets vault.
2. **GitHub Token** to select a personal access token from Obsidian's built-in secrets vault.

| Setting | Description |
|---------|-------------|
| **GitHub Token** | Select a GitHub token from Obsidian's secrets vault. Fine-grained tokens work as long as they can read your account and write to the selected repository. |
| **Repository owner** | Your GitHub username or organisation name |
| **Repository name** | The repo to sync your vault into |
| **Branch** | Branch to sync against (default: `main`) |
| **Sync on save** | Push changes automatically whenever you save a file |
| **Auto-sync interval** | Sync every N minutes in the background (0 = disabled) |
| **Sync on startup** | Pull latest changes when Obsidian starts |
| **Conflict strategy** | `local` / `remote` / `newer` — how to resolve conflicts |
| **Exclude patterns** | Glob patterns to skip (`.obsidian/`, `.trash/`, `.DS_Store` excluded by default) |
| **Notification location** | `notice` (popup) / `statusbar` / `none` |

If you do not already have a secret saved in Obsidian, click the token selector in plugin settings and use the built-in secret picker to add one.

---

## End-to-End Encryption

Quilden Sync uses the Web Crypto API — the same API built into every modern browser and Obsidian's Electron environment.

### How it works

1. You enter a password in the plugin settings
2. A key is derived using **PBKDF2** (SHA-256, 600,000 iterations) with a salt computed from `SHA-256("quilden:{github_login}/{owner}/{repo}")` — deterministic, nothing stored in the repo
3. Every file is encrypted with **AES-256-GCM** before being committed to GitHub
4. Encrypted files are prefixed with `QENC:1:` so both the plugin and the Quilden web editor can identify and decrypt them
5. When you open your vault in [Quilden](https://quilden.com), enter the same password — decryption happens entirely in your browser

### Encryption scope

Choose what gets encrypted:

| Scope | What's encrypted |
|-------|-----------------|
| `markdown` (default) | `.md` files only |
| `media` | Markdown + images and PDFs |
| `all` | Every file in the vault |

### Security properties

- Password is never transmitted or stored anywhere outside your device
- Salt is deterministic — no need to share or back it up
- GitHub, Quilden's servers, and any third party see only random-looking base64 ciphertext
- A small verification token (`QENC:1:...` encoding of a known string) is stored in plugin data to detect wrong passwords on re-entry without exposing plaintext

---

## Sync History & File Restore

Every sync is logged locally with:

- ISO timestamp
- Number of files pushed and pulled
- List of changed file paths

Access the history from the plugin's ribbon icon or command palette. To restore a file:

1. Open **Quilden Sync → Sync History**
2. Find the entry you want to restore to
3. Click **Restore** next to the entry or the specific file

Restores use GitHub's commit history, so no data is ever permanently lost.

---

## Connecting to Quilden (Web Editor)

[Quilden](https://quilden.com) is a web-based Obsidian-style editor that reads your GitHub-backed vault directly from the browser.

1. Install Quilden Sync and complete at least one sync
2. Visit [quilden.com](https://quilden.com) and sign in with GitHub
3. Select the repository your vault syncs to
4. If encryption is enabled, enter your password — decryption is local, nothing is sent to Quilden's servers
5. Edit your notes from any browser — work computer, school Chromebook, or borrowed device

---

## Commands

| Command | Description |
|---------|-------------|
| `Quilden Sync: Sync now` | Manual full sync (push + pull) |
| `Quilden Sync: Push changes` | Push local changes only |
| `Quilden Sync: Pull changes` | Pull remote changes only |
| `Quilden Sync: Open sync history` | Browse the sync log |
| `Quilden Sync: Open file history` | View commit history for the active file |

---

## Troubleshooting

**Sync fails with authentication error**
- Re-select your GitHub secret in the plugin settings if the stored secret was removed or renamed
- Regenerate your GitHub token and ensure it can write to the selected repository
- Tokens from the plugin settings page have the correct scope automatically

**Files not syncing**
- Check your exclude patterns — `.obsidian/` is excluded by default
- Verify the repository owner, name, and branch are correct

**Wrong password / decryption fails**
- Re-enter your password in **Settings → Quilden Sync → Encryption**
- The verification token will confirm whether it matches

**Conflicts**
- Adjust the conflict strategy in settings to match your workflow
- Conflict copies are saved alongside the original with `_conflict` in the filename

---

## Privacy

- No vault content is transmitted to Quilden's servers during sync — all traffic goes directly to GitHub's API
- With encryption enabled, Quilden's web editor decrypts in your browser; plaintext never leaves your device
- The plugin stores only sync state and settings in Obsidian's local plugin data folder

---

## License

MIT — see [LICENSE](./LICENSE)

---

## Contributing

Issues and pull requests are welcome. Please open an issue before submitting large changes.
