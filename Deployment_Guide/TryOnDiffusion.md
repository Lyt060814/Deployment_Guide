# TryOnDiffusion

# TryOnDiffusion 部署指南

本指南提供了在 macOS 环境下部署 TryOnDiffusion 项目的完整步骤，包括可能遇到的错误及解决方案。

## 1. 创建 Conda 环境

```bash
conda create -n tryondiffusion python=3.10
conda activate tryondiffusion

```

## 2. 安装基础依赖

```bash
python -m pip install --upgrade pip

```

## 3. 安装项目依赖

### 3.1 安装 xformers 依赖

在 macOS 上安装 `xformers` 可能会遇到 OpenMP 相关错误：

```
clang: error: unsupported option '-fopenmp'

```

> 解决方案：安装 libomp 并使用 gcc 编译器进行安装。
> 
> 
> ```bash
> # 安装 OpenMP 支持
> brew install libomp
> 
> # 使用 gcc 编译器安装 xformers
> CC=/opt/homebrew/opt/gcc/bin/gcc-15 CXX=/opt/homebrew/opt/gcc/bin/g++-15 CXXFLAGS="-I/usr/local/include" LDFLAGS="-L/usr/local/lib -lomp" pip install xformers
> 
> ```
> 

### 3.2 安装其他依赖

```bash
# 安装除 xformers 外的其他依赖
pip install --no-deps -r requirements.txt --extra-index-url <https://pypi.tuna.tsinghua.edu.cn/simple>

# 安装缺失的 Pillow 库
pip install Pillow

# 安装 TensorBoard 相关依赖
pip install absl-py grpcio markdown packaging protobuf 'tensorboard-data-server<0.8.0,>=0.7.0' werkzeug

# 安装 Hugging Face 相关依赖
pip install huggingface_hub psutil pyyaml safetensors

# 安装其他可能缺失的依赖
pip install wcwidth Jinja2 opencv-python

```

## 4. 安装项目

```bash
pip install -e .

```

## 5. 验证安装

```bash
python -c "import torch, torchvision, accelerate, xformers, ftfy, tensorboard, Jinja2, scipy, opencv_python, sentencepiece, ema_pytorch, einops, h5py, chardet; print('All major imports OK')"

```

如果遇到 `ModuleNotFoundError: No module named 'tryondiffusion'` 错误，可能是因为项目未正确安装或未添加到 Python 路径。

> 解决方案：
> 
> 
> ```bash
> # 确保在项目根目录下运行
> # cd /path/to/tryondiffusion
> 
> # 将当前目录添加到 Python 路径
> PYTHONPATH=. python -c "import tryondiffusion; print('tryondiffusion imported successfully')"
> 
> ```
> 

## 6. 运行示例

```bash
# 运行示例脚本，设置超时以避免无限等待
PYTHONPATH=. timeout 120 python ./examples/test_tryon_imagen_trainer.py || echo 'Example ran or timed out (this is expected for long training loops)'

```

## 7. 常见问题及解决方案

### 7.1 找不到 PIL 模块

> 错误：ModuleNotFoundError: No module named 'PIL'
> 
> 
> **解决方案**：
> 
> ```bash
> pip install Pillow
> 
> ```
> 

### 7.2 找不到 tryondiffusion 模块

> 错误：ModuleNotFoundError: No module named 'tryondiffusion'
> 
> 
> **解决方案**：
> 
> ```bash
> # 确保在项目根目录下
> # cd /path/to/tryondiffusion
> 
> # 方法1: 设置 PYTHONPATH
> PYTHONPATH=. python your_script.py
> 
> # 方法2: 重新安装项目
> pip install -e .
> 
> ```
> 

### 7.3 Shell 语法错误

> 错误：bash: syntax error near unexpected token '('
> 
> 
> **解决方案**：
> 
> ```bash
> # 使用正确的引号格式
> echo "Setup complete! To start developing or experimenting:"
> 
> ```
> 

## 8. 使用指南

成功部署后，您可以：

1. 查看 `./examples/` 目录获取使用示例。
2. **训练模型**：编辑并运行示例脚本或创建自己的脚本，使用 `tryondiffusion.TryOnImagenTrainer`。
3. **生成样本**：使用 `tryondiffusion.TryOnImagen` 如 README 所示。
4. 对于 Apple Silicon GPU 加速，torch 将自动使用 Apple Metal 后端。
5. 如需故障排除或更多选项，请阅读 `README.md` 和 `./examples/` 中的注释。

## 9. 完整部署流程总结

1. 创建并激活 conda 环境。
2. 安装 OpenMP 支持。
3. 使用特定编译器安装 `xformers`。
4. 安装其他项目依赖。
5. 安装项目本身 (`pip install -e .`)。
6. 验证安装。
7. 运行示例脚本。

按照上述步骤操作，您应该能够成功部署 TryOnDiffusion 项目。