# 训练细节

## 数据

| 项目 | 值 |
|---|---|
| 原始条数 | 972 条 |
| 总时长 | 67.8 分钟 |
| 采样率 | 32 kHz 单声道 |
| 平均单条 | 4.19 秒 |
| 训练用 | **754 条 / 60.7 分钟**（筛选 3–10 秒区间） |
| 文本 | 全量日文台词（已有转写，无需 ASR） |
| 音频质量 | 游戏原声，无 BGM、无混响 |

数据集分布（按声学聚类，非语义标签）：

| 类型 | 占比 | 说明 |
|---|---|---|
| 理性陈述类 | ~73% | 讲解、分析、判断（含 6 个声学子簇） |
| 冷静说教 | ~19% | 带自夸与不耐烦 |
| 强情绪 | ~8% | 恼怒吐槽 / 悲愤质问 |

> **客观限制：素材以理性语气为主，情绪化样本偏少。** 这决定了模型在"平静讲解"上表现最稳，
> 在"强烈情绪"上表现受限于参考音频。

## 预处理

标准 GPT-SoVITS 三步流程（`prepare_datasets/`）：

1. `1-get-text.py` — 音素 + 词级对齐
2. `2-get-hubert-wav32k.py` — hubert 特征 + 重采样 32k
3. `3-get-semantic.py` — 语义 token（需要 s2G 底模）

关键环境变量：

```bash
export inp_text=logs/kurisu/kurisu.list
export inp_wav_dir=/path/to/wav
export exp_name=kurisu
export i_part=0
export all_parts=1
export opt_dir=logs/kurisu
export version=v5dev
export bert_pretrained_dir=GPT_SoVITS/pretrained_models/chinese-roberta-wwm-ext-large
export cnhubert_base_dir=GPT_SoVITS/pretrained_models/chinese-hubert-base
export pretrained_s2G=GPT_SoVITS/pretrained_models/gsv-v5-pretrained/s2Gv5dev.pth
export s2config_path=GPT_SoVITS/configs/s2.json
```

> 注意：这三个脚本 `import text.xxx` 和 `import tools.xxx`，运行时
> `cwd` 需为 `GPT_SoVITS/`，且 `PYTHONPATH` 同时包含仓库根目录与 `GPT_SoVITS/`。

## SoVITS 训练

```bash
python -s GPT_SoVITS/s2_train_v3_lora.py --config tmp_s2.json
```

| 参数 | dev | turbo |
|---|---|---|
| version | `v5dev` | `v5turbo` |
| pretrained_s2G | `s2Gv5dev.pth` | `s2Gv5turbo.pth` |
| epochs | 12 | 12 |
| batch_size | 8 | 8 |
| lora_rank | 32 | 32 |
| text_low_lr_rate | 0.4 | 0.4 |

训练速度：96 iter/epoch，约 8 it/s → **12 轮约 4 分钟**（RTX PRO 6000）。

## GPT 训练

```bash
python -s GPT_SoVITS/s1_train.py --config_file tmp_s1.yaml
```

| 参数 | 值 |
|---|---|
| pretrained_s1 | `s1v3.ckpt` |
| epochs | 15 |
| batch_size | 8 |
| if_dpo | false |
| hz | 25hz |

**v5dev 与 v5turbo 共用此 GPT 权重**——因为两者的 GPT 底模都是 `s1v3.ckpt`。

训练速度：**15 轮约 6 分钟**。

## 产物

```
SoVITS_weights_v5dev/kurisu_e6_s576_l32.pth       72MB
SoVITS_weights_v5dev/kurisu_e9_s864_l32.pth       72MB
SoVITS_weights_v5dev/kurisu_e12_s1152_l32.pth     72MB   ← 采用
SoVITS_weights_v5turbo/kurisu_e12_s1152_l32.pth   73MB   ← 采用
GPT_weights_v5dev/kurisu-e15.ckpt                 149MB  ← 采用
```

> **注意：LoRA 训练产出的 `.pth` 是增量权重**，推理时必须同时提供对应的
> `s2Gv5dev.pth` / `s2Gv5turbo.pth` 基座，否则会报
> `FileNotFoundError: ... 底模缺失，无法加载相应 LoRA 权重`。

## 踩过的坑（复现者可能遇到）

| 现象 | 原因 | 解决 |
|---|---|---|
| `No module named 'text'` / `'tools'` | sys.path 不含 GPT_SoVITS/ 或仓库根 | 设 PYTHONPATH 两者都要 |
| `FileNotFoundError: 'ffmpeg'` | 未装 ffmpeg | `apt install ffmpeg` |
| `No module named 'opencc'` | Linux 上 opencc 需编译 | `pip install opencc-python-reimplemented` |
| `No module named 'pyopenjtalk'` | 无 Linux 预编译轮子 | `pip install pyopenjtalk`（需 cmake + gcc） |
| `No module named 'resampy'` | 推理时重采样 | `pip install resampy` |
| `No module named 'onnxruntime'` | G2PW 多音字模型依赖 | `pip install onnxruntime` |
| `fast-langdetect: Cache directory not found` | 缺语言检测模型 | 放 `lid.176.ftz` 到 `pretrained_models/fast_langdetect/` |
| 保存权重报 `FileNotFoundError: SoVITS_weights_xxx/` | 导出目录不存在 | 先 `mkdir -p SoVITS_weights_v5dev` |
