# So-Vits-SVC

# So-Vits-SVC 部署指南

本指南提供了在 Apple Silicon Mac 上部署 So-Vits-SVC（歌声转换系统）的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n sovits-svc python=3.8 -y
conda activate sovits-svc

```

> 注意：Python 3.8.9 是推荐版本，但任何 3.8.x 版本都可以工作。
> 

### 2. 安装系统依赖

```bash
brew install ffmpeg

```

> 说明：ffmpeg 是处理音频所必需的，用于重采样和音频处理。
> 

## 安装 Python 依赖

### 3. 安装 PyTorch 和基础依赖

```bash
pip install --upgrade pip
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cpu

```

> 错误处理：如果遇到 pip 版本问题，可以尝试降级 pip：
> 
> 
> ```bash
> pip install pip==24.0
> 
> ```
> 

### 4. 安装项目依赖

```bash
pip install -r requirements.txt

```

> 错误处理：如果遇到 omegaconf 和 hydra-core 的依赖冲突，可以尝试以下解决方案：
> 
> 1. 先安装特定版本的 omegaconf：
>     
>     ```bash
>     pip install omegaconf==2.0.0
>     
>     ```
>     
> 2. 然后安装 hydra-core：
>     
>     ```bash
>     pip install hydra-core==1.0.7
>     
>     ```
>     
> 3. 最后安装 fairseq：
>     
>     ```bash
>     pip install fairseq==0.12.2
>     
>     ```
>     

## 创建必要目录

### 5. 创建模型和日志目录

```bash
mkdir -p pretrain
mkdir -p pretrain/nsf_hifigan
mkdir -p logs/44k
mkdir -p logs/44k/diffusion

```

> 说明：这些目录用于存储预训练模型、声码器和训练日志。
> 

## 下载预训练模型

### 6. 下载 ContentVec 编码器模型

```bash
wget -P pretrain/ https://huggingface.co/lj1995/VoiceConversionWebUI/resolve/main/hubert_base.pt -O pretrain/checkpoint_best_legacy_500.pt

```

> 错误处理：如果文件不存在，请确保下载命令正确执行，并检查网络连接。
> 

### 7. 下载 NSF HiFiGAN 声码器

```bash
wget -O pretrain/nsf_hifigan_20221211.zip 'https://github.com/openvpi/vocoders/releases/download/nsf-hifigan-v1/nsf_hifigan_20221211.zip'
unzip -od pretrain/nsf_hifigan pretrain/nsf_hifigan_20221211.zip

```

> 说明：这个声码器可以提高生成音频的质量。
> 

## 验证安装

### 8. 验证依赖安装

```bash
python -c "import torch; import fairseq; import librosa; import soundfile; print('Core dependencies import successful')"
python -c "import sys; print('Python version:', sys.version)"
python -c "import torchcrepe; print('torchcrepe import successful')"
python -c "import pyworld; print('pyworld import successful')"

```

### 9. 检查模型文件

```bash
ls -lh pretrain/checkpoint_best_legacy_500.pt

```

> 错误处理：如果文件不存在，请重新执行下载命令：
> 
> 
> ```bash
> wget -P pretrain/ https://huggingface.co/lj1995/VoiceConversionWebUI/resolve/main/hubert_base.pt -O pretrain/checkpoint_best_legacy_500.pt
> 
> ```
> 

## 使用指南

### 10. 音频预处理

```bash
python resample.py --skip_loudnorm

```

> 说明：重采样音频文件，准备训练数据。
> 

### 11. 生成数据集配置

```bash
python preprocess_flist_config.py --speech_encoder vec768l12

```

> 说明：分割数据集并生成配置文件。
> 

### 12. 特征提取

```bash
python preprocess_hubert_f0.py --f0_predictor dio

```

> 说明：提取音频特征，包括基频（F0）和内容特征。
> 

### 13. 模型训练

```bash
python train.py -c configs/config.json -m 44k

```

> 说明：开始训练模型，使用配置文件和 44kHz 采样率。
> 

### 14. 推理（声音转换）

```bash
python inference_main.py -m logs/44k/G_0.pth -c configs/config.json -n <wavfile> -t 0 -s <spkname>

```

> 说明：使用训练好的模型进行声音转换，将源音频转换为目标说话人的声音。
> 

## 故障排除

1. **依赖冲突**：如果遇到 `omegaconf` 和其他包的依赖冲突，尝试降级 pip 或单独安装冲突的包。
2. **模型下载失败**：检查网络连接，或尝试从备用源下载模型文件。
3. **CUDA 错误**：在 Apple Silicon 上，确保使用 CPU 版本的 PyTorch，而不是 CUDA 版本。
4. **音频处理错误**：确保 ffmpeg 正确安装，并且可以从命令行访问。
5. **内存不足**：减小批处理大小或使用较小的模型配置。

通过按照上述步骤操作，您应该能够成功部署 So-Vits-SVC 并开始进行声音转换实验。