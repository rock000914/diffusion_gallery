# Diffusion Gallery

🌐 [English](README.en.md) · **日本語** · [简体中文](README.zh-CN.md) · [한국어](README.ko.md)

Stable Diffusion などの AI 生成画像を整理・閲覧するためのビューワーソフトです（旧名: PNG Metadata Viewer）。

PNG を中心に、メタデータ表示、LoRA / モデル系統の管理、サムネイル一覧、タグ集計、重複検出、ロゴオーバーレイ、そして BGM・ビート同期付きスライドショーまでを 1 つのデスクトップアプリにまとめています。

AI 画像の制作・整理・確認と相性がよく、生成時のプロンプトや LoRA 情報を見ながら運用しやすいのが特徴です。

## Highlights

- `PySide6` ベースの 3 ペイン GUI
- 高速なスキャン、閲覧、サムネイル表示
- PNG メタデータ解析と表示
- LoRA / モデル情報の一覧管理と検索
- モデル系統の自動判定: SD/SDXL/Pony/Illustrious/NoobAI/Flux/Anima など 17 系統をモデル名・生成環境指紋・LoRA 名から分類し、バッジ表示・フィルタ・手動タグ付けで管理
- 拡張検索: タグ・ファイル名・除外語・完全一致フレーズ・正規表現・日付/解像度/サイズ/Steps/CFG 範囲・検索履歴（「🔍詳細」パネル）
- 重複検出、評価、Favorites、タグ集計
- スライドショー & BGM: 拍・フレーム数・速度を調整できる再生。フェード/スライド/ズーム/パルス等の切替アニメ、🎵 BGM と連動し、波形同期・ビート検出・BPM 拍同期に対応
- ロゴオーバーレイ
- キャッシュを活かした軽快なフォルダ運用
- 英語・中国語（簡体）・韓国語対応（環境設定の「言語 / Language」タブで選択。再起動で反映）

## Tech Points

- `QThread` / `QRunnable` / `QThreadPool` を使って UI を止めない設計
- `QSplitter` ベースの柔軟なレイアウト
- Pillow と Qt を組み合わせた画像処理
- `librosa` による波形・ビート解析（窓分割で低メモリ、結果をキャッシュ）
- `QMediaPlayer` と `QGraphicsScene` / `QTimer` によるスライド演出と音声同期
- ローカルキャッシュと永続化設定で再起動後も使いやすい構成
- 単体スクリプトとして配布しやすい実装

## Requirements

- Python 3.11+
- Windows 10 / 11

```bash
pip install -r requirements.txt
python png_viewer.py
```

## Optional Environment Variables

- `PNG_VIEWER_DEFAULT_FOLDER`
  - 起動時に最初に開くフォルダを指定します（環境変数名は旧名のまま据え置き）

```powershell
$env:PNG_VIEWER_DEFAULT_FOLDER = "D:\Images"
python .\png_viewer.py
```
