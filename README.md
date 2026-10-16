# EchidnaWarsDX Android - Fullscreen / No-Popup Build

Maintained under `cute-furry-dev-porter/ewdx-fullscreen-no-popup`.
This build removes the boot-time "previous run" dialog and its 10-second hold, and starts the game fullscreen. Crash and error dialogs are unchanged.

The binary patch was applied directly to the shipped `libmain.so`. The source edits below reproduce the same behavior on rebuild.

## Layout

- `apk-parts/` - shipped APK split for GitHub hosting (`EchidnaWarsDX_fullscreen.apk.part00/01`). Join to restore.
- `patched-libs/arm64-v8a/libmain.so`, `patched-libs/armeabi-v7a/libmain.so` - patched natives, ready to swap into an APK.
- `patch_libmain.py` - reapplies the binary patch to clean `libmain.so` files (requires `lief`).
- `ewdx_ndk/` - native engine source (SDL2 + GLES2 translation layer, HSP runtime glue, boot/staging).
- `android/` - Android Gradle wrapper.
- `tools/` - asset and script analysis utilities.

## Binary patch reference

Applied to clean `libmain.so`:

arm64-v8a:
- `0x3b43c` → `1f2003d5` (NOP `SDL_ShowSimpleMessageBox` for "Previous run stopped")
- `0x3b444` → `1f2003d5` (NOP `SDL_Delay(10000)`)
- `0x2cfac` `48048052` → `68048052` (window flags `0x22` → `0x23`, adds fullscreen)

armeabi-v7a:
- `0x1f5f6` `35f09ceb` → `00bf00bf`
- `0x1f5fe` `35f0a0eb` → `00bf00bf`
- `0x14fe8` `2220` → `2320`

Usage:

```sh
pip install lief
python3 patch_libmain.py <dir containing arm64-v8a/ and armeabi-v7a/>
```

## Equivalent source changes

- `ewdx_ndk/ewdx_gles.cpp` (~line 239): add fullscreen to window flags at `SDL_CreateWindow`.
- `ewdx_ndk/ewdx_boot.cpp` (~lines 1347-1375): remove the "Previous run stopped at" message box and hold. Boot log is still written to Downloads. Leave the error path (~line 775) as-is.

## Reassemble the APK

Windows:

```bat
copy /b apk-parts\EchidnaWarsDX_fullscreen.apk.part00+apk-parts\EchidnaWarsDX_fullscreen.apk.part01 EchidnaWarsDX_fullscreen.apk
```

Linux/macOS:

```sh
cat apk-parts/EchidnaWarsDX_fullscreen.apk.part* > EchidnaWarsDX_fullscreen.apk
```

Joined size is 25,165,824 bytes (`PK` header). Uninstall any prior build first; the APK is signed with a debug key.

## Rebuild from source

1. Edit `ewdx_ndk/ewdx_boot.cpp` and `ewdx_ndk/ewdx_gles.cpp` as above.
2. Build `libmain.so` for `arm64-v8a` and `armeabi-v7a` via the NDK (`ewdx_ndk/CMakeLists.txt`).
3. Replace the libraries in the APK with store (no) compression for `.so` entries.
4. Sign, e.g. `java -jar uber-apk-signer.jar -a your.apk --allowResign`.
