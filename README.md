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

## AI 模型

AI 模型存放在独立仓库：[avc_model](https://github.com/xxxzhou/avc_model)
