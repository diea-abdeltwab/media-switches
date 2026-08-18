<div align="center">

# 🔀 media-switches

### *A one-word command to flip an annoying system setting on or off — no config files, no memorizing flags.*

[![Made with Bash](https://img.shields.io/badge/Made%20with-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)](https://www.gnu.org/software/bash/)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%28Arch%2Fomarchy%29-1793D1?style=for-the-badge&logo=linux&logoColor=white)](#)
[![License](https://img.shields.io/badge/License-MIT-a6e3a1?style=for-the-badge)](#-license)

[![Typing SVG](https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=600&size=22&duration=2500&pause=800&color=4EAA25&background=00000000&center=true&vCenter=true&width=650&lines=cloudflare-switch+%E2%86%92+toggle+Cloudflare+DNS;social-switch+%E2%86%92+block+social+media+system-wide;status+%C2%B7+on+%C2%B7+off+%E2%80%94+that%27s+the+whole+interface)](https://git.io/typing-svg)

</div>

---

## 📌 About

**media-switches** is a pair of small, dependency-free Bash scripts for people who want a one-word command to flip an annoying (or distracting) system setting on or off — instead of remembering which config file to edit every time.

Every script shares the same DNA:

- ⚡ **Single self-contained file** — no install script, no dependencies beyond standard `systemd`/coreutils tools already on your system
- 🔌 **Three verbs, always**: `status`, `on`, `off` — no flags to memorize
- 💾 **Remembers your choice** — the state you set is saved and re-applied automatically, so a reboot (or, for `cloudflare-switch`, anything that resets your network config) can't silently undo it
- 🐧 Built for **Arch Linux / [omarchy](https://omarchy.org/)**, but portable to any `systemd`-based distro
- 🔒 Every file edit is backed up before it's touched, and reverted cleanly on `off`

---

## 📦 The Scripts

| Script | For | Highlights |
|---|---|---|
| ☁️ [`cloudflare-switch`](#️-cloudflare-switch) | Toggling the Cloudflare (`1.1.1.3`) DNS override | Self-installs a boot-time `systemd` service so your `on`/`off` choice survives reboots even if the network stack resets it |
| 🚫 [`social-switch`](#-social-switch) | Blocking social media system-wide via `/etc/hosts` | Blocks 12 platforms (Facebook, Instagram, X/Twitter, TikTok, Reddit, Discord, Twitch, and more) with per-platform exceptions |

Each script is fully independent — copy just the one you need, or both.

---

## 🚀 Quick Start

Every script follows the same three steps:

```bash
chmod +x <script-name>
sudo mv <script-name> /usr/local/bin/<script-name>
```

Then use it anywhere:

```bash
<script-name> status       # check current state, no root needed
sudo <script-name> on      # turn it on
sudo <script-name> off     # turn it off
```

---

## ☁️ `cloudflare-switch`

Toggles the Cloudflare DNS override (`1.1.1.3` / `1.0.0.3`) for `systemd-networkd` setups by commenting/uncommenting `DNS=`/`FallbackDNS=` lines in your network config and `resolved.conf`.

```bash
cloudflare-switch status
# → shows whether the override is ACTIVE or INACTIVE right now

sudo cloudflare-switch on
# → re-enables Cloudflare DNS

sudo cloudflare-switch off
# → falls back to router/DHCP DNS

cloudflare-switch doctor
# → status + sanity checks: resolvectl, /etc/resolv.conf, a live lookup test
```

<details>
<summary><b>How persistence works</b></summary>

The first time you run `on` or `off`, the script installs its own `cloudflare-switch-restore.service` under `systemd` — no separate unit file to place by hand. On every boot, it re-applies whatever you last chose, so the override can't quietly come back on its own.

</details>

---

## 🚫 `social-switch`

Blocks social media system-wide by writing a clearly marked block into `/etc/hosts` (leaving the rest of the file untouched), with `chattr +i` locking so apps can't silently edit it back.

> `on` means social media is **on** (allowed); `off` means it's **off** (blocked) — like a light switch for the sites themselves, not for the block feature.

```bash
social-switch status
# → per-platform block status

social-switch list
# → lists every available platform key

sudo social-switch off
# → blocks everything

sudo social-switch off --except discord
# → blocks everything except Discord

sudo social-switch except add reddit
# → permanently allows Reddit, even the next time you run "off"

social-switch doctor
# → diagnoses duplicate /etc/hosts entries or a DNS-over-HTTPS bypass
```

<details>
<summary><b>All commands</b></summary>

| Command | Description |
|---|---|
| `status` | Per-platform block status |
| `list` | List available platform keys |
| `on` / `off` | Allow / block everything |
| `off --except <platform>` | Block everything except one platform |
| `except add <platform>` | Permanently allow a platform, even when "off" |
| `except remove <platform>` | Re-include a platform in future blocks |
| `except list` | Show current exceptions |
| `doctor` | Diagnose duplicate entries / DoH bypass |

**Platforms covered:** Facebook, Messenger, Instagram, Threads, X/Twitter, TikTok, Reddit, Snapchat, Pinterest, Tumblr, Discord, Twitch.

Exceptions persist in `/etc/social-switch/exclude` and are re-applied automatically the next time blocking is refreshed.

</details>

> [!NOTE]
> If `doctor` shows a domain is blocked at the OS level but it still loads in your browser, the browser is almost certainly using **Secure DNS / DNS-over-HTTPS** (on by default in Firefox in some regions, optional in Chrome), which bypasses `/etc/hosts` entirely. Turn it off in the browser's privacy settings if you want the block to be airtight there too.

---

## 🏗️ Design pattern (shared)

Both scripts follow the same internal shape:

```text
<script-name>
├── status / list        ← read-only, no root required
├── on / off              ← the two state-changing verbs (root required)
├── state persistence     ← /etc/<script-name>/state (+ exclude list for social-switch)
└── backup-before-edit    ← original config always saved before the first change
```

No frameworks, no package manager, no config format to learn — just `bash`, `sed`, and standard system tools.

---

## 📜 License

MIT — use it, fork it, tweak it however you like. See [`LICENSE`](./LICENSE).

<div align="center">

## 🧑‍💻 About the author

**Diea Abdeltwab** — Data Engineer · Software Engineer · Cairo, Egypt 🇪🇬

*Turning raw data into reliable systems — and, on the side, building the local-first tools I actually want to use.*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/diea-abdeltwab/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/diea-abdeltwab)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:dieaabdeltwab2024@gmail.com)

⭐ **If either of these scripts helps you stay focused, a star helps a lot!** ⭐

</div>
