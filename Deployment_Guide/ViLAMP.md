# ViLAMP

# ViLAMP 项目部署指南

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n vilamp python=3.8 -y
conda activate vilamp

```

> 注意：强烈建议使用 Python 3.8 版本，以确保最大兼容性。
> 

### 2. 安装系统依赖

```bash
# 安装视频处理所需的系统库
brew install ffmpeg
brew install libavif

```

> 可能的问题：如果 brew 安装过程中出现权限问题，可以尝试：
> 
> 
> ```bash
> sudo chown -R $(whoami) $(brew --prefix)/*
> 
> ```
> 

## 安装 Python 依赖

### 3. 更新 pip 并安装依赖

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt

```

> 常见问题：
> 
> - **decord 安装失败**：
>     
>     ```
>     ERROR: No matching distribution found for decord
>     
>     ```
>     
>     **解决方案**：从 requirements.txt 中移除 decord，然后重新安装其他依赖
>     
>     ```bash
>     sed -i '' '/^decord/d' requirements.txt
>     pip install -r requirements.txt
>     
>     ```
>     
> - **av 安装失败**：
>     
>     ```
>     ERROR: Failed building wheel for av
>     
>     ```
>     
>     **解决方案**：确保已安装 ffmpeg 和 libavif，然后尝试单独安装：
>     
>     ```bash
>     pip install av --no-binary :all:
>     
>     ```
>     
> - **deepspeed 安装问题**：
>     
>     如果遇到 deepspeed 安装错误，可以尝试：
>     
>     ```bash
>     TORCH_CUDA_ARCH_LIST="8.6" pip install deepspeed
>     
>     ```
>     

## 下载模型

### 4. 下载预训练模型

```bash
# 创建模型目录
mkdir -p models

# 下载模型（可能需要较长时间，取决于网络速度）
git lfs install
git clone https://huggingface.co/orange-sk/ViLAMP-llava-qwen models/ViLAMP-llava-qwen

```

> 可能的问题：
> 
> - 如果没有安装 git-lfs：
>     
>     ```bash
>     brew install git-lfs
>     
>     ```
>     
> - 如果下载速度过慢，可以考虑使用 Hugging Face CLI：
>     
>     ```bash
>     pip install huggingface_hub
>     huggingface-cli download orange-sk/ViLAMP-llava-qwen --local-dir models/ViLAMP-llava-qwen
>     
>     ```
>     

## 验证安装

### 5. 验证核心依赖

```bash
python -c "import transformers, av, accelerate, peft, diffusers; print('Core dependencies import successful.')"

```

> 注意：如果 decord 已从依赖中移除，上述命令中不要包含 decord。
> 

### 6. 验证推理脚本

```bash
timeout 120s python exp_vMME.py --help || true

```

## 使用指南

### 7. 推理示例

```bash
python exp_vMME.py \
  --dataset_path dataset/Video-MME/videomme/test-00000-of-00001.parquet \
  --video_dir dataset/Video-MME/data \
  --output_dir dataset/Video-MME/output \
  --version models/ViLAMP-llava-qwen \
  --split 1_1 \
  --max_frame_num 600

```

### 8. 训练指南

请参考项目中的 `scripts/train/README.md` 和 `README.md` 获取有关数据集格式和配置的详细信息。

## 故障排除

### 内存问题

如果在处理大型视频时遇到内存错误，可以尝试：

- 减少 `max_frame_num` 参数值
- 增加系统交换空间
- 使用更小的批处理大小

### GPU 相关问题

如果遇到 CUDA 相关错误：

- 确保 CUDA 驱动程序与 PyTorch 版本兼容
- 尝试使用 `CUDA_VISIBLE_DEVICES` 环境变量指定特定 GPU
- 对于 Apple Silicon，可能需要使用 MPS 后端而非 CUDA

### 视频处理问题

如果视频处理失败：

- 确保 ffmpeg 安装正确且在 PATH 中
- 检查视频格式是否受支持
- 尝试预先转换视频为更兼容的格式（如 MP4 使用 H.264 编码）