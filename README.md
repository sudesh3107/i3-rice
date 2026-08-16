# i3-rice — BreadOnPenguins

A dwm-inspired i3 rice themed after the BreadOnPenguins echo colorscheme.

![colorscheme](https://img.shields.io/badge/colorscheme-BreadOnPenguins%20echo-F38B58)

- dwm-style keybindings (`mod4`), split layout (no tabs), gaps
- pywal-driven theming: wallpaper → palette → i3, i3status-rust, st/dmenu (xrdb), alacritty
- square corners everywhere, dual-kawase blur (picom, glx backend)

## Dependencies

```
i3 (4.23+)        window manager
st                terminal (custom build, see below)
dmenu 5.4+        launcher (built from git.suckless.org, stock = no -c patch)
i3status-rust     status bar (v0.36.1, manual install)
picom             compositor (v10, glx backend)
pywal16           wal (fork: https://github.com/eylles/pywal16) — install to ~/.local/bin
feh               wallpaper
dunst             notifications
xss-lock + i3lock screen lock
playerctl         media keys
flameshot         screenshots
pcmanfm, nvim, htop, qutebrowser, termusic, darktable — app bindings
```

Install `wal` (pywal16) into `~/.local/bin` so it shadows any distro pywal.

## Installation

```bash
# 1. clone the dots
git clone https://github.com/sudesh3107/i3-rice
cd i3-rice

# 2. copy configs
cp -r .config/i3 ~/.config/
cp -r .config/i3status-rust ~/.config/
cp -r .config/picom ~/.config/

# 3. copy scripts into PATH
mkdir -p ~/.local/bin
cp scripts/wallpapermenu scripts/wal-i3colors ~/.local/bin/
cp scripts/pywal16 ~/scripts/   # or anywhere; adjust SCRIPT= in wallpapermenu

# 4. i3status-rust theme files MUST exist on disk (manual installs ship none)
mkdir -p ~/.config/i3status-rust/themes
cp .config/i3status-rust/themes/*.toml ~/.config/i3status-rust/themes/

# 5. wallpaper + st/dmenu
mkdir -p ~/Pictures/wallpapers && cp wallpapers/* ~/Pictures/wallpapers/
# the i3 config autostarts feh with ~/Pictures/wallpapers/squarebw.jpg (included,
# a 4K-downscaled copy of the original 173MB PNG — drop in your own files too)
# build st (custom build with Ctrl+Shift+C/V copy-paste) and dmenu 5.4

# 6. reload
i3-msg restart
```

## Keybindings

| Key | Action |
|---|---|
| mod+Return | terminal (st) |
| mod+d | dmenu launcher |
| mod+j / mod+k | focus down / up |
| mod+h / mod+l | resize width (dwm setmfact) |
| mod+1-0 | workspaces |
| mod+q / mod+Shift+q | kill / kill others |
| mod+Control+Shift+q | restart i3 |
| mod+Shift+BackSpace | exit nagbar |
| mod+space | move window center |
| mod+Shift+w | wallpaper menu (pywal) |
| mod+F1 / mod+Shift+F1 | flameshot gui / full |
| mod+F5 / F6 | volume down / up |
| mod+F8 | i3lock |
| mod+F10 / F11 / F12 | playerctl prev / play-pause / next |
| XF86Audio* | volume + media keys |
| mod+minus / mod+equal | gaps inner -/+ |
| mod+Shift+minus / + | gaps reset 10 / 0 |
| mod+Shift+b | bar toggle |

## Wallpaper → colors pipeline

```
mod+Shift+w (wallpapermenu)
  └─ XDG_CURRENT_DESKTOP=i3 wal -i <image> -o ~/scripts/pywal16
       ├─ wal generates palette, sets wallpaper via feh (--bg-fill)
       └─ pywal16: xrdb (st/dmenu) + alacritty colors
            └─ wal-i3colors: rewrites # wal-begin…# wal-end block in i3/config
                 + overrides line in i3status-rust/config.toml
                      └─ i3-msg restart (new bar colors, windows preserved)
```

pywal color mapping: `$bg`=background, `$fg`=foreground, `$border`=color8,
`$urgent`=color3, selbg=fg / selfg=bg. i3status-rust: idle=fg, info=color4,
good=color2, warning=color3, critical=color1, separator=color8.

## Gotchas (hard-learned)

- **i3status-rs 0.36.1 schema**: no `[theme] id`, use `theme = "plain"` + single-line
  `overrides` (multiline inline tables and trailing commas fail to parse).
  Placeholders use `$` syntax (`$utilization`), not `{...}`. Theme files must exist on disk.
- **picom**: MUST use `backend = "glx"` — `egl` black-screens on NVIDIA X sessions.
  libconfig parser rejects trailing commas in lists.
- **XDG_CURRENT_DESKTOP gotcha**: an i3 session started from GDM inherits
  `XDG_CURRENT_DESKTOP=zorin:GNOME`, so `wal` writes the background to GNOME dconf
  and nothing changes on the X root. wallpapermenu runs wal as
  `XDG_CURRENT_DESKTOP=i3 wal -i …` to force the feh path.
- **dmenu colors in bindings**: quote hex colors (`-nb '#30272b'`) — an unquoted
  `#` is eaten as a shell comment by i3's `sh -c`, truncating the command.
- **i3 include directive** does not propagate variable definitions — colors live in
  the `# wal-begin`/`# wal-end` marker block instead.
- **sound block can freeze** at the value sampled at session boot (Pulse subscription
  goes stale) — fix with `i3-msg restart`.
- Status bar restart: `pkill i3status-rs` alone does not reliably bring the bar
  back — use `i3-msg restart`.

See `AGENTS.md` in this repo for the full session memory / troubleshooting notes.

## Credits

- BreadOnPenguins echo colorscheme
- [pywal16](https://github.com/eylles/pywal16) wallpaper + palette generator
- [i3status-rust](https://github.com/greshake/i3status-rust)
- suckless st / dmenu