# AroDCHUB — User Manual

This guide explains how to install, configure, and use the AroDCHUB hub on a daily basis. It is intended for **hub administrators** and **end users** (those connecting with a DC++ client).

---

## 1. Quick Start

1. **Extract** the archive to a folder of your choice (e.g., `C:\AroDCHub\`).
2. Run **`AroDCHUB.exe`**.
3. The main window opens. Click **Start** (toolbar or **File** menu).
4. The hub is now listening on:
   - **Port 411** (NMDC) — classic DC++ clients
   - **Port 5111** (ADC) — modern clients
5. Minimize to the **system tray** (icon near the clock) to keep it running in the background.

> **Tip**: On first launch, the \configs\` and `\logs\` folders are created automatically with a default configuration.

---

## 2. Main Interface

| Area | Description |
|------|-------------|
| **Toolbar** | Start / Stop, switch hub (multi-hub), open configuration |
| **User list** | Nick, protocol (NMDC/ADC), connection type, IP, share, status |
| **Log** | Real-time activity (connections, commands, errors) |
| **Status bar** | Server state, user count, admins, total share, bans, uptime, traffic |

The **rules bot** (default `Hub-Security`) is always present in the user list: it sends the rules via private message to every new connection.

### Appearance / theme (Onyx Blue)

The interface comes in two looks:

- **Windows** (default): native light Windows appearance;
- **Onyx Blue**: a dark blue-slate theme (DevExpress *Calypso*-inspired) applied to every window.

Switch instantly through the menu **Tools ➜ Onyx Blue** (checkable item). The choice is stored in `StyleName` (section `[Hub]` of `AroDCHub.ini`) and is **per hub** — switching the active hub re-applies its own theme.

---

## 3. Configuration via the GUI (Recommended)

No need to edit files manually: everything is in **Configuration ➜ Hub Settings**.

### Main Tabs

| Tab | What you configure |
|-----|-------------------|
| **General** | Hub name, description, NMDC/ADC ports, max users, max connections per IP, files (MOTD, rules), operators & passwords, hublist, security (S1/S2/S3) |
| **Anti-flood** | Enable, message/time thresholds, action (kick/ban), ban duration |
| **Disclaimer** | Rules text, mandatory acceptance (`!accept`), bot name |
| **News / Events / Rules** | Automated periodic announcements |
| **Commands** | Enable/disable each command, required level (guest/operator) |
| **Registered Users** | Manage accounts (see § 6) |
| **Fake Clients** | Version detection, share check, whitelist/blacklist |
| **Hub** | Topic, redirect, nick length limits, **bandwidth (QoS) limit** |
| **Operators** | Per-user levels (guest/op/admin/owner) |
| **Limits** | Slots, DNSBL, blocked countries (GeoIP) |
| **Search** | Search limits, file types, anti-moglo, `!time` bot, per-user search quota |
| **Log** | Log rotation (max size, backup count) |
| **Security / TLS** | Certificate, TLS ports, runtime toggle |

> Click **OK** to apply. If the server is running, it restarts cleanly.

---

## 4. Chat Commands (for all users)

Type `!help` in main chat to see commands available at your level.

### Guest commands (everyone)

| Command | Description |
|---------|-------------|
| `!help` | List accessible commands |
| `!rules` | Re-display rules / disclaimer |
| `!accept` | Accept rules (if required) |
| `!stats` | Show user count and uptime |
| `!register [password]` | Create your account (protects your nick) |
| `!passwd <old> <new>` | Change your password |

### Operator commands

| Command | Description |
|---------|-------------|
| `!kick <nick> [reason]` | Kick a user |
| `!ban <nick> [minutes]` | Ban user's IP (0 = permanent) |
| `!unban <ip|nick>` | Lift a ban |
| `!bans` | List active bans |
| `!gag <nick> [reason]` | Mute user (main chat + PM) without disconnecting |
| `!ungag <nick>` | Restore a muted user |
| `!gagged` | List muted users |
| `!nickban <nick> [reason]` | Ban a nick (wildcards `*` `?` allowed) |
| `!nickunban <nick>` | Lift a nick ban |
| `!nickbans` | List banned nicks |
| `!rangeban <ip1> <ip2> [min]` | Ban an IP range (auto CIDR) |
| `!rangeunban <ip1> <ip2>` | Lift an IP range ban |
| `!addreg <nick> <pwd> <level>` | Add a registered account (guest/op) |
| `!delreg <nick>` | Delete a registered account |
| `!reglist` | List all registered accounts |
| `!relevel <nick> <level>` | Change a registered account's level (`guest`/`op`/`admin`/`owner`) |
| !op <nick> | Grant **temporary** operator rights (survives only the session; configurable level) |
| !deop <nick> | Revoke temporary operator rights |
| !massmsg <text> | Send a message to all users (admin) |

> **Tip**: Right-click a user in the list ➜ context menu (kick, ban, gag, details, PM).

### Operator channel (`#OpChat`)

Operators see a `#OpChat` room. Messages sent to it (directly in the room, or with `//message` in main chat) reach all operators.

By default the hub does **not** send your own message back to you (`OpChatEchoSelf=0` in `[Hub]`). This avoids the duplicate you would otherwise see when the DC++ client also mirrors main chat into the open `#OpChat` tab — the same behaviour as PtokaX room scripts (`if sNick ~= sSelfNick`). Set `OpChatEchoSelf=1` to restore the classic PtokaX `Core.SendToOpChat` behaviour (echo to the sender).

---

## 5. Registered User Accounts

Accounts protect nicks: only the owner can connect with that nick (password required).

### For the user
- `!register myPassword` — create account
- `!passwd old new` — change password

### For the operator (menu **Configuration ➜ Registered Users**)
- Add / remove / list accounts
- Level: `guest` (normal user) or `op` (operator)

### Grant a temporary operator (GUI)
Right-click a connected user in the **Users** list — either on the main window or in **Configuration ➜ User Management** — and choose **Grant temporary operator** / **Revoke temporary operator**. This is the GUI equivalent of `!op` / `!deop`: the status lasts only for the current session and is not saved to the configuration.

Passwords are **stored hashed** (PBKDF2-HMAC-SHA256, `pw2$` prefix) in `configs\registered_users.txt`. Legacy plaintext passwords are auto-migrated on load.

---

## 6. Important Security Features (enable as needed)

| Feature | INI parameter (`[Hub]`) | Description |
|---------|------------------------|-------------|
| **S1 — Strict `$Key`** | `NMDCKeyStrict=1` | Reject NMDC clients that don't send the full key (compat mode by default) |
| **S2 — Anti brute-force** | `MaxPassFails=5` `PassFailBanSec=300` | After 5 failed `$GetPass` attempts from same IP → IP banned 5 min |
| **S3 — Audit log** | `OpLogFileName=oplog.txt` | All operator actions (kick, ban, gag, nickban…) timestamped in `configs\oplog.txt` |

> **Recommendation**: On a public hub, enable `OpsRequirePassword=1` (General tab) and set a password for each operator in the `[OpPasswords]` section.

---

## 7. Hublist — Make Your Hub Public

To appear in public hublists (hublist.org, etc.):

1. **General** tab ➜ **Hublist** section
2. Check **Enable hublist registration**
3. Fill **Public address** (e.g., `myhub.example.com:411`)
4. Optional: **Hublist URL** (HTTP endpoint) and **Hublist list** (one per line)

The hub registers automatically every **10 minutes**. For ADC, the address is announced via the `HH` tag in `ISUP`.

---

## 8. Encrypted Connections (TLS / nmdcs:// / adcs://)

To protect passwords and traffic:

1. **Security / TLS** tab ➜ **Manage TLS…**
2. Generate a certificate (PFX) or pick one from Windows store
3. Set TLS ports (e.g., 42188 NMDC, 42189 ADC)
4. Check **Enable TLS** (or menu **Tools ➜ TLS ➜ Enable TLS**)

DC++ clients then connect via `nmdcs://host:port` or `adcs://host:port`.
Use the URL with `kp=SHA256/<keyprint>` to validate the certificate.

---

## 9. Country Blocking (GeoIP)

1. Download `.zone` files from [ipdeny.com](https://www.ipdeny.com/ipblocks/) (e.g., `ru.zone`, `cn.zone`)
2. Place them in `bin\geoip\`
3. Menu **Configuration ➜ Blocked countries…** ➜ add ISO codes (e.g., `RU`, `CN`)
4. IPs from these countries are refused at login (bot message)

---

## 10. Automatic Backups (Peace of Mind)

- **Configuration** (`AroDCHub.ini`): on every save, rotated to `.bak1`, `.bak2`, `.bak3`
- **Nick bans** (`nickbanlist.txt`): same rotation
- On startup: if the INI file is **missing or empty**, the first available backup is restored automatically

---

## 11. Connection Limit per IP

Parameter `MaxConnectionsPerIP` (General tab, default `0` = disabled).

Example: `MaxConnectionsPerIP=5` → a single IP cannot open more than 5 simultaneous connections. This limit is **independent of the anti-flood system** (works even when anti-flood is disabled).

---

## 12. Bandwidth Limit (QoS, Hub Outbound Traffic)

The hub can cap its own outbound traffic (chat, user lists, search results,
forwarding to linked hubs).

- **Hub tab** of **Configuration ➜ Hub Settings**:
  - `MaxUploadKBps` — maximum bytes the hub may send, in KB/s (`0` = unlimited, default)
  - `BurstKB` — allowed burst in KB (`0` = automatic, one second of the limit, minimum 16 KB)

Important:

- **Client to client file transfers do not go through the hub** — they are not
  affected by this limit.
- A limit well below the size of one user list slows down logins: keep a margin
  (for example 128 KB/s or more).
- The cap uses a token bucket; while throttled, the hub stays responsive and
  does not spin on the CPU.
- Closing messages (`$Quit` / `IQUI`) are never throttled.

---

## 13. Key Configuration Files

All in `bin\configs\`:

| File | Contents |
|------|----------|
| `AroDCHub.ini` | Main configuration (created/updated by the GUI) |
| `motd.txt` | Welcome message (one line = one message) |
| `disclaimer.txt` | Rules / disclaimer sent via PM on connect |
| `banlist.txt` | Persistent IP bans (CIDR) |
| `nickbanlist.txt` | Banned nicks (wildcards `*` `?`) |
| `registered_users.txt` | User accounts (hashed `pw2$`) |
| `illegal_nick.txt` | Nicks forbidden at login |
| `blocked_countries.txt` | ISO codes of blocked countries |
| `oplog.txt` | Operator action audit log (S3) |
| `hublist.txt` | Hublist endpoints for auto-registration |

---

## 14. Common Troubleshooting

| Issue | Solution |
|-------|----------|
| **Port already in use** | Change `PortNMDC`/`PortADC` or `TlsPortNMDC`/`TlsPortADC` in General / TLS tabs |
| **TLS won't start** | Verify TLS ports are non-zero, distinct from plaintext ports, and certificate is valid |
| **Client "certificate verify failed"** | Use URL `nmdcs://host:port/?kp=SHA256/<keyprint>` or accept cert in DC++ |
| **Hub still running after close** | Check system tray (icon near clock) |
| **Lost configuration** | Check `AroDCHub.ini.bak1` / `.bak2` / `.bak3` in `configs\` |

---

