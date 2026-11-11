# Tile-Toggle-Steam

A tiny script to control how the **Steam** client opens in Hyprland on Omarchy.

Omarchy floats Steam by default, with the main client at 1100x700, centered, and
the Friends List at 460x800. This script keeps a small override in
`~/.config/hypr/steam.lua` so the main Steam window **and Friends List** can open
tiled like most other apps, and lets you flip back to the stock floating
behaviour at any time.

Dialogs and the login window are left alone, so they keep opening as floating
panels.

## Usage

```bash
./tile-toggle-steam            # toggle between tiled and floating
./tile-toggle-steam tile       # main window and Friends List open tiled
./tile-toggle-steam float      # open them floating (Omarchy default)
./tile-toggle-steam status     # show the saved setting and open Steam windows
./tile-toggle-steam uninstall  # remove the override
```

The script reloads Hyprland and updates the already-open Steam windows, so you do
not need to restart Steam or the session.

## What it changes

| File | Purpose |
| --- | --- |
| `~/.config/hypr/steam.lua` | Managed override module (tile rule or comment) |
| `~/.config/hypr/hyprland.lua` | Adds `require("hypr.steam")` once, after making a timestamped backup |

Nothing under `/usr/share/omarchy/` is touched.

## How it works

Omarchy's stock rules live in
`/usr/share/omarchy/default/hypr/apps/steam.lua`:

```lua
o.window("steam", { float = true, idle_inhibit = "fullscreen" })
o.window({ class = "steam", title = "Steam" }, { center = true, size = { 1100, 700 } })
o.window({ class = "steam", title = "Friends List" }, { size = { 460, 800 } })
```

The override adds a later rule that matches the two windows:

```lua
o.window({ class = "steam", title = "^(Steam|Friends List)$" }, { tile = true })
```

Notes on the rule:

- `tile = true` is used instead of `float = false`; a false float rule does not
  unfloat a window, while the `tile` effect does.
- Matching on the titles keeps dialogs and the login window floating. Omarchy
  already uses those titles to size the two windows, so they are stable.
- Window rules are applied when a window opens. The script also dispatches a
  live `float` change to either window if it is already open.
- On float, the main window is restored to 1100x700 and centered, and the
  Friends List to 460x800, matching Omarchy's stock sizes.

## After an Omarchy refresh

`omarchy refresh hyprland` resets `hyprland.lua` and drops the `require` line.
Run any `tile`/`float`/`toggle` command again and the script re-adds it.
`status` warns when the module is not loaded.

## Optional keybinding

Put the script on `PATH`:

```bash
ln -s "$PWD/tile-toggle-steam" ~/.local/bin/tile-toggle-steam
```

Then add a binding in `~/.config/hypr/bindings.lua`:

```lua
o.bind("SUPER + ALT + T", "Toggle Steam tiling", "tile-toggle-steam")
```

`SUPER + ALT + T` was free when this was written; check your own bindings first:

```bash
omarchy menu keybindings --print
```

## Requirements

- Hyprland with the Lua config (Omarchy)
- `hyprctl`
- `jq` (for live window inspection and updates)
