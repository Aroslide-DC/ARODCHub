 # ARODCHub


A Direct Connect hub compatible with NMDC and ADC, written in Object Pascal (Delphi 11 / Studio 22.0), targeting Win32, and compilable with dcc32.

AroDCHUB is inspired by PtokaX while remaining a compact and readable project. Its main goal is to remain perfectly responsive even under very heavy traffic: all network input/output is handled outside the VCL thread.

## Features

Two protocols served simultaneously, each on its own port:
NMDC (default port 411)
ADC (default port 5111)
The listening socket on which the connection arrives determines the protocol unambiguously: no timeout-based detection.
NMDC and ADC communities are isolated: an NMDC client does not see ADC clients (and vice versa). On the other hand, nickname uniqueness is global across all protocols.
Configurable operators, welcome message (MOTD), logging.
Connection disclaimer: rules text sent as a private message by a bot that is always present in the user list (a virtual nickname per community, NMDC and ADC), with optional mandatory acceptance (chat, private messages, and searches are blocked until the client types !accept). The bot name is configurable; if empty, it is called Hub-Security.
The user list displays the connection type (NMDC).
Configurable anti-flood: message limit per interval, action (kick or temporary IP ban), and ban duration, all adjustable.
Persistent bans: list of banned IPs saved in configs\banlist.txt. Operator commands: !ban <nickname> [minutes], !unban <ip|nickname>, !bans. A ban of 0 minutes = permanent.
IP range ban: !rangeban <ip1> <ip2> [minutes] bans a range of addresses by automatically splitting it into CIDR blocks (!rangeunban <ip1> <ip2> removes it).
Moderation without disconnect:
gag — !gag <nickname> [reason] mutes a user (public chat and private messages) without disconnecting them; !ungag <nickname> restores them, and !gagged lists muted users;
nickname ban (nick ban) — !nickban <nickname> [reason] rejects a nickname at login regardless of the client (NMDC or ADC) and IP; !nickunban <nickname> removes it, and !nickbans displays the list. The list is persistent (configs\nickbanlist.txt) and accepts wildcards * and ? (case-insensitive comparison).
Configuration menu: a menu bar allows you to control the server and open a tabbed dialog (general, anti-flood, warning, commands). Chat commands can be enabled/disabled and their access level modified without recompilation.
Administrator chat: a discussion window (button Chat... or menu Tools → Admin chat…) allows reading the main chat and private messages sent to administration, and replying either under the Admin nickname (a virtual identity created on demand and removed when closed), or under the hub bot nickname. Capture is active only while the window is open; if the Admin nickname is already held by a user, sending falls back to the bot nickname.
Built-in protections:
send buffer cap (1 MB) → a slow client is disconnected;
receive buffer cap (64 KB);
configurable rate limit (flood) (default 120 lines per second), with kick or temporary ban;
hub upstream bandwidth cap (QoS, MaxUploadKBps) with configurable burst — see the QoS section below;
maximum configurable number of users (200 by default);
anti-impersonation check (the stated nickname must match the validated one; the address announced via $ConnectToMe must match the sender’s real IP).
Anti-freeze architecture
This is the central point of the project.

A single dedicated thread (THubServer, derived from TThread) performs all socket I/O operations.
Sockets are non-blocking and monitored by WSAPoll — not select, whose FD_SETSIZE limit (64) would have prevented exceeding 64 sockets.
The VCL thread never performs network I/O or waits.
No TThread.Synchronize: this would block the network thread while waiting for the main thread to process a message, which is exactly the mechanism that freezes classic hubs under load.
Instead:
log messages are queued in a thread-safe queue (TLogQueue, bounded queue);
the interface drains them in limited batches via a TTimer (300 ms);
the log TMemo is capped;
the user TListView is rebuilt only when the user version counter changes;
the UI reads only copies/snapshots (TUserManager.Snapshot), never the live shared state used by the network thread.
