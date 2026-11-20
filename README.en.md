# Diffusion Gallery

🌐 **English** · [日本語](README.md) · [简体中文](README.zh-CN.md) · [한국어](README.ko.md)

A desktop viewer for organizing and browsing AI-generated images (Stable Diffusion and similar). Formerly known as PNG Metadata Viewer.

Built around PNG files, it brings metadata display, LoRA management, thumbnail browsing, tag aggregation, duplicate detection, and logo overlays together in a single desktop app.

It fits the workflow of creating, organizing, and reviewing AI images — you can work while checking each image's generation prompt and LoRA information.

## Highlights

- 3-pane GUI built on `PySide6`
- Fast scanning, browsing, and thumbnail rendering
- PNG metadata parsing and display
- LoRA / model list management and search
- Advanced search: tags, file name, excluded words, exact phrases, regular expressions, date / resolution / size / Steps / CFG ranges, and search history (the "🔍 Detail" panel)
- Duplicate detection, ratings, Favorites, and tag aggregation
- Snappy folder operation powered by caching
- English / Simplified Chinese / Korean UI (choose it in Settings under the "言語 / Language" tab; applies after restart)

## Tech Points

- Keeps the UI responsive using `QThread` / `QRunnable` / `QThreadPool`
- Flexible layout based on `QSplitter`
- Image processing that combines Pillow and Qt
- Local cache and persisted settings stay usable across restarts
- Implemented as a single script for easy distribution

## Requirements

- Python 3.11+
- Windows 10 / 11

```bash
pip install -r requirements.txt
python png_viewer.py
```

## Optional Environment Variables

- `PNG_VIEWER_DEFAULT_FOLDER`
  - Sets the folder opened first at launch (the variable name is kept from the previous name)

```powershell
$env:PNG_VIEWER_DEFAULT_FOLDER = "D:\Images"
python .\png_viewer.py
```
