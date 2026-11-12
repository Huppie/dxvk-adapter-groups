# DXVK dual-screen fix for Supreme Commander (unofficial)

**Unofficial** build of [DXVK](https://github.com/doitsujin/dxvk) with adapter-group patches,
for Supreme Commander and Forged Alliance players on Linux. Not affiliated with DXVK or FAF.
Report problems here, not to DXVK or FAF.

Adapter-group support was merged into upstream DXVK on 12 November 2026
([#6024](https://github.com/doitsujin/dxvk/pull/6024)) but is not in a DXVK release yet; the
latest release is 2.7.1. These builds bring it to DXVK 2.7.1 and add a fix for three or more
screens that upstream doesn't have.

Download a build from [Releases](../../releases). There are two kinds:

- **`v2.7.1-adaptergroups.N`**: DXVK 2.7.1, the version faf-linux ships, plus the backported
  adapter-group patches and the three-screen fix. Recommended for faf-linux: only `d3d9.dll`
  differs from what you already run.
- **`master-<sha8>-adaptergroups.N`**: upstream DXVK master (commit `<sha8>`) plus the patches
  it still lacks; since the merge that is only the three-screen fix. Use it only with a newer
  DXVK, for example in a recent Proton.

Take the highest `N` of the kind you need.

## What it fixes

Supreme Commander / Forged Alliance draws its second-screen view through one D3D9
*adapter-group* device (`D3DCREATE_ADAPTERGROUP_DEVICE`, two heads, one `Present`).
DXVK up to 2.7.1 doesn't support that, so under Wine/Proton the second screen stays dark or the
main screen drops to a low resolution. The adapter-group patches add that support to DXVK's
d3d9 (`d3d9.adapterGroups`).

- **`v2.7.1` builds**: switched on for `SupremeCommander.exe` and `ForgedAlliance.exe` through
  DXVK's app profile; other games are unaffected.
- **`master` builds**: on by default, as in upstream DXVK. Other D3D9 games then see each
  display as its own adapter, as on Windows, and only get a multi-screen device if they ask
  for one.

Requirements: both monitors on the same GPU, every head fullscreen.

**Three or more screens**: the game sets up two screens, while D3D9 expects one per display
on the GPU. With a third display on the same GPU, device creation fails and the game falls back
to one screen at a low resolution. Windows has the same problem; players there disable the third
display. These builds limit the game to two screens (`d3d9.maxAdapterGroupHeads = 2`), so the
third display can stay on; upstream DXVK doesn't have this limit. The game gets the primary
display and the next one in the system's display order; which one that is can't be chosen yet.

## Install

The game is 32-bit, so only the 32-bit `d3d9.dll` matters. Close the game first.

**faf-linux**

```bash
cd ~/faf-linux/prefix/drive_c/windows/syswow64
cp d3d9.dll d3d9.dll.bak
cp ~/Downloads/d3d9.dll d3d9.dll
```

Roll back with `cp d3d9.dll.bak d3d9.dll`, or reinstall faf-linux's stock DLL from
`~/faf-linux/dxvk-<version>/x32/d3d9.dll`.

**Other Proton**: replace `files/lib/wine/dxvk/i386-windows/d3d9.dll` in the Proton folder
(keep a backup). Don't put it in FAF's `bin` folder; the FAF client manages that.

**Windows**: untested. These are standard DXVK DLLs, so putting the 32-bit `d3d9.dll` next to
`ForgedAlliance.exe` should work if your GPU and driver meet DXVK's
[requirements](https://github.com/doitsujin/dxvk/wiki/Driver-support), but the FAF client may
remove files it doesn't know from its `bin` folder. If you only need dual screen with a third display
connected, disabling that display in Windows' display settings is simpler.

The release archives are a drop-in for the official DXVK archive: `dxvk-<release>/` with
`x32/` and `x64/`, plus the patches and the DXVK license. For example,
`dxvk-2.7.1-adaptergroups.3.tar.gz` extracts to `dxvk-2.7.1-adaptergroups.3/`. The game itself
only needs the 32-bit `d3d9.dll`.

## Enable the second screen

In-game: **Options → Video → Secondary Adapter** → pick your second monitor's resolution,
then restart the game.

- **Integrated GPU**: on machines that also have an iGPU (most Ryzen desktops and laptops)
  the second screen may map to the iGPU and not join. Add `dxvk.hideIntegratedGraphics = True`
  to your DXVK config (faf-linux: `~/faf-linux/dxvk-local.conf`).
- **Anti-aliasing gotcha**: changing some other graphics options, anti-aliasing for example,
  can make the game save Secondary Adapter as disabled again. If the second screen stops
  showing the game, check that setting first.

## Reporting problems

Open an issue with:

- what worked and what didn't (menu, skirmish, mouse on both screens, alt-tab);
- GPU and driver, distro, desktop, Wayland or X11;
- monitor layout (resolutions, primary, any portrait screen);
- the DXVK log: run with `DXVK_LOG_LEVEL=info DXVK_LOG_PATH=$HOME/dxvk-logs` (the folder must
  exist) and attach `ForgedAlliance_d3d9.log`. The line
  `Creating adapter group device with 2 heads` means the patch is active.

## Source and builds

There is one patch series per DXVK base: [`patches/master/`](patches/master) holds what
upstream master still lacks (the three-screen limit), [`patches/v2.7.1/`](patches/v2.7.1) is
the full backport to DXVK 2.7.1. The adapter-group patches themselves went upstream in
[doitsujin/dxvk#6024](https://github.com/doitsujin/dxvk/pull/6024).
[`build.yml`](.github/workflows/build.yml) applies a series to upstream DXVK and builds the
32-bit and 64-bit DLLs on Debian 13.1 with mingw-w64 (posix threads), with the same options
as upstream's `package-release.sh`. A build of ref `X` uses `patches/X/` if it exists, else
`patches/master/`.

- **Push to `main`**: builds each patch set the push changed (`patches/master/` builds DXVK
  master, `patches/v2.7.1/` builds v2.7.1). Pushes that touch no patches build every set;
  sets already released with identical patches are skipped.
- **Manual run**: run the workflow with `dxvk_ref` (tag, branch or commit; empty = latest
  DXVK release), for example `gh workflow run build -f dxvk_ref=v2.7.1`.
- **Weekly run**: paused until a DXVK release the patches apply to; re-enable the `schedule:`
  line in `build.yml`.

## License

DXVK is © Philip Rebohle and contributors, under the zlib/libpng license; see
[`licenses/DXVK-LICENSE`](licenses/DXVK-LICENSE). The patches are under the same license.
