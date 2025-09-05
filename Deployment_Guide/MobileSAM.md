# MobileSAM

# MobileSAM 部署指南

本指南提供了在 macOS/arm64 系统上部署 MobileSAM 的完整步骤，包括可能遇到的错误及解决方案。

## 1. 环境准备

### 创建 Conda 环境

```bash
conda create -n mobilesam python=3.8
conda activate mobilesam

```

## 2. 安装基础依赖

```bash
pip install --upgrade pip
pip install torch torchvision --index-url <https://download.pytorch.org/whl/cpu>

```

**可能的错误**：在某些系统上，可能会出现 PyTorch 安装失败的情况。
**解决方案**：尝试不指定 index-url，直接使用 `pip install torch torchvision`。

## 3. 安装项目依赖

```bash
pip install timm opencv-python gradio
pip install -e .

```

**可能的错误**：如果出现 `pip install -e .` 失败，可能是权限问题或当前目录不是项目根目录。
**解决方案**：确认当前目录是项目根目录，如有必要使用 `sudo pip install -e .`。

## 4. 安装可选依赖

```bash
pip install matplotlib pycocotools

```

### ONNX 相关依赖（可选）

```bash
pip install onnx --upgrade
pip install onnxruntime==1.13.1

```

**可能的错误**：安装特定版本的 onnx (1.12.0) 可能会失败，出现 CMake 错误。
**解决方案**：使用 `pip install onnx --upgrade` 安装最新版本，通常可以避免编译问题。

### 开发工具（可选）

```bash
pip install flake8 isort black mypy

```

## 5. 下载模型权重

```bash
mkdir -p weights
wget -O 'weights/mobile_sam.pt' <https://raw.githubusercontent.com/ChaoningZhang/MobileSAM/master/weights/mobile_sam.pt>

```

**可能的错误**：如果 wget 不可用或下载失败。
**解决方案**：

- 如果没有 wget：`curl -o weights/mobile_sam.pt <https://raw.githubusercontent.com/ChaoningZhang/MobileSAM/master/weights/mobile_sam.pt`>
- 或者手动从 GitHub 下载模型文件并放入 weights 目录

## 6. 验证安装

```bash
python -c "import torch; import mobile_sam; print('MobileSAM import successful')"

```

**可能的错误**：可能会出现一些关于 timm 库的警告，但这些通常不影响功能。
**解决方案**：这些警告可以忽略，只要最后显示 "MobileSAM import successful" 即可。

## 7. 运行演示应用

```bash
cd app
python app.py

```

**可能的错误**：可能会出现 `FileNotFoundError: [Errno 2] No such file or directory: '../weights/mobile_sam.pt'`。
**解决方案**：

1. 确保已经下载了模型权重文件并放在正确的位置
2. 检查 [app.py](http://app.py/) 中的模型路径是否正确，可能需要修改为绝对路径

## 8. 使用示例

### 程序化调用

```python
from mobile_sam import sam_model_registry, SamPredictor

# 加载模型
model = sam_model_registry['vit_t'](checkpoint='./weights/mobile_sam.pt')
predictor = SamPredictor(model)

# 处理图像
# predictor.set_image(your_image)
# masks, _, _ = predictor.predict(...)

```

### ONNX 导出（可选）

```bash
python scripts/export_onnx_model.py --checkpoint ./weights/mobile_sam.pt --model-type vit_t --output ./mobile_sam.onnx

```

## 注意事项

1. 确保模型权重文件 (`mobile_sam.pt`) 已正确下载并放置在 weights 目录中
2. 在 Apple Silicon Mac 上，PyTorch 会自动使用 MPS 加速，无需额外配置
3. 如果遇到内存不足问题，可以尝试减小批处理大小或使用较小的输入图像
4. 项目中的警告信息（关于 timm 库）不影响功能，可以忽略

## 故障排除

如果遇到其他问题：

1. 检查 Python 和依赖库版本是否兼容
2. 确保已安装所有必需的依赖项
3. 查看项目 GitHub 仓库的 Issues 部分，寻找类似问题的解决方案
4. 尝试在干净的环境中重新安装