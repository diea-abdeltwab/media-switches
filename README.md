[![media-switches banner](https://capsule-render.vercel.app/api?type=waving&color=0:111214,100:5A1A1F&height=210&section=header&text=media-switches&fontSize=42&fontColor=EDEDEC&animation=fadeIn&fontAlignY=36&desc=Two%20tiny%20Bash%20tools%3A%20block%20social%20media%20and%20filter%20DNS%20%E2%80%94%20on%20any%20Linux&descAlignY=55&descColor=C99BA0&descSize=16)](https://capsule-render.vercel.app)

[![Typing SVG](https://readme-typing-svg.herokuapp.com/?font=Fira+Code&weight=600&size=20&duration=3000&pause=800&color=EDEDEC&background=00000000&center=true&vCenter=true&width=700&lines=social-switch+%E2%86%92+block+social+media+via+%2Fetc%2Fhosts;cloudflare-switch+%E2%86%92+Cloudflare+DNS+filter+on%2Foff;status+%C2%B7+on+%C2%B7+off.+That%27s+the+whole+interface.;One+file+each.+No+dependencies+beyond+Bash.)](https://git.io/typing-svg)

<p align="center">
  <a href="https://www.gnu.org/software/bash/"><img src="https://img.shields.io/badge/Bash%20%E2%89%A5%204.4-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white" alt="Bash"></a>
  <a href="#-requirements"><img src="https://img.shields.io/badge/Any%20Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux"></a>
  <a href="#️-cloudflare-switch"><img src="https://img.shields.io/badge/NetworkManager%20%C2%B7%20systemd--resolved-4A86CF?style=for-the-badge&logo=linux&logoColor=white" alt="DNS backends"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-7A2E33?style=for-the-badge" alt="License"></a>
</p>

---

## 📖 About

**media-switches** is a pair of small, dependency-free Bash tools for people who want a one-word command to flip an annoying (or distracting) system setting — instead of remembering which config file to edit every time.

| Tool | What it flips | How |
|---|---|---|
| 🚫 [`social-switch`](#-social-switch) | Social media, system-wide | A clearly marked block in `/etc/hosts`, generated from a saved list |
| ☁️ [`cloudflare-switch`](#️-cloudflare-switch) | A Cloudflare DNS filter (family / malware / standard) | NetworkManager, systemd-resolved or `resolv.conf` — whichever your system uses |

They share the same DNA: **one self-contained file each**, the same verbs (`status`, `on`, `off`), a read-only `status` that needs no root, and changes that are fully reversible.

> **Want a button instead of a terminal?** The same two scripts power a **GNOME Shell extension** (and an Omarchy widget) in [**social-switch-widget**](https://github.com/diea-abdeltwab/social-switch-widget) — a top-bar lock icon with one tile per platform.

---

## ✨ Features

- ⚡ **One file per tool** — no framework, no package manager, no config format to learn
- 🔌 **Same verbs everywhere** — `status` (no root), `on`, `off`; `sudo` is invoked for you when needed
- 🎯 **Per-platform control** — block Instagram and TikTok only, everything except Discord, or any website by URL
- 💾 **The saved choice is the source of truth** — `/etc/hosts` is generated from it, and `sync` repairs it if another tool rewrites the file
- 🛡️ **Safe edits** — atomic writes, `flock` locking, ownership/mode/SELinux label of `/etc/hosts` preserved, only the managed block is ever touched
- 🌍 **Works on any Linux** — `cloudflare-switch` auto-detects NetworkManager, systemd-resolved, or plain `resolv.conf`
- ↩️ **Reversible by design** — `off` removes only what `on` added; if DNS dies after `cloudflare-switch on`, it rolls back by itself
- 🧰 **Machine-readable** — `status --json` on both tools

---

## 📦 Requirements

- **Bash ≥ 4.4**, `awk`, `flock` (util-linux), `getent`, and `sudo` — present on practically every distribution
- `social-switch`: optionally `chattr` / `lsattr` (e2fsprogs) for the immutable-flag lock
- `cloudflare-switch`: one of **NetworkManager** (`nmcli`), **systemd-resolved** (`resolvectl`), or a plain `/etc/resolv.conf`

---

## 🚀 Installation

```bash
git clone https://github.com/diea-abdeltwab/media-switches.git
cd media-switches
sudo install -m 0755 bin/social-switch bin/cloudflare-switch /usr/local/bin/
```

Each tool is independent — install only the one you need:

```bash
sudo install -m 0755 bin/social-switch /usr/local/bin/
```

> [!TIP]
> Updating from an older copy? Compare `type -a social-switch` with `sudo bash -c 'type -a social-switch'`. On Fedora, **`sudo` searches `/usr/local/sbin` before `/usr/local/bin`**, so an old copy sitting there silently wins whenever you type `sudo social-switch`.

---

## 🚫 `social-switch`

Blocks social media system-wide by writing a clearly marked block into `/etc/hosts` (`0.0.0.0` and `::` for every domain, with and without `www.`). Every platform is independently **blocked** or **allowed**.

> **Wording:** `on` = social media is **ON** (allowed) · `off` = **OFF** (blocked) — a light switch for the sites themselves, not for the blocking feature.

```bash
social-switch status                 # per-platform state, no root needed
social-switch list                   # the platform keys

sudo social-switch off               # block ALL 12 platforms
sudo social-switch off --except discord,twitch
sudo social-switch off instagram tiktok     # block just these two
sudo social-switch on instagram             # allow it again
sudo social-switch on                       # allow everything

sudo social-switch off example.com          # any website — URLs work too
sudo social-switch off https://news.ycombinator.com/item?id=1
sudo social-switch on news.ycombinator.com  # unblock it again
social-switch sites                         # list custom sites

sudo social-switch sync                     # re-apply the saved state to /etc/hosts
social-switch doctor                        # diagnose problems
```

<details>
<summary><b>Example output</b></summary>

```text
$ social-switch status
===== Social Media Block Status =====
Managed block in /etc/hosts : yes
Social media            : mixed   (on = allowed, off = blocked)
/etc/hosts locked (chattr +i): true

PLATFORM       STATUS     SAMPLE DOMAIN
YouTube        blocked    youtube.com
Facebook       allowed    facebook.com
Instagram      blocked    instagram.com
Threads        allowed    threads.net
X / Twitter    allowed    x.com
TikTok         blocked    tiktok.com
Reddit         allowed    reddit.com
Snapchat       allowed    snapchat.com
Pinterest      allowed    pinterest.com
Kick           allowed    kick.com
Discord        allowed    discord.com
Twitch         allowed    twitch.tv

3 / 12 platforms currently blocked.
```
</details>

<details>
<summary><b>All commands</b></summary>

| Command | Description | Root |
|---|---|:-:|
| `status [--json]` | Per-platform state (and custom sites) | – |
| `list` | List platform keys | – |
| `off` / `on` | Block / allow **all** platforms | ✔ |
| `off <platform>…` / `on <platform>…` | Block / allow only these platforms | ✔ |
| `off --except a,b` | Block everything except `a` and `b` | ✔ |
| `block` / `unblock <platform>…` | Same as `off` / `on`, platforms required | ✔ |
| `off <URL>…` / `on <URL>…` | Block / unblock **any** site (also its `www.` form) | ✔ |
| `sites` | List custom (non-platform) sites | – |
| `sync` | Re-apply the saved state to `/etc/hosts` | ✔ |
| `doctor` | Duplicate entries, DNS-over-HTTPS bypass, resolver-level filtering | – |
| `version` | Print the version | – |

**Platforms:** YouTube, Facebook, Instagram, Threads, X/Twitter, TikTok, Reddit, Snapchat, Pinterest, Kick, Discord, Twitch.
**Aliases:** `x` = twitter, `fb` = facebook, `ig` = instagram, `all` = every platform.
`except add` and `except remove` from older versions still work but are deprecated — use `on` / `off <platform>`.

</details>

<details>
<summary><b>How it stays correct</b></summary>

- `/etc/social-switch/blocked` (and `custom`) is the **single source of truth**; `/etc/hosts` is a reflection of it. If another tool rewrites `/etc/hosts`, `status` reports the platform as `broken` and `sync` repairs it.
- Edits are atomic (temp file in `/etc`, then `mv`), serialised with `flock`, and keep the ownership, mode and **SELinux label** of the original file (matters on Fedora/RHEL).
- Only lines made *entirely* of managed domains are ever removed from `/etc/hosts` — your own entries are untouched. The original file is kept once as `/etc/hosts.bak`.
- While something is blocked, `/etc/hosts` is made immutable (`chattr +i`) so apps can't silently edit it back. Opt out with `LOCK_HOSTS=no` in `/etc/social-switch/config`.
- Platforms older versions had but this one doesn't (**Messenger**, **Tumblr**) are migrated to custom sites on first run, so an upgrade can never silently unblock them.

</details>

<details>
<summary><b><code>status --json</code> (schema 2)</b></summary>

```json
{"schema":2,"version":"2.2.0","state":"off","mode":"mixed",
 "blocked_count":3,"wanted_count":3,"total":12,"immutable":true,
 "platforms":[{"key":"youtube","label":"YouTube","blocked":true,"wanted":true,"status":"blocked"}, …],
 "custom":[{"domain":"news.ycombinator.com","blocked":true,"status":"blocked"}]}
```

| field | meaning |
|---|---|
| `state` | two-value view: `off` as soon as *any* platform is blocked |
| `mode` | `on` (nothing blocked) · `mixed` · `off` (all blocked) |
| `wanted` / `blocked` | what you asked for / what `/etc/hosts` really contains |
| `status` | `allowed` · `blocked` · `broken` (wanted but missing → `sync`) · `external` (blocked by an entry we don't manage) |

</details>

> [!NOTE]
> If `doctor` shows a domain is blocked but it still loads in your browser, the browser is almost certainly using **Secure DNS / DNS-over-HTTPS** (on by default in Firefox in some regions, optional in Chrome), which bypasses `/etc/hosts` entirely. Turn it off in the browser's privacy settings.

---

## ☁️ `cloudflare-switch`

Turns a Cloudflare DNS filter on or off for the whole machine, on **any Linux**.

> **Wording:** here `on` = the filter is **active** — the opposite habit of `social-switch`. `enable` / `disable` are accepted aliases.

```bash
cloudflare-switch status              # saved state + whether it is really in effect
sudo cloudflare-switch on             # family: 1.1.1.3 / 1.0.0.3 (malware + adult content)
sudo cloudflare-switch on malware     # 1.1.1.2 / 1.0.0.2
sudo cloudflare-switch on standard    # 1.1.1.1 / 1.0.0.1 (no filtering)
sudo cloudflare-switch off            # back to the DNS you had before
sudo cloudflare-switch sync           # re-apply (e.g. after joining a new Wi-Fi profile)
cloudflare-switch doctor              # conflicts, leftovers, uncovered profiles
```

<details>
<summary><b>Example output</b></summary>

```text
$ cloudflare-switch status
===== Cloudflare DNS filter =====
Filter    : on (Cloudflare Security (malware))
Backend   : resolvconf
In effect : yes
DNS used  : 1.0.0.2 1.1.1.2 2606:4700:4700::1002 2606:4700:4700::1112
```

`Backend` shows what your system uses (`nm`, `resolved` or `resolvconf`); `In effect` checks the servers the machine is *actually* using, not just the saved choice.
</details>

### Presets

| Preset | Servers | Blocks |
|---|---|---|
| `family` *(default)* | `1.1.1.3` / `1.0.0.3` | malware + adult content |
| `malware` | `1.1.1.2` / `1.0.0.2` | malware only |
| `standard` | `1.1.1.1` / `1.0.0.1` | nothing |

IPv6 addresses are set alongside the IPv4 ones.

### Backends (auto-detected)

| Backend | Used when | What it changes |
|---|---|---|
| `nm` | NetworkManager is running (Fedora, Ubuntu, Debian, Mint … desktops) | `ipv4.dns` / `ipv6.dns` + `ignore-auto-dns` of saved Ethernet/Wi-Fi/mobile profiles. Originals are stored in `/var/lib/cloudflare-switch/nm/` and restored by `off`. VPN profiles are never touched. |
| `resolved` | systemd-resolved without NetworkManager (Arch, Omarchy, networkd servers) | one drop-in: `/etc/systemd/resolved.conf.d/90-cloudflare-switch.conf` |
| `resolvconf` | neither (Alpine, Void, minimal installs) | `/etc/resolv.conf`, original saved; refuses if it is a symlink managed by something else |

Force one with `--backend nm|resolved|resolvconf`.

<details>
<summary><b>Design rules</b></summary>

- **Declarative:** `off` removes only what `on` added; it never edits files it doesn't own, so it can't fight the distribution or other tools. No boot service is needed — drop-ins and NetworkManager profiles persist.
- **Safe:** if name resolution worked before `on` and stops working afterwards, the change is rolled back automatically (a firewall or captive portal that blocks Cloudflare can't leave you offline).
- **It never touches `/etc/hosts`** — that belongs to `social-switch`.
- `doctor` lists DNS settings it doesn't own, leftovers of the old v1 (which edited systemd-networkd files and installed a boot service — removed on the next `on`/`off`), and NetworkManager profiles created after `on` (fix: `sync`).

</details>

> [!NOTE]
> Browsers with **Secure DNS / DoH** bypass system DNS too, so the filter won't apply inside them until you turn that setting off.

---

## 🤝 Using both together

| Concern | `social-switch` | `cloudflare-switch` |
|---|---|---|
| **Touches** | `/etc/hosts` (managed block only) | DNS settings (NetworkManager / resolved / `resolv.conf`) |
| **Never touches** | DNS servers | `/etc/hosts` |
| **State lives in** | `/etc/social-switch/` | `/etc/cloudflare-switch/`, `/var/lib/cloudflare-switch/` |
| **Lock** | `/run/social-switch.lock` | `/run/cloudflare-switch.lock` |
| **Undo** | `social-switch on` | `cloudflare-switch off` |

Two things worth knowing:

- **Opposite habits.** `social-switch on` means *allowed*; `cloudflare-switch on` means *filter active*. Both print their meaning.
- **`0.0.0.0` can come from either layer.** Cloudflare Family DNS also answers `0.0.0.0` for some domains. `social-switch doctor` tells hosts-file blocking apart from resolver-level filtering instead of blaming `/etc/hosts`.

---

## 🩺 Troubleshooting

<details>
<summary><strong>Sites still load although they show as blocked</strong></summary>
<br>

Browsers with Secure DNS / DNS-over-HTTPS ignore `/etc/hosts`. Turn it off in the browser, then run `social-switch doctor`.
</details>

<details>
<summary><strong><code>sudo social-switch</code> behaves differently from <code>social-switch</code></strong></summary>
<br>

`sudo` uses its own `secure_path`, which on Fedora/RHEL lists `/usr/local/sbin` before `/usr/local/bin`. Compare `type -a social-switch` with `sudo bash -c 'type -a social-switch'` and `sudo social-switch version`; remove the stale copy.
</details>

<details>
<summary><strong>The status shows <code>broken</code> for a platform</strong></summary>
<br>

Another tool rewrote `/etc/hosts` and removed blocks you asked for. Run `sudo social-switch sync`.
</details>

<details>
<summary><strong>Another program can't edit <code>/etc/hosts</code></strong></summary>
<br>

While anything is blocked, `social-switch` makes `/etc/hosts` immutable (`chattr +i`). To opt out, put `LOCK_HOSTS=no` in `/etc/social-switch/config`.
</details>

<details>
<summary><strong><code>cloudflare-switch on</code> says it rolled back</strong></summary>
<br>

DNS stopped working after the change, so the previous settings were restored. Cloudflare's resolvers are probably unreachable from this network (firewall, captive portal). Nothing was changed; try another network.
</details>

<details>
<summary><strong>I upgraded from cloudflare-switch v1</strong></summary>
<br>

Run `sudo cloudflare-switch on` (or `off`) once — it removes the old `cloudflare-switch-restore.service`. Then `cloudflare-switch doctor` shows any `DNS=` lines left in your `.network` files or `/etc/systemd/network/.dns_switch_backups`, which v2 doesn't use.
</details>

---

## 🗂️ Project Structure

```
media-switches/
├── bin/
│   ├── social-switch        # hosts-file blocker  (v2.2)
│   └── cloudflare-switch    # DNS filter switch   (v2.0, multi-backend)
├── LICENSE
└── README.md
```

---

## 🧭 Roadmap

- [x] Per-platform control and any-website blocking
- [x] `cloudflare-switch` on any Linux (NetworkManager / systemd-resolved / `resolv.conf`)
- [ ] Shell completions (bash / zsh / fish)
- [ ] Optional per-platform schedule (e.g. auto-block during work hours)
- [ ] Config-driven platform list instead of editing the script

Looking for a GUI? See [**social-switch-widget**](https://github.com/diea-abdeltwab/social-switch-widget) — a GNOME Shell extension and an Omarchy widget built on these exact scripts, with offline test suites for both.

---

## 🤝 Contributing

Issues and PRs are welcome — especially reports from distributions these haven't been tried on yet. Please include the output of `social-switch doctor` / `cloudflare-switch doctor` and your distro.

---

## 📜 License

Released under the [MIT License](LICENSE).

---

### 👤 Author

**Diea Abdeltwab** — Data Engineer · Software Engineer · Cairo, Egypt 🇪🇬

*Turning raw data into reliable systems — and, on the side, building the local-first tools I actually want to use.*

<p align="center">
  <a href="https://www.linkedin.com/in/diea-abdeltwab/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
  <a href="https://github.com/diea-abdeltwab"><img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"></a>
  <a href="mailto:diea.abdeltwab@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"></a>
</p>

⭐ **If either of these tools helps you stay focused, a star helps a lot!** ⭐

[![media-switches footer](https://capsule-render.vercel.app/api?type=waving&color=0:5A1A1F,100:111214&height=100&section=footer)](https://capsule-render.vercel.app)

