# darwin/opencv

macOS(arm64) 版 OpenCV 4.13.0, 供 avox 的 avox_opencv / avox_cv / avox_ocr / avox_avatar / avox_calib 插件使用。
目录关系对齐 `library/windows/opencv`(avc_library 库仓), 但 macOS 无 vc16 那层:

```
darwin/opencv/
  include/opencv2/   # opencv2/**.hpp (含构建生成的 cvdef/版本头)
  lib/               # libopencv_world4130.a (静态, 已 libtool 合并 3rdparty 解码库, 自包含)
```

> 产物 2026-09-23 由同级 `../opencv` 源码构建 (tag 4.13.0, Ninja, arm64, deployment target 11.0)。

## 为什么不能走 Homebrew

- 本机 brew 判定 Tier 3(自定义 prefix + macOS 26.6), 无 bottle → 26 个依赖全部源码编译(含已坏的 ffmpeg 与巨型 openvino), 极慢且大概率挂
- brew 的 opencv 现在是 **5.0.0**, 与 `cmake/FindOpenCV.cmake` 的 `OpenCV_VERSION 4.13.0` 与库名 `opencv_world4130` 不匹配
- 官方 release 无 macOS 预编译包(只有 windows.exe / android-sdk / ios-framework / linux-x64.sh)
- `script/opencv/download_opencv.py` 的 `detect_platform()` 把 darwin 映射成 `ios`(拿 iOS framework, macOS 链不了)

## 重建步骤

```bash
cd /Volumes/PSSD/work/github && git clone --depth 1 --branch 4.13.0 https://github.com/opencv/opencv.git
cd opencv
# 模块集按 avox 插件实际引用裁剪(opencv.hpp 是按 HAVE_OPENCV_* 条件包含的, 缺模块不报错);
# BUILD_LIST 排除 highgui 时必须同时 -DBUILD_opencv_highgui=OFF, 否则 world/CMakeLists.txt:95
# 调到不存在的 ocv_highgui_configure_target 直接配置失败
PATH="$HOME/.homebrew/bin:$PATH" cmake -B build/macos -G Ninja \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=11.0 -DCMAKE_INSTALL_PREFIX=$PWD/install \
  -DBUILD_SHARED_LIBS=OFF -DBUILD_opencv_world=ON \
  -DBUILD_LIST=core,imgproc,imgcodecs,calib3d,features2d,flann,objdetect \
  -DBUILD_opencv_highgui=OFF -DBUILD_opencv_videoio=OFF -DBUILD_opencv_dnn=OFF \
  -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_EXAMPLES=OFF -DBUILD_opencv_apps=OFF \
  -DBUILD_JAVA=OFF -DBUILD_opencv_java=OFF -DBUILD_opencv_python3=OFF \
  -DWITH_IPP=OFF -DWITH_CUDA=OFF -DWITH_OPENCL=OFF -DWITH_1394=OFF \
  -DWITH_VTK=OFF -DWITH_TESSERACT=OFF -DWITH_OPENVINO=OFF \
  -DWITH_GTK=OFF -DWITH_QT=OFF -DWITH_FFMPEG=OFF -DWITH_GSTREAMER=OFF \
  -DWITH_ITT=OFF -DWITH_EIGEN=OFF -DWITH_LAPACK=OFF
cmake --build build/macos -j 8 && cmake --install build/macos
# world.a 不含 3rdparty 解码库(jpeg/png/webp/tiff/zlib/openjp2/IlmImf/kleidicv/tegra_hal),
# libtool 合并成单一自包含库, 保住 FindOpenCV「单库」契约(否则消费方要多链 10 个库)
/usr/bin/libtool -static -o build/macos/merged/libopencv_world4130.a \
  install/lib/libopencv_world.a install/lib/opencv4/3rdparty/*.a
rm -rf include/opencv2 lib/libopencv_world4130.a
cp -R install/include/opencv4/opencv2 include/
cp build/macos/merged/libopencv_world4130.a lib/
```

实测: 配置 26s + 编译 1m04s(888 target, 模块裁剪后) + 合并拷贝, 产物 25MB。
`nm -u` 核查无 vImage/CF*/NS* 等系统框架依赖, 仅 libc/libm —— 静态链不需要额外 `-framework`。

## 消费方注意

- avox 核心不链 opencv: `src/avox/video/ImageIO.cpp` 的 `#ifdef AVOX_ENABLE_OPENCV` 只走
  `imageProcHub.create("opencv")` 运行期动态调用, `libavox.a` 无 cv 符号 → 宿主(avoxtest 等)
  链 `libavox.a` 不需要 opencv
- 只有插件动态库链接它, `register_plugin(LIBS ${OpenCV_LIBRARIES})` 单库即闭环
- 插件 dylib 体积参考: avox_opencv 12MB / calib 13MB / cv 6MB / ocr 6MB / avatar 5MB
