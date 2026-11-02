[Português](README.md) · **English** · [Русский](README.ru.md)

# AuroraLogin

Player authentication for Paper: registration, login, IP sessions, account protection and an in-game panel to tune everything. One jar, no dependencies, no external website, no remote database.

**Version:** 1.10.2  
**Server:** Paper 1.20.1 or newer  
**Java:** 17, or whatever your server version already requires  
**Dependencies:** none  
**Price:** free

## Why it exists

Every server running without online-mode has the same problem: anyone can join with anyone's name and that's it. AuroraLogin closes that hole with registration and passwords, but without making you set up a website, a remote database or half a dozen companion plugins. Accounts live in a SQLite file inside the plugin's own folder, passwords become PBKDF2-SHA256 hashes, and after configuring it once you usually never open the config file again, because everything is in the in-game panel.

It also handles the part that normally takes work: **players with a paid Minecraft account get in on their own**, without registering, even on a server with `online-mode=false`. That runs inside the jar itself, no ProtocolLib, no verification plugin.

## Quick summary

- Register, login, change password and logout, with an IP session.
- Paid accounts get in on their own; on an offline server the plugin checks with Mojang while the player is already in, without holding the connection, and a name nobody bought isn't bothered at all — it gets in on the first try.
- Bedrock gets in on its own when Floodgate is installed.
- `/auroraauth panel` with the settings, the player list and the IP strike list.
- Progressive strikes against brute force: kick, longer kick, IP ban.
- Portuguese, English and Russian, command names included.
- A commented config, and the comments don't disappear when the panel saves.

## What joining feels like

A player arrives and is held in place: no inventory, no breaking blocks, no item use, and they get the pre-login effects (blindness, slowness, mining fatigue) so they can tell they need to authenticate. If they walk before logging in, they go back to the spot where they joined.

Chat shows the button with the right command, already in the server's language, and the command list in tab shrinks to the authentication ones. They register with password confirmation, or log in with the password they already have. Once authenticated, the effects come off, the command list returns and they play normally.

Anyone coming back inside the session window gets straight in without typing again. A paid account skips all of it, and so does a Bedrock player with Floodgate. That's the whole idea in the end: whoever owns the account gets in, whoever doesn't stays outside.

## Paid accounts

With `premium-auto-login` on (it ships on), a paid Minecraft account gets in without registering and without a password.

- On a server with online-mode, the identity already arrives validated by Mojang and the plugin uses it as is.
- On an offline server without a proxy, the plugin decides before it challenges. First it asks Mojang's official name catalogue whether that name belongs to a paid account. A name nobody bought never gets an encryption request: it goes straight to registration, on the first join. A paid name is challenged, the player enters as soon as they accept the encryption and the session is confirmed with Mojang's session server in parallel, so nobody waits at the door. A name Mojang has already confirmed is noted down in the plugin's own database, so on the way back it is challenged immediately, with no network wait. All of this lives inside the jar.
- If you run Velocity or BungeeCord with forwarding, the plugin notices and doesn't compete with the proxy: it leaves the verification there and follows the normal login flow.
- If Mojang doesn't confirm, or the name catalogue is silent, slow or rate-limited, or the internal structure of a new version isn't recognised, the player falls back to the normal register-and-password flow. None of those failures locks anyone out and none of them slows the login: the catalogue gets 4.5 seconds to answer during login (3 to connect, 4 for the request), and with no answer the plugin simply doesn't challenge; the session check gets 5 seconds, and with no answer the password prompt arrives and the player registers as usual. Turn the whole thing off with `premium-verification`. One failure is remembered for 5 seconds and the global wait is 3, so a single error doesn't push the players behind it into the password flow. On start and on every `/auroraauth reload` the plugin opens the Mojang connection with a test name, so the first player doesn't pay for a cold connection, and the console warning comes once. `/auroraauth reload` also clears whatever the catalogue answered. If a paid name was used by someone who doesn't own the account, see [Clients with no Mojang session](#clients-with-no-mojang-session-the-pin).
- On the first join the plugin creates the internal account with a random password nobody ever sees, purely to keep the database consistent. If an account with that name already exists, it is linked to the player instead of duplicated.
- The UUID doesn't change, so the world, permissions and data you already have all stay valid.
- The account is marked as verified in the database and their profile in the panel shows it. **There is no badge in chat or in tab**: the name appears plain, which is probably what you'd rather have.
- And while the session hasn't answered? The player is already in the world, but held like anyone with no session: pre-login effects, no commands, no chat, and not one of the authenticated permissions. If the answer is yes they are let through without typing a thing; if it is no, the password prompt arrives in their own language. In practice that is a fraction of a second, and nobody becomes an authenticated player before Mojang says so.

### Clients with no Mojang session (the pin)

A paid name used by someone who doesn't own the account is still challenged, and that is on purpose: it is what stops somebody from walking onto your server with the real owner's identity. Their launcher gives up right there with the *Invalid session* message, client-side, before any password is involved. But only once.

The plugin stores the name and IP pair that died during the handshake, or stayed silent for 4 seconds, and stops challenging it. From the next connection on that client isn't even looked up in the catalogue: it walks straight into the register-and-password flow, as if the check had never applied to them. A player whose name nobody bought never goes through this, because they were never challenged.

- `/auroraauth premium` lists the recorded clients, with name and IP.
- `/auroraauth premium reset <name|ip>` clears one. After a reset the client is challenged again, and if it dies again it goes back into the pin.
- The pin lives in `accounts.db` and in memory. Restarting the server removes nothing.
- A client that answers the challenge is never pinned: ending up in the pin means it did not answer.
- `premium-verification: false` turns the whole check off, so no client is challenged and none is pinned.

Every pin write shows up in the console with the name and IP, whenever you want to look. You don't have to watch it: the plugin sorts this out by itself.

## Bedrock (Floodgate/Geyser)

With `floodgate-auto-login` on and Floodgate installed, anyone joining over Bedrock whose identity Floodgate has confirmed gets straight in, no password.

Identification uses Floodgate's own API (`isFloodgatePlayer`), never a prefix in the name, so a Java client can't pass itself off as a mobile player. Without Floodgate installed the bypass simply stays off and the Bedrock player registers with a password like anyone else, with no error and no console warning. The Floodgate UUID is preserved and the IP session applies the same way.

You can switch both automatic logins at once without opening a file: `/autologin off` turns everything off, `/autologin on` turns it back on, and with no argument the command shows how things stand. To control each one separately, use the panel under **General**. The command name follows the language too (`/autologin` in pt-br and English, `/автовход` in Russian).

## Brute-force protection

The block is per IP and it raises the tone gradually instead of banning on the first mistake. If an IP gets `max-attempts` wrong passwords inside the configured window:

1. first strike: kicked for 5 minutes;
2. second strike: kicked for 15 minutes;
3. third strike: IP banned for 24 hours.

All three durations are adjustable in the panel, the ban lifts on its own after the configured period without anyone having to unban, and strikes reset after 7 days without a new one. While the IP is blocked or banned, the connection never really gets in: the disconnect screen shows the IP, the time left and, on a ban, the reason plus the Discord invite you set. That is the player having somewhere to go instead of opening a thread on your forum.

Want to see what is holding someone out before the clock runs out? **Security → IPs and strikes**. The list brings every IP that touched the server — blocked, banned, expired, already free or with no strike at all — with the name used in its last connection attempt, the strikes and the status, newest activity first. The search takes part of an IP, the refresh button reloads without closing the menu, and each IP opens a screen with the head of whoever tried to enter last and two separate actions: **Clear block** frees the time right away and keeps the strikes, **Delete record** removes the IP from the database and zeroes the strikes. Neither one touches accounts or passwords. An IP stops showing up once it has spent `strike-reset-hours` without trying to enter.

On top of that, chat stays locked until the player authenticates, commands are restricted and world interactions are blocked (inventory, dropping, picking up, offhand swap, blocks and entities, portals, buckets, consuming items and shearing).

## Languages

Three complete languages: Portuguese (pt-br), English and Russian. Switching language changes the whole server, including the names of the authentication commands; the old ones stop existing and the new ones take effect immediately.

| Language | Register | Login | Logout | Change password | Auto login |
| --- | --- | --- | --- | --- | --- |
| pt-br | `/registrar` | `/entrar` | `/sair` | `/trocarsenha` | `/autologin` |
| en | `/register` | `/login` | `/logout` | `/changepassword` | `/autologin` |
| ru | `/регистрация` | `/вход` | `/выход` | `/сменитьпароль` | `/автовход` |

The admin command `/auroraauth` is the same in every language, and the text of each language sits loose in `lang/` for you to edit however you like.

One thing that used to annoy people, so it was handled separately: while a player is typing a password into `/register`, `/login` or `/changepassword`, tab suggests nobody's name. Nothing like leaking the player list while someone writes their password.

## The in-game panel

`/auroraauth panel` opens the menu. Every click writes to `config.yml` right away, no restart, and it doesn't leave the file a mess.

| Category | What you adjust |
| --- | --- |
| General | Session, automatic login for paid accounts, the Mojang check during login, time allowed to authenticate, password length, password matching the name, reuse, teleport and interaction blocking |
| Security | Brute force, attempts, window, the three strike durations, IP ban and the IP strike list, with search, refresh and two actions per IP |
| Visual | Pre-login effects, titles, sounds and action bar |
| Players | Account list, search by name, profile and admin actions |
| Language | Changes the language of the whole server |
| System | Audit, panel sounds and reloading the configuration |

On numeric settings, left click increases, right click decreases, and with shift the step is bigger.

### Players

The **Players** tab lists every account, including people who are offline, paginated, each one with the player's head.

- Search works by typing the name in chat after clicking Search (cancel with `cancelar`, `cancel` or `отмена`).
- The profile shows UUID, creation date, last login, last IP with its status and strikes, whether they're authenticated right now, how long the block lasts and whether the account was verified by Mojang.
- Resetting a password generates a temporary 12-character one, shown only to whoever ran the action, with a click to copy. The player isn't kicked: they keep playing, their IP session ends, and the new password is what they use at the next login.
- You can also disconnect them, clear the block and delete the account, the last one behind a confirmation screen. Clearing the block releases both locks at once — the time stored on the IP and the session counter — and keeps the strike, so the next excess still counts as the second one.

A specific player's profile opens directly with `/auroraauth panel <player>`.

## Commands

### Player commands

| Command | What it does |
| --- | --- |
| `/register <password> <password>` | Creates the account |
| `/login <password>` | Authenticates into the account |
| `/logout` | Ends the session |
| `/changepassword <current> <new> <new>` | Changes the password |

The names change with the language, as in the table above.

### Admin commands

Every subcommand has a short alias, so you don't have to type `/auroraauth` in front. Both formats use the same permission and answer with the same messages.

| Full command | Alias | What it does |
| --- | --- | --- |
| `/auroraauth info <player>` | `/info <player>` | Shows the account data |
| `/auroraauth unregister <player>` | `/unregister <player>` | Asks for confirmation to delete the account |
| `/auroraauth unregister <player> confirm` | `/unregister <player> confirm` | Deletes the account |
| `/auroraauth resetpassword <player>` | `/resetpassword <player>` | Generates a temporary password and resets it |
| `/auroraauth language <pt-br\|en\|ru>` | `/language <pt-br\|en\|ru>` | Changes the server language |
| `/auroraauth panel [player]` | `/panel [player]` | Opens the panel or a profile (in game only) |
| `/auroraauth autologin` | `/autologin` | Shows the automatic login state |
| `/auroraauth autologin <on\|off>` | `/autologin <on\|off>` | Turns the automatic logins on or off |
| `/auroraauth premium` | — | Lists the clients with no Mojang session (the pin) |
| `/auroraauth premium reset <name\|ip>` | — | Removes a client from the pin |
| `/auroraauth reload` | `/reload` | Reloads configuration and messages |

The namespaced name (`/auroralogin:reload`) always works. The short `/reload` reloads this plugin; for the server's datapacks, `/minecraft:reload` stays where it belongs. If another plugin has already claimed one of those aliases, AuroraLogin gives up the name, says so in console and carries on working through `/auroraauth`.

## Permissions

Everything is granted by `auroralogin.admin`, which already includes the rest. By default only ops have it.

| Permission | What it unlocks | Default |
| --- | --- | --- |
| `auroralogin.admin` | Full access | op |
| `auroralogin.admin.info` | Look accounts up | op |
| `auroralogin.admin.unregister` | Delete accounts | op |
| `auroralogin.admin.resetpassword` | Reset passwords | op |
| `auroralogin.admin.language` | Change the server language | op |
| `auroralogin.admin.panel` | Open the panel | op |
| `auroralogin.admin.autologin` | Turn automatic login on and off | op |
| `auroralogin.admin.premium` | View and clear the record of clients with no Mojang session | op |
| `auroralogin.admin.reload` | Reload configuration and messages | op |
| `auroralogin.admin.kick` | Disconnect a player from the panel | op |
| `auroralogin.admin.unblock` | Clear the attempt block, for a player and for an IP | op |
| `auroralogin.admin.unban` | Open the IP list and delete a record | op |

## Installation

1. Download the newest jar from the **Releases** tab.
2. Drop the file into your Paper server's `plugins/`.
3. Start the server. `plugins/AuroraLogin/` appears with the configuration and the languages.
4. Adjust from the panel with `/auroraauth panel`, or edit `config.yml` and run `/auroraauth reload`.

Requirements: Paper 1.20.1 or newer and Java 17 or newer. Nothing beyond that, because the paid-account check on an offline server already ships inside the jar. If you want Bedrock players joining without passwords, that's the one case where you need Floodgate (with Geyser); the plugin detects it by itself, and without it everything keeps working, Bedrock just uses passwords like anyone else.

## Configuration

`config.yml` ships with one explanation per option, and the panel preserves that: since each click rewrites only the value line, the comments stay exactly where they are. A server that already had an old configuration is updated on its own at first boot, reusing your values and keeping the previous file as `config.yml.bak`.

The full list, with every option explained and a sample file, is in [docs/configuracao.en.md](docs/configuracao.en.md) — and in [Portuguese](docs/configuracao.md) or [Russian](docs/configuracao.ru.md) if you prefer. The messages live in `plugins/AuroraLogin/lang/`, one file per language (`pt_br.yml`, `en.yml`, `ru.yml`).

## About your server's data

- A password becomes a PBKDF2-SHA256 hash with 600,000 iterations. It never exists as plain text in the database or in a log, and there is no command that shows someone's password.
- The admin reset creates a temporary 12-character one, shown only to whoever performed the action, with a click to copy. The player stays online.
- Accounts sit in `plugins/AuroraLogin/accounts.db`, local SQLite. A backup means copying that file with the server stopped.
- The same database holds the IP screen: for every IP, the date of its last connection attempt, the name used in it and that client's UUID. That is what lets you see who tried to enter through an IP before there was a single strike. It never leaves your server, and the row disappears on its own once the IP has spent `strike-reset-hours` without showing up.
- The audit log at `plugins/AuroraLogin/audit.log` records what the administration did, without touching passwords.
- The paid-account check only talks to two official Mojang addresses, during login: the name catalogue and the session server. Both are read-only lookups, no token and no key is stored, and the confirmation is worth only that one connection. The catalogue's answer is kept for a while precisely so the plugin doesn't hammer Mojang on every join, and the names Mojang confirmed as paid are noted in the plugin's own database — just the name and a date — so the owner's return doesn't depend on the network. There is no option to redirect either check to another address, and there won't be: if such a field existed, someone would point it at a machine that answers "yes" to any name.
- Nothing that happens on your server goes out, apart from those name and session lookups at Mojang. No website, no telemetry, no third-party service.

## Compatibility

Tested on Paper 1.20.1 and Paper 1.21.1 with no other plugin installed.

The paid-account check on an offline server rides the server's own network pipeline, which makes it sensitive to the internal structure of a version. If a future version changes too much and the plugin doesn't recognise it, it warns in console, leaves the check off and password login continues exactly as it is, without stuck joins for anybody. In that case it's a matter of waiting for the plugin update.

## Changelog

**1.10.2** — A real account no longer sits on the connection screen. The plugin used to wait for Mojang's `hasJoined` to answer before it released the login, and that was a cost per join, not an exception: in my measurement, around 420 ms on every paid entry, even with the name already noted in the catalogue. Now the login is released as soon as the client accepts the encryption, that connection is marked pending and Mojang is asked on the side. Confirmed, they are in on their own, without a prompt and without typing anything. Not confirmed, they still land in the password flow, only the message arrives after the verdict. Until the answer comes the player is in the world and held: pre-login effects, no commands, no chat, no authenticated permissions. In the test, with the answer forced to take 1.5 s, they were standing in the world 0.6 s before the verdict. Leaving before the answer raises no exception and leaves nothing pending in the database, and the verification timeout that used to hold that wait is gone along with the wait.

Players without a paid account get in on the first try. Every client used to receive the encryption request during login, and a launcher with no Mojang token gave up with the *Invalid session* message before the password flow ever started. Now, before challenging anyone, the plugin asks Mojang's official name catalogue whether that name belongs to a paid account: a name nobody bought receives no encryption request at all and goes straight to registration on the first join. A paid name is still challenged, because that is what stops somebody from walking in with the owner's identity, and whoever uses it without the account is recorded by name and IP after that single error and stops being challenged — the next login goes straight to the password flow. `/auroraauth premium` lists them and `/auroraauth premium reset <name|ip>` clears one. New permission: `auroralogin.admin.premium`. A catalogue that is down, slow or rate-limited doesn't challenge anybody either and doesn't delay anyone's login.

A real account got its automatic login back on a freshly started server. The first join after start paid for a cold connection to Mojang (DNS and TLS from scratch) and, against the old ceiling, the correct answer arrived after the deadline: the name went into the password flow even though it was paid. Now the plugin opens that connection on start and on every `/auroraauth reload` with a test name, and the budget is 3 seconds to connect and 4 for the request, with 4.5 seconds of waiting during login; one failure counts for 5 seconds and the global wait is 3. A name Mojang confirmed as paid is noted in `accounts.db` (the `premium_present` table, name and date only) and is challenged right away on the way back, with no network at all: even with the catalogue down, the owner still gets in alone. Catalogue answers are cached for 15 minutes for a paid name and 30 for a free one, and the reload clears both.

In the panel, the clear-block button now touches the block that is actually holding the player out. There were two locks, one in memory per player and one in the database per IP, and the profile only released the first: the admin clicked, got the confirmation and the player still fell on the access-blocked screen. The profile now releases both and shows the status and strikes of the account's last IP. And the IP screen became the entry log, not just the trouble list. Before, a row was only born at the first excess, so someone who simply touched a clean server never appeared; now every connection that reaches the login writes the IP, the name used in the attempt, the client's UUID and the time — no new config key and no switch to turn it off. A strike is still a strike: an attempt never zeroes, pushes or clears a block. The list brings in every IP in the database — blocked, banned, expired or already free — ordered by whichever activity is more recent, with a search that takes part of an IP and a refresh button, and every IP opens the screen with the head of whoever tried it last, the strikes, the status and two actions: **Clear block** frees the time right away and keeps the strikes, **Delete record** drops the row and zeroes them. That is how you catch an impostor before they authenticate: a paid name being tried by somebody else shows in the list without a single wrong password.

**1.10.1** — `config.yml` comments rewritten to explain what each option actually does, including what changes if you turn it off. Configuration documentation split out of the README.

**1.10.0** — Configuration reorganised into sections, with a header and one explanation per option, surviving any write made by the panel or the commands. Automatic migration of old configurations, with a backup in `config.yml.bak`. The Mojang session server address left the file: the check always queries the official server and there is no way to redirect it.

**1.9.2** — Breathing room on both sides of the panel's option grid.

**1.9.1** — Alignment of the first row of options in each category.

**1.9.0** — Progressive strikes per IP: 5-minute kick, 15-minute kick and 24-hour IP ban, all adjustable. Ban screen showing the IP, the time left, the reason and the Discord invite. List of banned IPs with unbanning from the panel and automatic expiry.

**1.8.3** — Aliases for every admin subcommand, without `/auroraauth` in front. The automatic login command follows the language too. While typing a password, tab stopped suggesting player names.

**1.8.2** — `/autologin on|off` controls both automatic logins at once and, with no argument, shows the current state.

**1.8.0** — Automatic login for Bedrock with Floodgate, identifying through Floodgate's API rather than a prefix in the name.

**1.7.3** — Paid account identified only in the database and in the panel profile, no badge in chat or in tab.

**1.7.2** — The reset temporary password is click to copy, and the player who was reset stays online.

**1.7.0** — Paid-account verification inside the plugin itself, single jar, no auxiliary protocol library, working on offline servers.

**1.5.5** — Automatic login for paid accounts, linking the account that already existed instead of duplicating it.

**1.5.0** — Authentication commands in the server's language, with the old names ceasing to exist after a switch.

**1.4.0** — **Players** tab: paginated account list of online and offline players, search by name in chat, full profile and admin actions. **System** tab with audit, panel sounds and reload.

**1.3.0** — Graphical in-game panel, by category, with toggles and numeric settings saving on the spot.

**1.2.0** — Portuguese, English and Russian, configurable pre-login effects, reduced command list until authentication, clickable chat buttons and a countdown in the action bar.

**1.1.0** — IP sessions, brute-force protection, password policy, interaction blocking before login, separate admin permissions, account deletion with confirmation and audit.

**1.0.0** — Registration, login, password change, logout and basic administration.

## Support

Found a bug, have a suggestion, or want to understand a strange behaviour before assuming it's the plugin? Open an issue in this repository, preferably with your Paper version, the plugin version and the console output from where it went wrong.

Site: https://auroraplugins.duckdns.org

Made by Kat.
