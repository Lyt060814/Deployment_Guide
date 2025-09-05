# Video-XL

# Video-XL 项目部署指南

本指南详细介绍了如何部署Video-XL、Video-XL-Pro和Video-XL-2三个子项目，包括环境配置、依赖安装和常见问题解决方案。

## 目录

1. [前期准备](https://grok.com/chat/3094e2f6-bc63-44e9-b981-b1cf8a911782#%E5%89%8D%E6%9C%9F%E5%87%86%E5%A4%87)
2. [Video-XL 部署](https://grok.com/chat/3094e2f6-bc63-44e9-b981-b1cf8a911782#video-xl-%E9%83%A8%E7%BD%B2)
3. [Video-XL-Pro 部署](https://grok.com/chat/3094e2f6-bc63-44e9-b981-b1cf8a911782#video-xl-pro-%E9%83%A8%E7%BD%B2)
4. [Video-XL-2 部署](https://grok.com/chat/3094e2f6-bc63-44e9-b981-b1cf8a911782#video-xl-2-%E9%83%A8%E7%BD%B2)
5. [常见问题与解决方案](https://grok.com/chat/3094e2f6-bc63-44e9-b981-b1cf8a911782#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E4%B8%8E%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)
6. [使用提示](https://grok.com/chat/3094e2f6-bc63-44e9-b981-b1cf8a911782#%E4%BD%BF%E7%94%A8%E6%8F%90%E7%A4%BA)

## 前期准备

确保系统满足以下要求：

- macOS (特别是Apple Silicon)或Linux系统
- 足够的RAM (建议96GB+)
- 充足的磁盘空间 (>200GB)
- 已安装Anaconda或Miniconda

## Video-XL 部署

### 1. 创建并激活conda环境

```bash
conda create -n videoxl python=3.10
conda activate videoxl

```

### 2. 安装PyTorch和基础依赖

```bash
pip install torch torchvision
pip install packaging ninja
pip install py-cpuinfo

```

### 3. 安装flash-attention (可选，在Mac上可能不支持)

```bash
pip install flash-attn --no-build-isolation --no-cache-dir || echo 'flash-attn not available on Mac, skipping.'

```

### 4. 克隆并安装Video-XL

```bash
git clone https://github.com/iejMac/Video-XL.git
cd Video-XL
pip install -r requirements.txt

```

### 5. 验证安装

```bash
PYTHONPATH=. python -c "import videoxl; print('Video-XL import successful')"

```

**常见问题**：

- 如果出现`ModuleNotFoundError: No module named 'videoxl'`，确保设置了正确的PYTHONPATH或在Video-XL目录中执行命令。

## Video-XL-Pro 部署

### 1. 创建并激活conda环境

```bash
conda create -n videoxlpro python=3.10
conda activate videoxlpro

```

### 2. 安装PyTorch和基础依赖

```bash
pip install torch torchvision
pip install packaging ninja
pip install py-cpuinfo psutil

```

### 3. 安装flash-attention (可选，在Mac上可能不支持)

```bash
pip install flash-attn --no-build-isolation --no-cache-dir || echo 'flash-attn not available on Mac, skipping.'

```

### 4. 克隆并安装Video-XL-Pro

```bash
git clone https://github.com/iejMac/Video-XL-Pro.git
cd Video-XL-Pro
pip install -e .

```

### 5. 验证安装

```bash
PYTHONPATH=$PWD python -c "import videoxlpro; print('Video-XL-Pro import successful')"

```

**常见问题**：

- 如果出现`ModuleNotFoundError: No module named 'videoxlpro'`，确保设置了正确的PYTHONPATH或在Video-XL-Pro目录中执行命令。

## Video-XL-2 部署

### 1. 创建并激活conda环境

```bash
conda create -n videoxl2 python=3.10
conda activate videoxl2

```

### 2. 安装PyTorch和基础依赖

```bash
pip install torch torchvision

```

### 3. 克隆Video-XL-2仓库

```bash
git clone https://github.com/iejMac/Video-XL-2.git

```

### 4. 处理requirements.txt中的文件路径依赖

在Mac上，需要移除requirements.txt中的本地文件路径依赖：

```bash
# 移除所有包含file:///的行
sed -i '' '/@ file:\\/\\//d' 'Video-XL-2/train/requirements.txt'

```

### 5. 安装依赖

```bash
cd Video-XL-2
pip install -r train/requirements.txt

```

### 6. 验证安装

```bash
python -c "import torch; print('Video-XL-2 torch import successful')"

```

## 常见问题与解决方案

### 1. 找不到模块错误

**问题**：`ModuleNotFoundError: No module named 'videoxl'`或类似错误

**解决方案**：

```bash
# 设置PYTHONPATH指向项目根目录
PYTHONPATH=. python -c "import videoxl; print('Video-XL import successful')"
# 或
cd 项目目录
python -c "import videoxl; print('Video-XL import successful')"

```

### 2. flash-attn安装失败

**问题**：在Mac上安装flash-attn时出现CUDA相关错误

**解决方案**：

- 在Mac上，特别是Apple Silicon上，可以跳过flash-attn安装
- 使用`-no-build-isolation --no-cache-dir`选项尝试安装，如果失败则跳过

### 3. requirements.txt中的文件路径依赖

**问题**：安装依赖时出现`ERROR: Could not install packages due to an OSError: [Errno 2] No such file or directory: '/apex'`

**解决方案**：

```bash
# 移除所有包含file:///的行
sed -i '' '/@ file:\\/\\//d' 'Video-XL-2/train/requirements.txt'

```

### 4. 找不到requirements.txt

**问题**：`ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'`

**解决方案**：

- 确保在正确的目录中执行命令
- 指定完整路径：`pip install -r /path/to/requirements.txt`

### 5. 安装editable包失败

**问题**：`ERROR: videoxl/.[train] is not a valid editable requirement`

**解决方案**：

- 确保在正确的目录中执行命令
- 使用正确的相对或绝对路径：`pip install -e .`

## 使用提示

1. **Apple Silicon/CPU注意事项**：
    - 在Mac上，torch将使用CPU/Apple GPU，预期推理速度较慢
    - 某些包（如flash-attn、bitsandbytes、torch-scatter等）可能在Mac上不可用
    - 如果安装失败，可以跳过这些包并尝试使用核心功能
2. **性能优化**：
    - 在Apple Silicon上，使用torch>=2.0和torchvision，但不要使用CUDA wheels
    - 使用特定子项目时，激活其conda环境（例如，`conda activate videoxl`）
3. **模型下载**：
    - 查看每个子项目的README获取HuggingFace模型下载和使用说明
4. **路径设置**：
    - 如果导入模块失败，请设置正确的PYTHONPATH：`PYTHONPATH=. python your_script.py`