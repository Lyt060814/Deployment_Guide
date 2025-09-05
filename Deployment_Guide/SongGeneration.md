# SongGeneration

# SongGeneration 部署指南

本指南提供了在 macOS（特别是 Apple Silicon）上部署 SongGeneration 项目的详细步骤，包括可能遇到的错误及解决方案。

## 环境准备

### 1. 创建 Conda 环境

```bash
conda create -n songgeneration python=3.10 -y
conda activate songgeneration

```

### 2. 升级基础工具

```bash
pip install --upgrade pip
pip install --upgrade wheel

```

## 安装依赖

### 3. 安装 PyTorch 及相关库

```bash
pip install torch==2.6.0 torchaudio==2.6.0 torchvision==0.21.0 --index-url https://download.pytorch.org/whl/cpu

```

> 注意：对于 Apple Silicon，这将安装支持 MPS 加速的版本。
> 

### 4. 安装项目依赖

```bash
pip install -r requirements.txt --no-deps
pip install -r requirements_nodeps.txt --no-deps

```

## 下载模型和资源

### 5. 从 Hugging Face 下载必要文件

```bash
pip install huggingface_hub
huggingface-cli repo clone waytan22/SongGeneration ./hf_tmp
cp -r ./hf_tmp/ckpt ./
cp -r ./hf_tmp/third_party ./
rm -rf ./hf_tmp

```

> 错误处理：如果 huggingface-cli 命令不可用，可以尝试：
> 
> 
> ```bash
> python -m huggingface_hub download waytan22/SongGeneration --local-dir ./hf_tmp
> 
> ```
> 
> 或者直接从网页界面下载并解压。
> 

## 验证安装

### 6. 验证核心依赖

```bash
python -c "import torch; import torchaudio; import transformers; print('Core dependencies imported successfully')"

```

## 解决 third_party.demucs 模块问题

### 7. 安装 demucs 并修复路径问题

```bash
pip install demucs

```

```bash
# 复制 demucs 到 third_party 目录
cp -r "$(python -c 'import site; print(site.getsitepackages()[0])')/demucs" "third_party/demucs"

```

```bash
# 修改 generate.py 中的导入路径
sed -i '' 's/from third_party.demucs.models.pretrained import get_model_from_yaml/from third_party.demucs.pretrained import get_model_from_yaml/' 'generate.py'

```

> 错误说明：如果运行时出现 ModuleNotFoundError: No module named 'third_party.demucs' 错误，上述步骤是必要的。这是因为 demucs 库的结构与项目预期不匹配。
> 

## 运行测试

### 8. 测试生成功能

```bash
sh generate.sh ckpt/songgeneration_base sample/lyrics.jsonl sample/output --not_use_flash_attn

```

> 注意：在 Mac 上使用 --not_use_flash_attn 标志以禁用 Flash Attention，这在 Apple Silicon 上可能不受支持。
> 
> 
> **错误处理**：如果生成过程超时，可以移除超时限制或增加超时时间：
> 
> ```bash
> timeout 300s sh generate.sh ckpt/songgeneration_base sample/lyrics.jsonl sample/output --not_use_flash_attn
> 
> ```
> 

### 9. 验证输出

```bash
if [ -d sample/output/audio ] && [ -f sample/output/jsonl ]; then
  echo '✅ 推理成功完成。找到输出音频和 jsonl 文件。'
else
  echo '❌ 推理可能失败。请检查上面的日志。'
fi

```

## 使用示例

```bash
# 低内存模式生成
sh generate.sh ckpt/songgeneration_base sample/lyrics.jsonl sample/output --low_mem --not_use_flash_attn

# 仅生成人声
sh generate.sh ckpt/songgeneration_base sample/lyrics.jsonl sample/output --vocal --not_use_flash_attn

# 启动 Gradio UI
sh tools/gradio/run.sh ckpt/songgeneration_base

```

## 常见问题解决

1. **内存不足**：使用 `-low_mem` 标志减少内存使用。
2. **GPU 相关错误**：在 Mac 上，确保使用 `-not_use_flash_attn` 标志。
3. **模块导入错误**：如果遇到其他模块导入错误，检查 `third_party` 目录结构，可能需要类似上述 demucs 的修复方法。
4. **生成超时**：对于复杂或长歌词，生成过程可能需要更长时间，可以增加超时限制或完全移除。
5. **依赖冲突**：如果遇到依赖冲突，尝试使用 `-no-deps` 安装特定版本的依赖。

---

按照上述步骤，您应该能够成功部署并运行 SongGeneration 项目。如有其他问题，请参考项目的 README.md 文件或提交 issue。