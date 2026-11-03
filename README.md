# DXVK with adapter groups (unofficial)

**Unofficial** build of [DXVK](https://github.com/doitsujin/dxvk) with an adapter-group patch,
for Supreme Commander and Forged Alliance players on Linux. Not affiliated with DXVK or FAF.
Report problems here, not to DXVK or FAF.

Download the latest build from [Releases](../../releases).

## What it fixes

Supreme Commander / Forged Alliance draws its second-screen view through one D3D9
*adapter-group* device (`D3DCREATE_ADAPTERGROUP_DEVICE`, two heads, one `Present`).
Upstream DXVK doesn't support that, so under Wine/Proton the second screen stays dark or the
main screen drops to a low resolution. These patches add adapter-group support to DXVK's
d3d9. It switches itself on for `SupremeCommander.exe` and `ForgedAlliance.exe` through
DXVK's app profile (`d3d9.adapterGroups`); other games are unaffected.

Requirements: both monitors on the same GPU, every head fullscreen.

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

The release archives use the official DXVK layout (`dxvk-<version>-adaptergroups/x32/*.dll`)
and also contain the patches and the DXVK license.

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

The patches are in [`patches/`](patches); their development branch is
[Huppie/dxvk `adapter-groups-master`](https://github.com/Huppie/dxvk/tree/adapter-groups-master).
[`build.yml`](.github/workflows/build.yml) applies them to upstream DXVK and builds the
32-bit DLLs on Debian 13.1 with mingw-w64 (posix threads), with the same options as upstream's
`package-release.sh`. It runs weekly against the latest DXVK release and opens an issue
when the patches stop applying. To build another upstream tag, branch or commit, run the
workflow manually with `dxvk_ref`.

## License

DXVK is © Philip Rebohle and contributors, under the zlib/libpng license; see
[`licenses/DXVK-LICENSE`](licenses/DXVK-LICENSE). The patches are under the same license.
