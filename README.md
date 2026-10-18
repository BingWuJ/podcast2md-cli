# podcast2md

**播客链接转章节文字稿的 Windows CLI。下载音频，调用本地 CapsWriter-Offline 转录，按已有章节生成 Markdown。**

**A Windows CLI that turns podcast URLs into chapter-organized Markdown using yt-dlp and local CapsWriter-Offline transcription.**

[中文](#中文) · [English](#english) · [详细使用说明 / Detailed usage](PODCAST2MD.md)

## 中文

### 项目摘要

`podcast2md.py` 是单文件 Python 编排工具，而不是独立语音模型或 CapsWriter-Offline 的完整发行版。

- **下载**：通过本机 `yt-dlp` 下载单集音频并保存元数据。
- **本地转录**：调用已安装的 `start_server.exe`、`start_client.exe`；未运行时尝试启动服务端。
- **章节整理**：优先使用 yt-dlp 章节，其次描述中的时间戳，再尝试网页 HTML 时间轴；均无结果时只生成一章「全文」。
- **交付**：音频、`toc.json`、`.txt/.srt/.json` 和 `chapters.md`。
- **保护原稿**：默认不润色；不调用云端润色模型、不读取 API Key。默认拒绝覆盖非空成品目录。

网页和音频下载需要网络；语音转录使用本地模型。章节按 SRT 条目起点归属，不会在跨章条目内部逐字切分。

### 环境与安装

1. Windows 10/11、Python 3.10+，以及 PATH 中可调用的 `yt-dlp`、`ffmpeg`。
2. 先安装 [CapsWriter-Offline](https://github.com/HaujetZhao/CapsWriter-Offline/releases)，按其说明准备运行库与模型。本项目在配置为 `fun_asr_nano` 的环境中验证；脚本不会修改引擎配置。
3. 克隆或下载本仓库，将 **`podcast2md.py` 复制到 CapsWriter-Offline 安装根目录**，与 `start_server.exe`、`start_client.exe`、`config_server.py` 放在一起。仅克隆本仓库不能直接转录。

```powershell
python -m pip install --upgrade yt-dlp
# FFmpeg 须另行安装并加入 PATH
ffmpeg -version
yt-dlp --version
```

### 使用方式

在 CapsWriter-Offline 安装目录打开 PowerShell：

```powershell
# 将链接替换为要转录的单集页面或音频直链
python .\podcast2md.py "https://www.youtube.com/watch?v=jNQXAC9IVRw"

# 查看所有参数
python .\podcast2md.py --help

# 可选：基础规则清理；不是模型润色，建议先保留默认原稿
python .\podcast2md.py "单集URL" --polish basic

# 长音频可增加等待时长，单位为秒
python .\podcast2md.py "单集URL" --transcribe-timeout 10800
```

成品默认位于安装目录的 `work/<slug>/`，阅读 `chapters.md` 即可。`--force` **会删除同名成品目录后重跑**，确认内容可丢弃时再使用。

也可以把链接发给能够运行本地命令的 AI 助手，说：

> 把这期播客转成文字，按已有章节整理，不要缩成摘要。

由助手替你调用 CLI、检查产物并按要求整理。**自然语言入口由 AI 助手提供，CLI 本身不解析自然语言。**

### 验证与边界

- 已验证短音频和长播客流程，但不代表所有网站或任意时长均可用。公开仓库不包含个人测试记录和转录内容。
- HTML 时间轴提取依赖网页格式，属尽力而为；不保证任意网页通用。
- `--polish basic` 仅做规则删减，完整端到端润色链路尚未验证，不能替代人工校对。
- 不含说话人分离、GUI、MCP 或通用语音引擎安装器。
- 服务端启动后默认保持运行，可从托盘退出；不要结束其他任务的进程。
- 仓库不包含 CapsWriter 程序、模型、运行配置、下载音频或转录内容。上游组件的授权以各自项目为准。

## English

### Summary

`podcast2md.py` is a single-file Python orchestration tool, not a standalone speech model or a full distribution of CapsWriter-Offline.

- **Download:** use the locally installed `yt-dlp` to download an episode and save its metadata.
- **Transcribe locally:** invoke the installed `start_server.exe` and `start_client.exe`, starting the server if needed.
- **Organize chapters:** prefer yt-dlp chapters, then timestamps in the description, then an HTML timeline. If none is available, produce one full-transcript chapter instead of inventing headings.
- **Deliver:** audio, `toc.json`, `.txt/.srt/.json`, and `chapters.md`.
- **Preserve the source:** no polishing by default, no cloud polishing calls, and no API-key reads. Existing nonempty output directories are protected unless explicitly overridden.

Downloading pages and audio requires network access; speech recognition uses local models. Chapters are assigned by each SRT cue's start time, not by splitting individual characters inside a cue that crosses a chapter boundary.

### Prerequisites and installation

1. Windows 10/11, Python 3.10+, and `yt-dlp` and `ffmpeg` available on PATH.
2. Install [CapsWriter-Offline](https://github.com/HaujetZhao/CapsWriter-Offline/releases), including its required runtime and models. The tested environment uses `fun_asr_nano`; this script does not change the engine configuration.
3. Clone or download this repository, then **copy `podcast2md.py` into the CapsWriter-Offline installation root**, alongside `start_server.exe`, `start_client.exe`, and `config_server.py`. Cloning this repository alone is not enough to run transcription.

```powershell
python -m pip install --upgrade yt-dlp
# Install FFmpeg separately and add it to PATH
ffmpeg -version
yt-dlp --version
```

### Usage

Open PowerShell in the CapsWriter-Offline installation directory:

```powershell
# Replace this URL with your episode page or direct audio URL
python .\podcast2md.py "https://www.youtube.com/watch?v=jNQXAC9IVRw"

# Show all options
python .\podcast2md.py --help

# Optional rule-based cleanup, not model-based polishing
python .\podcast2md.py "EPISODE_URL" --polish basic

# Increase the transcription timeout for long recordings (seconds)
python .\podcast2md.py "EPISODE_URL" --transcribe-timeout 10800
```

Outputs are saved under `work/<slug>/` in the installation directory. Open `chapters.md` to read the result. **`--force` deletes the existing episode output directory before rerunning**; use it only if those files can be discarded.

You can also send a URL to an AI assistant with permission to execute local commands:

> Transcribe this episode, organize it using the existing chapters, and do not summarize it.

The assistant can run the CLI, check its outputs, and prepare the result. **Natural-language interaction is provided by the assistant, not by the CLI itself.**

### Validation and limitations

- Short-audio and long-podcast workflows have been exercised, but support is not guaranteed for every site or duration. Personal test records and transcripts are not included in this public repository.
- HTML timeline extraction is format-dependent and best-effort, not a general-purpose parser for every website.
- `--polish basic` only applies deletion-based rules. Its complete end-to-end path has not been validated and it does not replace human proofreading.
- No speaker diarization, GUI, MCP server, or general-purpose speech-engine installer is included.
- A server started by the CLI remains running for reuse. Exit it through its tray menu; do not stop processes belonging to other tasks.
- CapsWriter binaries, models, runtime configuration, downloaded audio, and transcripts are not included. Upstream components remain subject to their respective licenses.

## 文档 / Documentation

- [中英文详细使用说明 / Detailed bilingual usage](PODCAST2MD.md)

公开版本仅包含 CLI、双语说明及文件忽略规则，不包含内部开发记录或私人数据。

The public version includes only the CLI, bilingual documentation, and ignore rules—not internal development records or private data.

## 致谢 / Acknowledgments

[CapsWriter-Offline](https://github.com/HaujetZhao/CapsWriter-Offline) · [yt-dlp](https://github.com/yt-dlp/yt-dlp) · [FFmpeg](https://ffmpeg.org/)
