# CFLD

# CFLD 项目部署指南

本指南提供了CFLD项目的完整部署流程，包括环境配置、依赖安装、脚本运行以及可能遇到的问题及解决方案。

## 1. 环境配置

### 创建Conda环境

```bash
# 从environment.yaml创建CFLD环境
conda env create -f environment.yaml

```

**可能的问题与解决方案**:

- **问题**: 在Mac或非CUDA环境中，pytorch-cuda相关包可能导致安装失败
- **解决方案**:
    
    ```bash
    # 移除environment.yaml中的pytorch-cuda依赖
    sed -i '' '/pytorch-cuda/d' environment.yaml
    # 移除xformers特定版本依赖(Mac环境可能不兼容)
    sed -i '' '/xformers=0.0.21/d' environment.yaml
    # 重新创建环境
    conda env create -f environment.yaml
    
    ```
    

### 激活环境

```bash
conda activate CFLD

```

## 2. 安装依赖包

```bash
# 如果环境创建后缺少某些包，可以更新环境
conda env update --file environment.yaml --prune

# 特别是xformers可能需要单独安装
pip install xformers

```

**可能的问题与解决方案**:

- **问题**: 导入torch、diffusers或xformers时出现`ModuleNotFoundError`
- **解决方案**:
    
    ```bash
    # 手动安装缺失的包
    pip install torch torchvision
    pip install diffusers
    pip install xformers
    
    ```
    

## 3. 验证环境配置

```bash
# 验证PyTorch安装
python -c "import torch; print(f'Torch version: {torch.__version__}, CUDA available: {torch.cuda.is_available()}')"

# 验证Diffusers安装
python -c "import diffusers; print('Diffusers version:', diffusers.__version__)"

# 验证xformers安装
python -c "import xformers; print('xformers import successful')"

```

## 4. 数据集准备

### 测试数据集处理脚本

```bash
# 测试数据集处理脚本是否可运行
python generate_fashion_datasets.py --help || echo 'Script has no --help; trying dry run.'
python generate_fashion_datasets.py || echo 'Expected error (missing data); script is present.'

```

**可能的问题与解决方案**:

- **问题**: 运行`generate_fashion_datasets.py`时出现`AssertionError: fashion is not a valid directory`
- **解决方案**: 这是预期的错误，因为需要先下载DeepFashion数据集。请按以下步骤操作：
    1. 从DeepFashion官网下载`Img/img_highres.zip`并解压到`./fashion`目录
    2. 下载DPTN的`train.lst`、`test.lst`及相关csv标注文件，放到`./fashion`目录
    3. 确保fashion目录结构完整后再运行数据集生成脚本

## 5. 数据集生成

```bash
# 确保数据集文件已正确放置后，运行数据集生成脚本
python generate_fashion_datasets.py

```

## 6. 模型训练

```bash
# 单GPU训练
bash scripts/single_gpu/pose_transfer_train.sh 0

# 多GPU训练
# bash scripts/multi_gpu/pose_transfer_train.sh

```

**可能的问题与解决方案**:

- **问题**: 在Mac或无CUDA环境中训练速度很慢
- **解决方案**: 这是预期行为，因为训练默认使用GPU。在CPU或Apple Silicon上训练会慢很多。

## 7. 模型推理

```bash
# 单GPU推理
bash scripts/single_gpu/pose_transfer_test.sh 0 MODEL.PRETRAINED_PATH checkpoints

# 多GPU推理
# bash scripts/multi_gpu/pose_transfer_test.sh MODEL.PRETRAINED_PATH checkpoints

```

**可能的问题与解决方案**:

- **问题**: 找不到预训练模型
- **解决方案**: 需要手动下载预训练模型并放置在`./pretrained_models`目录中

## 8. 注意事项

1. DeepFashion数据集需要从官方网站申请并下载，由于许可限制无法自动下载
2. 预训练模型同样需要手动下载并放置在正确位置
3. 在Apple Silicon (M1/M2) Mac上，训练和推理将使用CPU或Metal后端，速度会比CUDA慢
4. 确保有足够的磁盘空间和内存用于数据集处理和模型训练

按照此指南操作，您应该能够成功部署CFLD项目并进行训练和推理。如有其他问题，请参考项目的README文件或提交issue。