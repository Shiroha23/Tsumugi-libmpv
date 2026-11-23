# Tsumugi-libmpv

[Tsumugi](https://github.com/Shiroha23/Tsumugi) 使用的 libmpv / FFmpeg 原生库构建：全部从正式发布版源码编译，每个 Release 附对应源码。

## 用途

Tsumugi 的视频播放基于 libmpv。本仓库提供它随包分发的 libmpv：

| 平台 | 产物 | 状态 |
|---|---|---|
| Windows x64 | `libmpv-2.dll`（静态链接 FFmpeg 及全部依赖，只依赖 Windows 系统 DLL） | 可用 |
| Android / iOS | — | 计划中 |

Tsumugi 按固定的 Release 地址与 sha256 下载并校验产物。

## 构建

Windows：在 Ubuntu 24.04 上用 mingw-w64 交叉编译。

```sh
sudo apt-get install gcc-mingw-w64-x86-64-posix g++-mingw-w64-x86-64-posix binutils-mingw-w64-x86-64 \
  mingw-w64-tools nasm meson ninja-build pkgconf python3 curl xz-utils zip
bash windows/build.sh <工作目录> <输出目录>
```

- `windows/sources.tsv`：全部组件的版本、许可证、下载地址与 sha256；脚本只用校验通过的源码包。
- `windows/build.sh`：配置与编译步骤，最后检查许可证（FFmpeg 必须是 LGPL、不含 GPL / nonfree 组件，mpv 以 `-Dgpl=false` 构建）
  和导入表（只允许 Windows 系统 DLL）。
- 输出 `<TAG>.zip`（`libmpv-2.dll`、`libmpv.dll.a`、`include/*.h`、`BUILD_INFO.txt`）、`<TAG>-sources.tar`（对应源码）与 `SHA256SUMS`。

## 发布

- 标签形如 `libmpv-win-<mpv 版本>-ff<FFmpeg 版本>-r<修订号>`，取自 `windows/VERSION`。
- 由 GitHub Actions（`.github/workflows/windows.yml`，手动运行）在 `main` 上构建并发布，附构建来源证明
  （`gh attestation verify <文件> --repo Shiroha23/Tsumugi-libmpv`）。
- 发布只增不改（不可变 Release）：任何组件更新或构建改动都递增修订号，发新的 Release。

## 许可证

- 本仓库的构建脚本：GPL-3.0-or-later（见 `LICENSE`）。
- 产物：mpv 为 LGPL-2.1-or-later，FFmpeg 为 LGPL-3.0-or-later；其余组件的许可证见 `windows/sources.tsv`，
  许可证全文与版权声明随源码包中的各组件源码提供。
