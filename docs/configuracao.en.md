[Português](configuracao.md) · **English** · [Русский](configuracao.ru.md)

# Configuration

`config.yml` sits in `plugins/AuroraLogin/`. Every option already carries an explanation inside the file itself, so you can work out what a field does without opening any documentation.

If you would rather click than edit text, `/auroraauth panel` does the same thing from inside the game: anything you save there goes into the file immediately, without restarting the server. And because the panel rewrites only the line holding the value that changed, all the comments are still there afterwards.

A server that was already running with an old configuration is updated on its own the first time it boots after you change version: the plugin reuses your values, writes the previous file as `config.yml.bak` and puts the new commented one in place. The comments are written without accents on purpose, so the file opens correctly in any editor, including the ones that save as ANSI. The text the player actually sees, which does carry accents, lives in the files under `lang/`.

## The whole file

The block below is the file with its comments translated, so you can read it without switching windows. One catch worth knowing: the file the plugin actually writes in `plugins/AuroraLogin/` keeps these comments in Portuguese, because there is a single template inside the jar — so copy values from here, not the whole block.

```yaml
# ============================================================
#  AuroraLogin - configuration file
# ============================================================
#
#  Everything the plugin does on your server is here. You can
#  edit it by hand or open the in-game panel with /auroraauth
#  panel and click through the options.
#
#  Edited this file? Run /auroraauth reload. No need to restart
#  the server.
#
#  When the panel changes an option, the plugin rewrites only
#  that value's line. So the comments and the order of the
#  sections stay put afterwards.
#
#  On your server these comments come out in Portuguese and
#  without accents: there is a single template inside the jar,
#  and that way the file opens correctly in any editor,
#  including the ones that save as ANSI. The text the player
#  sees, which does carry accents, lives in the files under lang/.
#
#  Site, support and a detailed explanation of each option:
#  https://auroraplugins.duckdns.org
# ============================================================

# Version of this format. It lets the plugin notice, after an
# update, that the old file needs the new comments. Leave it as
# it is.
config-version: 1

# ------------------------------------------------------------
#  Login and account
# ------------------------------------------------------------
settings:
  # Language of the whole server: messages, panel and even the
  # names of the login commands. Switching to en turns
  # /registrar into /register and /entrar into /login.
  # Options: pt_br, en, ru.
  default-language: pt_br

  # A paid Minecraft account gets in on its own, without
  # registering and without typing a password. Turn it off if you
  # want passwords for everyone, including the people who bought
  # the game.
  premium-auto-login: true

  # Before challenging a client, the plugin asks Mojang whether
  # that name belongs to a paid account. A name nobody bought gets
  # no challenge at all: the player lands in the password flow on
  # the very first join, with no launcher warning. A paid name does
  # get challenged, and that is what lets the real owner walk in on
  # their own. Someone using a paid name without owning the account
  # sees the warning exactly once; on the next connection the
  # plugin knows the name and the IP, stops challenging, and the
  # player registers normally. That record clears with
  # /auroraauth premium reset <name or ip>. If Mojang is
  # unavailable, nobody is challenged at that moment and the login
  # goes through the password flow without stalling. With a proxy
  # doing the checking, this switches itself off.
  premium-verification: true

  # Only has an effect with Floodgate installed. With it, the
  # player on mobile, console and Windows gets in without a
  # password. Without Floodgate this option sits still and the
  # bedrock player registers with a password like anyone else.
  floodgate-auto-login: true

  # Session per IP: someone who already logged in comes back over
  # the same IP and gets straight in, no password again. Great for
  # the player, bad if your server runs in a lan house, where
  # everybody shares the same IP.
  session-enabled: true

  # How long the session is worth, in minutes. After that, the
  # password is asked for again.
  session-minutes: 30

  # Time, in seconds, the player has to authenticate after
  # joining. Once it runs out, they are disconnected.
  timeout-seconds: 60

  # Password size, applying to registration and to changing it.
  password-min-length: 6
  password-max-length: 64

  # Does not allow your own name as the password, which is the
  # first guess of anyone trying to get into another person's
  # account.
  password-deny-name: true

  # When changing the password, does not let you go back to the
  # one that was there before.
  password-reuse: true

  # Did the player walk before logging in? They go back to the
  # spot where they joined. Turn it off if you prefer leaving them
  # standing where they are.
  teleport-on-move: true

  # Before authenticating, the player opens no inventory, breaks
  # or places no block, uses no item and no entity. This is the
  # protection for someone who gets into another person's account
  # and immediately walks off with the chest.
  block-interactions: true

  # Effects on anyone who has not authenticated yet, so the person
  # realises they are waiting on login instead of thinking the
  # server froze. Each one turns on and off separately.
  pre-login-effects:
    # Master key: with this off, no effect is applied at all.
    enabled: true
    # Blinds the player.
    blindness: true
    # Makes them walk slowly.
    slowness: true
    # Slowness level, from 1 to 5.
    slowness-level: 2
    # Takes longer to break a block.
    mining-fatigue: true
    # Mining fatigue level, from 1 to 5.
    mining-fatigue-level: 1

# ------------------------------------------------------------
#  Security
# ------------------------------------------------------------
security:
  # Protection against anyone guessing passwords one at a time.
  # The block is per IP and it grows every time the limit
  # overflows: first time a short kick, second a longer kick,
  # third an IP ban.
  brute-force:
    # Master key of the protection.
    enabled: true
    # How many wrong passwords the IP may send, inside the window
    # below, before it takes the first strike.
    max-attempts: 5
    # Counting window, in minutes.
    window-minutes: 5
    # Kick length on the first excess, in minutes.
    block-minutes: 5
    # Kick length on the second excess, in minutes.
    second-kick-minutes: 15
    # Length of the IP ban on the third excess, in hours. The ban
    # lifts on its own after that, with nobody needing to unban.
    ban-hours: 24
    # After this long without any excess, the IP's strikes go back
    # to zero. In hours (168 = one week).
    strike-reset-hours: 168
    # With it off, the third time becomes just another kick at the
    # second length instead of banning the IP.
    ip-ban: true
    # Discord invite link, shown on the ban screen so the player
    # has somewhere to appeal. Leave it empty ('') to show no
    # button at all.
    discord-invite: ''

# ------------------------------------------------------------
#  Interface
# ------------------------------------------------------------
visual:
  # Title on screen when the player joins and when they authenticate.
  titles: true
  # Sound on register, login and password change.
  sounds: true
  # Action bar with the time left to authenticate.
  action-bar: true
  # How often the bar is refreshed, in ticks (20 ticks = 1
  # second). 40 means every 2 seconds.
  action-bar-interval-ticks: 40

# ------------------------------------------------------------
#  Panel
# ------------------------------------------------------------
panel:
  # Click sound when navigating /auroraauth panel.
  sounds: true

# ------------------------------------------------------------
#  Administration
# ------------------------------------------------------------
admin:
  # Writes to audit.log what the administration did: who reset a
  # password, removed an account, disconnected a player or unbanned
  # an IP. A password never appears there.
  audit: true

# ------------------------------------------------------------
#  Storage
# ------------------------------------------------------------
storage:
  # SQLite database inside plugins/AuroraLogin/, holding the
  # accounts, the sessions, the banned IPs and the record of
  # clients with no Mojang session. For a backup, copy this file
  # with the server stopped.
  file: accounts.db
```

## Login and account

`default-language` picks the language of the whole server: messages, panel and even the names of the authentication commands. Switching to English turns `/registrar` into `/register` and `/entrar` into `/login`; the old names stop existing immediately.

`premium-auto-login` is the automatic entry for paid Minecraft accounts. The player arrives, the plugin checks the account and opens the door without asking for a registration or a password. `premium-verification` is the check itself, the thing that makes that work on a server with `online-mode=false`: before it challenges a client it asks Mojang whether that name belongs to a paid account, and a name nobody bought never gets the challenge at all — it goes straight into the password flow, on the first join. If you run Velocity or BungeeCord doing that verification, the plugin notices and switches itself off, though you can turn it off by hand too. `floodgate-auto-login` is the same idea for anyone joining over Bedrock, and it only counts when Floodgate is installed.

One case is left over: a paid name used by somebody who doesn't own the account. Their launcher gives up at the encryption request with the *Invalid session* message, and that happens exactly once. The plugin records the client that died in the handshake — name and IP — and stops challenging it: the next connection goes straight to registration or login without touching Mojang. `/auroraauth premium` shows what is recorded, `/auroraauth premium reset <name|ip>` clears it, and after a reset that client is challenged again. A client that answers the encryption request never ends up in that record, and neither does a client whose name nobody bought, because it was never challenged. If Mojang doesn't answer, nobody is challenged at that moment: the plugin waits a few seconds, warns once in the console, and the login carries on normally without stalling. `/auroraauth reload` clears the answers that stayed in cache.

`session-enabled` together with `session-minutes` lets a player come back over the same IP without retyping the password until the session expires. `timeout-seconds` is how long they have to authenticate after joining before they get disconnected.

The four password fields (`password-min-length`, `password-max-length`, `password-deny-name`, `password-reuse`) build the policy: length, not allowing a password equal to the name, and not letting someone go back to the password they had before when changing it.

`teleport-on-move` returns the player to the spot where they joined if they walk before authenticating. `block-interactions` is what stops inventory, breaking blocks and item use during that window. And `pre-login-effects` applies blindness, slowness and mining fatigue to anyone still waiting on login, so they understand they are being held rather than thinking the server froze; the levels run from 1 to 5, and each effect turns on and off on its own.

## Security

The brute-force protection works per IP and escalates. `max-attempts` is how many wrong passwords fit inside `window-minutes`; once it overflows, the IP takes its first strike and is kicked for `block-minutes`. The second strike kicks for `second-kick-minutes`. The third applies the IP ban for `ban-hours`, as long as `ip-ban` is on; with it off, the third time is just another kick at the second duration. `strike-reset-hours` is how long an IP has to stay under the limit for its strikes to return to zero.

The ban expires on its own after `ban-hours`, and it deletes neither accounts nor passwords: the person can play again when the time passes or when you free the IP. The path is **Security → IPs and strikes** in the panel, which lists every IP in the database — blocked, banned, expired or already free — with its strikes, status and the date of the last excess, newest first. The search takes part of an IP, the refresh button reloads without closing the menu, and each IP opens a screen with two actions: **Clear block** frees the time right away and keeps the strikes, **Delete record** removes the IP from the database and zeroes the strikes. A ban whose time has run out stays in the list marked as expired; it disappears on its own once the IP goes `strike-reset-hours` without another excess.

`discord-invite` takes the link to your Discord. It is shown on the ban screen, next to the IP and the time left, so the player has somewhere to go. Leave it `''` to show no button at all.

## Interface, panel and administration

`visual.titles`, `visual.sounds` and `visual.action-bar` control what appears on screen: the title on join and on authenticating, the sounds for the login actions, and the action bar with the time remaining. `action-bar-interval-ticks` is how often that bar is refreshed, in ticks, 20 ticks being one second.

`panel.sounds` turns the click on or off when navigating the panel. `admin.audit` writes the administration's actions to `plugins/AuroraLogin/audit.log`: who reset a password, removed an account, disconnected a player, freed an IP or deleted its record. A password is never written there.

## Storage

`storage.file` is the SQLite database inside `plugins/AuroraLogin/`, holding the accounts, the sessions and the IPs with strikes. For a backup, copy that file with the server stopped.

## Language files

The messages live in `plugins/AuroraLogin/lang/`, one file per language: `pt_br.yml`, `en.yml` and `ru.yml`. You can edit the text as much as you like, including the authentication command names, which sit in the keys `command-register`, `command-login`, `command-logout` and `command-changepassword`. After editing, run `/auroraauth reload` to apply.
