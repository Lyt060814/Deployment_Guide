# SAM

# Segment Anything 部署指南

本指南提供了部署 Segment Anything 项目的完整步骤，包括可能遇到的错误及解决方案。

## 1. 环境准备

### 创建 Conda 环境

```bash
conda create -n segment-anything python=3.10
conda activate segment-anything

```

## 2. 安装依赖

### 安装 PyTorch 和核心依赖

```bash
pip install torch torchvision
pip install matplotlib pycocotools opencv-python onnx onnxruntime
pip install -e .

```

> 注意：在 Apple Silicon 设备上，这将安装 CPU 版本的 PyTorch，以确保最佳兼容性。
> 

## 3. 下载模型检查点

```bash
curl -L -o sam_vit_h_4b8939.pth <https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth>

```

> 可能的错误：下载可能因网络问题失败。
> 
> 
> **解决方案**：可以尝试使用浏览器手动下载，或使用代理。
> 

## 4. 导出 ONNX 模型和图像嵌入

### 创建必要的目录

```bash
mkdir -p demo/model
mkdir -p demo/src/assets/data

```

> 错误：如果不创建目录，会出现 FileNotFoundError: [Errno 2] No such file or directory: 'demo/model/sam_onnx_example.onnx'
> 
> 
> **解决方案**：确保在运行导出脚本前创建所需目录。
> 

### 导出 ONNX 模型

```bash
python scripts/export_onnx_model.py --checkpoint sam_vit_h_4b8939.pth --model-type vit_h --output demo/model/sam_onnx_example.onnx

```

### 量化 ONNX 模型

```bash
python -c "from onnxruntime.quantization import quantize_dynamic, QuantType; quantize_dynamic('demo/model/sam_onnx_example.onnx','demo/model/sam_onnx_quantized_example.onnx', per_channel=False, reduce_range=False, weight_type=QuantType.QUInt8)"

```

> 错误：原命令中包含 optimize_model=True 参数，但会导致 TypeError: quantize_dynamic() got an unexpected keyword argument 'optimize_model'
> 
> 
> **解决方案**：移除 `optimize_model=True` 参数。
> 

### 生成图像嵌入

```bash
python -c "import cv2; import numpy as np; from segment_anything import SamPredictor, sam_model_registry; sam = sam_model_registry['vit_h'](checkpoint='sam_vit_h_4b8939.pth'); predictor = SamPredictor(sam); image = cv2.imread('demo/src/assets/data/dogs.jpg'); predictor.set_image(image); emb = predictor.get_image_embedding().cpu().numpy(); np.save('demo/src/assets/data/dogs_embedding.npy', emb)"

```

> 注意：确保 demo/src/assets/data/dogs.jpg 文件存在。如果不存在，需要先复制或下载示例图片。
> 

## 5. 验证 Python 安装

```bash
python -c "import segment_anything; print('segment_anything import successful')"

```

## 6. 前端设置

### 安装 Yarn

```bash
npm install --g yarn

```

> 错误：可能会遇到权限问题 npm error code EACCES
> 
> 
> **解决方案**：使用 sudo 安装
> 
> ```bash
> sudo npm install --g yarn
> 
> ```
> 

### 安装前端依赖

```bash
cd demo && yarn

```

> 错误：如果找不到 demo 目录，会出现 bash: cd: demo: No such file or directory
> 
> 
> **解决方案**：确保你在项目根目录下，或使用完整路径
> 
> ```bash
> cd /完整/路径/到/segment-anything/demo && yarn
> 
> ```
> 

## 7. 运行演示应用

```bash
cd demo && yarn start

```

> 注意：这将启动一个开发服务器，你可以在浏览器中访问 http://localhost:8081/ 查看演示。
> 

## 8. 故障排除

### 检查文件是否存在

确保以下文件存在：

- `demo/model/sam_onnx_example.onnx`
- `demo/model/sam_onnx_quantized_example.onnx`
- `demo/src/assets/data/dogs_embedding.npy`

### 路径问题

如果遇到路径相关错误，请确保你在正确的目录中执行命令，或使用绝对路径。

### 内存问题

导出模型和生成嵌入需要大量内存。如果遇到内存错误，请关闭其他应用程序释放内存。

## 9. 使用说明

成功部署后：

- 访问 [http://localhost:8081/](http://localhost:8081/) 查看演示
- 要为自己的图像导出嵌入/模型，请参考 demo/README.md
- 如果遇到问题，请检查 ONNX 模型和嵌入是否存在于 demo/model 和 demo/src/assets/data 中