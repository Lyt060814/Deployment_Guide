# Real-ESRGAN

# Real-ESRGAN 部署指南

本指南提供了在 macOS (特别是 Apple Silicon) 上部署 Real-ESRGAN 的完整步骤，包括可能遇到的错误及其解决方案。

## 1. 环境准备

### 安装 Miniforge (适用于 Apple Silicon)

```bash
curl -L -o Miniforge3-MacOSX-arm64.sh <https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh>
bash Miniforge3-MacOSX-arm64.sh -b -p $HOME/miniforge3
eval "$($HOME/miniforge3/bin/conda shell.bash hook)"
conda config --set auto_activate_base false

```

### 创建并激活 conda 环境

```bash
conda create -n realesrgan-env python=3.10
conda activate realesrgan-env

```

## 2. 安装依赖

### 安装 PyTorch 和基础依赖

```bash
pip install --upgrade pip
pip install torch torchvision torchaudio --index-url <https://download.pytorch.org/whl/cpu>

```

> 注意：如果遇到 PyTorch 安装问题，可以尝试直接从官方网站获取适合 Apple Silicon 的安装命令。
> 

### 安装其他依赖

```bash
pip install opencv-python Pillow tqdm

```

### 安装 basicsr 和其他关键依赖

```bash
# 清除可能存在的冲突包
pip uninstall -y basicsr scipy
pip cache purge

# 使用 PEP 517 安装 basicsr
pip install --use-pep517 basicsr

# 安装额外依赖
pip install filterpy numba

```

> 错误解决：如果遇到 tb-nightly 依赖错误，可以尝试直接安装项目而不是通过 requirements.txt。
> 

### 安装项目

```bash
python setup.py develop

```

> 错误解决：如果遇到 torchvision.transforms.functional_tensor 模块缺失错误，需要更新 torchvision 并修复导入路径：
> 
> 
> ```bash
> pip install --upgrade torchvision
> sed -i '' 's/from torchvision.transforms.functional_tensor import rgb_to_grayscale/from torchvision.transforms.functional import rgb_to_grayscale/' /path/to/conda/env/lib/python3.10/site-packages/basicsr/data/degradations.py
> 
> ```
> 
> 请将 `/path/to/conda/env` 替换为你的 conda 环境路径，通常是 `~/miniforge3/envs/realesrgan-env`。
> 

## 3. 下载预训练模型

### 创建模型权重目录

```bash
mkdir -p weights

```

### 下载照片超分辨率模型

```bash
wget <https://github.com/xinntao/Real-ESRGAN/releases/download/v0.1.0/RealESRGAN_x4plus.pth> -P weights

```

### 下载动漫超分辨率模型

```bash
wget <https://github.com/xinntao/Real-ESRGAN/releases/download/v0.2.2.4/RealESRGAN_x4plus_anime_6B.pth> -P weights

```

## 4. 验证安装

### 测试导入

```bash
python -c "import realesrgan; print('Import successful')"

```

### 创建测试图像

```bash
mkdir -p inputs
python -c "from PIL import Image; Image.new('RGB', (64,64), color='white').save('inputs/test.png')"

```

### 运行照片超分辨率推理

```bash
python inference_realesrgan.py -n RealESRGAN_x4plus -i inputs/test.png --fp32

```

### 运行动漫超分辨率推理

```bash
python inference_realesrgan.py -n RealESRGAN_x4plus_anime_6B -i inputs/test.png --fp32

```

## 5. 使用指南

Real-ESRGAN 设置完成后，可以使用以下命令处理图像：

- **处理照片文件夹**：
    
    ```bash
    python inference_realesrgan.py -n RealESRGAN_x4plus -i <input_folder> --fp32
    
    ```
    
- **处理动漫图像文件夹**：
    
    ```bash
    python inference_realesrgan.py -n RealESRGAN_x4plus_anime_6B -i <input_folder> --fp32
    
    ```
    
- **使用面部增强处理照片**：
    
    ```bash
    python inference_realesrgan.py -n RealESRGAN_x4plus -i <input_folder> --face_enhance --fp32
    
    ```
    
- 结果将保存在 `./results/` 目录中。
- 在 Apple Silicon 上，PyTorch MPS 后端会自动使用以获得最佳速度。
- 更多高级选项（平铺、输出比例、视频等）请参考 `README.md`。

## 常见问题解决

1. **bash 特殊字符问题**：如果在运行包含感叹号的 `echo` 命令时遇到 `event not found` 错误，请使用单引号包裹整个字符串。
2. **依赖冲突**：如果遇到依赖冲突，尝试先卸载冲突包，清除 pip 缓存，然后重新安装。
3. **模块导入错误**：对于 `ModuleNotFoundError`，检查是否需要更新相关包或修复导入路径。
4. **安装失败**：如果 `pip install -r requirements.txt` 失败，可以尝试单独安装每个依赖。