# SwinIR

# SwinIR 部署指南

本指南提供了在 macOS (特别是 Apple Silicon) 上部署 SwinIR 图像恢复项目的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n swinir python=3.10 -y
conda activate swinir

```

### 2. 安装 PyTorch 和基础依赖

```bash
pip install torch torchvision torchaudio

```

**可能的错误**：如果安装过程中出现网络问题或超时
**解决方案**：尝试使用国内镜像源或增加超时时间

```bash
pip install torch torchvision torchaudio -i <https://pypi.tuna.tsinghua.edu.cn/simple> --timeout 300

```

### 3. 安装项目依赖

```bash
# 如果项目中有 requirements.txt
if [ -f requirements.txt ]; then
    pip install -r requirements.txt
else
    pip install numpy pillow requests tqdm scikit-image opencv-python
fi

```

### 4. 安装缺失的依赖

```bash
pip install timm

```

**可能的错误**：`ModuleNotFoundError: No module named 'timm'`**解决方案**：如上所示，手动安装 timm 库

## 验证安装

### 1. 验证 PyTorch 安装

```bash
python -c "import torch; print('PyTorch:', torch.__version__, 'MPS available:', torch.backends.mps.is_available())"

```

**预期输出**：显示 PyTorch 版本和 MPS (Apple GPU) 是否可用

### 2. 验证项目模块导入

```bash
python -c "import main_test_swinir; print('main_test_swinir import: SUCCESS')"

```

**可能的错误**：如果不在项目根目录下运行
**解决方案**：确保在项目根目录下运行命令

```bash
cd /path/to/SwinIR

```

## 运行测试

### 1. 经典超分辨率测试

```bash
python main_test_swinir.py --task classical_sr --scale 2 --training_patch_size 48 --model_path model_zoo/swinir/001_classicalSR_DIV2K_s48w8_SwinIR-M_x2.pth --folder_lq testsets/Set5/LR_bicubic/X2 --folder_gt testsets/Set5/HR

```

**可能的错误**：内存不足
**解决方案**：添加 `--tile 400` 参数分块处理图像

```bash
python main_test_swinir.py --task classical_sr --scale 2 --training_patch_size 48 --model_path model_zoo/swinir/001_classicalSR_DIV2K_s48w8_SwinIR-M_x2.pth --folder_lq testsets/Set5/LR_bicubic/X2 --folder_gt testsets/Set5/HR --tile 400

```

**可能的错误**：找不到模型文件
**解决方案**：模型会自动下载，但如果下载失败，可以手动创建目录并下载

```bash
mkdir -p model_zoo/swinir
# 然后手动下载模型到该目录

```

## 常用命令示例

### 1. 经典超分辨率 (x2)

```bash
python main_test_swinir.py --task classical_sr --scale 2 --training_patch_size 48 --model_path model_zoo/swinir/001_classicalSR_DIV2K_s48w8_SwinIR-M_x2.pth --folder_lq testsets/Set5/LR_bicubic/X2 --folder_gt testsets/Set5/HR

```

### 2. 轻量级超分辨率

```bash
python main_test_swinir.py --task lightweight_sr --scale 2 --model_path model_zoo/swinir/002_lightweightSR_DIV2K_s64w8_SwinIR-S_x2.pth --folder_lq testsets/Set5/LR_bicubic/X2 --folder_gt testsets/Set5/HR

```

### 3. 真实世界超分辨率

```bash
python main_test_swinir.py --task real_sr --scale 4 --model_path model_zoo/swinir/003_realSR_BSRGAN_DFO_s64w8_SwinIR-M_x4_GAN.pth --folder_lq testsets/RealSRSet+5images --tile

```

### 4. 降噪

```bash
python main_test_swinir.py --task gray_dn --noise 15 --model_path model_zoo/swinir/004_grayDN_DFWB_s128w8_SwinIR-M_noise15.pth --folder_gt testsets/Set12

```

## 注意事项

1. 对于 Apple Silicon (M1/M2/M3) 用户，PyTorch 现已支持 MPS 加速。如果 `MPS available: False`，将自动回退到 CPU 模式。
2. 处理大图像时可能会遇到内存不足问题，此时可以使用 `-tile` 参数指定分块大小。
3. 首次运行时，模型会自动下载，请确保网络连接正常。
4. 如果遇到其他依赖问题，可以根据错误信息使用 `pip install` 安装缺失的包。