---
license: apache-2.0
language:
  - ja
  - zh
library_name: gpt-sovits
base_model:
  - lj1995/GPT-SoVITS
tags:
  - text-to-speech
  - voice-cloning
  - gpt-sovits
  - anime
  - japanese
  - chinese
  - lora
pipeline_tag: text-to-speech
---

# KurisuMakiseTTS — 牧濑红莉栖 中文/日语 TTS 微调模型

基于 **GPT-SoVITS v5**（dev / turbo 双版本）微调的角色语音模型。

> ## ⚠️ 使用前必读
>
> **这是一个同人（fan-made）研究项目，模型权重以 Apache-2.0 发布，允许商业使用。**
>
> - **权重许可**：本项目产出的模型参数、配置与文档以 Apache-2.0 授权，可用于商业产品
> - **但角色与声音的权利不属于本项目作者**：
>   - 角色「牧濑红莉栖」及原作品版权归 **5pb. / MAGES. / Nitroplus** 所有
>   - 声优本人的声音权益归声优本人所有
>   - 因此，**将本模型用于商业场景（商业配音、产品内置、直播带货等）前，
>     需你自行获得角色与声音权利方的授权**，本项目不代为授予该权利
> - 请勿用于冒充真人、伪造语音、诈骗等违法用途
> - 如权利方要求，本项目将立即下架

---

## 模型信息

| 项目 | 说明 |
|---|---|
| 基座 | GPT-SoVITS v5dev + v5turbo |
| 训练数据 | 972 条 / 67.8 分钟（32kHz 单声道，游戏原声） |
| 实际使用 | 754 条 / 60.7 分钟（3–10 秒黄金区间） |
| 训练方式 | SoVITS LoRA (rank 32) + GPT 全量微调 |
| 语言 | 日语（母语）/ 中文（跨语种） |
| 采样率 | 输出 48kHz |

### 两个版本怎么选

| | v5dev | v5turbo |
|---|---|---|
| 音质 | 更好 | 略逊（4 步蒸馏） |
| GPU RTF | ~1.08 | **~0.74–0.82** |
| **CPU RTF** | **~32（不可用）** | **~3.2（可用）** |
| 4 秒台词的 CPU 耗时 | ~131 秒 | **~12 秒** |
| 适用 | 有 GPU / 离线批量生成 | 纯 CPU 实时场景 |

> 两者**共用同一个 GPT 权重**（`gpt/kurisu_v5_gpt.ckpt`），因为 v5dev 与 v5turbo 的 GPT 底模都是 `s1v3.ckpt`。

### 主观听感评价（作者个人测评）

在相同参考音频、相同文本下的主观对比：

| 对比 | 结论 |
|---|---|
| **turbo vs dev** | turbo 的音色还原与自然度约为 dev 的 **80%–95%**，日常对话场景差异不明显；长句和情绪起伏处 dev 更稳 |
| **dev vs IndexTTS-2.5** | **在「牧濑红莉栖」这一角色上，dev 明显优于 IndexTTS-2.5** —— 微调模型的音色贴近度显著更高，IndexTTS 受限于单条参考音频的零样本克隆 |
| IndexTTS 的优势 | 仅在于「用自然语言描述控制情绪」，音色还原度不是它的强项 |

> 以上为作者的主观听测，非客观指标。IndexTTS-2.5 的权重不随本仓库分发（其许可为 bilibili Model Use License）。

---

## 测试与基准环境

所有训练与性能数据均在以下两台机器上实测：

### GPU 机器（训练 + GPU 推理基准）

| 项目 | 规格 |
|---|---|
| **GPU** | **NVIDIA RTX PRO 6000 Blackwell Server Edition** |
| 显存 | 96 GB（97,887 MiB） |
| 计算能力 | sm_120（Blackwell） |
| 驱动 / CUDA | 580.95.05 / CUDA 13.0 |
| CPU | 22 vCPU（Xeon Platinum 8470Q） |
| 内存 | 110 GB |
| PyTorch | 2.8.0+cu128 |
| 精度 | fp16（`is_half: true`） |

### CPU 机器（CPU 推理基准）

| 项目 | 规格 |
|---|---|
| **CPU** | **AMD Ryzen 7 255 w/ Radeon 780M**（Zen 4） |
| 核心 / 线程 | 8 核 / 16 线程，最高 3.8 GHz |
| 内存 | 21.8 GB DDR5 |
| 核显 | Radeon 780M（未参与计算，本次为纯 CPU 推理） |
| PyTorch | 2.14.1+cpu |
| 精度 | fp32（`is_half: false`） |

> 该 CPU 与常见迷你主机/开发板上的 **7840HS** 同属 Zen 4 架构，
> 因此本文的 CPU RTF 数据可近似参考 7840HS 平台。


---

## 📊 客观评测（held-out 测试集，未参与训练）

| 系统 | 日文 SS↑ | 日文 CER↓ | 中文 SS↑ | 中文 CER↓ |
|---|---|---|---|---|
| v5dev 底模（零样本） | 82.18 | 17.31 | 82.24 | 13.19 |
| v2Pro 微调（上一代公开模型） | 81.10 | 13.53 | 80.12 | 14.00 |
| **v5dev 微调（本仓库）** | 82.85 | 13.59 | 82.18 | **7.41** |
| **v5turbo 微调（本仓库）** | **83.11** | **12.71** | **82.38** | 12.04 |
| IndexTTS-2.5（零样本，仅对比） | 80.40 | 13.21 | 79.34 | **5.56** |
| 真实音频参照 | 83.89 | 10.04 | — | — |

> SS = 说话人嵌入余弦相似度（×100，越接近真实音频基线越好）；
> CER = Whisper-small 转写字错误率（×100，越低越可懂；10.04 为真实音频上的 ASR 底线）。
> 评测集为 6 条日文 + 4 条中文，全部未参与训练。方法与复现见 docs/benchmark.md。

**结论：** 微调后中文可懂度（CER）较底模几乎减半，turbo 的说话人相似度逼近真实音频基线。
IndexTTS-2.5 中文可懂度最高但音色相似度最低——零样本泛化与微调贴合的经典取舍
（其许可为 bilibili Model Use License，权重不随本仓库分发，数据仅作对比）。

---

## 🔊 试听样例

**日文（同一句，两种版本对比）**

dev 版：
<audio controls src="https://huggingface.co/FishelKET/KurisuMakiseTTS/resolve/main/samples/dev_ja.wav"></audio>

turbo 版：
<audio controls src="https://huggingface.co/FishelKET/KurisuMakiseTTS/resolve/main/samples/turbo_ja.wav"></audio>

turbo 版（**CPU 直出**，无 GPU）：
<audio controls src="https://huggingface.co/FishelKET/KurisuMakiseTTS/resolve/main/samples/turbo_ja_cpu.wav"></audio>

**中文（跨语种）**

dev 版：
<audio controls src="https://huggingface.co/FishelKET/KurisuMakiseTTS/resolve/main/samples/dev_zh.wav"></audio>

turbo 版：
<audio controls src="https://huggingface.co/FishelKET/KurisuMakiseTTS/resolve/main/samples/turbo_zh.wav"></audio>

**参考音频（`ref_audio/`，用于推理时提供语气锚点）**

<audio controls src="https://huggingface.co/FishelKET/KurisuMakiseTTS/resolve/main/ref_audio/ref1.wav"></audio>

---

## 快速开始

### 1. 准备 GPT-SoVITS 环境

```bash
git clone https://github.com/RVC-Boss/GPT-SoVITS.git
cd GPT-SoVITS
pip install -r requirements.txt
```

### 2. 下载必需的基座模型

以下模型**不在本仓库内**，请从官方获取，放到 `GPT-SoVITS/GPT_SoVITS/pretrained_models/`：

| 文件 | 用途 | 来源 |
|---|---|---|
| `chinese-hubert-base/` | 音频特征 | `TencentGameMate/chinese-hubert-base` |
| `chinese-roberta-wwm-ext-large/` | 文本特征 | `hfl/chinese-roberta-wwm-ext-large` |
| `s1v3.ckpt` | GPT 基座 | `lj1995/GPT-SoVITS` |
| `gsv-v5-pretrained/s2Gv5dev.pth` | dev 版 SoVITS 基座 | 同上 |
| `gsv-v5-pretrained/s2Gv5turbo.pth` | turbo 版 SoVITS 基座 | 同上 |
| `gsv-v5-pretrained/vocoder.pth` | 声码器 | 同上 |
| `sv/pretrained_eres2netv2w24s4ep4.ckpt` | 说话人验证 | 同上 |

### 3. 放置本模型

```
GPT-SoVITS/GPT_SoVITS/pretrained_models/
└── kurisu/
    ├── kurisu_v5_gpt.ckpt
    ├── kurisu_v5_sovits.pth
    └── kurisu_v5turbo_sovits.pth
```

### 4. 配置并启动

用 `config/` 下的模板，把路径改成你的实际路径：

```bash
# dev 版
python api_v2.py -a 127.0.0.1 -p 9880 -c config/tts_infer_v5dev.yaml

# turbo 版
python api_v2.py -a 127.0.0.1 -p 9880 -c config/tts_infer_v5turbo.yaml
```

### 5. 合成

```bash
curl http://127.0.0.1:9880/tts -H "Content-Type: application/json" -d '{
  "text": "早上好，我是牧濑红莉栖。",
  "text_lang": "zh",
  "ref_audio_path": "ref_audio/ref1.wav",
  "prompt_text": "あなたに会いに来たの、岡部凛太郎さん。じゃなくて…",
  "prompt_lang": "ja",
  "media_type": "wav"
}' -o out.wav
```

> **参考音频是必需的**——GPT-SoVITS 架构上参考音频参与推理（音色锚点 + 韵律起点）。
> 但音色已经微调进权重，所以换参考音频主要影响**语气**，而不是"像不像"。

---

## 参考音频建议

`ref_audio/` 提供 3 条 5–6 秒的干净样本。挑选原则：

- **5–10 秒**，不要超过 10 秒（api_v2 会拒绝）
- 干净、无 BGM、无混响
- **想要什么语气就选什么语气的样本**——合成会继承参考的语气
- `prompt_text` 必须和音频内容**逐字一致**

这批素材以「理性陈述」为主（占比约 73%），情绪化样本较少，这是选型时的客观限制。

---

## 训练细节

见 [docs/training.md](docs/training.md)。

**一览：** SoVITS 12 轮 LoRA(r32) + GPT 15 轮，单张 RTX PRO 6000 上约 8 分钟（SoVITS）+ 6 分钟（GPT）。

## 性能

见 [docs/benchmark.md](docs/benchmark.md)。

---

## 致谢与许可

- **代码**：[GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) — MIT License
- **基座模型**：lj1995 / RVC-Boss
- **训练音频与文本转写**：[`bysq/TTS-KurisuMakise`](https://huggingface.co/bysq/TTS-KurisuMakise) — **Apache-2.0**
  （本项目的 972 条语音素材及台词标注均来自该仓库，按其 Apache-2.0 许可使用）
- **本项目新增部分**（LoRA 权重、GPT 微调权重、配置、脚本、文档、评测数据）— **Apache-2.0**

完整许可见 [LICENSE](LICENSE) 与 [NOTICE](NOTICE)，使用限制见 [DISCLAIMER.md](DISCLAIMER.md)。

**仍需注意**：角色「牧濑红莉栖」及原作品的知识产权归 5pb. / MAGES. 等权利方所有，
音频来源仓库的许可声明不代表原权利方的授权。本仓库仅为技术研究与同人交流目的分发模型参数。
