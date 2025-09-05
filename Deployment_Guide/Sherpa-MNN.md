# Sherpa-MNN

# Sherpa-MNN 部署指南

本指南提供了在 macOS 上部署 Sherpa-MNN 语音识别框架的完整步骤，包括可能遇到的错误及解决方案。

## 1. 环境准备

```bash
# 创建 conda 环境
conda create -n sherpa-mnn python=3.9
conda activate sherpa-mnn

# 安装必要的依赖
brew install cmake
brew install portaudio
brew install wget

```

## 2. 构建 MNN

```bash
# 克隆 MNN 仓库（如果尚未克隆）
git clone https://github.com/alibaba/MNN.git
cd MNN

# 构建 MNN
mkdir -p build && cd build
cmake .. -DMNN_LOW_MEMORY=ON -DMNN_SEP_BUILD=OFF -DCMAKE_INSTALL_PREFIX=. -DMNN_BUILD_CONVERTER=ON
make -j$(sysctl -n hw.ncpu)
make install

```

**可能的错误**：

- 如果编译失败，可能是内存不足，尝试减少并行编译任务数：`make -j4`
- 如果找不到 CMakeLists.txt，确保你在正确的目录中

## 3. 下载和准备模型

```bash
# 进入 sherpa-mnn 目录
# 注意：根据你的实际路径调整，下面是一个示例
cd /path/to/MNN/apps/frameworks/sherpa-mnn

# 下载模型
mkdir -p sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20
wget https://github.com/k2-fsa/sherpa-onnx/releases/download/asr-models/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2 -O sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2
tar xf sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20.tar.bz2 -C sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20

# 创建 MNN 模型目录
mkdir -p sherpa-mnn-models

```

**可能的错误**：

- 如果 `cd` 命令报错 `No such file or directory`，请使用绝对路径：
    
    ```bash
    cd "/Users/username/path/to/MNN/apps/frameworks/sherpa-mnn"
    
    ```
    

## 4. 转换模型

```bash
# 使用 MNNConvert 将 ONNX 模型转换为 MNN 格式
# 注意：使用 MNNConvert 的完整路径
/path/to/MNN/build/MNNConvert -f ONNX --modelFile sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/encoder-epoch-99-avg-1.onnx --MNNModel sherpa-mnn-models/encode.mnn --weightQuantBits=8 --weightQuantBlock=64

/path/to/MNN/build/MNNConvert -f ONNX --modelFile sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/decoder-epoch-99-avg-1.onnx --MNNModel sherpa-mnn-models/decode.mnn --weightQuantBits=8 --weightQuantBlock=64

/path/to/MNN/build/MNNConvert -f ONNX --modelFile sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/joiner-epoch-99-avg-1.onnx --MNNModel sherpa-mnn-models/joiner.mnn --weightQuantBits=8 --weightQuantBlock=64

```

**可能的错误**：

- 如果 `MNNConvert` 命令报错 `No such file or directory`，请确保使用 MNNConvert 的完整路径
- 如果模型路径错误，检查解压后的目录结构，可能需要调整路径

## 5. 构建 Sherpa-MNN

```bash
# 在 sherpa-mnn 目录中
mkdir -p build && cd build

# 配置构建
cmake .. -DMNN_LIB_DIR="/absolute/path/to/MNN/build" -DSHERPA_MNN_ENABLE_TTS=ON -DSHERPA_MNN_ENABLE_SPEAKER_DIARIZATION=ON -DSHERPA_MNN_ENABLE_PORTAUDIO=ON -DSHERPA_MNN_ENABLE_WEBSOCKET=ON -DSHERPA_MNN_ENABLE_C_API=ON -DSHERPA_MNN_BUILD_C_API_EXAMPLES=ON

# 编译
make -j$(sysctl -n hw.ncpu)

```

**可能的错误**：

- 如果 `cmake` 报错找不到 MNN 库，请使用 MNN 构建目录的绝对路径
- 如果遇到 CMake 版本问题，可能需要修改依赖库的 CMakeLists.txt 文件：
    
    ```bash
    # 修改 kaldi_native_fbank 的 CMake 最低版本要求
    sed -i '' 's/cmake_minimum_required(VERSION 3.3 FATAL_ERROR)/cmake_minimum_required(VERSION 3.5 FATAL_ERROR)/' "/path/to/kaldi_native_fbank-src/CMakeLists.txt"
    
    # 修改 openfst 的 CMake 最低版本要求
    sed -i '' 's/cmake_minimum_required(VERSION [0-9]\.[0-9]*/cmake_minimum_required(VERSION 3.5/' "/path/to/openfst-src/CMakeLists.txt"
    
    # 修改 portaudio 的 CMake 最低版本要求
    sed -i '' 's/CMAKE_MINIMUM_REQUIRED *(VERSION [0-9]\.[0-9]*/CMAKE_MINIMUM_REQUIRED(VERSION 3.5/' "/path/to/portaudio-src/CMakeLists.txt"
    
    # 修改 cppjieba 的 CMake 最低版本要求
    sed -i '' 's/cmake_minimum_required(VERSION [0-9]\.[0-9]*/cmake_minimum_required(VERSION 3.5/' "/path/to/cppjieba-src/CMakeLists.txt"
    
    ```
    

## 6. 测试 Sherpa-MNN

```bash
# 测试语音识别
./bin/sherpa-mnn \
  --tokens=../sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20/tokens.txt \
  --encoder=../sherpa-mnn-models/encode.mnn \

```