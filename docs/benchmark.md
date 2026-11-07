# 性能基准

## 测试环境

| | GPU | CPU |
|---|---|---|
| 硬件 | **NVIDIA RTX PRO 6000 Blackwell Server Edition** | **AMD Ryzen 7 255 w/ Radeon 780M** |
| 显存 | 96 GB（97,887 MiB） | —（共享系统内存） |
| 计算能力 | sm_120 | — |
| 核心 / 线程 | 22 vCPU（Xeon Platinum 8470Q） | 8 核 / 16 线程 @ 3.8 GHz |
| 内存 | 110 GB | 21.8 GB DDR5 |
| 驱动 / CUDA | 580.95.05 / CUDA 13.0 | — |
| PyTorch | 2.8.0+cu128 | 2.14.1+cpu |
| 精度 | fp16 (is_half=true) | fp32 (is_half=false) |

> CPU 侧为 Zen 4 架构，与 **7840HS** 同代，RTF 数据可近似参考该平台。
> 本次测试中 Radeon 780M 核显**未参与计算**（纯 CPU 推理）。

测试文本：`そう言えば、まともに自己紹介してませんでしたね。`（日文，约 4 秒音频）
参考音频：5.06 秒日文样本

## 结果

### RTF（Real-Time Factor，越低越快）

| 模型 | GPU | CPU | CPU/GPU 差距 |
|---|---|---|---|
| **v5turbo** | **0.74 – 0.82** | **3.19 – 3.60** | ~4.4x |
| v5dev | 1.08 | ~32 | ~30x |

> RTF = 生成耗时 / 音频时长。RTF < 1 表示比实时快。

### 实际耗时（4 秒左右的句子）

| 模型 | GPU | CPU |
|---|---|---|
| v5turbo | ~3 秒 | **~12–13 秒** |
| v5dev | ~9 秒 | ~131 秒 |

### 关键结论

1. **turbo 在 CPU 上比 dev 快约 10 倍**（13s vs 131s）
   — 4 步蒸馏的收益在 CPU 上被放大，因为每步采样开销占比更高

2. **turbo + CPU 是可用的组合**
   — 4 秒台词 ~13 秒出，异步 bot 场景完全能接受

3. **dev 在 CPU 上不可用**（RTF 32，一句话 2 分钟）
   — 除非你有 GPU，否则 dev 只适合离线批量预生成

4. **v5 整体比 v2Pro 慢很多**
   — v2Pro 在本机 CPU 上 RTF 约 1.64，v5turbo 是 3.19。
   — v5 音质明显更好，代价是速度。

## 选型建议

| 场景 | 推荐 |
|---|---|
| 有 GPU（≥8GB 显存） | v5dev（音质优先） |
| 只有 CPU，要能接受 10 秒级延迟 | **v5turbo** |
| 只有 CPU，要求近实时（<3 秒） | v2Pro 微调版 |
| 高频固定台词 | v5dev 预生成 + 磁盘缓存 |


## 客观评测（held-out 测试集）

测试集：6 条日文 + 4 条中文，**均未参与训练**（训练集筛选了 3–10 秒片段，
评测集全部来自 2–3 秒的训练集外片段）。指标：

- **SS（Speaker Similarity）↑**：resemblyzer VoiceEncoder（256 维）说话人嵌入余弦相似度，
  与 8 条真实红莉栖音频的平均值，×100
- **CER（Character Error Rate）↓**：Whisper-small 转写后与原文本的字错误率，×100

| 系统 | ja SS↑ | ja CER↓ | zh SS↑ | zh CER↓ |
|---|---|---|---|---|
| v5dev 底模（零样本，未微调） | 82.18 | 17.31 | 82.24 | 13.19 |
| v2Pro 微调（上一代公开模型） | 81.10 | 13.53 | 80.12 | 14.00 |
| **v5dev 微调（本仓库）** | 82.85 | 13.59 | 82.18 | **7.41** |
| **v5turbo 微调（本仓库）** | **83.11** | **12.71** | **82.38** | 12.04 |
| 真实音频参照 | 83.89 | 10.04¹ | — | — |

¹ 10.04 是 Whisper-small 在**真实音频**上的转写误差，即本评测的 ASR 底线；
低于此值理论上不可能。83.89 为真实片段互相之间的 SS 基线。

**要点：**

1. **SS：v5turbo 合成语音与真实片段的相似度（83.11）几乎追平真实片段之间的基线（83.89）**
   —— 在说话人嵌入层面接近"真假难辨"
2. **中文可懂度：v5dev 的 CER 7.41，较零样本（13.19）几乎减半**
   —— 注意 CER 衡量的是字词正确率；主观上的"日本腔"属于韵律问题，两者独立
3. 全部微调版本的日文 CER 均优于零样本底模

### 复现

评测集与脚本见仓库 `eval/` 目录（合成音频 + 指标脚本），
Whisper-small 作为 ASR 评审，字级 Levenshtein 计算 CER，
resemblyzer 计算 256 维说话人嵌入余弦相似度。

## 主观听感评测（作者个人，非客观指标）

测试条件：同一参考音频（`ref_audio/ref1.wav`）、同一文本、各模型默认参数。

| 对比项 | 结论 |
|---|---|
| **turbo vs dev** | turbo 达到 dev 的 **80%–95%** 水平。短句、日常对话几乎听不出差别；长句、情绪起伏、句尾收束上 dev 更自然 |
| **dev vs v2Pro（上一代微调）** | dev 明显更好，作者原话："这个效果比 v2Pro 好太多太多了" |
| **dev vs IndexTTS-2.5** | **在牧濑红莉栖这一角色上 dev 明显更优**。微调模型对音色的还原度显著高于 IndexTTS 的单参考零样本克隆 |
| IndexTTS 唯一优势 | 可用自然语言描述控制情绪（见 `samples/` 与 IndexTTS 的 C1–C4 对比），但音色不像本人 |

### 结论

- 追求**音色还原**：GPT-SoVITS 微调（dev > turbo >> IndexTTS）
- 追求**情绪可控**：IndexTTS 有独特价值，但需要接受音色损失
- 两者不是替代关系，理论上可并存（音色走 GPT-SoVITS，强情绪段落走 IndexTTS）

## 与 IndexTTS-2.5 的对比（参考）

同一台 PRO 6000 上：

| 模型 | 参数量 | GPU RTF |
|---|---|---|
| IndexTTS-2.5 | 0.8B | 0.36 – 0.93 |
| GPT-SoVITS v5turbo | — | 0.74 – 0.82 |
| GPT-SoVITS v5dev | — | 1.08 |

> IndexTTS 未做许可兼容性评估，其权重不随本仓库分发。
> 其优势是"用文本描述控制情绪"，劣势是音色还原度取决于单条参考音频。

## 复现方法

```bash
# GPU
python api_v2.py -a 127.0.0.1 -p 9880 -c config/tts_infer_v5turbo.yaml

# CPU（把 config 里的 device 改成 cpu、is_half 改 false）
python api_v2.py -a 127.0.0.1 -p 9880 -c config/tts_infer_v5turbo.yaml

# 计时
time curl http://127.0.0.1:9880/tts -H "Content-Type: application/json" \
  -d @request.json -o out.wav
```

> **首次调用**会包含模型懒加载，耗时远高于稳态。测速请跑第二次。
