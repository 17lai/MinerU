## 在C500+MACA上部署并使用Mineru

### 获取MACA镜像，包含torch-maca,maca,sglang-maca

镜像获取地址：https://developer.metax-tech.com/softnova/docker ,
选择maca-c500-pytorch:2.33.0.6-ubuntu22.04-amd64

若在docker上部署镜像则需要启动GPU设备访问
```bash
docker run --device=/dev/dri --device=/dev/mxcd....
```

#### 注意事项

由于此镜像默认开启TORCH_ALLOW_TF32_CUBLAS_OVERRIDE，会导致backed:vlm-transformers推理结果错误

```bash
unset TORCH_ALLOW_TF32_CUBLAS_OVERRIDE
```

### 安装MinerU

使用--no-deps，去除对一些cuda版本包的依赖，后续采用pip install-r requirements.txt 安装其他依赖
```bash
pip install -U "mineru[core]" --no-deps
```

```tex
boto3>=1.28.43
click>=8.1.7
loguru>=0.7.2
numpy==1.26.4
pdfminer.six==20250506
tqdm>=4.67.1
requests
httpx
pillow>=11.0.0
pypdfium2>=4.30.0
pypdf>=5.6.0
reportlab
pdftext>=0.6.2
modelscope>=1.26.0
huggingface-hub>=0.32.4
json-repair>=0.46.2
opencv-python>=4.11.0.86
fast-langdetect>=0.2.3,<0.3.0
transformers>=4.51.1
accelerate>=1.5.1
pydantic
matplotlib>=3.10,<4
ultralytics>=8.3.48,<9
dill>=0.3.8,<1
rapid_table>=1.0.5,<2.0.0
PyYAML>=6.0.2,<7 
ftfy>=6.3.1,<7
openai>=1.70.0,<2
shapely>=2.0.7,<3
pyclipper>=1.3.0,<2
omegaconf>=2.3.0,<3
transformers>=4.49.0,!=4.51.0,<5.0.0
fastapi
python-multipart
uvicorn
gradio>=5.34,<6
gradio-pdf>=0.0.22
albumentations
beautifulsoup4
scikit-image==0.25.0
outlines==0.1.11
magika>=0.6.2,<0.7.0
mineru-vl-utils>=0.1.6,<1
```
上述内容保存为requirments.txt,进行安装
```bash
pip install -r requirments.txt
```
安装doclayout_yolo，这里doclayout_yolo会依赖torch-cuda,使用--no-deps
```bash
pip install doclayout-yolo --no-deps
```
### 在线使用
**基础使用命令为:mineru -p <input_path> -o <output_path> -b vlm-transformers**

- `<input_path>`: Local PDF/image file or directory
- `<output_path>`: Output directory
- -b  --backend [pipeline|vlm-transformers|vlm-vllm-engine|vlm-http-client] (default:pipeline)<br/>

其他详细使用命令可参考官方文档[Quick Usage - MinerU](https://opendatalab.github.io/MinerU/usage/quick_usage/#quick-model-source-configuration)

### 离线使用

**所用模型为本地模型，需要设置环境变量和config配置文件**<br/>
#### 下载模型到本地
通过mineru交互式命令行工具进行下载，下载完后会自动更新mineru.json配置文件
```bash
mineru-models-download
```
也可以在[HuggingFace](http://www.huggingface.co.)或[ModelScope](https://www.modelscope.cn/home)找到所需模型源（PDF-Extract-Kit-1.0和MinerU2.5-2509-1.2B）进行下载，
下载完成后，创建mineru.json文件，按如下进行修改
```json
{
    "models-dir": {
        "pipeline": "/path/pdf-extract-kit-1.0/",
        "vlm": "/path/MinerU2.5-2509-1.2B"
    },
    "config_version": "1.3.0"
}
```
path为本地模型的存储路径，其中models-dir为本地模型的路径，pipeline代表backend为pipeline时，所需要的模型路径，vlm代表backend为vlm-开头，所需要的模型路径

#### 修改环境变量

```bash
export MINERU_MODEL_SOURCE=local
export MINERU_TOOLS_CONFIG_JSON=/path/mineru.json   //此环境变量为配置文件的路径
```
修改完成后即可正常使用<br/>