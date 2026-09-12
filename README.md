# AVC Library

为 [avplay](https://github.com/xxxzhou/avplay) 项目存放大型二进制文件（dll、lib 等）。

## 目录结构

```
avc_library/
├── build/                  # WebRTC 构建产物 (release 无符号库, 直存 git 不走 LFS)
│   ├── android/            # Android libwebrtc_nosym.a
│   ├── darwin/             # macOS libwebrtc_nosym.a
│   ├── ios/                # iOS libwebrtc_nosym.a
│   └── windows/            # Windows webrtc_nosym.lib
├── 3rdparty/               # 第三方库
│   └── library/            # ONNX Runtime 等
└── src/                    # WebRTC 源码（git submodule）
```

## WebRTC 库说明

- git 只管 **release 无符号版** (`*_nosym`)，均在 100MB 内，已配置绕过 LFS 直存；avox 的 FindWebRTC.cmake release 构建优先链接它。
- 带符号版 (`libwebrtc.a`/`webrtc.lib`) 含调试信息体积大 4~14 倍，**不进版本管理**（见 .gitignore），留在各构建机上用于崩溃符号化。
- 重编用 avox 仓库 [script/webrtc/](../avox/script/webrtc) 下各平台脚本；Windows Debug 库（build/windows/debug/）同样不入库，需本机自编 `.\webrtc_build_windows.ps1 debug`。

## 3rdparty 预编译库说明

- **windows/onnxruntime**: `onnxruntime-win-x64-MT-Release-1.23.2`，来自 [csukuangfj/onnxruntime-libs](https://github.com/csukuangfj/onnxruntime-libs/releases/tag/v1.23.2)（静态 CRT /MT 版）。曾以 `onnxruntime-win-x64-1.23.2`（形似微软官方命名）入库，内容即此 MT 构建，现改名对齐 FindONNX.cmake 优先目录与 avox 下载脚本的解压目录名。微软官方 MD 版在 avox 的 plugins/ 下 LoadLibrary 时 DllMain 初始化失败 (err=1114)，勿入库。
- **android/onnxruntime**: 已移除（静态 .a 解压后 1.3GB，占 LFS 存储 73%）。需要编译 Android 时在 avox 仓库执行 `python script/onnx/down_onnxruntime_android.py`（上游 zip 仅 180MB，解压到 avox/3rdparty/library/android/onnxruntime）。
- **windows/opencv**: 精简为 release-only —— 已删 debug 库（opencv_world4130d 127MB）、java/python 绑定、重复的顶层 ffmpeg dll。avox 的 FindOpenCV.cmake 找不到 debug 库时自动全配置链 release 库，Debug 构建不受影响。

## AI 模型

AI 模型存放在独立仓库：[avc_model](https://github.com/xxxzhou/avc_model)
