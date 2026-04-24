# AVC Library

音视频通信库，基于 WebRTC 构建。

## 目录结构

```
avc_library/
├── src/                    # 源代码
│   ├── api/                # 公共 API
│   ├── audio/              # 音频处理
│   ├── video/              # 视频处理
│   ├── pc/                 # PeerConnection
│   ├── p2p/                # P2P 传输
│   ├── media/              # 媒体引擎
│   └── modules/            # 功能模块
├── 3rdparty/               # 第三方库
│   └── library/            # ONNX Runtime 等
├── build/                  # 构建产物
│   ├── android/            # Android 构建产物
│   ├── ios/                # iOS 构建产物
│   └── windows/            # Windows 构建产物
└── assets/                 # 资源文件
```

## AI 模型

AI 模型已迁移到独立仓库：[avc_model](https://github.com/xxxzhou/avc_model)

包含：
- STT 语音识别模型 (sherpa-onnx)
- 日中翻译模型 (opus-mt-ja-zh)
- 水印检测/修复模型

### 使用模型

```bash
# 克隆模型仓库到 assets/models 目录
git clone https://github.com/xxxzhou/avc_model.git assets/models
```

## 构建

### Windows

```bash
# 使用 GN 生成项目
gn gen out/Debug
ninja -C out/Debug
```

### Android

```bash
# 使用 GN 生成项目
gn gen out/Android --args="target_os=\"android\" target_cpu=\"arm64\""
ninja -C out/Android
```

## 依赖

- WebRTC
- ONNX Runtime (AI 推理)
- OpenSSL
- libyuv

## License

MIT
