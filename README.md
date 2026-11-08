# LIBWRT GitHub Build

这是一个用于在 GitHub Actions 上编译 `IPQ60XX` 平台 `LIBWRT/OpenWrt` 固件的仓库。

当前工作流基于上游源码：

- 源码仓库：`https://github.com/laipeng668/openwrt-6.x.git`
- 默认分支：`25.12-nss`
- 默认目标：`qualcommax/ipq60xx`
- 默认时区：`Asia/Shanghai`

## 项目特点

- 使用 GitHub Actions 在线编译，无需本地搭建完整编译环境。
- 默认编译 `6.12` 内核的 `NSS` 固件，并启用 `ATH11K NSS Mesh` 支持。
- 采用「入口工作流 + 可复用构建工作流」结构，构建逻辑集中在 `Build-OpenWrt.yml`，便于复用。
- 使用 `configs/IPQ60XX.config` 和 `configs/General.config` 组合生成最终 `.config`。
- 通过 `scripts/Custom.sh` 在 feeds 安装后替换部分软件包、添加第三方插件并修改默认系统设置。
- 编译完成后自动发布到 GitHub Releases。
- 使用 `actions/cache` 缓存 `.ccache` 和 `staging_dir`，减少重复编译耗时。

## 工作流说明

工作流包含两个文件：

- [IPQ60XX-LibWrt.yml](/D:/libwrt-github-build/.github/workflows/IPQ60XX-LibWrt.yml)：入口工作流，负责接收触发并向构建工作流传递参数。
- [Build-OpenWrt.yml](/D:/libwrt-github-build/.github/workflows/Build-OpenWrt.yml)：可复用构建工作流（`workflow_call`），完成实际的编译与发布。

当前触发方式（入口工作流）：

- `workflow_dispatch`：手动触发
- `workflow_call`：供其他工作流调用

注意：

- 当前没有启用 `push` 自动编译。
- 当前定时任务 `schedule` 处于注释状态。

入口工作流向构建工作流传递的主要参数：

- `repo_url` / `repo_branch`：上游源码地址与分支（`laipeng668/openwrt-6.x` 的 `25.12-nss`）
- `config_file`：`configs/IPQ60XX.config`
- `firmware_tag` / `artifact_prefix`：均为 `IPQ60XX`
- `kernel_file`：`kernel-6.12`
- `manifest_file`：`libwrt-qualcommax-ipq60xx.manifest`
- `release_summary`：Release 描述

`Build-OpenWrt.yml` 支持的可选输入参数：

- `remove_sqm_scripts_nss`：移除 `sqm-scripts-nss` 包（默认 `false`）
- `rename_qualcommax_to_nowifi`：将固件文件名中的 `qualcommax` 改为 `nowifi`（默认 `false`）
- `delete_old_workflows`：发布后删除 2 天前的旧工作流记录（默认 `false`）
- `package_extension`：软件包扩展名，`apk` 或 `ipk`（默认 `apk`）

## 使用方法

1. Fork 本仓库，或新建一个仓库后上传本项目全部文件。
2. 打开仓库的 `Settings -> Actions -> General`。
3. 将 `Workflow permissions` 设置为 `Read and write permissions`。
4. 进入 `Actions -> IPQ60XX-LibWrt`。
5. 点击 `Run workflow` 手动开始编译。
6. 编译完成后，到 `Releases` 下载固件和附带文件。

## 权限要求

构建工作流在 Job 级别显式声明了以下权限：

```yaml
permissions:
  contents: write
  actions: write
```

原因：

- `contents: write`：用于创建或更新 GitHub Release。
- `actions: write`：用于删除旧的 GitHub Actions cache（以及可选的旧工作流记录）。

如果仓库未开启 `Read and write permissions`，常见报错包括：

- `403 Resource not accessible by integration`（发布 Release 失败）
- `Failed to delete cache: HTTP 403`（删除 cache 失败）

## 编译流程

构建工作流运行在 `ubuntu-24.04` 上，整体流程如下：

1. 检查服务器性能（CPU、内存、磁盘）。
2. 初始化编译环境并安装依赖，设置时区为 `Asia/Shanghai`。
3. 检出本项目代码。
4. 克隆上游 `openwrt-6.x` 源码到 `/mnt/openwrt`。
5. 用 `configs/IPQ60XX.config` 生成目标平台、子平台等变量，并据此构造缓存前缀与缓存键。
6. 恢复 `.ccache` 和 `staging_dir` 缓存。
7. 刷新 `staging_dir` 中的 `stamp` 时间戳。
8. 安装 feeds。
9. 执行 `scripts/Custom.sh` 做定制化修改。
10. 合并 `configs/IPQ60XX.config` 与 `configs/General.config`，执行 `make defconfig` 并下载源码包。
11. 编译固件（失败时自动降级重试）。
12. 检查空间使用情况。
13. 整理产物（重命名、收集独立 `.apk` 包并打包）。
14. 发布固件到 GitHub Releases。
15. 删除旧缓存并输出 cache 占用情况。
16. （可选）删除 2 天前的旧工作流记录。

## 配置文件说明

### `configs/IPQ60XX.config`

设备与目标平台相关配置，当前主要内容包括：

- 目标平台：`qualcommax`
- 子平台：`ipq60xx`
- 多设备编译：`CONFIG_TARGET_MULTI_PROFILE=y`
- 已启用多个 `ipq60xx` 设备 profile（如 `jdcloud_re-cs-02`、`redmi_ax5`、`xiaomi_ax1800`、`netgear_wax610`、`linksys_mr7350` 等）
- `jdcloud_re-cs-02` 额外追加：
  - `luci-app-athena-led`
  - `luci-i18n-athena-led-zh-cn`

同时包含例如：

- `CONFIG_ATH11K_MEM_PROFILE_512M=y`
- `CONFIG_NSS_FIRMWARE_VERSION_11_4=y`
- `CONFIG_ATH11K_NSS_MESH_SUPPORT=y`

### `configs/General.config`

通用功能配置，主要包括：

- `CONFIG_CCACHE=y`
- `CONFIG_TARGET_PER_DEVICE_ROOTFS=y`
- `-O3` 编译优化与 cortex-a53 相关指令集
- 默认系统设置/中国区默认设置 `default-settings-chn`，界面附加 `zh_Hant` 语言
- 使用 **nginx-ssl 替代 uhttpd**（配套 `nginx-mod-luci`、`uwsgi` 等），同时关闭 `uhttpd` 及其核心/UI 插件
- 常用 LuCI 应用：
  - `acme`、`ddns`、`ttyd`、`upnp`、`wol`
  - `samba4`、`vlmcsd`、`aria2`、`autoreboot`
  - `cpufreq`、`frps`、`frpc`、`diskman`、`hd-idle`
  - `3cat`、`watchcat`、`openclash`、`wireguard`
- 常用工具与系统组件：
  - `tmux`、`htop`、`btop`、`nano-full`、`dmesg`
  - `fdisk`、`cfdisk`、`sgdisk`、`tune2fs`、`fstrim`
  - `autocore`、`coremark`、`hdparm`、`smartmontools-drivedb`
  - `ariang`、`zram-swap`（LZ4 后端）、`zoneinfo-asia`
  - `openssh-sftp-server`
  - `acme-acmesh-dnsapi`、`ddns-scripts-cloudflare`

说明：

- `sqm-scripts-nss` 明确不启用。
- 主题相关：`luci-theme-argon` / `luci-app-argon-config` 为启用状态；`luci-theme-aurora` / `luci-app-aurora-config` 由 `Custom.sh` 克隆引入，当前配置文件中置为不启用。

PassWall 相关：

- 使用 `Nftables_Transparent_Proxy` 透明代理，并启用 `Geoview` 与 `Xray`。
- 关闭了多数额外后端组件（Hysteria、NaiveProxy、SingBox、Trojan、tuic 等）。

## 自定义脚本说明

自定义脚本文件：

- [Custom.sh](/D:/libwrt-github-build/scripts/Custom.sh)

脚本主要做了这些事情：

- 修改默认 LAN 地址为 `192.168.2.1`
- 修改默认主机名为 `libwrt`
- 在系统状态页固件版本位置加入编译时间和 Releases 链接
- 删除 feeds 中部分原有包，替换为第三方仓库版本
- 拉取并加入额外插件与主题
- 清理 PassWall 的 `chnlist` 规则文件
- 重新执行一次 `feeds update/install`

当前脚本会引入或替换的主要内容包括：

- 主题与配置：
  - `luci-theme-argon`、`luci-app-argon-config`
  - `luci-theme-aurora`、`luci-app-aurora-config`
- 更替的核心包：
  - `aria2`、`ariang`、`nginx`、`golang`、`frp`
  - `luci-app-frpc`、`luci-app-frps`、`luci-app-upnp`、`luci-app-wol`、`miniupnpd`
- 额外插件：
  - `luci-app-openlist2`、`luci-app-lucky`、`luci-app-wechatpush`
  - `OpenAppFilter`、`luci-app-gecoosac`、`luci-app-athena-led`
  - `openwrt-passwall-packages`、`luci-app-passwall`、`luci-app-passwall2`、`luci-app-openclash`

## 编译产物

编译成功后，Release 中通常会包含：

- 固件镜像文件（`IPQ60XX` 前缀）
- `IPQ60XX.config`
- `IPQ60XX.config.buildinfo`
- `IPQ60XX.manifest`
- `IPQ60XX.Packages.tar.gz`

其中：

- `IPQ60XX.manifest` 可用于确认最终镜像实际包含了哪些包。
- `IPQ60XX.Packages.tar.gz` 中包含本次构建生成的独立 `.apk` 软件包。

## Cache 说明

当前缓存项：

- `OPENWRT_PATH/.ccache`
- `OPENWRT_PATH/staging_dir`

缓存键由「上游仓库名 + 分支 + 目标平台 + 子平台 + 源码 hash」组成，同一前缀下可复用最近一次的缓存。

作用：

- `.ccache` 用于加速重复编译。
- `staging_dir` 用于复用工具链和中间产物。

`Refresh The Cache` 步骤会刷新 `staging_dir` 中部分 `stamp` 文件的时间戳，以减少恢复缓存后的依赖判定问题。

如果遇到以下情况，建议清缓存或重新编译：

- 更换上游源码分支
- 调整 target 或 subtarget
- 修改较多底层工具链相关配置
- 出现异常但难以复现的编译错误

## 常见问题

### 1. 为什么 push 后不会自动编译

因为当前工作流没有配置 `push` 触发器，只支持手动运行或被其他工作流调用。

### 2. 为什么会出现 Release 403

通常是以下原因之一：

- 仓库 `Workflow permissions` 不是 `Read and write permissions`
- 工作流没有 `contents: write`
- 运行来源受限

### 3. 为什么会出现删除 cache 的 403

通常是以下原因之一：

- 工作流没有 `actions: write`
- 仓库 Actions 权限不足

## 目录结构

```text
.
├─ .github/workflows/IPQ60XX-LibWrt.yml
├─ .github/workflows/Build-OpenWrt.yml
├─ configs/IPQ60XX.config
├─ configs/General.config
├─ scripts/Custom.sh
└─ README.md
```
