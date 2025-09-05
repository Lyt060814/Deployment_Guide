# BSRGAN

# BSRGAN 部署指南

本指南提供了在 macOS (特别是 Apple Silicon) 上部署 BSRGAN 图像超分辨率项目的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n bsrgan python=3.9 -y
conda activate bsrgan

```

### 2. 安装 PyTorch 和依赖包

```bash
pip install torch torchvision torchaudio --index-url <https://download.pytorch.org/whl/cpu>
pip install numpy pillow scikit-image opencv-python tqdm

```

**可能的错误**：如果安装过程中出现网络超时，可以尝试使用国内镜像源：

```bash
pip install torch torchvision torchaudio --index-url <https://download.pytorch.org/whl/cpu> -i <https://pypi.tuna.tsinghua.edu.cn/simple>
pip install numpy pillow scikit-image opencv-python tqdm -i <https://pypi.tuna.tsinghua.edu.cn/simple>

```

### 3. 安装额外依赖

```bash
pip install requests matplotlib

```

**说明**：`requests` 用于下载预训练模型，`matplotlib` 用于图像处理和可视化。

## 获取预训练模型

### 方法一：使用下载脚本（推荐）

```bash
python main_download_pretrained_models.py

```

**可能的错误**：如果脚本不存在或下载失败，请使用方法二或方法三。

### 方法二：直接下载模型文件

```bash
mkdir -p model_zoo
wget -O model_zoo/BSRGAN.pth <https://github.com/cszn/KAIR/releases/download/v1.0/BSRGAN.pth>
wget -O model_zoo/BSRNet.pth <https://github.com/cszn/KAIR/releases/download/v1.0/BSRNet.pth>

```

**可能的错误**：如果 `wget` 命令不可用，可以使用 `curl`：

```bash
curl -L <https://github.com/cszn/KAIR/releases/download/v1.0/BSRGAN.pth> -o model_zoo/BSRGAN.pth
curl -L <https://github.com/cszn/KAIR/releases/download/v1.0/BSRNet.pth> -o model_zoo/BSRNet.pth

```

### 方法三：手动下载

从以下链接手动下载模型文件，并放置在 `model_zoo/` 目录下：

- Google Drive: [https://drive.google.com/drive/folders/13kfr3qny7S2xwG9h7v95F5mkWs0OmU0D?usp=sharing](https://drive.google.com/drive/folders/13kfr3qny7S2xwG9h7v95F5mkWs0OmU0D?usp=sharing)
- Github Release: [https://github.com/cszn/KAIR/releases/tag/v1.0](https://github.com/cszn/KAIR/releases/tag/v1.0)

## 修复代码兼容性问题

对于 Apple Silicon 或无 GPU 环境，需要修改 `main_test_bsrgan.py` 文件以避免 CUDA 相关错误：

```bash
sed -i '' "s/logger.info('{:>16s} : {:<d}'.format('GPU ID', torch.cuda.current_device()))/logger.info('{:>16s} : {:<s}'.format('GPU ID', str(torch.cuda.current_device()) if torch.cuda.is_available() else 'cpu'))/g" main_test_bsrgan.py

```

**可能的错误**：如果 `sed` 命令失败，可以手动编辑文件，将：

```python
logger.info('{:>16s} : {:<d}'.format('GPU ID', torch.cuda.current_device()))

```

替换为：

```python
logger.info('{:>16s} : {:<s}'.format('GPU ID', str(torch.cuda.current_device()) if torch.cuda.is_available() else 'cpu'))

```

## 测试部署

### 运行测试图像

```bash
python main_test_bsrgan.py --input testsets/RealSRSet/oldphoto2.png --model_path model_zoo/BSRGAN.pth --output results/oldphoto2_BSRGAN_out.png

```

**可能的错误**：

1. 如果出现 `No module named 'utils'`，确保你在项目根目录下运行命令。
2. 如果出现 CUDA 相关错误，确保已执行上述代码修复步骤。
3. 如果出现 `FileNotFoundError`，确保测试图像路径正确，可能需要创建 `results` 目录：
    
    ```bash
    mkdir -p results
    
    ```
    

## 使用指南

### 1. 对自定义图像进行超分辨率处理

```bash
python main_test_bsrgan.py --input path/to/your_image.png --model_path model_zoo/BSRGAN.pth --output path/to/sr_image.png

```

### 2. 在 Python 代码中使用降质模型

```python
from utils import utils_blindsr as blindsr
lq, hq = blindsr.degradation_bsrgan(img, sf=4, lq_patchsize=72)

```

### 3. 批量处理图像

创建一个包含多张图像的目录，然后使用以下脚本处理：

```bash
for img in /path/to/images/*.png; do
    filename=$(basename -- "$img")
    python main_test_bsrgan.py --input "$img" --model_path model_zoo/BSRGAN.pth --output "results/${filename%.*}_BSRGAN.png"
done

```

## 故障排除

1. **内存不足**：处理大图像时可能遇到内存不足问题，可以尝试裁剪图像或使用较小的批处理大小。
2. **模型加载失败**：确保模型文件路径正确，并且文件完整下载。
3. **CUDA 错误**：如果在 GPU 环境中遇到 CUDA 错误，可能需要检查 PyTorch 和 CUDA 版本的兼容性。
4. **图像格式问题**：如果输入图像无法正确加载，尝试转换为常见格式如 PNG 或 JPEG。

## 进阶资源

- 训练自己的模型：参考项目 README 和 KAIR 仓库 [https://github.com/cszn/KAIR](https://github.com/cszn/KAIR)
- 更多预训练权重：查看 [https://github.com/cszn/KAIR/releases/tag/v1.0](https://github.com/cszn/KAIR/releases/tag/v1.0)