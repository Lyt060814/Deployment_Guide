# PLM

# PLM 部署指南

本指南提供了在 macOS 系统上部署 PLM 模型的完整步骤，包括可能遇到的错误及解决方案。

## 1. 创建 Conda 环境

```bash
conda create -n plm python=3.10
conda activate plm

```

**可能的错误**:

- **错误**: `conda: command not found`
    - **解决方案**: 安装 Miniconda 或 Anaconda，并确保已添加到 PATH 中

## 2. 安装依赖库

```bash
pip install --upgrade pip
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install transformers==4.41.2
pip install huggingface-hub

```

**可能的错误**:

- **错误**: `ERROR: Could not find a version that satisfies the requirement transformers==4.41.2`
    - **解决方案**: 尝试安装最新版本 `pip install transformers`
- **错误**: PyTorch 安装失败
    - **解决方案**: 对于 Apple Silicon，可以尝试 `pip install torch torchvision torchaudio`（不带额外参数）

## 3. 下载模型

使用 Hugging Face Hub 下载模型：

```bash
# 创建模型目录
mkdir -p models

# 使用 huggingface-cli 下载模型
huggingface-cli download PLM-Team/PLM-1.8B-Instruct --local-dir models/PLM-1.8B-Instruct

```

**可能的错误**:

- **错误**: `huggingface-cli: command not found`
    - **解决方案**: 确认 `huggingface-hub` 已安装，或尝试 `python -m huggingface_hub download PLM-Team/PLM-1.8B-Instruct --local-dir models/PLM-1.8B-Instruct`
- **错误**: 下载中断或网络问题
    - **解决方案**: 重新运行命令，下载会从断点继续
- **错误**: 磁盘空间不足
    - **解决方案**: 确保有至少 5GB 的可用空间

## 4. 验证模型加载和推理

```bash
python -c "import torch; from transformers import AutoTokenizer, AutoModelForCausalLM; tokenizer = AutoTokenizer.from_pretrained('models/PLM-1.8B-Instruct'); model = AutoModelForCausalLM.from_pretrained('models/PLM-1.8B-Instruct', torch_dtype=torch.bfloat16); input_text = 'Tell me something about reinforcement learning.'; inputs = tokenizer(input_text, return_tensors='pt'); output = model.generate(inputs['input_ids'], max_new_tokens=32); print(tokenizer.decode(output[0], skip_special_tokens=True))"

```

**可能的错误**:

- **错误**: `AttributeError: module 'torch' has no attribute 'bfloat16'`
    - **解决方案**: 在较旧的 PyTorch 版本中，将 `torch.bfloat16` 替换为 `torch.float16`
- **错误**: `OSError: models/PLM-1.8B-Instruct/pytorch_model.bin not found`
    - **解决方案**: 确认模型下载完整，或重新下载模型
- **错误**: 内存不足
    - **解决方案**: 添加 `device_map="auto"` 参数到 `AutoModelForCausalLM.from_pretrained()` 调用中

## 5. 使用 MPS 加速（Apple Silicon）

如果您使用的是 Apple Silicon Mac，可以启用 MPS 加速：

```bash
python -c "import torch; from transformers import AutoTokenizer, AutoModelForCausalLM; tokenizer = AutoTokenizer.from_pretrained('models/PLM-1.8B-Instruct'); model = AutoModelForCausalLM.from_pretrained('models/PLM-1.8B-Instruct', torch_dtype=torch.float16, device_map='mps'); input_text = 'Tell me something about reinforcement learning.'; inputs = tokenizer(input_text, return_tensors='pt').to('mps'); output = model.generate(inputs['input_ids'], max_new_tokens=32); print(tokenizer.decode(output[0], skip_special_tokens=True))"

```

**可能的错误**:

- **错误**: `RuntimeError: MPS does not support...`
    - **解决方案**: 某些操作在 MPS 上不支持，尝试使用 `device_map="cpu"` 或更新 PyTorch 到最新版本
- **错误**: `torch.cuda.is_available()` 返回 False
    - **解决方案**: 在 Mac 上，应使用 `torch.backends.mps.is_available()` 检查 MPS 可用性

## 6. 高级：使用 llama.cpp（可选）

如果需要量化模型或更高性能，可以使用 llama.cpp：

```bash
# 克隆 llama.cpp 仓库
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp

# 编译（针对 Apple Silicon）
LLAMA_METAL=1 make

# 转换模型（需要先下载原始模型）
python convert.py ../models/PLM-1.8B-Instruct/ --outtype f16

# 量化模型（可选）
./quantize ../models/PLM-1.8B-Instruct/ggml-model-f16.bin ../models/PLM-1.8B-Instruct/ggml-model-q4_0.bin q4_0

# 运行推理
./main -m ../models/PLM-1.8B-Instruct/ggml-model-f16.bin -p "Tell me something about reinforcement learning."

```

**可能的错误**:

- **错误**: 编译失败
    - **解决方案**: 确保已安装 Xcode 命令行工具 (`xcode-select --install`)
- **错误**: 模型转换失败
    - **解决方案**: 确保 Python 环境中安装了 `torch` 和 `sentencepiece`
- **错误**: Metal 相关错误
    - **解决方案**: 尝试不使用 Metal 编译 `make` 而不是 `LLAMA_METAL=1 make`

## 注意事项

1. 对于生产环境，建议使用专用服务器或云实例
2. 模型占用约 3GB 磁盘空间，运行时内存使用约 4-6GB
3. 首次加载模型可能较慢，之后会更快
4. 对于更长的对话或更复杂的提示，可能需要调整 `max_new_tokens` 参数

现在您已成功部署 PLM 模型，可以在应用程序中集成或进行进一步的实验！