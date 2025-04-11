# TL;DR — Komorebi.exe: the Chad WM for Windows 10+

# komorebi

Tiling Window Management for Windows.

 

![screenshot](https://user-images.githubusercontent.com/13164844/184027064-f5a6cec2-2865-4d65-a549-a1f1da589abf.png)

## Why it's cool:
- **Tiling windows** = less alt-tabbing, more doing.
- **CLI-powered** for automation gods.
- **Plays nice** with AutoHotKey and whkd for custom hotkeys.
- **Zero invasive OS tweaks**. It’s like a guest that brings their own snacks and doesn’t trash your place.

## Core Features (aka the Senpai Stats)

| Feature              | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| 🌐 Multi-monitor     | Manages all your displays like a true command center.                      |
| 🎮 CLI interface     | Script everything. Yes, even window positioning.                           |
| 🎯 Workspace support | Virtual desktops like a boss.                                              |
| 🪄 Low OS interference| Default config = no surprises.                                             |
| 🔌 Extensibility     | Named pipe event subscriptions = roll your own widgets, like a real hacker.|
| 🧱 Status bars       | Integrate your fav bar (yasb, custom JSON readers, etc.)                   |
| 📦 Install via Scoop/Winget | No 5-hour build instructions from the abyss.                      |


# Demonstrations

[@amnweb](https://github.com/amnweb) showing _komorebi_ `v0.1.28` running on Windows 11 with window borders,
unfocused window transparency and animations enabled, using a custom status bar integrated using
_komorebi_'
s [Window Manager Event Subscriptions](https://github.com/LGUG2Z/komorebi?tab=readme-ov-file#window-manager-event-subscriptions).

https://github.com/LGUG2Z/komorebi/assets/13164844/21be8dc4-fa76-4f70-9b37-1d316f4b40c2

[@haxibami](https://github.com/haxibami) showing _komorebi_ running on Windows
11 with a terminal emulator, a web browser and a code editor. The original
video can be viewed
[here](https://twitter.com/haxibami/status/1501560766578659332).

https://user-images.githubusercontent.com/13164844/163496447-20c3ff0a-c5d8-40d1-9cc8-156c4cebf12e.mp4

[@aik2mlj](https://github.com/aik2mlj) showing _komorebi_ running on Windows 11
with multiple workspaces, terminal emulators, a web browser, and the
[yasb](https://github.com/DenBot/yasb) status bar with the _komorebi_ workspace
widget enabled. The original video can be viewed
[here](https://zhuanlan.zhihu.com/p/455064481).

https://user-images.githubusercontent.com/13164844/163496414-a9cde3d1-b8a7-4a7a-96fb-a8985380bc70.mp4


# Window Manager Event Subscriptions

## Named Pipes

It is possible to subscribe to notifications of every `WindowManagerEvent` and `SocketMessage` handled
by `komorebi` using [Named Pipes](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes).

First, your application must create a named pipe. Once the named pipe has been created, run the following command:

```powershell
komorebic.exe subscribe-pipe <your pipe name>
```

Note that you do not have to include the full path of the named pipe, just the name.

If the named pipe exists, `komorebi` will start pushing JSON data of successfully handled events and messages:

```json lines
{"event":{"type":"AddSubscriber","content":"yasb"},"state":{}}
{"event":{"type":"FocusWindow","content":"Left"},"state":{}}
{"event":{"type":"FocusChange","content":["SystemForeground",{"hwnd":131444,"title":"komorebi – README.md","exe":"idea64.exe","class":"SunAwtFrame","rect":{"left":13,"top":60,"right":1520,"bottom":1655}}]},"state":{}}
{"event":{"type":"MonitorPoll","content":["ObjectCreate",{"hwnd":5572450,"title":"OLEChannelWnd","exe":"explorer.exe","class":"OleMainThreadWndClass","rect":{"left":0,"top":0,"right":0,"bottom":0}}]},"state":{}}
{"event":{"type":"FocusWindow","content":"Right"},"state":{}}
{"event":{"type":"FocusChange","content":["SystemForeground",{"hwnd":132968,"title":"Windows PowerShell","exe":"WindowsTerminal.exe","class":"CASCADIA_HOSTING_WINDOW_CLASS","rect":{"left":1539,"top":60,"right":1520,"bottom":821}}]},"state":{}}
{"event":{"type":"FocusWindow","content":"Down"},"state":{}}
{"event":{"type":"FocusChange","content":["SystemForeground",{"hwnd":329264,"title":"den — Mozilla Firefox","exe":"firefox.exe","class":"MozillaWindowClass","rect":{"left":1539,"top":894,"right":1520,"bottom":821}}]},"state":{}}
{"event":{"type":"FocusWindow","content":"Up"},"state":{}}
{"event":{"type":"FocusChange","content":["SystemForeground",{"hwnd":132968,"title":"Windows PowerShell","exe":"WindowsTerminal.exe","class":"CASCADIA_HOSTING_WINDOW_CLASS","rect":{"left":1539,"top":60,"right":1520,"bottom":821}}]},"state":{}}
```

You may then filter on the `type` key to listen to the events that you are interested in. For a full list of possible
notification types, refer to the enum variants of `WindowManagerEvent` in `komorebi` and `SocketMessage`
in `komorebi::core`.

Below is an example of how you can subscribe to and filter on events using a named pipe in `nodejs`.

```javascript
const { exec } = require("child_process");
const net = require("net");

const pipeName = "\\\\.\\pipe\\komorebi-js";
const server = net.createServer((stream) => {
  console.log("Client connected");

  // Every time there is a workspace-related event, let's log the names of all
  // workspaces on the currently focused monitor, and then log the name of the
  // currently focused workspace on that monitor

  stream.on("data", (data) => {
    let json = JSON.parse(data.toString());
    let event = json.event;

    if (event.type.includes("Workspace")) {
      let monitors = json.state.monitors;
      let current_monitor = monitors.elements[monitors.focused];
      let workspaces = monitors.elements[monitors.focused].workspaces;
      let current_workspace = workspaces.elements[workspaces.focused];

      console.log(
        workspaces.elements
          .map((workspace) => workspace.name)
          .filter((name) => name !== null)
      );
      console.log(current_workspace.name);
    }
  });

  stream.on("end", () => {
    console.log("Client disconnected");
  });
});

server.listen(pipeName, () => {
  console.log("Named pipe server listening");
});

const command = "komorebic subscribe-pipe komorebi-js";

exec(command, (error, stdout, stderr) => {
  if (error) {
    console.error(`Error executing command: ${error}`);
    return;
  }
});
```

## Unix Domain Sockets

It is possible to subscribe to notifications of every `WindowManagerEvent` and `SocketMessage` handled
by `komorebi` using [Unix Domain Sockets](https://devblogs.microsoft.com/commandline/af_unix-comes-to-windows/).

UDS are also the only mode of communication between `komorebi` and `komorebic`.

First, your application must create a socket in `$ENV:LocalAppData\komorebi`. Once the socket has been created, run the
following command:

```powershell
komorebic.exe subscribe-socket <your socket name>
```

If the socket exists, komorebi will start pushing JSON data of successfully handled events and messages as in the
example above in the Named Pipes section.

## Rust Client

As of `v0.1.22` it is possible to use the `komorebi-client` crate to subscribe to notifications of
every `WindowManagerEvent` and `SocketMessage` handled by `komorebi` in a Rust codebase.

Below is a simple example of how to use `komorebi-client` in a basic Rust application.

```rust
// komorebi-client = { git = "https://github.com/LGUG2Z/komorebi", tag = "v0.1.35"}

use anyhow::Result;
use komorebi_client::Notification;
use komorebi_client::NotificationEvent;
use komorebi_client::UnixListener;
use komorebi_client::WindowManagerEvent;
use std::io::BufRead;
use std::io::BufReader;
use std::io::Read;

pub fn main() -> anyhow::Result<()> {
  let socket = komorebi_client::subscribe(NAME)?;

  for incoming in socket.incoming() {
    match incoming {
      Ok(data) => {
        let reader = BufReader::new(data.try_clone()?);

        for line in reader.lines().flatten() {
          let notification: Notification = match serde_json::from_str(&line) {
            Ok(notification) => notification,
            Err(error) => {
              log::debug!("discarding malformed komorebi notification: {error}");
              continue;
            }
          };

          // match and filter on desired notifications
        }
      }
      Err(error) => {
        log::debug!("{error}");
      }
    }
  }

}
```

A read-world example can be found
in [komokana](https://github.com/LGUG2Z/komokana/blob/feature/komorebi-uds/src/main.rs).

## Licensing & Usage
- **Personal use**: Free, educational, but no redistribution/hard-forking.
- **Commercial use**: Requires a paid license (respect the hustle).
- **Moral clause**: Consider donating to Gaza relief before supporting dev. Mad respect for that.

## Integration Potential (for mad dev scientists 🧪)
- Real-time event hooks via named pipes? **Check.**
- Exportable state as JSON? **Check.**
- Modding-friendly? **Triple check.** Fork away for personal tweaks or PR-worthy enhancements.
- Easter egg releases on physical media? **Yep.** That’s peak hipster-dev energy.

## Why Not FancyZones?
Let’s be honest — FancyZones is great if your idea of "customization" is dragging boxes with a mouse and calling it productivity. Komorebi is for devs who talk in keybinds and breathe JSON configs.

## Final Verdict
- **Rating**: 10/10 Would Ctrl+Win+H again.

Komorebi is like an anime protagonist’s training arc. It starts quiet and unassuming, then hits you with high-octane efficiency and a sleek layout that makes you feel like you’re hacking the Matrix — even if you’re just moving Notepad.

# Appreciations 
-- to the original komorebi creater i just modified a little 😼
🔍 **See the original documentation and get started here:** [https://github.com/LGUG2Z/komorebi](https://github.com/LGUG2Z/komorebi)
