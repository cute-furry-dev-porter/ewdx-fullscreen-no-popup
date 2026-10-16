# hmm_spy - PC-side draw trace for Echidna Wars DX

A replacement `hmm.dll` that forwards all 114 exports of the original DLL to
`hmm_real.dll` while logging the DG draw stream in the same format used by the
Android port logs (`[pc-maw]`, `[pc-prim]`, `[pc-dump]`). PC output and Android
`[maw]`/`[prim]` logs can then be compared line by line.

## Install (on the PC that runs the game)

1. Open the game folder (`Echidna Wars DX [ENG-JAP]/`).
2. Rename the real `hmm.dll` → `hmm_real.dll` (keep it!).
3. Copy this build's `hmm.dll` into the folder.
4. Run `EchidnaWarsDX.exe` normally.

Output: `hmm_spy.log` in the game folder (appends across runs).

## What gets logged

- Full stream into a 4 MB ring buffer: every DGCOLOR / DGGSEL / DGCLEAR /
  DGRECT / DGPOS / DGSCALEANDANGLE (with 8.8→float) / DGBLENDMODE / DGTEXTURE /
  DGLINE / DGBUFFER / DGREDRAW / DGADDPRIMITIVE / DGDRAWPRIMITIVE /
  DGCREATEPRIMITIVE, plus `DGGCOPY id flags`.
- **The ring flushes to `hmm_spy.log` whenever `DGGCOPY 5` fires** - the composite
  call - so each episode produces a complete trace, marked
  `[pc-dump] FLUSH ep=N`. Menu output outside these flushes is not retained.

## Diffing against the port

Port side (Android): the `[maw]`/`[prim]` lines in `ewdx-boot*.log` use the
same call sequence. Compare per POV episode:

```
grep "\[pc-" hmm_spy.log | sed -n '1,400p'
```

vs the port's `[maw]` lines around an `id=5` copy. Any divergence in draw
order, flags, scale, blend, or target is the port bug.

## Rebuild

```
python3 tools/hmm_exports.py | grep -oE "^_[A-Z0-9]+@16" | sort -u > tools/hmm_spy/hmm_names.txt
python3 tools/hmm_spy/gen_spy_c.py
python3 -m ziglang cc -target x86-windows-gnu -shared -O2 -o tools/hmm_spy/hmm.dll tools/hmm_spy/hmm_spy.c
```

(zig via `pip install ziglang`; `-target x86-windows-gnu` = 32-bit, the game
is x86. Regenerate `hmm_spy.c` after any edit to `gen_spy_c.py` only.)

## Uninstall

Delete the spy `hmm.dll`, rename `hmm_real.dll` back to `hmm.dll`.

## Files

- `hmm_spy.zig` - earlier revision, retained for reference
- `gen_spy_c.py` - generates `hmm_spy.c` from `hmm_names.txt`
- `hmm_names.txt` - the 114 export names (from `tools/hmm_exports.py`)
- `hmm_spy.c` -> `hmm.dll` - the built shim (x86)
