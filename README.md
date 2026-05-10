# LIBWRT GitHub Build

This project builds IPQ60XX LIBWRT/OpenWrt firmware on GitHub Actions.

## Usage

1. Create a new GitHub repository (for example: `libwrt-build`).
2. Upload all files in this folder to that repository.
3. Open the repository on GitHub.
4. Go to `Actions` -> `IPQ60XX-LibWrt` -> `Run workflow`.
5. Start the build.
6. Download firmware from `Releases` after the workflow finishes.

## Default settings

- Source repo: `https://github.com/laipeng668/openwrt-6.x.git`
- Default branch: `main-nss`
- Default target: `qualcommax/ipq60xx`
- Default profile: `jdcloud_re-cs-02`

## Custom layout

- `configs/IPQ60XX.config` stores the device-specific build config.
- `configs/General.config` stores extra shared config appended before the final `make defconfig`.
- `scripts/Roc-script.sh` is the custom hook script executed after feeds are installed.
- `.github/workflows/IPQ60XX-LibWrt.yml` is the main GitHub Actions workflow.

## Notes

- If you want another device, duplicate `configs/IPQ60XX.config` and update the workflow environment variables.
- Compiling OpenWrt/LIBWRT can take a long time on GitHub runners.
