# SwapNet

# SwapNet 部署指南

本指南提供了在 macOS 系统上部署 SwapNet 项目的详细步骤，包括可能遇到的错误及解决方案。

## 1. 创建 Conda 环境

```bash
conda create -n swapnet python=3.7
conda activate swapnet

```

> 可能的错误:
> 
> - `EnvironmentNameNotFound: Could not find conda environment: swapnet`
>     - **解决方案**: 确认环境创建成功，或使用 `conda info --envs` 查看可用环境列表。

## 2. 安装依赖包

```bash
pip install adabound==0.0.5 chardet==3.0.4 cycler==0.10.0 cython==0.29.6 dominate==2.4.0 \\
    easydict==1.9 idna==2.8 kiwisolver==1.0.1 matplotlib==3.0.3 msgpack==0.6.1 \\
    pandas==0.24.2 protobuf==3.7.1 pyparsing==2.3.1 pytz==2019.1 pyyaml==5.1 \\
    requests==2.22.0 scipy==1.2.1 tensorboardx==1.6 torchfile==0.1.0 tqdm==4.31.1 \\
    urllib3==1.25.3 visdom==0.1.8.8 websocket-client==0.56.0 seaborn

```

> 可能的错误:
> 
> - `No matching distribution found for opencv-python==4.0.0.21`
>     - **解决方案**: 使用较新版本 `pip install opencv-python==4.3.0.38`。
> - **安装 pandas 失败**
>     - **解决方案**: `pip install --no-build-isolation pandas==1.1.5`。
> - **其他包安装失败**
>     - **解决方案**: 安装兼容的版本 `pip install numpy==1.19.5 seaborn==0.11.1`。

## 3. 安装 PyTorch 和 torchvision

```bash
pip install torch==1.9.0 torchvision==0.10.0 -f <https://download.pytorch.org/whl/torch_stable.html>

```

> 可能的错误:
> 
> - `No matching distribution found for torch==1.9.0+cpu`
>     - **解决方案**: 移除 `+cpu` 后缀，直接安装 `torch==1.9.0 torchvision==0.10.0`。

## 4. 克隆并构建 ROI 依赖

```bash
git clone <https://github.com/jwyang/faster-rcnn.pytorch.git>
cd faster-rcnn.pytorch
git checkout pytorch-1.0
pip install -r requirements.txt

```

> 可能的错误:
> 
> - `fatal: destination path 'faster-rcnn.pytorch' already exists and is not an empty directory`
>     - **解决方案**: 如果目录已存在，直接进入目录继续后续步骤。
> - `bash: cd: faster-rcnn.pytorch: No such file or directory`
>     - **解决方案**: 确认当前工作目录，使用绝对路径 `cd /path/to/faster-rcnn.pytorch`。

## 5. 构建 pycocotools

```bash
cd faster-rcnn.pytorch/lib/pycocotools
curl -O <https://raw.githubusercontent.com/muaz-urwa/temp_files/master/setup.py>
python setup.py build_ext --inplace

```

> 可能的错误:
> 
> - `bash: cd: faster-rcnn.pytorch/lib/pycocotools: No such file or directory`
>     - **解决方案**: 确认目录结构，可能需要先创建目录 `mkdir -p lib/pycocotools`。

## 6. 构建 ROI 库

```bash
cd faster-rcnn.pytorch/lib
python setup.py build develop

```

> 可能的错误:
> 
> - **构建失败**
>     - **解决方案**: 检查编译错误信息，可能需要安装额外的编译工具或依赖。

## 7. 链接 ROI 库到项目

```bash
cd /path/to/SwapNet
ln -s $(pwd)/faster-rcnn.pytorch/lib $(pwd)/lib

```

> 可能的错误:
> 
> - `ln: ... No such file or directory`
>     - **解决方案**:
>         
>         ```bash
>         rm -rf "$(pwd)/lib"
>         ln -s "$(pwd)/faster-rcnn.pytorch/lib" "$(pwd)/lib"
>         
>         ```
>         
> - **如果链接后仍无法导入 ROI 库**
>     - **解决方案** (复制文件而不是链接):
>         
>         ```bash
>         git clone <https://github.com/jwyang/faster-rcnn.pytorch.git> ../faster-rcnn.pytorch-official
>         rm -rf ./lib
>         cp -r ../faster-rcnn.pytorch-official/lib ./lib
>         cd lib && python setup.py build develop
>         
>         ```
>         

## 8. 验证安装

```bash
python -c "import torch; import torchvision; import visdom; print('PyTorch:', torch.__version__, '| CUDA:', torch.cuda.is_available()); import sys; sys.path.append('./lib'); print('ROI lib import:', end=' '); import roi_data_layer; print('OK')"

```

> 可能的错误:
> 
> - `ModuleNotFoundError: No module named 'roi_data_layer'`
>     - **解决方案**: 检查 `lib` 目录是否正确链接或复制，可能需要重新构建 ROI 库或调整 Python 路径。

## 9. 使用说明

### 训练 warp 阶段

```bash
python train.py --name deep_fashion/warp --model warp --dataroot data/deep_fashion

```

### 训练 texture 阶段

```bash
python train.py --name deep_fashion/texture --model texture --dataroot data/deep_fashion

```

### 运行推理 (下载检查点后)

```bash
python inference.py --checkpoint checkpoints/deep_fashion --dataroot data/deep_fashion --shuffle_data True

```

### 启动 Visdom 服务器可视化进度

```bash
python -m visdom.server

```

然后在浏览器中访问 [http://localhost:8097](http://localhost:8097/)

## 注意事项

- 此设置在 CPU 上运行，训练/推理会很慢。要获得完整的 GPU 支持，请使用支持 CUDA 的 Linux 系统。
- 参考 README 获取数据集下载/预处理步骤。
- 在 macOS 上，特别是 Apple Silicon 芯片上，某些 CUDA 相关功能可能无法正常工作。