# ginger's system — i3/BreadOnPenguins themed setup (Zorin OS 24.04 based)

This file gives new opencode sessions instant context. Read it before helping with any
i3/window-manager/theming work. The user should NOT have to re-explain any of this.

## System

- Linux (Ubuntu 24.04 base), Zorin OS, GDM login manager, hybrid laptop (Intel iGPU + NVIDIA dGPU, NVIDIA driver 580.x, PRIME offload).
- Default desktop: Zorin (GNOME Wayland). The user ALSO has an i3 X11 session ("i3" in the GDM session list, from /usr/share/xsessions/i3.desktop).
- i3 is X11-only — it cannot run inside the GNOME Wayland session. To test i3, the user must log out and pick "i3" at GDM.
- X sessions run on the NVIDIA proprietary driver (nvidia_drv.so); the Xorg user log is at ~/.local/share/xorg/Xorg.1.log.
- Session errors for the last X session live in journalctl (e.g. `journalctl -b -1 | grep gdm-x-session`).

## i3 setup (~/.config/i3/config)

- dwm-inspired layout: mod4 ($mod) bindings, DEFAULT (splith) workspace layout — NOT tabbed (user wants dwm-style separate windows, not tabs), gaps (inner 10, top/bottom 30, left/right 20), focus_follows_mouse no.
- BreadOnPenguins echo colorscheme: bg #30272b, fg #F38B58, border #443c40, urgent #DBB77E.
- Notable bindings: mod+Return = st (term), mod+d = dmenu_run, mod+j/k = focus down/up, mod+h/l = resize width, mod+space = `move position center`, mod+1-0 workspaces, mod+q kill, mod+Shift+q killothers, mod+Control+Shift+q restart, mod+Shift+BackSpace exit nagbar, mod+Shift+w = wallpapermenu, mod+F1 flameshot gui, mod+F5/F6 volume, mod+F8 i3lock, mod+F10/F11/F12 playerctl.
- Autostart: picom -b, dunst, xss-lock -- i3lock, feh --bg-fill ~/Pictures/BOP_Images/squarebw.png (a 173MB black-and-white PNG — large, slow to decode).
- Themed binaries live in ~/.local/bin: st, i3status-rs, killothers, wallpapermenu, playerctl, fzf, bat, batman.
- st (~/.local/src/st, custom build): copy = Ctrl+Shift+C or Alt+C, paste = Ctrl+Shift+V or Alt+V/Shift+Insert. Rebuild with `make && cp st ~/.local/bin/st` (use mv if "Text file busy").

## i3status-rust (~/.config/i3status-rust/config.toml)

- Manually installed version 0.36.1 (NOT the distro package) — its config schema changed vs older versions:
  - NO `[theme] id = "echo"` — that crashes it with "unknown field `id`".
  - Correct format: `[theme] theme = "plain"` plus a single-line `overrides = { idle_fg = ..., ... }` (multiline inline tables and trailing commas FAIL to parse).
  - Blocks must NOT set `theme = "echo"` either.
  - Placeholders use `$` syntax now (e.g. `$utilization`), NOT `{...}`.
  - THEME FILES MUST EXIST ON DISK: the manual install ships no themes; `theme = "plain"` fails with "Theme 'plain' not found" unless ~/.config/i3status-rust/themes/plain.toml exists (downloaded from GitHub v0.36.1).
  - To test: run `i3status-rs` and check JSON output (empty `[]` rows in the first seconds are normal startup; blocks fill in after ~2s).
- Blocks: cpu, memory, disk_space(/), battery, sound, net, time. Bar is `status_command i3status-rs` (no path needed; defaults to config.toml).

## picom (~/.config/picom/picom.conf)

- BreadOnPenguins style: dual_kawase blur, shadows, fading. User wants SQUARE corners everywhere — `corner-radius = 0` (Aug 2026, user complained bar AND terminal looked rounded).
- MUST use `backend = "glx"` (not "egl") — the egl backend black-screens on the NVIDIA X session.
- `rounded-corners-exclude = [ "window_type = 'dock'" ]` keeps the i3bar square (user dislikes rounded bar ends). NOTE: picom v10's libconfig parser rejects trailing commas in lists — write exclude lists as `[ "cond" ]` with NO trailing comma.
- vsync = true.

## Known gotchas / history

- Aug 2026: user reported i3 showing a black screen. Root causes fixed: (1) i3status-rs 0.36.1 schema mismatch crashed the status bar, (2) picom egl backend + NVIDIA, (3) invalid binding `move window to position 0` (now `move position center`).
- If a future "black screen" report comes in, check in order: i3status-rs config parse (run `i3status-rs` and read output), picom backend, feh wallpaper decode, then journalctl for the last gdm-x-session boot.
- The wallpaper image squarebw.png is black-and-white by design — a "mostly black" desktop is expected when the bar is dead.
- Aug 2026 (session 2): dmenu (mod+d) usage error root cause — `#` in `-nb #30272b` was eaten as a shell comment by i3's `sh -c`, truncating args so dmenu got a dangling `-nb`. FIX: quote hex colors in bindings: `-nb '#30272b'`. Also stock dmenu 5.4 has NO `-c` (centered) option — removed `-c` from dmenunotes/dmenuvids/dmenutemp/wallpapermenu.
- Sound block VOL can freeze at the value it sampled at session boot (e.g. stuck at "0%") — the Pulse subscription goes stale. Fix: `DISPLAY=:0 i3-msg restart`. NOTE: `pkill i3status-rs` alone does NOT bring the bar back (i3bar does not reliably restart the status command) — use the i3 restart.
- Media-key bindings (XF86AudioRaise/Lower/Mute → `amixer -D pulse sset Master 5%±`, XF86AudioPlay/Next/Prev → playerctl) verified working — amixer↔PipeWire default sink volumes track each other (amixer shows % , wpctl shows 0.x).
- Wallpaper → color pipeline (Aug 2026): `mod+Shift+w` wallpapermenu → `wal -i` (pywal16 fork at ~/.local/bin/wal) → `~/scripts/pywal16` post-script refreshes xrdb (st/dmenu), alacritty.toml colors, then `wal-i3colors` (in ~/.local/bin) rewrites the `# wal-begin`…`# wal-end` block in ~/.config/i3/config and the `overrides = {…}` line (MUST stay single-line) in ~/.config/i3status-rust/config.toml, then `DISPLAY=:0 i3-msg restart` (PID stays same, i3 re-execs; i3status-rs gets a new PID). Do NOT edit the wal-begin block by hand.
- pywal color mapping: $bg=background, $fg=foreground, $border=color8, $urgent=color3, selbg=fg/selfg=bg. i3status-rs: idle=fg, info=color4, good=color2, warning=color3, critical=color1, separator=color8.
- NOTE: i3's `include` directive does NOT propagate variable definitions into the including file — use the wal-begin/wal-end marker block instead.
- WALLPAPER-SET GOTCHA (Aug 2026): the i3 session inherits `XDG_CURRENT_DESKTOP=zorin:GNOME`, so `wal` picks its GNOME branch and writes the background to dconf via gsettings — NOTHING changes on the i3 root window (and `~/.fehbg` stays untouched; wal still logs "Set the new wallpaper."). FIX: wallpapermenu runs wal as `XDG_CURRENT_DESKTOP=i3 wal -i …` so wal uses set_wm_wallpaper → `feh --bg-fill` (updates ~/.fehbg + root pixmap). Diagnose by checking `~/.fehbg` and `xprop -root _XROOTPMAP_ID`.
- BOP_Images files are huge (up to ~250MB PNGs) — feh takes seconds to decode; the wallpaper change appears with a delay.

## Communication preferences

- User wants to avoid re-typing context every session. Use this file + `opencode --continue` for session continuity. Keep updates to this file when the setup changes.