# GroundingDINO

# GroundingDINO 部署指南

本指南详细介绍了如何在 macOS 系统上部署 GroundingDINO 项目，特别适用于 Mac ARM 架构（M1/M2/M3）。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n groundingdino python=3.9 -y

```

> 注意：虽然 Python 3.10 也可以工作，但 3.9 与大多数依赖库兼容性更好。
> 

### 2. 激活环境并升级 pip

```bash
source $(conda info --base)/etc/profile.d/conda.sh
conda activate groundingdino
python -m pip install --upgrade pip

```

## 安装依赖

### 3. 安装 PyTorch 和其他依赖

```bash
pip install torch torchvision torchaudio
pip install transformers addict yapf timm numpy opencv-python supervision>=0.22.0 pycocotools

```

> 注意：在 Mac ARM 上，这将自动安装 MPS 加速的 PyTorch 版本，无需 CUDA。
> 

### 4. 安装 GroundingDINO 包

直接安装可能会因为 `setup.py` 中的自动安装脚本而出错。

```bash
pip install -e .

```

> 可能的错误：如果遇到 No module named 'pip' 或 No module named 'torch' 错误，这是由于 setup.py 中的自动安装 torch 功能导致的。
> 

**解决方案**：

1. **创建 `pyproject.toml` 文件**：
为了更好地管理构建依赖，可以创建一个 `pyproject.toml` 文件。
    
    ```bash
    echo '[build-system]
    requires = ["setuptools>=64", "wheel", "torch", "torchvision"]
    build-backend = "setuptools.build_meta"' > pyproject.toml
    
    ```
    
2. **修改 `setup.py` 文件**：
移除 `setup.py` 文件中自动安装 torch 的函数。
    
    ```bash
    sed -i '' '/def install_torch()/,/install_torch()/d' setup.py
    
    ```
    
3. **重新尝试安装**：
    
    ```bash
    pip install -e .
    
    ```
    

## 下载预训练模型

### 5. 创建权重目录并下载模型

```bash
mkdir -p weights
curl -L -o weights/groundingdino_swint_ogc.pth <https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha/groundingdino_swint_ogc.pth>

```

## 验证安装

### 6. 验证 Python 导入

```bash
python -c "import groundingdino; print('GroundingDINO import OK')"

```

> 可能的错误：如果导入失败，检查是否有 C++ 扩展编译错误。
> 

### 7. 运行演示推理

```bash
mkdir -p logs/test_output
python demo/inference_on_a_image.py --cpu-only -c groundingdino/config/GroundingDINO_SwinT_OGC.py -p weights/groundingdino_swint_ogc.pth -i .asset/cat_dog.jpeg -o logs/test_output -t "cat . dog ."

```

> 注意：
> 
> - `-cpu-only` 参数在 Mac 上很重要，确保使用 CPU 而不是尝试使用 CUDA。
> - 如果想使用 Mac 的 MPS 加速，可能需要修改源代码以支持 MPS 设备，因为某些操作可能不被原生支持。

## 使用指南

### 自定义图像推理

```bash
python demo/inference_on_a_image.py --cpu-only -c groundingdino/config/GroundingDINO_SwinT_OGC.py -p weights/groundingdino_swint_ogc.pth -i <your_image.jpg> -o <output_dir> -t "object ."

```

### 启动 Gradio Web UI 演示

```bash
python demo/gradio_app.py --cpu-only -c groundingdino/config/GroundingDINO_SwinT_OGC.py -p weights/groundingdino_swint_ogc.pth

```

### Python API 使用

查看 `README` 或 `groundingdino/util/inference.py` 了解如何使用 `load_model`, `predict` 等函数。

## 常见问题解决

1. **安装 C++ 扩展失败**：
    - 确保已安装 Xcode 命令行工具：`xcode-select --install`
    - 确保 PyTorch 和 torchvision 已正确安装。
2. **内存错误**：
    - 减小批处理大小或输入图像尺寸。
    - 使用较小的模型变体。
3. **推理速度慢**：
    - 在 Mac 上，CPU 模式预期会比 GPU 慢。
    - 考虑使用较小的输入图像尺寸。
4. **找不到模型文件**：
    - 确保模型文件路径正确，使用绝对路径可能更可靠。
5. **Python 版本兼容性问题**：
    - 如果 Python 3.9 有问题，可以尝试 Python 3.10，但可能需要调整某些依赖项。