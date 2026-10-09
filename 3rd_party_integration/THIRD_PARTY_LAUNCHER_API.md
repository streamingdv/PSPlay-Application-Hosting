# Starting PXPlay from another application

For a launcher or frontend that wants to stream a PlayStation console through PXPlay: PXPlay is asked for one
console, it streams it, and the process ends when the session does.

Two things shape the interface. The Windows executable is a windowed binary, so nothing it prints can be read
by the caller: what you get back is an **exit code** and a few small **files PXPlay writes**. And everything
here sits behind one token, `--integration-api=1`, so a PXPlay older than your launcher, or a launcher older
than PXPlay, is never answered in a language it does not speak.

## The command line

`--key=value` and `--key value` are both accepted. Anything PXPlay does not recognize is ignored.

| Argument | Meaning |
|---|---|
| `--integration-api=1` | Opts into everything on this page. Without it PXPlay behaves exactly as it did before this interface existed. A version PXPlay does not speak exits `10`. |
| `--launch-mode=direct-stream` | Stream one console instead of showing the application. |
| `--console-id=<mac>` | Which console. `AA:BB:CC:DD:EE:FF`, `aa-bb-cc-dd-ee-ff` and `aabbccddeeff` all name the same one. Required by `direct-stream`. |
| `--launcher=<name>` | Who is starting PXPlay, for the log. It grants nothing. Pick any name; `steam` and `desktop` are the entries PXPlay writes for itself, so use your own instead of one of those. |
| `--on-stream-exit=standby\|nothing` | What happens to the console when the stream ends. Worth passing: the shipped default asks the user in a dialog, which is not what you want in a window you are about to close. |
| `--window-mode=window\|fullscreen\|borderless` | How the stream window opens. `window` is a normal window, as large as the monitor allows. `fullscreen` is the fullscreen of the platform, with the screen mode change that comes with it. `borderless` covers the monitor without a title bar and without a mode change, so other windows stay switchable. Left out, the user's own setting decides. |
| `--panel-window-mode=borderless\|fullscreen` | What kind of window the **loading panel** covers the screen with. Only means something for a stream that opens in the fullscreen of the platform, and you almost certainly do not need it. See [the loading panel and the screen](#the-loading-panel-and-the-screen). |
| `--console-ip=<a.b.c.d>` | Connect to this address and nothing else, waking the console first if it is in rest mode. A modifier on `--console-id`, which still has to be there. Read [how the console is found](#how-pxplay-finds-the-console) before using it. |
| `--title-id=<id>` | Ask the console to start this game before the stream opens, so the picture arrives on the game. PS5 only. See [starting a game](#starting-a-game-with-the-stream). |
| `--activate-license --license-email=<address> --license-key=<key>` | Activate a license and exit without starting anything. Cannot be combined with `direct-stream`: one invocation has one outcome, so one exit code can describe it. |

`--on-stream-exit`, `--window-mode`, `--panel-window-mode` and `--title-id` hold for the one session they were
passed for. Nothing on this page is written to the user's preferences, and the next start is unaffected.

The call you will make for every launch:

```
PXPlay.exe --integration-api=1 --launcher=mylauncher --launch-mode=direct-stream ^
           --console-id=aabbccddeeff --on-stream-exit=standby --window-mode=borderless
```

A stream started this way opens without any of PXPlay around it: a loading panel while the console is found
and connected to, then the picture. While that panel is up it carries a small PXPlay logo and "Remote Play by
PXPlay" in the bottom middle, so a user who started the console from your library knows what is streaming it.
It goes as soon as the picture arrives.

PXPlay puts that window in front itself, and the stream window after it, so there is nothing for you to raise.
What it does not do is stay there: it asks for the front rather than holding it, so a launcher that raises its
own window *after* starting PXPlay ends up in front of the stream, and in front of a message waiting to be
acknowledged. Raise yourself before the start, not after.

## The loading panel and the screen

When the start covers the screen, whichever of the two fullscreens that is, the loading panel covers it with a
frameless window the size of the monitor rather than by asking the platform for a fullscreen. It looks the same
and it behaves better: a window the platform keeps fullscreen is one it also lowers the moment the focus leaves
it, and the focus goes to the stream window while that is being built, which had the panel blinking away in the
middle of the connection. Only the stream window asks the platform for the real thing, because by the time it
appears it is the window with the focus.

`--panel-window-mode=fullscreen` asks for the real one anyway:

```
PXPlay.exe --integration-api=1 --launcher=mylauncher --launch-mode=direct-stream ^
           --console-id=aabbccddeeff --window-mode=fullscreen --panel-window-mode=fullscreen
```

It is honoured only for a stream that opens in the fullscreen of the platform, because a panel is never more of
a window than the stream behind it. That means either `--window-mode=fullscreen`, or no `--window-mode` at all
on a machine where the user configured a real fullscreen themselves. With `borderless` or `window` it changes
nothing. Left out, you get the frameless window described above, which is what every launch got before this
argument existed.

The blinking is worked around rather than avoided in this mode: the panel is held above other windows for as
long as it is the window on the screen, and let go of the moment it steps aside for the stream or has something
to tell the user, so there is no moment to see. Three things to know if you use it. The panel is therefore
genuinely on top while the console is being found, not merely in front, so a user cannot put another window
over it until the stream arrives — which is the point on a handheld and may not be what you want on a desktop.
A platform may refuse to hold a window in front and say nothing about it, in which case PXPlay takes the front
back within about a frame instead, so a flicker is possible rather than impossible. And on macOS a real
fullscreen is a space of its own, which means the panel opens one space and the stream opens another; it works,
but `borderless` is the smoother choice there.

Reach for this if your own window is a real fullscreen and a frameless window beside it does not sit right, or
on a handheld where the two are visibly different. Otherwise leave it alone.

## Finding PXPlay

On Windows the installer writes a key of its own, in the **64 bit view** of the registry:

```
HKLM\Software\PXPlay
  InstallPath  REG_SZ  C:\Program Files\PXPlay
  Version      REG_SZ  2.1.0
```

`InstallPath` is the directory; the executable is `InstallPath\PXPlay.exe`. Prefer this over the uninstall
entry: that one is called `PXPlay_is1` because of the installer we happen to use today, and both the suffix and
the name in front of it would move if that ever changed, leaving anything keyed to it quietly finding nothing.

Two things about reading it:

- **A 32 bit process must open the key with `KEY_WOW64_64KEY`.** PXPlay installs 64 bit, so a 32 bit reader
  without that flag is redirected to `Software\WOW6432Node\PXPlay`, finds nothing, and concludes PXPlay is not
  installed. On .NET that is `RegistryKey.OpenBaseKey(RegistryHive.LocalMachine, RegistryView.Registry64)`.
- **Installing needs elevation.** It is a per-machine install under `Program Files`, so a one-click install
  from your side gets a UAC prompt whatever else is true, signed installer or not.

## The registered consoles

`consoles.json`, in `integration/` inside the PXPlay base directory:

| Platform | Base directory |
|---|---|
| Windows | `%AppData%\PXPlay` |
| Linux | `~/PXPlay` |
| macOS | `~/Library/Application Support/PXPlay` |

```json
{ "protocol": 1, "updatedAt": 1757000000000, "consoles": [
  { "consoleId": "aabbccddeeff", "displayName": "Living Room",
    "nickname": "PS5-123", "deviceType": "PS5" }
] }
```

Rewritten whenever the user's registrations change and once at every start, so build your picker from it and
nobody has to type a MAC. `consoleId` is what goes into `--console-id`. `displayName` is the name the user
gave the console, falling back to the one it calls itself. `deviceType` is `PS4` or `PS5`. Every field is
always present, and every string except `consoleId` may be empty.

**Pairing stays in PXPlay.** Registering a console needs a PSN sign-in and a PIN read off the console screen,
so there is no command line for it: send the user to PXPlay once and this file is there afterwards. It
deliberately carries nothing that could stream a console on its own — no registration key, no RP key, no PSN
token, and no address.

## The license

PXPlay needs a valid license to stream. Nothing on this page changes that: no argument, file or flag
influences it, and `--launcher` grants nothing. A start without a license exits `11`.

If you sell or bundle keys, `--activate-license` saves sending the user into PXPlay to type one. It starts no
window, and it cannot run while PXPlay is running, which answers `24`:

```
PXPlay.exe --integration-api=1 --launcher=mylauncher --activate-license ^
           --license-email=user@example.com --license-key=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
```

`0` means the license is active on this machine now. `20` means it already had one and your key was left
unused, so running this at every start is safe. `21` to `24` say what else was wrong. The key is passed on the
command line and nowhere else: it reaches no log file and none of the files described below.

Before it leaves it rewrites `consoles.json`, and `status.json` as well whenever it got as far as asking the
server, so after a `0` you can confirm the license and build your console picker straight away instead of
sending the user into PXPlay for a list that is already known.

## How PXPlay finds the console

The same sequence every stream started from PXPlay's own window goes through. It decides which exit code you
get and how long you wait.

1. The MAC is matched against the registered consoles. No match is `12`, a match whose registration is
   unusable is `13`.
2. The console is looked for on the local network for **5 seconds**. One that answers from standby is woken
   first, with the PSN credentials of the account it is registered to. There is nothing to send beforehand.
3. If it did not answer, a connection **over the internet is tried automatically**. That needs the remote
   connection enabled in PXPlay and working PSN credentials for the account the console is registered to.
   Without either you get `15` rather than a wait.
4. Steps 2 and 3 together are capped at **30 seconds**, after which you get `14`. A console that answered
   from standby is given **50 seconds** instead, since that is what coming out of rest mode can take.

`--console-ip` replaces steps 2 and 3 with that one address. PXPlay asks the console there what state it is
in, which took 0.6 seconds when it answered and 1.5 when nothing did, and then:

- **awake**: it connects straight away.
- **in rest mode**: it is woken exactly as in step 2 and the stream starts once the console reports itself
  up, within the same 50 seconds.
- **no answer**: it connects anyway, because an address can let a stream through while answering no questions
  of its own.

Nothing is searched for on the network in any of the three, and there is no remote fallback either, so an
address with nothing on it runs into the session request timeout, which was 72 seconds when measured, and
fails as `16`.

## Starting a game with the stream

`--title-id=<id>` asks the console for a game before anything looks for it on the network, so the picture
arrives on the game instead of on the home screen:

```
PXPlay.exe --integration-api=1 --launcher=mylauncher --launch-mode=direct-stream ^
           --console-id=aabbccddeeff --title-id=PPSA01234_00 --on-stream-exit=standby
```

The command goes to the console through PSN, not over your network, so it reaches one that is still in rest
mode and brings it up on the game. The id is Sony's, as `PPSA01234_00` or `CUSA12345`; case does not matter.

**PS5 only.** Sony's launch endpoint speaks for PS5 consoles, so a PS4 named here is refused with `18` rather
than streamed. It also needs the PSN sign-in of the account the console is registered to, which PXPlay already
has if the user set up remote connection or ever looked at their games.

Two outcomes to tell apart, because they are not the same thing:

- **PXPlay could not ask at all** — a PS4, no PSN sign-in for that console, or PXPlay cannot work out which
  console on the account this is. Exit code `18`, and **nothing is streamed**: a game your user picked from a
  library is not answered with a home screen, and all three are things they can fix. The message on screen says
  which of the three it was.
- **The console was asked and did not start it** — PSN or the console turned the command down, or it took too
  long. The console is **streamed anyway**, exactly as it is when a game is started from PXPlay's own window,
  and `titleLaunched: false` in `session.json` says the picture is not on the game.

The whole step is capped at **25 seconds**, after which the console is connected to regardless. It runs before
the console is looked for, so it adds to the waits in the section above.

## Exit codes

The exit code is the authoritative answer about what happened. The numbers never change; new ones are only
ever added.

| Code | Name | What happened |
|---|---|---|
| 0 | `SUCCESS` | the stream ran and the user ended it, or there was nothing to do |
| 1 | `NATIVE_INITIALIZATION_FAILED` | a native library could not be loaded, which stops any kind of start |
| 10 | `INVALID_LAUNCH_ARGUMENTS` | the arguments did not describe something that could be started |
| 11 | `LICENSE_UNAVAILABLE` | no valid license, so there was no stream |
| 12 | `CONSOLE_NOT_FOUND` | no registered console has that MAC |
| 13 | `CONSOLE_CONFIGURATION_INVALID` | it is registered, but the registration lacks what streaming needs |
| 14 | `CONNECTION_FAILED` | the console could not be reached |
| 15 | `REMOTE_CONNECTION_UNAVAILABLE` | not on the local network and no usable remote connection |
| 16 | `STREAM_START_FAILED` | the connection was made but the stream never started |
| 17 | `STREAM_DISCONNECTED_WITH_ERROR` | the stream ran and then the console cut it short |
| 18 | `TITLE_LAUNCH_UNAVAILABLE` | a `--title-id` was given that this console cannot be asked for at all, so nothing was streamed |
| 19 | `INTERNAL_LAUNCHER_ERROR` | something failed in a way it has no code for |
| 20 | `LICENSE_ALREADY_ACTIVE` | activation was asked for on a copy that is already licensed |
| 21 | `LICENSE_EMAIL_MISSING` | activation without an email address |
| 22 | `LICENSE_KEY_INVALID` | no key was passed, or not one this version accepts |
| 23 | `LICENSE_ACTIVATION_REFUSED` | the email and the key were not accepted |
| 24 | `LICENSE_ACTIVATION_UNAVAILABLE` | the activation could not be carried out, and nothing changed |
| 30 | `STREAM_HANDED_OFF` | PXPlay was already running and took the request; **this process exiting is not the end of the stream** |

Everything above `19` is only ever returned when `--integration-api=1` was given. Without it a handed-off
request still exits `0`, exactly as before.

**Code `30` needs handling.** Only one PXPlay may hold the preferences and the console connections, so when
one is already running, the process you started passes the request to it and leaves at once. The stream is in
the other process and `session.json` names it. A handed-off request that never becomes a stream says so in
the same file, with `state: "ended"` and the `exitCode` a launch of its own would have left with, so after
`30` that file is the whole answer and there is nothing for you to time out on.

**A failure is shown to the user before the process leaves.** Anything that goes wrong puts a message in
PXPlay's window and waits for it to be acknowledged, so a launcher closing that window cannot take the
explanation with it. The exit code therefore arrives only once that has been clicked away. You do not have to
wait for it: the reason is in `session.json` as `state: "ended"` with the `exitCode` the moment the failure is
decided, so watch the file and show your own message instead.

## The files PXPlay writes

All in `integration/` inside the base directory named above. Every file carries `protocol` and `updatedAt`,
and each is written beside its own name and then moved over it, so a reader sees the whole of the old one or
the whole of the new one. On a file system that cannot do that in one step, a home directory on a network
share for instance, treat a document that will not parse as one to read again rather than as an error. PXPlay
writes them and never reads them back.

**`session.json`** — written while a stream *you* asked for is running, and never for one the user started
themselves.

```json
{ "protocol": 1, "updatedAt": 1757000000000, "state": "streaming",
  "consoleId": "aabbccddeeff", "pid": 24680, "windowHandle": 1183554,
  "panelWindowHandle": 0 }
```

- `state` is `starting`, `streaming` or `ended`.
- `pid` is the process the stream is in, which after a `30` is the PXPlay that was already running rather than
  the one you started. It is not the process id you got back from starting `PXPlay.exe` either: that
  executable is a thin wrapper that starts PXPlay and waits for it, which is why its exit code is PXPlay's.
  Wait on the process you started, and use this `pid` when you need to name the process the stream is in.
- `windowHandle` is the window the stream is drawn in: the `HWND` on Windows, the X11 window id under X11, and
  `0` on Wayland and macOS, where a window is not something another process can be handed by number. It is
  what to reparent if you show the stream inside your own interface, and only meaningful while `state` is
  `streaming`.
- `panelWindowHandle` is the window the **loading panel** is in, read the same way and `0` on the same
  platforms. It is a different window from `windowHandle`, see [two windows, not one](#two-windows-not-one).
  Non-zero only while `state` is `starting`, which is exactly as long as it is a window worth embedding: `0`
  from `streaming` onwards, and `0` again on `ended`, so a handle you find in a file left behind by a session
  that is over is never one you might reparent after the window was destroyed.
- `titleLaunched` is there only when `--title-id` was given, and says whether the console started that game.
  `true` means the picture is on the game, `false` means it is not and you are looking at whatever the console
  had in front. Written before the console is even looked for, so it is there by the time `state` is
  `streaming`.
- `exitCode` is the only other field that is not always there, and it is written once per session. A launch that
  reports its own outcome, which is every one that did not answer `30`, puts it there with `state: "ended"` as
  soon as that outcome is known. After a `30` the stream is in a process you never started and whose exit you
  will not see, so the code is there for a request that **failed** before it streamed and not for a session
  that ran: watch for `state: "ended"` rather than for a code in that case, since it is the state that tells
  you the session is over.

  With a process of your own to wait on, the reason to read it here is that it is earlier. It is written the
  moment a failure is decided, while the process is still holding the message about it on screen for a user who
  is looking at your window rather than at PXPlay's, so it can be minutes ahead of the exit code that
  eventually says the same thing.

The file is left behind after a session, so compare `updatedAt` against the moment you started PXPlay before
believing a `streaming` you did not cause.

### Two windows, not one

A session uses two top-level windows, one after the other, and if you show PXPlay inside your own interface you
will reparent both:

| | Window | Field | Live while |
|---|---|---|---|
| 1 | the loading panel | `panelWindowHandle` | `state` is `starting` |
| 2 | the stream | `windowHandle` | `state` is `streaming` |

The second is not the first resized. The renderers draw through D3D11, Vulkan or Metal into a window they
create themselves, so it is a new window with a new handle, and the panel is **hidden, not destroyed**, at the
moment it appears. Nothing of it is left on screen behind yours.

So: watch `session.json` rather than reading it once. Take `panelWindowHandle` as soon as it is non-zero, take
`windowHandle` when `state` becomes `streaming`, and expect the two to differ. `updatedAt` moves on every
write, which is what makes watching worthwhile.

The panel window can come back: a failure is shown in it, so if you reparented it, the message appears inside
your interface, which is where the user is looking. `panelWindowHandle` is `0` by then all the same, whether
the failure came before the stream or after it, because a window you were already handed is not one to hand out
again — the handle you took is still that window.

One window for the whole session would be the better contract and is not what PXPlay does today.

**`status.json`** — advisory, for deciding whether to show something of your own, such as a "buy a license"
link. `licensed` is what PXPlay last recorded and it decides nothing: a launch answering `11` is the authority
on whether a stream may start.

```json
{ "protocol": 1, "updatedAt": 1757000000000, "appVersion": "2.1.0", "licensed": true }
```

Rewritten at every start, and by `--activate-license` before it exits.

## Putting it together

1. Read `consoles.json` and let the user pick a console.
2. Start PXPlay with the arguments above and keep the process handle.
3. On exit code `30` the stream is in another process: take `pid` from `session.json` and watch that instead.
4. Watch `session.json` from there. `panelWindowHandle` while `state` is `starting`, then `windowHandle` once
   `state` is `streaming`. A second or two between them is normal, since a native renderer unpacks libplacebo
   first, and longer when you passed `--title-id`.
5. If `state` becomes `ended` with an `exitCode` instead, that is the failure, and you have it before the user
   has dismissed PXPlay's message about it.
6. The exit code of the process you started says how the session ended. `0` means the user ended it. After a
   `30` there is no such code, because the process the stream is in was there before you and stays after: the
   session is over when `state` becomes `ended`.

## What is not there yet

- **One instance at a time.** PXPlay holds the preferences and the console connections, so two streams cannot
  run in parallel. That is what code `30` is about.
- **No remote control of a running session.** There is no way to stop or sleep a running stream other than
  ending the process that holds it, the `pid` in `session.json` — ending the `PXPlay.exe` you started removes
  only the wrapper. Ending it this way also skips what PXPlay does on the way out, including the standby
  request `--on-stream-exit=standby` asked for.
- **No `--parent-hwnd`.** PXPlay opens its own top-level windows and you reparent them, rather than PXPlay
  drawing into one of yours. Two windows over the life of a session, see above.
- **One window for the whole session.** The loading panel and the stream are separate windows, so there are two
  reparents per launch instead of one.
