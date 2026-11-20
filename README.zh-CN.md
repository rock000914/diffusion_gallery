# Diffusion Gallery

🌐 [English](README.en.md) · [日本語](README.md) · **简体中文** · [한국어](README.ko.md)

用于整理和浏览 AI 生成图像（Stable Diffusion 等）的查看器软件（旧名：PNG Metadata Viewer）。

以 PNG 为中心，将元数据显示、LoRA / 模型谱系管理、缩略图列表、标签统计、重复检测、Logo 叠加，以及 BGM·节拍同步的幻灯片播放等功能整合到一个桌面应用中。

非常适合 AI 图像的制作、整理与确认，可在查看生成提示词和 LoRA 信息的同时进行操作。

## 亮点

- 基于 `PySide6` 的三栏式界面
- 快速的扫描、浏览与缩略图显示
- PNG 元数据解析与显示
- LoRA / 模型信息的列表管理与搜索
- 模型谱系自动判定：依据模型名、生成环境指纹与 LoRA 名，自动分类为 SD/SDXL/Pony/Illustrious/NoobAI/Flux/Anima 等 17 个谱系，支持徽章显示、筛选与手动打标签
- 高级搜索：标签、文件名、排除词、精确短语、正则表达式、日期 / 分辨率 / 尺寸 / Steps / CFG 范围、搜索历史（"🔍 详细" 面板）
- 重复检测、评分、收藏、标签统计
- 幻灯片与 BGM：可调节拍 / 帧数 / 速度的播放，切换动画（淡入淡出 / 滑动 / 缩放 / 脉冲等），与 🎵 BGM 联动，支持波形同步、节拍检测与 BPM 拍同步
- Logo 叠加
- 借助缓存实现轻快的文件夹操作
- 支持英语、简体中文、韩语（在设置的 "言語 / Language" 选项卡中选择；重启后生效）

## 技术要点

- 使用 `QThread` / `QRunnable` / `QThreadPool`，界面不会卡顿
- 基于 `QSplitter` 的灵活布局
- 结合 Pillow 与 Qt 的图像处理
- 用 `librosa` 进行波形与节拍分析（分窗加载以节省内存，结果缓存）
- 用 `QMediaPlayer` 和 `QGraphicsScene` / `QTimer` 实现幻灯片动画与音频同步
- 本地缓存与持久化设置，重启后依旧好用
- 以单文件脚本形式实现，便于分发

## 运行要求

- Python 3.11+
- Windows 10 / 11

```bash
pip install -r requirements.txt
python png_viewer.py
```

## 可选环境变量

- `PNG_VIEWER_DEFAULT_FOLDER`
  - 指定启动时首先打开的文件夹（环境变量名沿用旧名）

```powershell
$env:PNG_VIEWER_DEFAULT_FOLDER = "D:\Images"
python .\png_viewer.py
```
