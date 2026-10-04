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

## 硬件要求

本项目当前以 **Apple Silicon Mac** 为主要支持目标。推荐配置：

- **推荐起点：M1 / M2 / M3 / M4 + 16GB 统一内存**
- **更宽裕：24GB 或以上统一内存**，适合同时运行浏览器、Agent、TTS 和 ASR
- **8GB：可以尝试，但不作为推荐配置**，长文本任务时余量更小
- **Intel Mac：不作为本项目当前推荐目标**
- **磁盘空间：至少预留 10GB，可用空间 20GB 以上更合适**

仓库作者当前已验证的参考机器是 **Mac mini M4 16GB**。M1 16GB 级别的 Apple Silicon 机器可作为较低的推荐档位，但不同芯片、散热、后台负载和模型版本都会影响速度，不能保证与 M4 16GB 有相同表现。

本地模型、缓存、分块 WAV 和 ASR 诊断记录都会持续占用磁盘空间，因此磁盘余量通常比单纯看 CPU 更容易被忽略。

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

## Agent 接管与安装

如果你把这个仓库地址交给另一个 Agent，请让它先完整阅读 `AGENTS.md`，里面包含接管、安装、配置、验收和日常操作流程。
