# 运行命令

- 项目：Whisper-WebUI
- 生成时间：2026-09-08
- 运行方式：直接运行（uv + Python 3.12）
- 硬件评估：**满足**
  - 官方默认 faster-whisper fp16 约 4755MB 显存；openai/whisper 约 11325MB
  - 本机 RTX 4080 / 16GB；评估时已用约 3.5GB（含其他 python 进程），剩余约 12.8GB，够默认路径
  - 内存约 32GB，磁盘 E: 充足
  - 若同时开 UVR/说话人分离/openai-whisper large，显存会明显变紧，勿与其他占卡任务硬叠

## 环境准备

```powershell
# ffmpeg（本机已有）
$env:Path = "E:\Programs\ffmpeg-master-latest-win64-gpl\bin;$env:Path"

# 国内源：PyPI 官方超时，HF 官方超时
$env:UV_INDEX_URL = "https://pypi.tuna.tsinghua.edu.cn/simple"
$env:HF_ENDPOINT = "https://hf-mirror.com"

# Python 3.10–3.12；勿用系统 3.14
cd E:\AI\local-voice\Whisper-WebUI
uv venv --python 3.12 .venv
uv pip install -r requirements.txt --index-url https://pypi.tuna.tsinghua.edu.cn/simple --extra-index-url https://download.pytorch.org/whl/cu128
```

`requirements.txt` 默认 CUDA 12.8 的 torch 轮子；PyTorch 直连可用。模型首次在 UI 选择时经 HF 镜像下载到 `models/`。

## 启动

- 推荐：`pwsh -NoProfile -File .\start.ps1`
- 说明：单服务，跳过菜单；端口从 7860 起占用则顺延
- 等价手动命令：

```powershell
$env:Path = "E:\Programs\ffmpeg-master-latest-win64-gpl\bin;$env:Path"
$env:HF_ENDPOINT = "https://hf-mirror.com"
.\.venv\Scripts\python.exe app.py --server_name 127.0.0.1 --server_port 7860 --inbrowser false
```

Docker 备选（镜像约 7GB）：`docker compose up`（需 NVIDIA Container Toolkit）。

## 验证

1. 浏览器打开启动日志中的 `http://127.0.0.1:<port>`
2. 上传短音频，选 faster-whisper + 小模型（如 `base`/`small`），能出字幕即通
3. `Test-NetConnection 127.0.0.1 -Port <port>` 应 Succeeded

## 备注

- 镜像：PyPI → 清华；HF → `https://hf-mirror.com`；PyTorch cu128 直连
- ffmpeg：`E:\Programs\ffmpeg-master-latest-win64-gpl\bin`
- 说话人分离需 Hugging Face token，并接受 pyannote 协议
- - 首次启动时 7860/7861 已被占用，实际端口为 **7862**（start.ps1 会自动顺延）
- 缺依赖 `audioread` 会导致 UVR BGM 分离不可用；已补装进 `.venv`，若需 UVR 请重启 WebUI
- 本目录 `RUN.md` / `start.ps1` 为本机记录，默认不提交
