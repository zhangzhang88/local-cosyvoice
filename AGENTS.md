# AGENTS.md

本文件面向接手 `local-cosyvoice` 的 AI Agent。普通用户安装和使用请先看 `README.md`；真正的 TTS 执行规则以 `skill/SKILL.md` 为准。

## 1. 接管顺序

接手后先只读检查，不要立即改文件或安装依赖：

1. 阅读 `README.md`，理解项目定位、安装方式和用户工作流。
2. 阅读 `AGENTS.md`，理解 Agent 约束。
3. 完整阅读 `skill/SKILL.md`，以它作为运行规则来源。
4. 检查 Git 状态、分支、remote 和最近提交。
5. 检查本机是否已有：
   - `speech`
   - `ffmpeg`
   - CosyVoice3 缓存
   - 用户自己的参考音频与准确 transcript
6. 不要假设仓库作者机器上的绝对路径在另一台机器仍然有效。

如果只是接管作者现有机器，优先复用已有环境、缓存和参考声音；不要重复安装或下载。

## 2. 当前工作流

当前默认流程：

- 使用 `speech` CLI + CosyVoice3 做个人声音 TTS。
- Markdown 先转成朗读文本并做数字朗读规范化。
- 长文本切成较小语义 Block。
- 每个 Block 独立生成并持久保留 WAV。
- 默认不使用 ASR。
- 首次每个 Block 使用 seed 42。
- 生成 `timeline.md`，把最终音频时间点映射到 Block。
- 用户人工试听后，如果指出某个时间点或 Block 有问题，只重生成该 Block。
- 第一次人工修复使用 seed 43，第二次使用 seed 44。
- 新 attempt 生成后，先把新 Block WAV 路径给用户试听。
- **只有用户确认“正确”后，才更新 manifest、重算 timeline，并重新合并最终 WAV。**
- 用户说“不正确”时，继续使用下一 seed；不要覆盖旧 attempt。

不要自行把 ASR、二次识别、投票或其他复杂校验重新加回默认流程。

## 3. 文件职责

- `README.md`：面向普通用户，介绍安装、使用、Block/timeline 和常见问题。
- `AGENTS.md`：面向 AI Agent，说明接管、边界和维护规则。
- `skill/SKILL.md`：正式执行规则。运行 TTS 时以它为准。

不要把 README 的安装教程重复塞进 Skill，也不要让 AGENTS 复制整份 README。

## 4. 安装到另一台 Mac 时

当前主要目标是 Apple Silicon Mac。

如果需要帮助新用户安装：

- 先确认硬件、可用磁盘和现有环境。
- 缺少 Homebrew、`speech`、`ffmpeg` 或模型时，先告诉用户会发生什么，再执行安装或下载。
- 不要顺手下载额外模型。
- 默认流程不需要 Qwen3-ASR 或 ForcedAligner。
- 每个用户必须使用自己的参考声音和逐字对应 transcript。

安装 Skill 时，可将仓库里的：

`skill/SKILL.md`

复制到宿主 Agent 的 Skill 目录，例如：

`~/.agents/skills/my-voice-tts/SKILL.md`

然后只修改本机相关配置，例如：

- 参考音频路径
- reference transcript
- CLI 路径
- 用户自己的风格预设

不要把用户的个人声音复制回仓库。

## 5. 首次验收

不要直接从长文章开始。

先验证短文本：

- 能否正常生成 WAV
- 声音是否像用户本人
- 语速是否可接受
- 参考声音与 transcript 是否匹配

然后再做长文本验收：

- Markdown 清理是否正确
- 数字/百分比朗读规范化是否正确
- 80–200 字符左右的语义分块是否完整覆盖全文
- 每个 Block WAV 是否保留并能单独试听
- 首次 seed 是否统一为 42
- `timeline.md` 是否准确映射最终时间点和 Block
- 用户指出问题后，是否只生成对应 Block 的新 attempt
- 新 attempt 是否先给用户试听
- 未经用户确认时，是否保持当前 manifest/timeline/最终 WAV 不变
- 用户确认后，是否正确更新 manifest、重算 timeline 并重新合并

## 6. 日常修复行为

如果用户说：

> 4:18 左右读错了

Agent 应：

1. 读取当前任务的 `timeline.md`。
2. 找到唯一对应 Block。
3. 读取该 Block 的文本和 manifest 状态。
4. 只生成该 Block 的下一 attempt。
5. 给用户新的 Block WAV 路径试听。
6. 等待用户明确确认。

如果用户回复：

> 正确

再：

1. 更新 manifest 当前选用 attempt。
2. 重新读取所有当前选用 Block 的实际时长。
3. 重算完整 `timeline.md`。
4. 重新合并一个新的最终 WAV。
5. 验证最终 WAV 并报告新路径。

如果用户回复：

> 不正确

则使用下一 seed 只重生成同一个 Block，不更新最终版本。

如果时间点正好落在 Block 边界、无法唯一定位，才需要向用户确认。

## 7. 隐私与禁止提交

仓库不能包含：

- 个人参考声音
- 生成的 WAV / MP3
- 模型权重和缓存
- Python/虚拟环境
- Speech Studio 安装包
- 第三方完整源码 checkout
- API Key、Token、密码或其他秘密

提交前检查 `.gitignore` 和 `git status`。

不要因为调试方便就把个人音频或模型临时提交进 Git。

## 8. 修改原则

- 优先简单、可解释、可复现。
- 先确认真实问题，再改 Skill。
- 不因为一次听感问题就增加复杂自动规则。
- 不自动删除用户可能需要试听或回退的 Block attempt。
- 不更换模型，除非用户明确要求。
- 修改正式 Skill 后，要同步仓库里的 `skill/SKILL.md` 备份。
- 修改执行逻辑时，README / AGENTS / Skill 中相关说明要保持一致。
- 不要把文档调整扩散成无关代码重构。

## 9. 给新 Agent 的最短接管指令

> 阅读 README.md、AGENTS.md 和 skill/SKILL.md，先只读检查环境和 Git 状态。确认 speech、ffmpeg、CosyVoice3 缓存和用户自己的参考声音是否存在。缺少安装或模型时先说明并取得用户同意。默认不使用 ASR。长文本保留每个 Block、timeline 和 manifest；用户指出某处错误时，只生成对应 Block 的新 attempt，先让用户试听，确认正确后才更新 manifest、timeline 并重新合并。不要上传个人声音、生成音频、模型、缓存或秘密。
