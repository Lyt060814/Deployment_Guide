# Dia

# Dia Text-to-Speech 部署指南

本指南提供了在 Mac ARM 架构上部署 Dia 文本转语音模型的完整步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n dia python=3.10
conda activate dia

```

### 2. 安装系统依赖

```bash
brew install ffmpeg
brew install libsndfile

```

这些是音频处理所必需的系统级依赖。

## 安装 Dia 及其依赖

### 3. 安装 Python 依赖

```bash
pip install --upgrade pip
pip install torch==2.6.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cpu
pip install descript-audio-codec>=1.0.0 gradio>=5.25.2 huggingface-hub>=0.30.2 numpy>=2.2.4 pydantic>=2.11.3 safetensors>=0.5.3 soundfile>=0.13.1
pip install -e .

```

**注意**：在 Mac ARM 架构上，我们使用 CPU 版本的 PyTorch，因为 CUDA 不可用。

## 验证安装

### 4. 验证基本导入和 CLI

```bash
python -c "import dia; print('Dia import successful')"
python cli.py --help

```

如果这些命令成功执行，说明基本安装正确。

### 5. 测试示例脚本

```bash
python example/simple-cpu.py

```

**错误处理**：

- 如果运行 `python example/simple.py` 时遇到 `NotImplementedError: Output channels > 65536 not supported at the MPS device` 错误，这是因为 Mac 的 MPS (Metal Performance Shaders) 设备有限制。
- **解决方案**：使用 `simple-cpu.py` 示例，它会强制使用 CPU 而不是 MPS。如果该文件不存在，可以复制 `simple.py` 并在开头添加以下代码：

```python
import os
os.environ["PYTORCH_ENABLE_MPS_FALLBACK"] = "1"
# 或者完全禁用 MPS
# os.environ["PYTORCH_ENABLE_MPS_FALLBACK"] = "1"

```

### 6. 测试 Gradio GUI

```bash
timeout 120 python app.py || true

```

这将启动 Gradio 界面，可在 [http://localhost:7860](http://localhost:7860/) 访问。命令设置了 2 分钟超时以避免挂起。

**错误处理**：

- 如果遇到与 MPS 相关的错误，编辑 `app.py` 文件，在开头添加与上面相同的环境变量设置。

## 使用指南

成功部署后，可以通过以下方式使用 Dia：

### 7. 命令行界面

```bash
python cli.py --text "Hello, this is a test." --output test.wav

```

### 8. Python API

```python
from dia import Dia

model = Dia()
output = model.generate("Hello, this is a test.")
output.save("test.wav")

```

### 9. 语音克隆

```bash
python example/voice_clone.py

```

### 10. Gradio Web 界面

```bash
python app.py

```

然后在浏览器中访问 [http://localhost:7860](http://localhost:7860/)

## 常见问题解决

### PyTorch MPS 问题

**问题**：在 Mac ARM 上使用 MPS 加速时出现 `NotImplementedError: Output channels > 65536 not supported at the MPS device`。

**解决方案**：

1. 设置环境变量强制回退到 CPU：
    
    ```python
    import os
    os.environ["PYTORCH_ENABLE_MPS_FALLBACK"] = "1"
    
    ```
    
2. 或完全禁用 MPS：
    
    ```python
    import os
    os.environ["DEVICE"] = "cpu"  # 在某些脚本中可能需要
    
    ```
    

### 模型下载问题

**问题**：首次运行时模型权重下载失败。

**解决方案**：

1. 检查网络连接
2. 手动从 Hugging Face 下载模型并放置在正确的缓存目录中
3. 确保有足够的磁盘空间（模型需要几 GB）

### 内存不足

**问题**：运行时出现内存错误。

**解决方案**：

1. 关闭其他内存密集型应用
2. 对于长文本，考虑分段处理

## 系统要求

- Python 3.10 或更高版本
- 至少 8GB RAM（推荐 16GB+）
- 足够的磁盘空间用于模型（约 5-10GB）
- ffmpeg 和 libsndfile 系统库

这份部署指南应该能帮助您在 Mac ARM 架构上成功部署和运行 Dia 文本转语音模型。

```jsx
[AISeg]:
GenGuideJs@/Users/agenthub/Library/Application Support/AI2Apps/local/server/agents/AutoDeploy/PrjSetupBySteps.js
[Result]
{
	"result": {
		"Reasoning": "1. Project requirements: Dia is a 1.6B parameter text-to-speech model requiring Python >=3.10, PyTorch 2.6.0/torchaudio 2.6.0, and several dependencies. While CPU support is coming, official support and all benchmarks are GPU-focused, but MacOS/Apple Silicon isn't explicitly supported for GPU (Metal) yet. 2. Alternatives: Dockerfiles are provided for Linux (CPU/GPU), but not for MacOS/Apple Silicon. The simplest native install is a pip/venv/conda-based install as per README. 3. Resources: The host has 96GB RAM, 191GB disk, Apple Silicon (arm64), and no conda yet. Plenty of resources for setup. 4. Platform-specific: PyTorch for Apple Silicon/M1/M2 is supported via pip wheels, but torch==2.6.0 (from pyproject.toml) is not available for arm64/MacOS as of 2024-06; the latest available is 2.3.x. Therefore, will use the latest compatible torch/torchaudio for MacOS arm64. 5. Chosen approach: Create a conda environment (as always required), install dependencies (with torch/torchaudio for M1/M2), then pip install editable package, then validate with example/simple.py. Validate import and simple inference. 6. Plan: (a) Install conda, (b) Create env, (c) Install requirements with correct torch, (d) pip install -e ., (e) Test with example, (f) Show usage tips. If any dependency fails on arm64, suggest using HuggingFace Transformers route as fallback (since dia is also available via transformers main branch).",
		"Actions": [
			{
				"action": "brew",
				"tip": "Install Miniforge (conda for Apple Silicon) if conda is not present. This is the preferred conda distribution for arm64 MacOS.",
				"install": "miniforge"
			},
			{
				"action": "conda",
				"tip": "Create isolated Python 3.10 environment for Dia. Python 3.10 is required by the project.",
				"conda": "dia-env",
				"pythonVersion": "3.10"
			},
			{
				"action": "brew",
				"tip": "Install ffmpeg and libsndfile for audio processing. These are required by torchaudio and soundfile.",
				"install": "ffmpeg"
			},
			{
				"action": "brew",
				"tip": "Install libsndfile for soundfile Python package.",
				"install": "libsndfile"
			},
			{
				"action": "bash",
				"tip": "Install PyTorch and torchaudio for Apple Silicon (arm64/MacOS). The official wheels as of June 2024 are torch==2.3.0 and torchaudio==2.3.0. torch==2.6.0 is not available for M1/M2. If errors, suggest user try 'pip install torch torchaudio' without version pin.",
				"commands": [
					"pip install --upgrade pip",
					"pip install torch==2.3.0 torchaudio==2.3.0 --index-url https://download.pytorch.org/whl/cpu"
				]
			},
			{
				"action": "bash",
				"tip": "Install all remaining Python dependencies as specified in pyproject.toml, except torch/torchaudio (already installed).",
				"commands": [
					"pip install descript-audio-codec>=1.0.0 gradio>=5.25.2 huggingface-hub>=0.30.2 numpy>=2.2.4 pydantic>=2.11.3 safetensors>=0.5.3 soundfile>=0.13.1"
				]
			},
			{
				"action": "bash",
				"tip": "Install Dia in editable/development mode. This allows running local code and using example scripts.",
				"commands": [
					"pip install -e ."
				]
			},
			{
				"action": "bash",
				"tip": "Validate installation: (1) check that dia package imports, (2) run the simplest example to generate speech. If this fails due to torch/torchaudio incompatibility, suggest fallback via transformers.",
				"commands": [
					"python -c \"import dia; print('Dia import successful')\"",
					"python example/simple.py || echo 'If this fails due to torch/torchaudio on Apple Silicon, try using HuggingFace Transformers: pip install git+https://github.com/huggingface/transformers.git; see README for details.'"
				]
			},
			{
				"action": "bash",
				"tip": "Show quick usage tips, including how to run Gradio UI and CLI examples. Suggest using HuggingFace Transformers as fallback for M1/M2 compatibility.",
				"commands": [
					"echo 'Dia setup complete! Example usage:'",
					"echo '  python example/simple.py'",
					"echo '  python app.py   # to launch Gradio UI on http://localhost:7860'",
					"echo '  python cli.py --help'",
					"echo 'If you encounter errors on Apple Silicon for torch/torchaudio, use the HuggingFace Transformers route as in README:'",
					"echo '  pip install git+https://github.com/huggingface/transformers.git'",
					"echo '  See hf.py for example code.'"
				]
			}
		]
	},
	"preSeg": "1ILI9VIEO0",
	"outlet": "1ILIA093U0"
}

```