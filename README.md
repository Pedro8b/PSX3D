# PSX3D O3DS

**This is a fork of [PSX3D](https://github.com/BreadChaser/PSX3D) by BreadChaser.**
Its only goal is to get as much speed as possible out of the Old 3DS / 3DS XL / 2DS
(268 MHz, no L2 cache), so that at least 2D games play at full speed. Everything
here is experimental and tested on one Old 3DS so far; for the New 3DS, use the
original.

> **LLM notice.** Most of the code changes, analysis and text in this fork were
> written with the help of an LLM (Anthropic's Claude), guided and tested on real
> hardware by the fork's author. Review them with that in mind.

The fork installs next to the original (its own title ID, shown as "PSX3D O3DS")
and shares its folder `sdmc:/3ds/PSX3D/` (settings, saves, covers). It writes its
own log, `log_o3ds.txt`.

### What's different

Measured in Castlevania: Symphony of the Night on an Old 3DS, most of the frame
time went to drawing (7-28 ms of 18-35 ms), then events and disc reads, then the
PS1 CPU. The changes target that:

- **Frameskip by timing.** Frameskip `auto` now skips drawing the next frame
  whenever the current one started late, instead of waiting for the sound buffer
  to run low. The game keeps its speed; up to 3 frames in a row are skipped on
  Old 3DS. This was the biggest single win.
- **Sound that slows down instead of crackling.** When emulation can't keep up,
  playback glides slower (lower pitch) rather than running dry.
  `audio_stretch=30` in `config.ini` sets the maximum, in percent (0 = off).
- **"Skip every 2nd line" drawing** (Core options). Draws only every other line
  and shows each twice: +15-20% speed in heavy scenes, but small text becomes hard
  to read. Off is recommended for everyday play.
- **Idle loop skip** (Core options, "Idle loop skip (PSX3D)"). The dynarec spots
  short polling loops (a game waiting for the GPU, the CD, a timer) and lets
  emulated time pass in 1024-cycle steps there instead of spinning. Turn it off
  if a game misbehaves.
- **Disc read-ahead on the syscore.** The Old 3DS's second core is only lent to
  apps 30% of the time, but waiting on the SD card costs almost no CPU, so disc
  prefetching runs there. The main thread never waits for it.
- **Memory.** The dynarec code cache is 8 MB instead of 16 MB, which leaves room
  for save states and "Continue where I left off" on Old 3DS (they used to fail
  with "Not enough memory").
- **Profiler.** With `debug=1` in `config.ini`, the log shows once a second where
  frame time goes: drawing by primitive kind, events, disc and XA reads, picture
  output, frames skipped, and where the PS1 CPU spends its time.

Recommended Old 3DS setup: Frameskip type `auto`, Frameskip interval `3`,
"Skip every 2nd line" off (on for the heaviest games), 3D resolution `half`.
Old `core.*` lines left in `config.ini` from earlier versions override the
defaults; delete them if a setting doesn't seem to apply.

Core changes are kept as patches in `patches/` (0003 and 0004 are this fork's),
applied to the PCSX-ReARMed submodule at build time like the original's.

---

*The original README follows.*

# PSX3D

PSX3D is a standalone PlayStation emulator for the Nintendo 3DS. It runs the
[PCSX-ReARMed](https://github.com/libretro/pcsx_rearmed) emulation core directly,
without RetroArch, behind a frontend written for the 3DS.

<p align="center">
  <img src="docs/screenshots/library.png" alt="Game library" width="300">
  <img src="docs/screenshots/menu.png" alt="In-game menu" width="300">
</p>
<p align="center">
  <img src="docs/screenshots/crash.png" alt="Crash Bandicoot" width="400">
  <img src="docs/screenshots/silent-hill.png" alt="Silent Hill" width="400">
</p>

## Features

- Dynarec CPU core at 804 MHz with the L2 cache enabled; most titles run at full speed on New 3DS hardware
- Game library with box art, cleaned-up titles and multi-disc grouping
- Library loading that does not scan the whole SD card, so large cards (1 TB and up) work
- Save states (10 slots) and per-game memory cards
- Display modes: 4:3, stretched, and widescreen (renders a wider field of view in supported titles)
- Disc swapping for multi-disc games
- Bottom screen backlight turned off during play; touch the bottom screen to open the menu
- Safe handling of HOME, sleep and power-off at any point
- Old 3DS support: faster default settings and on-screen L2/R2 buttons
- No DSP1 needed: the DSP firmware is copied from the system on first start

## Requirements

- Any 3DS or 2DS model running Luma3DS. On a New 3DS, New 3DS XL or New 2DS XL
  most titles run at full speed. On an Old 3DS, 3DS XL or 2DS, 2D games run at
  full speed and 3D games run slower (Crash Bandicoot at around 35-40 FPS).
- Game images in `.cue/.bin`, `.chd`, `.pbp`, `.iso` or `.m3u` format. No games are included.
- Optional: a PlayStation BIOS image (for example `scph1001.bin`) in
  `sdmc:/3ds/PSX3D/bios/`. Without one, the core's HLE BIOS is used.

## Old 3DS

The Old 3DS, 3DS XL and 2DS have a 268 MHz CPU (the New 3DS runs at 804 MHz
with an extra cache), so expect the following:

- Demanding 3D games do not reach full speed, and some may not be playable.
- Defaults are tuned for speed and differ from the New 3DS:
  - Automatic frameskip is on, so motion is less smooth when the game falls behind.
  - 3D scenes are drawn at half vertical resolution.
  - Sound uses simple interpolation and no reverb.
  - GTE speed hacks are on, which can cause rare graphical glitches.

  All of these can be changed in Settings → Core options.
- Rewind is not available.
- There are no ZL/ZR buttons; L2 and R2 are on-screen buttons on the touch screen.

## Installation

1. Download `PSX3D.cia` from [Releases](https://github.com/BreadChaser/PSX3D/releases).
2. Install it with FBI or another CIA installer.
3. Place your games in `sdmc:/roms/psx/`. A different folder can be selected from
   the library with Y.

## Controls

| Context | Input | Action |
|---|---|---|
| Library | A | Start game |
| Library | Y | Select games folder |
| Library | SELECT | Rescan folder |
| Library | START | Exit |
| In game | A / B / X / Y | Circle / Cross / Triangle / Square |
| In game | L / R / ZL / ZR | L1 / R1 / L2 / R2 |
| In game | Circle Pad / C-Stick | D-pad or left analog / right analog |
| In game | Touch bottom screen | Open menu |

## Box art

Box art is downloaded from [libretro-thumbnails](https://github.com/libretro-thumbnails/libretro-thumbnails)
the first time a game is displayed and cached in `sdmc:/3ds/PSX3D/covers/`. An internet
connection is only required for the initial download. Custom art can be added as a PNG
in the same folder using the matching file name.

## File locations

| Path | Contents |
|---|---|
| `sdmc:/3ds/PSX3D/saves/` | Memory cards |
| `sdmc:/3ds/PSX3D/states/` | Save states |
| `sdmc:/3ds/PSX3D/covers/` | Box art cache |
| `sdmc:/3ds/PSX3D/bios/` | Optional BIOS |
| `sdmc:/3ds/PSX3D/config.ini` | Settings |
| `sdmc:/3ds/PSX3D/log.txt` | Log of the most recent session |

## Known issues

- Box art downloads are slow because of the 3DS network stack.
- Heavy Wi-Fi activity (for example NTR screen streaming) can cause audio clicks.
- Some titles require a real BIOS to boot.
- On an Old 3DS, the Continue screen shows an empty frame instead of the screenshot.

When reporting a bug, please include `log.txt` from the session where it happened.

## Building

Requires [devkitPro](https://devkitpro.org/) with devkitARM, libctru, citro2d and citro3d.

```bash
git clone --recursive https://github.com/BreadChaser/PSX3D.git
cd PSX3D
git -C core submodule update --init deps/libchdr deps/libretro-common
make        # psx3d/PSX3D.3dsx
make cia    # psx3d/PSX3D.cia (requires makerom and bannertool)
```

## Credits

- [PCSX-ReARMed](https://github.com/libretro/pcsx_rearmed): notaz, Exophase,
  senquack and the libretro contributors, based on PCSX, PCSX-Reloaded and the
  P.E.Op.S. plugins
- The authors of the PCSX-ReARMed 3DS port
- devkitPro: libctru, citro2d, citro3d
- [DSP1](https://github.com/zoogie/DSP1) by zoogie: the method for copying the DSP firmware
- Artwork and font credits: [tools/art/CREDITS.md](tools/art/CREDITS.md)

PSX3D is not affiliated with Sony, Nintendo, libretro or the PCSX-ReARMed developers.

## License

GPL-2.0, matching PCSX-ReARMed. See [LICENSE](LICENSE).
