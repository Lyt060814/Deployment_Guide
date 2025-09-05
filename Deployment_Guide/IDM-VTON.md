# IDM-VTON

# IDM-VTON 部署指南

本指南提供了在 macOS/arm64 平台上部署 IDM-VTON 虚拟试衣系统的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建并激活 conda 环境

```bash
conda env create -f environment.yaml
conda activate idm

```

**可能的错误**:

- **错误**: `ResolvePackageNotFound` 某些包无法找到
**解决方案**: 手动安装缺失的包，或更新 conda 源 `conda config --add channels conda-forge`
- **错误**: 环境创建过程中内存不足
**解决方案**: 关闭其他应用程序释放内存，或增加 swap 空间

### 2. 更新环境依赖

```bash
conda env update -f environment.yaml --prune
conda activate idm

```

**可能的错误**:

- **错误**: 某些包版本冲突
**解决方案**: 编辑 environment.yaml 文件，放宽版本限制或移除冲突包，之后手动安装

## 安装 Detectron2

### 3. 从源码安装 Detectron2 (macOS/arm64)

```bash
export CC=clang
export CXX=clang++
python -m pip install 'git+https://github.com/facebookresearch/detectron2.git'

```

**可能的错误**:

- **错误**: 编译失败，缺少 C++ 编译器
**解决方案**: 安装 Xcode 命令行工具 `xcode-select --install`
- **错误**: 编译过程中出现 PyTorch 相关错误
**解决方案**: 确保已安装正确版本的 PyTorch，对于 macOS/arm64，使用 `pip install torch torchvision torchaudio`
- **错误**: 内存不足导致编译失败
**解决方案**: 增加系统 swap 空间或关闭其他应用程序

## 验证安装

### 4. 验证核心模块导入

```bash
python -c "import torch; import detectron2; print('Detectron2 import successful')"
python -c "import diffusers; import accelerate; import gradio; print('Core IDM-VTON modules import successful')"

```

**可能的错误**:

- **错误**: `ImportError: No module named 'xxx'`**解决方案**: 手动安装缺失的模块 `pip install xxx`
- **错误**: Detectron2 导入错误，提示 CUDA 相关问题
**解决方案**: 在 macOS/arm64 上，Detectron2 将以 CPU 模式运行，可以忽略 CUDA 警告

## 下载模型和数据集

### 5. 下载预训练模型和检查点

```bash
# 创建模型目录
mkdir -p checkpoints

# 下载模型 (根据项目README中的链接)
# 示例:
wget -O checkpoints/model.ckpt <https://path-to-model-checkpoint>

```

**可能的错误**:

- **错误**: 下载失败或链接过期
**解决方案**: 检查项目 README 获取最新链接，或联系项目维护者

### 6. 准备数据集

```bash
# 创建数据目录
mkdir -p data

# 下载并解压数据集 (根据项目要求)
# 示例:
wget -O data/dataset.zip <https://path-to-dataset>
unzip data/dataset.zip -d data/

```

**可能的错误**:

- **错误**: 磁盘空间不足
**解决方案**: 清理磁盘空间或使用外部存储

## 运行应用

### 7. 运行训练 (可选)

```bash
accelerate launch train_xl.py --gradient_checkpointing --use_8bit_adam --output_dir=result --train_batch_size=6 --data_dir=DATA_DIR

```

**可能的错误**:

- **错误**: 内存不足
**解决方案**: 减小 batch_size，添加 `-gradient_accumulation_steps` 参数
- **错误**: `accelerate` 命令未找到
**解决方案**: 安装 accelerate `pip install accelerate`

### 8. 运行推理

```bash
accelerate launch inference.py --width 768 --height 1024 --num_inference_steps 30 --output_dir result --unpaired --data_dir DATA_DIR --seed 42 --test_batch_size 2 --guidance_scale 2.0

```

**可能的错误**:

- **错误**: 内存不足
**解决方案**: 减小 `test_batch_size` 或图像尺寸 (`width` 和 `height`)
- **错误**: 找不到模型文件
**解决方案**: 确保模型文件已下载并放置在正确位置，检查路径配置

### 9. 启动 Gradio 演示界面

```bash
python gradio_demo/app.py

```

**可能的错误**:

- **错误**: Gradio 相关导入错误
**解决方案**: 安装 Gradio `pip install gradio`
- **错误**: 端口被占用
**解决方案**: 修改 [app.py](http://app.py/) 中的端口号，或终止占用端口的进程

## 注意事项

1. 在 macOS/arm64 平台上，所有处理将在 CPU 模式下运行，速度会比 GPU 慢很多
2. 推理和训练过程可能需要较长时间，请耐心等待
3. 确保系统有足够的内存和磁盘空间
4. 参考项目 [README.md](http://readme.md/) 获取更多关于数据集设置和模型下载的详细信息

## 故障排除

如果遇到其他问题，可尝试以下通用解决方案:

1. 检查 Python 版本是否为 3.10
2. 确保 conda 环境已正确激活
3. 尝试重新安装有问题的依赖包
4. 查看项目 issues 页面，寻找类似问题的解决方案
5. 对于 macOS/arm64 特定问题，确保使用了适合 Apple Silicon 的 PyTorch 版本