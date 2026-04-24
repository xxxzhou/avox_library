# AVC Library

为 [avplay](https://github.com/xxxzhou/avplay) 项目存放大型二进制文件（dll、lib 等）。

## 目录结构

```
avc_library/
├── build/                  # WebRTC 构建产物
│   ├── android/            # Android .a 文件
│   ├── ios/                # iOS .a 文件
│   └── windows/            # Windows .lib 文件
├── 3rdparty/               # 第三方库
│   └── library/            # ONNX Runtime 等
└── src/                    # WebRTC 源码（git submodule）
```

## AI 模型

AI 模型存放在独立仓库：[avc_model](https://github.com/xxxzhou/avc_model)
