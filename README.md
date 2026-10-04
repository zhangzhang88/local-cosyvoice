# local-cosyvoice

本仓库用于备份我的本地个人声音 TTS 工作流。

核心流程：

1. 使用 speech CLI + CosyVoice3 做声音克隆与 TTS。
2. Markdown 先清理为朗读文本，再做数字朗读规范化。
3. 长文本按较小语义块生成，每块独立输出 WAV。
4. 使用本地 Qwen3-ASR 0.6B 4bit 对每块做完整性校验。
5. FAIL 块只重试当前块，并切换 seed；PASS / REVIEW 块不重复生成。
6. 所有块满足合并条件后，用 ffmpeg 合成为最终 WAV。

## 主要文件

- skill/SKILL.md：当前正在使用的 Pi / Agent Skill 规则备份。

## 本机依赖

当前工作流依赖本机已经安装或缓存的：

- speech CLI
- CosyVoice3
- Qwen3-ASR 0.6B 4bit
- ffmpeg

这些模型、虚拟环境和第三方源码不会提交到本仓库。

## 隐私

个人参考声音、生成的 WAV、Speech Studio 安装包、ASR 虚拟环境、第三方 whisper.cpp 源码均通过 .gitignore 排除，不上传 GitHub。

如果要在另一台电脑恢复使用，需要重新准备自己的参考声音，并根据新机器路径调整 skill/SKILL.md 中的本地配置。
