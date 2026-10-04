# AGENTS.md

本文件用于让新的 Agent 快速接管 `local-cosyvoice`，以及让任何拿到本仓库地址的人，能够让自己的 Agent 在 Apple Silicon Mac 上复现这套本地个人声音 TTS 工作流。

## 1. 项目目标

这个仓库不是模型仓库，也不是要把个人声音上传到 GitHub。

目标是维护一条可复用的本地工作流：

1. 使用 `speech` CLI + CosyVoice3 做个人声音 TTS。
2. Markdown 转朗读文本，并做十进制数字朗读规范化。
3. 长文本按较小语义块独立生成 WAV。
4. 用本地 Qwen3-ASR 0.6B 4bit 对每块做内容完整性校验。
5. FAIL 块只重试当前块，并切换 seed；PASS / REVIEW 块不重复生成。
6. 所有块满足门槛后，用 ffmpeg 合成最终 WAV。

核心规则位于：

`skill/SKILL.md`

## 2. 接管项目时先做什么

新的 Agent 接管后，先只读检查，不要直接改文件：

1. 阅读 `README.md`。
2. 完整阅读 `AGENTS.md`。
3. 完整阅读 `skill/SKILL.md`。
4. 检查 Git 状态、当前分支、remote 和最近提交。
5. 检查本机是否已有 `speech`、`ffmpeg`、参考音频、CosyVoice3 缓存和 Qwen3-ASR 缓存。
6. 不要假设仓库作者机器上的绝对路径在新机器上仍然有效。

若只是接管现有作者机器，优先复用已有安装、模型缓存和参考声音，不重复下载或安装。

## 3. 仓库里有什么，什么不会提交

仓库应包含：

- `README.md`
- `AGENTS.md`
- `skill/SKILL.md`
- 后续必要的脚本、配置模板和文档

仓库不应包含：

- 个人参考声音
- 生成的 WAV / MP3
- Python 虚拟环境
- 模型权重和模型缓存
- Speech Studio 安装包
- 第三方源码完整 checkout，例如 `.whisper-cpp/`
- API Key、Token、密码或其他秘密

提交前检查 `.gitignore`，并确认 `git status` 里没有个人音频、大模型、缓存、安装包或秘密文件。

## 4. 在另一台 Apple Silicon Mac 上安装

### 4.1 前提

本项目当前按 Apple Silicon Mac 设计。其他平台不要直接照搬，先验证兼容性。

推荐先检查：

```bash
uname -m
sw_vers
which brew
which speech
which ffmpeg
```

### 4.2 安装基础 CLI

如果 Homebrew 尚未安装，先让用户决定是否安装 Homebrew；不要静默修改系统。

已有 Homebrew 后：

```bash
brew install speech
brew install ffmpeg
```

安装后检查：

```bash
speech --help
speech speak --help
speech transcribe --help
ffmpeg -version
```

参数以本机 `--help` 为准，不要凭记忆猜测。

### 4.3 准备个人参考声音

每个使用者都必须使用自己的参考声音，不能直接复用仓库作者的声音。

建议准备：

- 一段约 20–30 秒、环境安静、自然语速的参考录音
- 与录音逐字对应的 reference transcript

参考音频与 transcript 必须准确对应。

把参考音频放在使用者自己选择的本地路径，不要提交 Git。

### 4.4 准备模型

当前工作流依赖：

- CosyVoice3
- Qwen3-ASR 0.6B 4bit

新机器第一次配置时，先检查本机缓存是否已存在。若缺失模型，Agent 必须先告诉用户将发生模型下载，并在得到用户明确同意后再下载所需模型；不要顺手下载额外模型。

Qwen3-ASR 当前目标模型：

`aufklarer/Qwen3-ASR-0.6B-MLX-4bit`

不要默认安装 ForcedAligner，当前工作流并不依赖它。

## 5. 安装 Skill

本仓库的 `skill/SKILL.md` 是可迁移规则，但其中可能包含仓库作者机器上的本地绝对路径和作者自己的 reference transcript。

在另一台机器上使用时：

1. 复制 `skill/SKILL.md` 到该 Agent 的 Skill 目录，例如：

   ```bash
   mkdir -p ~/.agents/skills/my-voice-tts
   cp skill/SKILL.md ~/.agents/skills/my-voice-tts/SKILL.md
   ```

2. 修改复制后的 Skill，而不是仓库里的原模板，至少替换：

   - 参考音频绝对路径
   - reference transcript
   - 如有需要，CLI 路径
   - 个人风格预设

3. 不要把使用者的参考音频复制回 Git 仓库。

4. 如果宿主 Agent 的 Skill 目录不是 `~/.agents/skills/`，按该 Agent 的实际 Skill 机制安装。

## 6. 首次验收

不要一上来就跑长文章。

### 阶段 A：短文本

先用一小段普通中文测试：

- 是否成功生成 WAV
- 是否像使用者本人
- 语速是否可接受
- 是否能正常使用参考声音和 reference transcript

### 阶段 B：风格

测试 A–E 菜单是否正常，尤其确认：

- A 自然模式不传 `--cosy-instruct`
- D 个人知识旁白是否符合使用者听感

### 阶段 C：长文本

再选一篇约 2000–4000 字、包含数字、英文缩写、标题和多个自然段的真实文章。

验证：

- Markdown 清理是否正确
- 十进制规范化是否正确
- 80–200 字符语义分块是否完整覆盖全文
- 每块是否执行 Qwen3-ASR 校验
- FAIL 是否只重试当前块
- 初始 seed 是否统一，重试是否切换 seed
- 有未解决 FAIL 时是否阻止最终合并
- 最终 WAV 是否可正常播放

## 7. 当前已验证的关键行为

以下结论来自当前项目的真实测试，不要随意删除：

- 长文本即使命令退出码为 0，也可能静默漏读句子或段落。
- 仅靠把块切小不能彻底避免漏读，所以必须保留 ASR 自检层。
- 整体覆盖率很高也可能漏掉一个完整短句，因此逐句检查是硬要求。
- Qwen3-ASR 对中文完整性检查速度足够快，适合作为本地验收层。
- 专名、缩写可能被 ASR 错识，因此存在 REVIEW 状态，不能把所有差异都判 FAIL。
- 相同文本 + 相同参数 + 相同 seed 可以得到完全相同的 PCM；所以 FAIL 重试必须换 seed。
- 当前长文默认 base seed 为 42，第一次和第二次 FAIL 重试分别使用 43、44。
- 固定 seed 提高可复现性，但不能保证不同语义块之间语速完全一致。

## 8. 日常使用方式

安装完成后，理想交互应该非常简单。

用户只需要提供文本或 Markdown 文件，并说类似：

> 用我的声音生成 TTS。

Agent 应读取 Skill，然后显示 A–E 菜单，等待用户明确选择后再生成。

长文完成后必须报告：

- 最终 WAV 路径
- 总块数
- PASS / REVIEW / FAIL 统计
- 重试块及 seed 序列
- TTS 耗时
- ASR 耗时
- 最终音频时长
- records 诊断目录

## 9. 修改项目的原则

- 优先简单、可解释、可复现，不要过度工程化。
- 先验证问题，再修改 Skill。
- 不因为一次听感问题就增加复杂规则。
- 不自动删除诊断记录。
- 不上传个人声音和生成音频。
- 不用新的模型替换现有模型，除非用户明确要求。
- 修改 Skill 后应同步更新仓库中的 `skill/SKILL.md` 备份，再提交 Git。

## 10. 给另一个 Agent 的最短接管指令

如果用户只把本仓库地址交给你，可以按下面执行：

> 克隆并阅读这个仓库的 README.md、AGENTS.md 和 skill/SKILL.md。先只读检查当前 Mac 环境，不修改系统。确认 speech、ffmpeg、CosyVoice3、Qwen3-ASR 和个人参考声音是否存在。缺少任何安装或模型下载时先告诉用户并取得同意。然后把 skill/SKILL.md 安装到本机 Agent 的 Skill 目录，替换为用户自己的参考音频路径、逐字 transcript 和个人风格预设。先做短文本测试，再做一篇 2000–4000 字长文测试，必须保留每块 ASR 完整性校验和 FAIL 局部重试。不要上传个人声音、生成音频、模型、缓存或秘密到 Git。

完成安装后，再进入日常“用我的声音生成 TTS”的使用流程。
