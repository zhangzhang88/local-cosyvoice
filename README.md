# local-cosyvoice

一个基于 **CosyVoice3 + speech CLI** 的本地个人声音 TTS 工作流。

目标：把 Markdown 文章转换成自己的声音朗读，并支持长文章分块、时间定位和局部修复。

## 功能

- 个人声音克隆 TTS
- Markdown 转朗读文本
- 数字朗读规范化
- 长文章自动分 Block
- 保存每个 Block WAV
- 生成 `timeline.md` 定位问题位置
- 只重新生成有问题的 Block
- 用户确认后重新合并最终 WAV

## 工作流程

```text
Markdown
  ↓
朗读文本整理
  ↓
Block 分段
  ↓
生成 Block WAV
  ↓
timeline.md
  ↓
合并最终 WAV
```

## 硬件要求

当前主要支持 Apple Silicon Mac：

- 推荐：M1 / M2 / M3 / M4 + 16GB 统一内存
- 更推荐：24GB 以上，适合同时运行 Agent、浏览器和 TTS
- 8GB：可以尝试，但不推荐长文本任务
- Intel Mac：未作为主要目标验证
- 磁盘：建议预留 20GB 以上

已验证机器：Mac mini M4 16GB。

## 安装

### 1. 安装依赖

需要：

- speech CLI
- CosyVoice3
- ffmpeg

例如：

```bash
brew install ffmpeg
```

`speech` 和模型安装请根据当前环境和 CLI 文档配置。

### 2. 准备自己的声音

需要：

- 约 20–30 秒参考录音
- 对应 transcript

不要把个人声音上传到 GitHub。

### 3. 安装 Skill

复制：

```bash
mkdir -p ~/.agents/skills/my-voice-tts
cp skill/SKILL.md ~/.agents/skills/my-voice-tts/SKILL.md
```

然后修改其中的个人路径配置。

## 使用

提供 Markdown 文件，然后让 Agent 调用 Skill。

长文本生成后会得到：

```text
records/
├── block-001/
├── block-002/
├── timeline.md
├── manifest.json
└── reading-text.txt
```

## 如何修复某一段

试听最终 WAV 时，如果发现：

> 4:18 左右读错了

直接告诉 Agent。

它会：

1. 根据 `timeline.md` 找到对应 Block
2. 生成新的 attempt
3. 提供试听
4. 用户确认后更新版本
5. 重新合并最终 WAV

不会重新生成整篇文章。

## 常见问题

### 生成失败

检查：

- speech CLI 是否可用
- ffmpeg 是否安装
- 模型缓存是否存在
- 磁盘空间是否足够

### 速度慢

第一次运行通常需要加载模型。

速度会受到：

- 芯片型号
- 内存
- 后台程序
- 模型版本

影响。

## 隐私

不要提交：

- 个人参考声音
- 生成 WAV
- 模型文件
- API Key / Token
- 本地缓存

## 给 Agent 的接管说明

如果另一个 Agent 接手本仓库：

1. 先阅读 `README.md`
2. 再阅读 `AGENTS.md`
3. 最后阅读 `skill/SKILL.md`

不要直接修改系统环境。
不要上传个人声音。
不要下载额外模型，除非用户明确同意。

详细 Agent 工作规则见 `AGENTS.md`。
