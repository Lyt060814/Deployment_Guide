# CatVTON

# CatVTON 项目部署指南

本指南提供了部署 CatVTON（基于扩散模型的虚拟试衣系统）的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

1. 创建并激活 Python 3.9 的 Conda 环境

```bash
conda create -n catvton python=3.9
conda activate catvton

```

**可能的错误**：

- 如果提示 `conda command not found`，请先安装 Miniconda 或 Anaconda
- 如果环境创建失败，检查是否有足够的磁盘空间或尝试更新 conda：`conda update -n base conda`

## 依赖安装

1. 更新 pip 并安装依赖

```bash
pip install --upgrade pip
pip install -r requirements.txt

```

**可能的错误**：

- 如果 `requirements.txt` 文件不存在，请确保您在项目根目录下
- 如果某些包安装失败，可能需要单独安装：
    
    ```bash
    pip install torch torchvision
    pip install git+https://github.com/huggingface/diffusers.git
    pip install gradio
    pip install -r requirements.txt --ignore-installed
    
    ```
    
- 对于 Apple Silicon Mac 用户，可能需要指定 PyTorch 版本：
    
    ```bash
    pip install torch torchvision --extra-index-url <https://download.pytorch.org/whl/cpu>
    
    ```
    

## 验证安装

1. 验证核心依赖是否正确安装

```bash
python -c "import torch; import diffusers; import gradio; print('Import successful')"

```

**可能的错误**：

- 如果导入失败，检查具体错误信息并重新安装相应的包
- 如果 CUDA 相关错误，对于 CPU 运行可忽略，或检查 CUDA 版本与 PyTorch 版本是否匹配

## 测试 Gradio 应用

1. 启动 Gradio 应用进行测试（2分钟后自动停止）

```bash
timeout 120 python app.py --output_dir="resource/demo/output" --mixed_precision="bf16" --allow_tf32 || echo 'Gradio test stopped after 2 minutes (expected for validation)'

```

**可能的错误**：

- 如果 `timeout` 命令不可用（Windows），可以直接运行 `python app.py` 并手动停止
- 如果出现 `-mixed_precision="bf16"` 相关错误，可以尝试使用 `-mixed_precision="no"` 或 `-mixed_precision="fp16"`
- 如果出现 `resource/demo/output` 目录不存在错误，请手动创建：
    
    ```bash
    mkdir -p resource/demo/output
    
    ```
    
- 首次运行时，模型会自动从 HuggingFace 下载，确保网络连接良好
- 如果下载模型时出现网络问题，可以尝试使用代理或手动下载模型并放置在正确位置

## 使用指南

1. 启动 Gradio 演示应用

```bash
python app.py --output_dir="resource/demo/output" --mixed_precision="bf16" --allow_tf32

```

**可能的错误**：

- 对于不支持 bf16 的设备，使用 `-mixed_precision="fp16"` 或 `-mixed_precision="no"`
- 如果 `-allow_tf32` 参数导致错误，可以移除此参数
1. 在 DressCode 或 VITON-HD 数据集上进行推理

```bash
python inference.py --dataset [dresscode|vitonhd] --data_root_path <path> --output_dir <path> --dataloader_num_workers 8 --batch_size 8 --seed 555 --mixed_precision [no|fp16|bf16] --allow_tf32 --repaint --eval_pair

```

**可能的错误**：

- 确保已下载并准备好数据集，按照 README 中的说明进行
- 对于内存较小的设备，可以减小 `-batch_size` 和 `-dataloader_num_workers` 的值
- 如果出现 CUDA 内存不足错误，减小 batch_size 或使用 CPU 模式
1. 计算推理后的评估指标

```bash
python eval.py --gt_folder <gt_image_folder> --pred_folder <predicted_image_folder> --paired --batch_size=16 --num_workers=16

```

**可能的错误**：

- 确保指定的文件夹路径正确且存在
- 对于内存较小的设备，可以减小 `-batch_size` 和 `-num_workers` 的值

## 重要提示

- 首次运行时，模型权重会自动从 HuggingFace 下载
- 对于推理，必须按照 README 中的说明下载并准备 DressCode 或 VITON-HD 数据集
- 如果您的设备没有强大的 GPU，推理过程可能会很慢，请耐心等待
- 对于 Apple Silicon Mac 用户，PyTorch 会自动使用 Metal 性能着色器进行加速，无需额外配置

通过以上步骤，您应该能够成功部署并运行 CatVTON 项目。如有其他问题，请参考项目的 GitHub 仓库或提交 issue。