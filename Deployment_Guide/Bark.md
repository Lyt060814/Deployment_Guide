# Bark

# Bark 文本转语音模型部署指南

本指南提供了在 Apple Silicon Mac (arm64) 上部署 Bark 文本转语音模型的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n bark-env python=3.10
conda activate bark-env

```

### 2. 安装音频处理依赖

```bash
brew install ffmpeg

```

> 如果已安装，此命令不会重复安装。成功后会显示 ffmpeg version ...。
> 

### 3. 安装基础依赖

```bash
python -m pip install --upgrade pip
pip install torch torchvision torchaudio --index-url <https://download.pytorch.org/whl/cpu>
pip install setuptools wheel

```

> 注意：对于 Apple Silicon Mac，这会安装原生支持的 PyTorch 版本。
> 

## 安装 Bark

### 4. 安装 Bark 及其依赖

```bash
pip install -e .

```

> 这会以可编辑模式安装 Bark，允许修改源代码。成功后会显示 Successfully installed bark ...。
> 

### 5. 安装可选依赖

```bash
pip install jupyter scipy

```

> 这些依赖对于开发和使用 Jupyter 笔记本很有用。
> 

## 模型预加载与验证

### 6. 预加载模型

```bash
python -c "from bark import preload_models; preload_models(); print('Model preloading ok')"

```

> 可能的错误：PyTorch 2.6+ 中的 weights_only 参数默认值变更导致的 UnpicklingError。
> 
> 
> **解决方案**：修改 `generation.py` 文件，将 `torch.load` 调用更改为显式设置 `weights_only=False`：
> 
> ```bash
> sed -i '' 's/torch.load(ckpt_path, map_location=device)/torch.load(ckpt_path, map_location=device, weights_only=False)/' '/path/to/bark/bark/generation.py'
> 
> ```
> 
> 替换 `/path/to/bark` 为实际的 Bark 安装路径。修改后重新运行预加载命令。
> 

### 7. 测试命令行接口

```bash
timeout 120 python -m bark --text "Hello, my name is Suno." --output_filename "example.wav"

```

> 这将生成一个示例音频文件。如果在 120 秒内未完成，命令将超时。
> 

### 8. 验证音频输出

```bash
ls -lh example.wav

```

> 确认生成的音频文件存在且非空。
> 

## 使用指南

### Python 代码示例

```python
from bark import SAMPLE_RATE, generate_audio, preload_models
preload_models()
audio = generate_audio("Hello, this is Bark!")
from scipy.io.wavfile import write as write_wav
write_wav("bark_generation.wav", SAMPLE_RATE, audio)

```

### 命令行使用

```bash
python -m bark --text "Hello, world!" --output_filename output.wav

```

### 低内存模式（适用于 MacBook 或仅 CPU 环境）

在导入 Bark 之前设置这些环境变量，可以减少内存使用。

```python
import os
os.environ["SUNO_OFFLOAD_CPU"] = "True"
os.environ["SUNO_USE_SMALL_MODELS"] = "True"

from bark import generate_audio
# ...

```

## 注意事项

1. **不要使用** `pip install bark`，这会安装一个不相关的包。
2. 在 Apple Silicon Mac 上，会自动使用原生 PyTorch 支持。
3. 首次运行时，模型会自动下载，这可能需要一些时间。
4. 如果没有 GPU，推理过程会比较慢。
5. 更多信息请参考项目的 `README.md` 和 GitHub 仓库：[https://github.com/suno-ai/bark](https://github.com/suno-ai/bark)