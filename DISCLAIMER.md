# 免责声明 / Disclaimer

## 中文

1. **本仓库是一个同人（fan-made）研究项目。** 它基于开源项目 GPT-SoVITS 微调而成，
   目的是探索语音合成与角色语音复现的技术实践。

2. **训练数据来源与许可。**
   - 本模型使用的 972 条语音素材与日文台词转写，来自 Hugging Face 仓库
     [`bysq/TTS-KurisuMakise`](https://huggingface.co/bysq/TTS-KurisuMakise)，
     该仓库声明以 **Apache-2.0** 协议发布。
   - 本项目依该协议使用这些素材训练模型，并在 `ref_audio/` 目录中附带少量样本用于演示。
   - **本仓库不包含完整的原始音频数据集。**

3. **角色与原作权利不属于本项目。**
   - 角色「牧濑红莉栖」（牧瀬紅莉栖 / Kurisu Makise）及其原作品的知识产权
     归 **5pb. / MAGES. / Nitroplus** 等权利方所有。
   - 声优对其声音享有声音权 / パブリシティ権。
   - **上游仓库的 Apache-2.0 声明，不构成原权利方的授权。** 使用者需自行承担相关风险。

4. **使用范围与责任划分。**
   - **本仓库产出的模型权重、配置与文档以 Apache-2.0 授权，允许商业使用**，
     可自由使用、修改、再分发，无需向本项目作者申请许可。
   - **但 Apache-2.0 只能授权本项目作者拥有的部分。** 角色形象、原作品与声优声音
     的权利不属于本项目作者，因此：
     - 若你计划将本模型用于**商业场景**（商业配音、产品内置、直播、广告、付费内容等），
       **需你自行获得角色与声音权利方（5pb. / MAGES. 及声优）的授权**；
       本项目不构成、也不替代该授权。
     - 因使用者未取得相应授权而产生的任何法律后果，由使用者自行承担。
   - **以下用途一律禁止**（与 Apache-2.0 无关，属强制性红线）：
     - 冒充真人、伪造证据、诈骗、骚扰
     - 生成涉及政治、色情、暴力等违法内容
     - 任何可能严重损害角色形象或声优声誉的用途

5. **无担保。** 本模型按「现状」提供，作者不对其准确性、合法性、适用性作任何担保，
   也不对使用者因使用本模型产生的任何后果承担责任。

6. **下架承诺。** 若任何权利方提出异议，本项目将立即删除相关内容。

7. **如果你不能接受以上条款，请勿使用本模型。**

---

## English

1. This is a **fan-made research project**, fine-tuned on top of the open-source
   GPT-SoVITS project for technical exploration of speech synthesis.

2. **Training data source & license.**
   - The 972 voice clips and Japanese transcripts used for training come from the
     Hugging Face repository [`bysq/TTS-KurisuMakise`](https://huggingface.co/bysq/TTS-KurisuMakise),
     which declares the **Apache-2.0** license.
   - This project uses those materials under that license and includes a small
     number of reference samples in `ref_audio/` for demonstration only.
   - **The full original audio dataset is NOT redistributed here.**

3. **Character and franchise rights do not belong to this project.**
   - The character "Kurisu Makise" and the original work are copyrighted by
     **5pb. / MAGES. / Nitroplus** and related rights holders.
   - The voice actress retains rights over her voice.
   - **The upstream repository's Apache-2.0 declaration does not constitute
     authorization from the original rights holders.** Users assume all related risk.

4. **Scope and liability.**
   - **The model weights, configs and docs produced by this project are licensed
     under Apache-2.0 and MAY be used commercially**, modified and redistributed
     without further permission from this project's author.
   - **However, Apache-2.0 can only license what this author owns.** The character
     likeness, the original work and the voice actress's voice are not ours, so:
     - If you intend to use this model **commercially** (commercial dubbing,
       shipping in a product, streaming, advertising, paid content, etc.), you must
       obtain your own authorization from the relevant rights holders
       (5pb. / MAGES. and the voice actress). This project does not grant, and does
       not substitute for, such authorization.
     - You are solely responsible for any legal consequences arising from your use.
   - **The following are prohibited outright** (independent of Apache-2.0):
     impersonation, fraud, harassment, generating illegal content, or any use that
     seriously harms the reputation of the character or the voice actress.

5. **No warranty.** Provided "as is" without warranty of any kind.

6. **Takedown.** This project will be removed immediately upon request from any
   rights holder.

7. **If you do not accept these terms, do not use this model.**
