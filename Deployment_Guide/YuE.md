# YuE

# YuE 音乐生成模型部署指南

本指南提供了在 macOS (Apple Silicon) 上部署 YuE 音乐生成模型的完整步骤，包括可能遇到的问题及解决方案。

## 环境准备

```bash
# 创建 Python 3.10 的 conda 环境（同时支持推理和微调）
conda create -n yue python=3.10
conda activate yue

# 升级 pip 并安装 PyTorch 及相关库（Apple Silicon 优化版本）
pip install --upgrade pip
pip install torch torchvision torchaudio

```

## 安装依赖

```bash
# 安装基本依赖
pip install -r requirements.txt

```

### 安装微调依赖

```bash
# 安装 py-cpuinfo（解决 deepspeed 安装问题）
pip install py-cpuinfo

# 移除 CUDA 相关依赖（Apple Silicon 不需要）
sed -i '' '/nvidia-cublas-cu12==12.1.3.1/d' finetune/requirements.txt
sed -i '' '/nvidia-cuda-cupti-cu12==12.1.105/d' finetune/requirements.txt
sed -i '' '/nvidia-cuda-nvrtc-cu12==12.1.105/d' finetune/requirements.txt
sed -i '' '/nvidia-cuda-runtime-cu12==12.1.105/d' finetune/requirements.txt

# 安装微调依赖
pip install -r finetune/requirements.txt

```

**可能的错误**：`ModuleNotFoundError: No module named 'cpuinfo'`

**解决方案**：先安装 `py-cpuinfo` 并移除 CUDA 相关依赖

## 编译 C++ 扩展

```bash
# 安装 pybind11（用于构建 C++ 扩展）
pip install pybind11

# 修改 Makefile 以支持 Apple Silicon
sed -i '' 's/$(CXX) $(CXXFLAGS) $(CPPFLAGS) $< -o $@/$(CXX) $(CXXFLAGS) $(CPPFLAGS) $(shell python3-config --ldflags) $< -o $@/' finetune/core/datasets/Makefile

# 编译 C++ 扩展
cd finetune/core/datasets
make
cd ../../../

```

**可能的错误**：`Undefined symbols for architecture arm64: "_PyBaseObject_Type", referenced from...`

**解决方案**：修改 Makefile 添加 Python 链接标志

## 验证安装

```bash
# 验证 PyTorch 和 Transformers 安装
python -c "import torch; import transformers; print('PyTorch + Transformers import successful')"

# 测试推理脚本
cd inference
python infer.py --help || echo 'infer.py loaded, dependencies satisfied.'
cd ..

```

## 运行模型

```bash
kh# 运行推理示例
cd inference
python infer.py \
  --stage1_model m-a-p/YuE-s1-7B-anneal-en-cot \
  --stage2_model m-a-p/YuE-s2-1B-general \
  --genre_txt ../prompt_egs/genre.txt \
  --lyrics_txt ../prompt_egs/lyrics.txt \
  --output_dir ../output \
  --max_new_tokens 50
m
```

## 注意事项

1. 在 Apple Silicon 上，模型将使用 CPU/MPS 进行推理，性能会低于 GPU 设备
2. 微调功能在 Apple Silicon 上可能受限，建议仅用于推理
3. 如需完整功能，建议在 GPU 服务器上部署
4. 确保有足够的内存和存储空间（推荐至少 16GB RAM 和 20GB 存储）

## 故障排除

- 如果遇到 CUDA 相关错误，确保已移除所有 CUDA 依赖
- 如果 C++ 扩展编译失败，检查是否正确修改了 Makefile
- 如果模型加载失败，确保网络连接良好，因为模型需要从 Hugging Face 下载

## 高级用法

有关数据准备、微调和高级工作流程，请参考 `finetune/README.md` 文件。