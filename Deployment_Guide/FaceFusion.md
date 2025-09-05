# FaceFusion

# FaceFusion 部署指南

本指南提供了在 macOS（特别是 Apple Silicon）上部署 FaceFusion 的完整步骤，包括可能遇到的错误及解决方案。

## 1. 创建 Conda 环境

```bash
conda create -n facefusion python=3.10 -y
conda activate facefusion

```

**可能的错误**:

- **conda 命令未找到**: 安装 Miniconda 或 Anaconda
    
    ```bash
    # 下载并安装 Miniconda
    curl -O <https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh>
    bash Miniconda3-latest-MacOSX-arm64.sh
    
    ```
    

## 2. 安装依赖

```bash
pip install -r requirements.txt

```

**可能的错误**:

- **requirements.txt 未找到**: 确保在项目根目录下运行命令
- **opencv-python 安装失败**: 在 Apple Silicon 上可能需要替代方案
    
    ```bash
    pip uninstall -y opencv-python
    pip install opencv-python-headless
    
    ```
    
- **numpy 版本冲突**: 指定兼容版本
    
    ```bash
    pip install numpy==1.24.3
    
    ```
    
- **onnxruntime 安装失败**: 为 Apple Silicon 安装特定版本
    
    ```bash
    pip install onnxruntime-silicon
    
    ```
    

## 3. 运行安装脚本

```bash
python install.py

```

**可能的错误**:

- **模块未找到错误**: 检查是否在正确的目录中
- **权限错误**: 确保有写入权限
    
    ```bash
    chmod +x install.py
    
    ```
    
- **网络错误**: 检查网络连接，可能需要重试或使用代理

## 4. 验证安装

```bash
python facefusion.py --version

```

**可能的错误**:

- **模块未找到**: 确保所有依赖都已正确安装
    
    ```bash
    pip install -r requirements.txt --ignore-installed
    
    ```
    
- **版本不显示**: 检查 [facefusion.py](http://facefusion.py/) 是否存在并有执行权限

## 5. 测试运行

```bash
timeout 120 python facefusion.py run

```

**可能的错误**:

- **timeout 命令未找到**: macOS 使用不同命令
    
    ```bash
    gtimeout 120 python facefusion.py run
    # 如果没有 gtimeout，安装 coreutils
    brew install coreutils
    
    ```
    
- **GUI 相关错误**: 可能需要安装额外的 GUI 依赖
    
    ```bash
    pip install PyQt5
    
    ```
    
- **模型下载失败**: 手动下载模型
    
    ```bash
    python facefusion.py force-download
    
    ```
    

## 6. 强制下载模型（如有需要）

```bash
python facefusion.py force-download

```

**可能的错误**:

- **网络超时**: 检查网络连接，可能需要使用代理
- **磁盘空间不足**: 确保有足够的磁盘空间（至少 2GB）

## 7. 使用示例

```bash
# 基本运行
python facefusion.py run

# 批处理模式
python facefusion.py batch-run

# 查看所有选项
python facefusion.py --help

```

**常见使用错误**:

- **缺少源图像或目标图像**: 确保指定了必要的参数
    
    ```bash
    python facefusion.py run --source 源图像路径 --target 目标图像路径
    
    ```
    
- **内存不足**: 减小处理的图像尺寸
    
    ```bash
    python facefusion.py run --source 源图像路径 --target 目标图像路径 --output-resolution 720
    
    ```
    

## 8. 故障排除

如果遇到其他问题，尝试以下步骤：

1. 检查 Python 版本是否为 3.10
    
    ```bash
    python --version
    
    ```
    
2. 确保 conda 环境已激活
    
    ```bash
    conda activate facefusion
    
    ```
    
3. 更新所有依赖
    
    ```bash
    pip install --upgrade -r requirements.txt
    
    ```
    
4. 清除缓存
    
    ```bash
    pip cache purge
    
    ```
    
5. 查看官方文档获取更多帮助
    
    ```bash
    echo "访问 <https://docs.facefusion.io> 获取更多信息"
    
    ```
    

祝您部署顺利！