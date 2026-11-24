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

Windows：在 Linux x86_64（CI 为 Ubuntu 24.04）上用 llvm-mingw（UCRT）交叉编译。工具链由脚本按 `windows/toolchain.tsv`
下载并校验 sha256，宿主上只需要 meson、ninja、nasm、pkg-config、python3、curl、xz、zip 与 binutils。

```sh
sudo apt-get install nasm pkg-config python3-venv curl xz-utils zip binutils
python3 -m venv venv && venv/bin/pip install meson==1.6.1 ninja==1.11.1.3 && export PATH=$PWD/venv/bin:$PATH
bash windows/build.sh <工作目录> <输出目录>
```

- `windows/sources.tsv`：全部组件的版本、许可证、下载地址与 sha256；脚本只用校验通过的源码包。
- `windows/toolchain.tsv`：交叉编译工具链的版本、下载地址与 sha256。
- `windows/patches/`：对上游发布包的补丁（每个文件开头写明原因），按文件名前缀应用到对应组件：
  - `mpv-0.42.0-d3d11-helpers-for-d3d-hwaccel.patch`：只开 `d3d-hwaccel` 时补编 `d3d11_helpers.c`（否则链接失败）。
  - `ffmpeg-8.1.3-no-env-http-proxy.patch`：FFmpeg 的 http 协议不从环境变量 `http_proxy` 取代理（Tsumugi 的播放器只连接本地媒体代理）。
- `windows/licenses/`：静态链接进 DLL 的工具链运行库的许可证文本。
- `windows/build.sh`：配置与编译步骤，最后检查许可证（FFmpeg 必须是 LGPL、不含 GPL / nonfree 组件，mpv 以 `-Dgpl=false` 构建）
  和导入表（只允许 Windows 系统 DLL）。
- 输出 `<TAG>.zip`（`libmpv-2.dll`、`libmpv.dll.a`、`include/*.h`、`licenses/`、`BUILD_INFO.txt`）、`<TAG>-sources.tar`
  （对应源码：`tarballs/` 全部源码发布包、`build-scripts/`、`licenses/`、`LICENSE`、`BUILD_INFO.txt`）与 `SHA256SUMS`。
- 从源码包重新构建：解开 `<TAG>-sources.tar` 后运行 `build-scripts/build.sh <工作目录> <输出目录>`，脚本优先使用包内的 `tarballs/`
  （同样校验 sha256）；交叉工具链仍按 `build-scripts/toolchain.tsv` 下载（或预先放入 `<工作目录>/dl/`）。

## 发布

- 标签形如 `libmpv-win-<mpv 版本>-ff<FFmpeg 版本>-r<修订号>`，取自 `windows/VERSION`。
- 由 GitHub Actions（`.github/workflows/windows.yml`，手动运行）在 `main` 上构建并发布，附构建来源证明
  （`gh attestation verify <文件> --repo Shiroha23/Tsumugi-libmpv`）。
- 发布只增不改（不可变 Release）：任何组件更新或构建改动都递增修订号，发新的 Release。

## 许可证

- 本仓库的构建脚本：GPL-3.0-or-later（见 `LICENSE`）。
- 产物：mpv 为 LGPL-2.1-or-later，FFmpeg 为 LGPL-3.0-or-later；其余组件的许可证见 `windows/sources.tsv`，
  许可证全文与版权声明随源码包中的各组件源码提供。静态链接进 DLL 的工具链运行库（mingw-w64 CRT 与 winpthreads、
  LLVM 的 libc++ / libunwind / compiler-rt）为宽松许可证，版本见 `BUILD_INFO.txt`。
