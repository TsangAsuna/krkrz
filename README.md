# krkrz (Kirikiri Z / KiriKiri Z)

[English](README.md) | [中文](README_zh.md)

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

krkrz is a cross-platform visual-novel engine derived from
[Kirikiri Z](https://github.com/krkrz/krkrz), extended with an SDL2 runtime
and native ports for Android, iOS, OpenHarmony, and desktop platforms. It
provides the TJS2 scripting virtual machine, a layered sprite compositor,
FAudio-based audio mixing, and a video overlay for opening movies -- the
complete runtime layer needed to ship a KAG/Kirikiri scenario as a native
application.

This repository is a clean extraction of the framework itself (no
game-specific assets), intended for building a standalone visual-novel
runtime around your own scenario data.

## Module Layout

| Directory    | Purpose                                                              |
| ------------ | -------------------------------------------------------------------- |
| `tjs2`       | TJS2 scripting virtual machine (bytecode interpreter)                |
| `base`       | Core runtime: streams, file system, thread pool, Window message      |
| `visual`     | Layered renderer: layers, sprites, transitions, OpenGL/SDL backend   |
| `sound`      | Audio subsystem: FAudio / SDL audio mixer for BGM and SFX            |
| `movie`      | Video overlay for opening/intro movies                               |
| `msg`        | Message/event pump and environment abstraction                       |
| `extension`  | Plugin registry for native TJS2 extensions                           |
| `environ`    | Per-platform abstraction (Win32, SDL2, Android, iOS, OHOS)           |
| `android`    | Android JNI project scaffolding                                      |
| `utils`      | Filesystem and string helpers                                        |

## Architecture Overview

![krkrz engine architecture](docs/architecture.png)

The interactive diagram is generated from
[`docs/architecture.dataflow.json`](docs/architecture.dataflow.json) using
the Archify renderer.

## Quick Start

See `HowToBulid.txt` for the original desktop build steps. The SDL2 and
native platform builds are driven from the engine core in `base` +
`environ`; platform-specific bootstrap lives under each `environ/`
subdirectory.

## Adding Game Content (Assets and Scenario Text)

krkrz loads content from an ARC/Xp3 archive or from the packed `data`
directory at runtime. To make your character graphics (standings/sprites)
and scenario text loadable, structure the game content as follows.

### 1. Content directory layout

Place your game data next to the executable (or inside the native app
bundle), following the conventional Kirikiri naming:

```
data/
├── scenario/          # KAG scenario scripts (.ks text files)
├── image/             # graphics: background, standing sprites, UI
│   ├── bgm/ ...       # (audio conventionally lives under sound/)
├── sound/             # BGM and SFX (OGG/WAV via FAudio)
├── video/             # opening movie (played via the movie overlay)
├── fnt/               # font resources
└── system/            # system scripts (init.tjs, etc.)
```

At startup the engine resolves `./data/*` through the configured data
source; on the Android/iOS port the runtime reports the extracted public
data directory to the native bridge before the VM boots.

### 2. Adding scenario text

Scenario text is plain UTF-8 (or Shift-JIS for legacy archives) `.ks` files
inside `data/scenario/`, written in KAG/KAGEX syntax:

```
; start.ks
*start
「Ciallo～(∠・ω< )⌒☆，世界。」
@close message
```

The scenario manager (base msg layer) dispatches these lines to the TJS2
VM; the `@`-prefixed tags drive text output, background transitions, and
message-window state.

### 3. Adding character sprites (standing art)

Place a standing-sprite image under `data/image/<character>/` (PNG, BMP, or
JPEG with alpha where needed), then display it from the scenario with the
layer-based API exposed by the visual subsystem:

```
; show a standing character at the default position
@bg storage="image/room.png"        ; background layer
@ld c="image/alf/alf_stand.png"    ; character layer
```

The visual renderer loads the image by its archive-relative path and
attaches it to the named layer; layer order, offsets and transitions are
controlled by `@ld` / `@bg`/`@lt` tags or directly through the `Layer`
TJS2 object.

### 4. Playing audio and opening movies

```
@bgm storage="sound/bgm01.ogg"      ; start music on the FAudio mixer
@movie storage="video/op.mp4"       ; play the opening via the movie overlay
```

The audio subsystem pauses/resumes on application background/foreground
with a short settle delay, matching iOS/Android platform lifecycle.

## Platforms

- Windows (Win32 / SDL2)
- Android (ARM64-v8a, armeabi-v7a)
- iOS (ARM64)
- OpenHarmony / HarmonyOS
- macOS / Linux (SDL2)

## License

MIT (see [LICENSE](LICENSE)). Upstream Kirikiri Z is distributed under the
MIT license as well.