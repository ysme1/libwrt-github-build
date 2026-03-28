# LIBWRT GitHub Build

This project builds LIBWRT firmware on GitHub Actions.

## Usage

1. Create a new GitHub repository (for example: `libwrt-build`).
2. Upload all files in this folder to that repository.
3. Open the repository on GitHub.
4. Go to `Actions` -> `Build LIBWRT` -> `Run workflow`.
5. Set inputs (branch/profile) and start build.
6. Download firmware from `Artifacts` after the workflow finishes.

## Default settings

- Source repo: `https://github.com/LiBwrt/openwrt-6.x.git`
- Default branch: `25.12-nss`
- Default target: `qualcommax/ipq60xx`
- Default profile: `jdcloud_ax6600`

## Notes

- If your profile name differs, edit the workflow input `device_profile`.
- Compiling OpenWrt/LIBWRT can take a long time on GitHub runners.
