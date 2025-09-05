# Vosk

# Vosk 语音识别工具部署指南

本指南提供了在 MacOS (arm64) 上部署 Vosk 语音识别工具的完整步骤，包括可能遇到的错误及解决方案。

## 1. 创建隔离环境

```bash
# 创建一个 Python 3.10 的 conda 环境
conda create -n vosk-test python=3.10
# 激活环境
conda activate vosk-test

```

## 2. 安装依赖

```bash
# 更新 pip
pip install --upgrade pip
# 安装 Vosk 依赖项
pip install cffi requests tqdm srt websockets

```

## 3. 安装 Vosk 包

```bash
# 进入 python 目录并以可编辑模式安装 Vosk
cd python && pip install -e . && cd ..

```

> 可能的错误：如果安装过程中出现依赖冲突或编译错误
> 
> 
> **解决方案**：
> 
> ```bash
> # 强制重新安装 Vosk
> pip install --force-reinstall vosk
> 
> ```
> 

## 4. 下载语音模型

```bash
# 下载小型英语模型 (约50MB)
curl -LO <https://alphacephei.com/vosk/models/vosk-model-small-en-us-0.15.zip>
# 解压模型
unzip -q vosk-model-small-en-us-0.15.zip
# 重命名模型目录
mv vosk-model-small-en-us-0.15 model

```

> 可能的错误：如果下载链接失效
> 
> 
> **解决方案**：
> 访问 [https://alphacephei.com/vosk/models](https://alphacephei.com/vosk/models) 获取最新模型链接
> 

## 5. 下载测试音频

```bash
# 下载测试用的 WAV 文件
curl -L -o test.wav <https://github.com/alphacep/vosk-api/raw/master/python/example/test.wav>

```

## 6. 验证安装

```bash
# 使用示例脚本测试
python python/example/test_simple.py python/example/test.wav

```

> 可能的错误：如果直接运行复杂的一行命令出现语法错误
> 
> 
> **解决方案**：
> 不要使用复杂的一行命令，而是使用仓库中提供的示例脚本，如上所示。
> 
> **可能的错误**：找不到模型或测试文件
> 
> **解决方案**：
> 
> ```bash
> # 确保当前目录是项目根目录
> cd "/Users/[用户名]/[项目路径]/vosk-api"
> # 检查文件是否存在
> ls -la model
> ls -la python/example/test.wav
> 
> ```
> 

## 7. 使用示例

完成安装后，可以通过以下方式使用 Vosk：

### 1. 使用命令行转录 WAV 文件

```bash
python -m vosk.transcriber --model model test.wav

```

### 2. 在 Python 代码中使用

创建一个 `test_vosk.py` 文件，内容如下:

```python
from vosk import Model, KaldiRecognizer
import wave
import json

model = Model("model")
wf = wave.open("test.wav", "rb")
rec = KaldiRecognizer(model, wf.getframerate())

results = []
while True:
    data = wf.readframes(4000)
    if len(data) == 0:
        break
    if rec.AcceptWaveform(data):
        results.append(json.loads(rec.Result()))
results.append(json.loads(rec.FinalResult()))

print("TRANSCRIPT:", results)

```

然后运行:

```bash
python test_vosk.py

```

## 8. 故障排除

1. **问题**：导入 `vosk` 模块失败
    
    **解决方案**：确认 vosk 已正确安装
    
    ```bash
    pip list | grep vosk
    # 如果未安装或版本不正确，重新安装
    pip install --force-reinstall vosk
    
    ```
    
2. **问题**：找不到模型目录
    
    **解决方案**：确认模型路径正确
    
    ```bash
    # 检查模型目录是否存在
    ls -la model
    # 如果不存在，重新下载并解压
    curl -LO <https://alphacephei.com/vosk/models/vosk-model-small-en-us-0.15.zip>
    unzip -q vosk-model-small-en-us-0.15.zip
    mv vosk-model-small-en-us-0.15 model
    
    ```
    
3. **问题**：运行示例脚本时出现路径错误
    
    **解决方案**：确保在正确的目录中运行命令
    
    ```bash
    # 导航到项目根目录
    cd "/Users/[用户名]/[项目路径]/vosk-api"
    
    ```
    
4. **问题**：ARM64 架构兼容性问题
    
    **解决方案**：确保使用最新版本的 Vosk，它支持 ARM64 架构
    
    ```bash
    pip install --upgrade vosk
    
    ```
    

完成以上步骤后，Vosk 语音识别工具应该已成功部署并可以使用。