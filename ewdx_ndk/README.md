# ewdx_ndk - EchidnaWarsDX Android translation layer

Native layer that runs the game script on Android. It implements the drawing,
input, text, audio, and system interfaces used by the game on top of SDL2 and
OpenGL ES 2.0, inside the OpenHSP NDK runtime.

## Components

| File | Responsibility |
|---|---|
| `ewdx_gles.h` / `ewdx_gles.cpp` | SDL and GLES2 setup, viewports, offscreen targets, present and blend state |
| `ewdx_batch.h` / `ewdx_batch.cpp` | Quad batching, color-key upload, primitive and line paths |
| `ewdx_dg.h` / `ewdx_dg.cpp` | Script-facing drawing interface, error reporting via `stat` |
| `ewdx_input.h` / `ewdx_input.cpp` | Touch, keyboard, and gamepad input mapped to the script joystick mask |
| `ewdx_text.h` / `ewdx_text.cpp` | Text rendering with SDL_ttf, string cache, SJIS to UTF-8 conversion |
| `ewdx_sjis_tab.h` | Generated CP932 conversion table observed in game data |
| `ewdx_audio.h` / `ewdx_audio.cpp` | Sound-effect bank, software mixer, OGG background-music streaming |
| `ewdx_register.h` / `ewdx_register.cpp` | Script command dispatch, variable save and restore |
| `ewdx_hspv.h` / `ewdx_hspv.cpp` | Save-data reader |
| `ewdx_boot.h` / `ewdx_boot.cpp` | Asset staging, `start.ax` preparation, VM boot and main loop |
| `ewdx_extcmd.h` / `ewdx_extcmd.cpp` | Screen, dialog, mouse, keyboard, and system-info support |
| `ewdx_ovplay.h` / `ewdx_ovplay.cpp` | Music playback hook |
| `ewdx_osd.h` / `ewdx_osd.cpp` | On-screen display helpers |
| `ewdx_paths.cpp`, `ewdx_log.h` | Path resolution and logging |
| `ewdx_supio_decl.h` / `ewdx_supio.cpp` | String and filesystem support for the VM core |
| `CMakeLists.txt` | Static-library build definition for the NDK build |
| `tests/` | Host-side verification for input and audio |
| `thirdparty/` | Vendored decoders (stb_vorbis, stb_image) |
| `vm/` | VM support sources |

## Implementation notes

- Save format follows the HSP `hspv` container. Verified against shipped map, model, and motion files.
- Blend modes follow the original Direct3D render-state usage. The script uses modes 0-4.
- Asset lookup uses `data/<sub>/` prefixes for the six content loaders. Images load through `DGLOADMEMORY`. BGM loop points are per track.
- Boot stages `start.ax` from the APK into the app files directory, normalizes path separators, sets the working directory, then starts the VM. The boot journal is written on every run.
- Window creation is in `ewdx_gles.cpp` (`SDL_CreateWindow`). Boot dialogs and the startup hold are in `ewdx_boot.cpp`. The error dialog path is separate from the startup notice.

## Build verification

- NDK: native translation units compile clean for `aarch64-linux-android21` under `-Wall -Wextra`. Third-party sources build with warnings suppressed.
- Host tests cover audio parsing, mixing, panning, loop handling, and input mapping.
- Device check: install the APK, confirm first launch unpacks `data/`, and logcat shows probe through `HSP VM ready` to title.
