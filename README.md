# LIBWRT GitHub Build

This project builds IPQ60XX LIBWRT/OpenWrt firmware on GitHub Actions.

## Usage

1. Create a new GitHub repository (for example: `libwrt-build`).
2. Upload all files in this folder to that repository.
3. Open the repository on GitHub and go to `Settings` -> `Actions` -> `General`.
4. Set `Workflow permissions` to `Read and write permissions`.
5. Open `Actions` -> `IPQ60XX-LibWrt` -> `Run workflow`.
6. Start the build.
7. Download firmware from `Releases` after the workflow finishes.

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

- This workflow is triggered by `workflow_dispatch` by default, so pushing commits will not start a build automatically.
- The workflow creates GitHub Releases and deletes old GitHub Actions caches, so it needs `contents: write` and `actions: write` permissions.
- If you see `403 Resource not accessible by integration`, first check repository `Workflow permissions` and then confirm the workflow `permissions` block is present.
- If you want another device, duplicate `configs/IPQ60XX.config` and update the workflow environment variables.
- Compiling OpenWrt/LIBWRT can take a long time on GitHub runners.
