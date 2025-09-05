# TinyLLaVA-Video

# TinyLLaVA-Video 部署指南

本指南提供了 TinyLLaVA-Video 项目的完整部署步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n tinyllava_video python=3.10
conda activate tinyllava_video

```

**说明**: 创建一个独立的 Python 3.10 环境，避免依赖冲突。

### 2. 安装项目依赖

```bash
pip install --upgrade pip
pip install -e .

```

**说明**: 升级 pip 并以可编辑模式安装项目及其依赖。

**可能的错误**:

- 如果出现 `ERROR: Directory '.' is not installable.`，确保你在项目根目录下执行命令。
- 解决方案: `cd /path/to/TinyLLaVA-Video` 然后重试。

### 3. 安装 flash-attn

```bash
pip install flash-attn --no-build-isolation || echo 'flash-attn install failed (likely not supported on Mac) safe to ignore unless you want GPU/optimized attention.'

```

**可能的错误**:

- 语法错误: `bash: syntax error near unexpected token ')'`
- 解决方案: 确保引号匹配正确，移除特殊字符，如上所示。

**注意**: 在 Apple Silicon Mac 上，flash-attn 安装可能会失败，这是预期的，不会影响基本功能。

### 4. 验证安装

```bash
python -c 'import torch; import tinyllava; print("TinyLLaVA-Video and torch import successful!")'

```

**可能的错误**:

- 语法错误: `bash: !': event not found`
- 解决方案: 在 bash 中使用单引号包裹 Python 代码，并在内部使用双引号，如上所示。
- 导入错误: `ModuleNotFoundError: No module named 'tinyllava'`
- 解决方案: 确保你在项目根目录下执行了 `pip install -e .`，并且安装成功。

## 使用指南

### 运行推理

```bash
# 编辑 eval.py 设置模型路径、提示和视频文件等
CUDA_VISIBLE_DEVICES=0 python eval.py

```

**说明**: 对于训练或在基准测试上进行评估，请编辑并运行 scripts/train 或 scripts/eval 中的相关脚本。

**可能的错误**:

- 如果在 Mac 上运行，可能需要移除 `CUDA_VISIBLE_DEVICES=0`，因为 Mac 不使用 CUDA。
- 解决方案: 直接运行 `python eval.py`

### 其他常见问题

1. **内存不足错误**:
    - 错误: `CUDA out of memory`
    - 解决方案: 减小批处理大小或使用较小的模型变体。
2. **模型下载问题**:
    - 错误: 无法下载预训练模型
    - 解决方案: 手动下载模型并放置在正确的目录中，或检查网络连接。
3. **依赖冲突**:
    - 错误: 版本冲突错误
    - 解决方案: 在全新的 conda 环境中安装，严格按照指定版本安装依赖。

## 参考资源

完成安装后，请参考项目的 README.md 获取有关数据集设置和高级用法的完整详细信息。