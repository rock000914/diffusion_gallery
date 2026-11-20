# CHANGELOG

PNG/画像メタデータビューア (PySide6版) — 更新履歴

---

## [v2.32.2] 2026-11-21

### バグ修正
- **最大化ウィンドウをタイトルバードラッグで解除すると縦幅が最小サイズに潰れる**問題を修正（Windows 実機で再現確認: 1600x900 画像で復元後 `936×124`、修正後 `1340×864`）。
  - **FullViewer**: `_start_maximized=True` 起動時に `showMaximized()` を呼ぶ前に、画像アスペクトと画面利用率 75% から妥当な「復元サイズ」を計算して `resize()` + `move()` + `_normal_size` 登録を追加 [png_viewer.py:4911-4935](file:///N:/マイドライブ/png_viwer/png_viewer.py#L4911-L4935)。Qt はこの事前ジオメトリを「normal geometry」として保持し、OS のドラッグ解除時にそこへ戻る。
  - **MainWindow**: `_restore_geometry()` で `restoreGeometry()` が失敗した場合（`geometry` が未保存 or 壊れている）に、`showMaximized()` 前に画面比 60%（最低 900×600）の復元サイズを登録するガードを追加 [png_viewer.py:9456-9476](file:///N:/マイドライブ/png_viwer/png_viewer.py#L9456-L9476)。
  - 両箇所とも、事前にウィンドウが表示済みで normal geometry が確立されている経路（`_toggle_fullscreen` など）は影響なし。

### 検証
- `.tmp/smoke_v232_2.py`: `windows` プラットフォームで実ウィンドウを作り、`showMaximized()` → `showNormal()` 後の geometry が width>=640 かつ height>=480 を確認。修正前: `h=124` で FAIL、修正後: `h=864` で PASS。
- 既存 `.tmp/smoke_v232.py` (音量 OSD) も引き続き `SMOKE_OK`。

---

## [v2.32.1] 2026-11-21

### バグ修正
- **没入モードで音量 OSD が見えなかった問題を修正**。v2.32 の音量ホットキー表示は `info_lbl`（ツールバー内）に書き込んでいたが、Space/F11 で `_toolbar.hide()` されると通知自体も消える。`.hide()` 前に `_show_image` の `setText` → `hide` 復元タイマーも走るため、ユーザーは音量変更を検知できなかった。
  - `FullViewer.__init__` の `_scene_widget` 直下に浮遊 `QLabel _vol_osd`（`WA_TransparentForMouseEvents`、暗色半透明背景+シアン文字）を追加。`_vol_osd_timer` (2 秒, singleShot) で自動 hide。
  - `_flash_volume(kind, value)` は OSD の `setText` → `_update_osd_pos` → `show/raise_` → `timer.start(2000)` の流れに統一。連続キー入力時はタイマーが再スタートし、2 秒表示が伸びる。OSD 未作成時は info_lbl にフォールバック。
  - `_update_osd_pos()` はシーン左下 12px マージン、極小サイズ時のみ右端に収める。viewport の `QEvent.Type.Resize` を処理する `eventFilter` 内で再配置。
  - info_lbl は画像情報のみに専念し、音量変更では触らない（二重表示回避）。

### 内部
- `APP_VERSION` を `v2.32` → `v2.32.1` へパッチ bumps。
- `.tmp/smoke_v232.py` に OSD 表示 + タイマー起動 + 連続押下時のテキスト切替アサーションを追加。

---

## [v2.32] 2026-11-21

### 新機能
- **ビュアーにズーム率を % 表示**。`FullViewer._show_image` の info_lbl が `zoom:1.0x` 形式だったが、`ズーム:100%`（`Te('ズーム:')` + `self._zoom*100:.0f`）へ変更 [png_viewer.py:5146](file:///N:/マイドライブ/png_viwer/png_viewer.py#L5146)。等倍=100%、2倍=200% が直感的に分かる。
- **音量調整ホットキー**を `keyPressEvent` に追加 [png_viewer.py:5583-5587](file:///N:/マイドライブ/png_viwer/png_viewer.py#L5583-L5587):
  - `[` / `]` = BGM 音量 −5% / +5%
  - `{` / `}` = 音声・セリフ 音量 −5% / +5%
  - 変更後は `🔊 BGM: 75%` 形式で info_lbl に 2 秒表示（`_flash_volume`）。
- **ツールバーに 🔊 / ⚙ ボタン**を追加 [png_viewer.py:4804-4808](file:///N:/マイドライブ/png_viwer/png_viewer.py#L4804-L4808)。
  - 🔊 = 音量・速度調整ポップアップ（`_build_volume_popup`）。BGM / 音声・セリフ それぞれに 0-100 スライダーと 0.5-2.0 速度スピン。設定タブを開かなくても一時的な微調整が可能。`Qt.WindowType.Popup` の QFrame で、`_build_adv_search_popup` と同じパターン。
  - ⚙ = 環境設定ダイアログを開く (`mw._open_settings()`)。
  - `HAS_MULTIMEDIA=False`（PySide6.QtMultimedia 無し環境）では 🔊 は自動で無効化 + ツールチップで理由表示。

### 内部
- `FullViewer._set_volume(kind, value)` / `_set_speed(kind, rate)` / `_get_volume(kind)` / `_adjust_bgm_volume(delta)` / `_adjust_voice_volume(delta)` / `_flash_volume(kind, value)` / `_open_settings_from_viewer()` / `_open_volume_popup()` / `_build_volume_popup()` メソッドを新設。音量・速度は `ss_bgm_volume` / `ss_voice_volume` / `ss_bgm_speed` / `ss_voice_speed` の 4 キーで settings.json に保存（既存の `save_cfg()` を利用）。
- i18n テーブルに 9 キー追加（`BGM` / `環境設定` / `ズーム:` / `※ ホットキー: ...` / `速度` / `音声` / `音声・セリフ` / `音量` / `音量・速度調整`）。英中韓訳付き、総エントリ数 618 → 627。
- `APP_VERSION` を `v2.31` → `v2.32` へバンプ。

### 検証
- `.tmp/smoke_v232.py`: offscreen で FullViewer を作り、🔊 ポップアップ初期値（80/100/1.0/1.0）、スライダー/スピン値変更 → cfg 反映、`[`/`{` ホットキー音量 ±5 と info_lbl 表示、⚙ で `mw._open_settings` 呼び出し、全てパス。

---

## [v2.31] 2026-11-21

### リファクタリング
- **`AUDIO_EXTS` 二重定義をモジュールレベル定数に集約**。v2.29/v2.30 で `FullViewer._AUDIO_EXTS`（クラス属性）と `SettingsDialog._on_auto_detect_bpm` 内のハードコードタプルに分散していた音声拡張子リストを、`SUPPORTED_EXTS` の直下 [png_viewer.py:1017-1019](file:///N:/マイドライブ/png_viwer/png_viewer.py#L1017-L1019) に定義した単一の `AUDIO_EXTS = (".mp3", ".wav", ".flac", ".ogg", ".m4a", ".aac", ".opus")` に統一。
  - `FullViewer._scan_audio_folder` の `self._AUDIO_EXTS` 参照 → `AUDIO_EXTS` [png_viewer.py:5352](file:///N:/マイドライブ/png_viwer/png_viewer.py#L5352)。
  - `SettingsDialog._on_auto_detect_bpm` のハードコード → `AUDIO_EXTS` [png_viewer.py:6656](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6656)。
  - 副次的に `_scan_audio_folder` は `self._AUDIO_EXTS` を参照しなくなったので、クラス属性に依存しない純粋なスキャン関数になった。
  - 将来 `.opus` や新形式を 1 箇所で追加でき、片方だけ追いつかないバグを防止。
- `APP_VERSION` を `v2.30` → `v2.31` へバンプ。

### 検証
- `python -m py_compile` OK。`grep _AUDIO_EXTS` で残存参照 0 件。
- `pv.AUDIO_EXTS` が 7 拡張子のタプルであること、`hasattr(FullViewer, '_AUDIO_EXTS')` が False であることを確認。
- `_scan_audio_folder(r'E:\eof1.8RC12')` → 65 ファイル検出（.mp3/.ogg/.wav の混合）、`_scan_audio_folder(r'J:\motorvcdatafaile\Aizawatomo')` → 3 wav 検出。
- 既存の v2.30 BPM 検出（中央値 123 BPM / 62曲）と整合（65 ファイル中 3 件は detect_bpm が None を返したため検出数 62）。

### 補足: ボリューム調整
- v2.29 で環境設定 > 🎵 音楽 タブに音量スライダーを実装済み（BGM 0-100 / 音声 0-100、`QAudioOutput.setVolume(0.0-1.0)` に変換して適用）。
- スライドショー実行中にツールバーから素早く音量調整する機能は未実装（ご希望あれば次回）。

---

## [v2.30] 2026-11-21

### 追加機能: BPM 自動検出（Phase 2）
ユーザー指定「ビートは自動検出と手動で」に対応。Phase 1 の BPM 手動入力に加え、**librosa による自動 BPM 検出**を追加。追加 pip install 不要（librosa 0.11.0 / soundfile / numpy / scipy は環境導入済み）。

#### 実装
- **librosa 遅延 import** [png_viewer.py:200-206](file:///N:/マイドライブ/png_viwer/png_viewer.py#L200-L206): 起動時に `import librosa` で存在チェックのみ（HAS_LIBROSA フラグ）。実際の librosa / numpy 呼び出しは `detect_bpm()` 内で遅延 import。librosa 自体の import は 5-15 秒かかるため、初回検出時のみ待たせる。
- **`detect_bpm(path, sample_sec=30)`** [png_viewer.py:6312-6354](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6312-L6354): 先頭 30 秒を sr=22050 mono で読み、`librosa.beat.beat_track` で BPM 推定。89 分メドレーでも数秒で終わる。
- **`bpm_cache.json`**（APP_DIR 直下）: `{path: {mtime, size, bpm}}` 形式。mtime + size 一致ならファイルキャッシュから即返す（0.01 秒）。原子的書き出しは tmp + `os.replace`。
- **`_BpmDetectWorker` (QThread)** [png_viewer.py:6357-6385](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6357-L6385): BGM フォルダ内の全曲を順に解析。`progress(done, total, current_path)` / `finished_all(bpms, median)` / `failed(msg)` シグナルで UI に通知。`cancel()` で中断可能。
- **`np_median(values)`** [png_viewer.py:6388-6396](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6388-L6396): numpy 非依存の中央値関数（偶数件は中央 2 件の平均）。
- **SettingsDialog 音楽タブ拡張** [png_viewer.py:6454-6471](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6454-L6471): BPM スピンの行を QHBoxLayout に変更、`🎵 自動検出` ボタンと結果ラベルを追加。`HAS_LIBROSA=False` 時はボタン無効化 + ツールチップで理由表示。
- **解析ハンドラ群** [png_viewer.py:6535-6585](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6535-L6585):
  - `_on_auto_detect_bpm()`: BGM フォルダ内の音楽ファイルを列挙、ワーカ起動、ボタン無効化、結果ラベルに「解析中 0/N...」表示。
  - `_on_bpm_progress(done, total, current_path)`: 進捗更新、ファイル名は 30 文字で切り詰め。
  - `_on_bpm_done(bpms, median)`: 中央値 BPM をスピンに書き込み、ラベルに「中央値 X BPM / N曲検出」表示。
  - `_on_bpm_failed(msg)`: ラベルに警告表示。
- **ワーカ安全停止**: `_save()` と `reject()` で `cancel() + wait(3000)`。ダイアログ破棄時のクラッシュを防止。
- **i18n**: 12 キー追加、テーブル 606 → **618 キー**。en/zh/ko 翻訳込み。プレースホルダ `{0}` `{1}` `{2}` は各言語で同じ位置に維持。
- `APP_VERSION` を `v2.29` → `v2.30` へバンプ。

### サンプル検証（`E:\eof1.8RC12\y2mate.com - 高確率で流れる定番人気EDMクラブミュージックメドレー.mp3`）
- 89 分 EDM メドレー、128kbps MP3、44100Hz stereo。
- QMediaPlayer 統合: `LoadedMedia` 状態、duration 5348310ms 正確、`play()` で `PlayingState` / position 941ms、error なし。
- `detect_bpm()` 単曲: 初回 5.88 秒（librosa import 含む）、キャッシュ後 0.0001 秒、ファイルキャッシュ復元 0.0128 秒。検出値 129 BPM。
- 地点別推定: 冒頭 129.2 BPM / 45分時点 129.2 BPM / 89分時点 107.7 BPM（メドレーなので曲ごとに異なる）。
- フォルダ一括検出: `E:\eof1.8RC12\` の 62 曲を `_BpmDetectWorker` で解析、**中央値 123 BPM / 62曲検出**、スピンに自動設定。

### Phase 3（将来）
- 曲ごとの BPM を検出してキャッシュし、曲切り替わり時に `_beat_interval_ms()` を動的再計算（メドレー対応）。
- librosa によるオンセット検出で、拍位置に正確に同期する位相合わせ。

### 検証
- `python -m py_compile` OK。`ast` で `_I18N_TABLE` 618 キー・新規 12 キーの存在を確認。
- offscreen で `SettingsDialog` 音楽タブ構築、`🎵 自動検出` ボタン・結果ラベル・HAS_LIBROSA フラグ表示を確認。
- offscreen で `_on_auto_detect_bpm()` → `_BpmDetectWorker` 起動 → `finished_all` シグナル受信 → 中央値 123 BPM がスピンに反映、ラベルに「中央値 123 BPM / 62曲検出」表示されることを確認。

---

## [v2.29] 2026-11-21

### 追加機能: スライドショー音楽再生（Phase 1）
ユーザー指定「スライドショーに音楽再生機能をつける。設定に音楽再生用のフォルダ選択できるようにする。2つ選択できるようにして音楽を2つ同時に流せるようにする。同時、片方のみの再生、設定でそれぞれの再生スピードを変更できるようにしたい」に対応。Phase 1 では **BPM 手動入力 + 拍同期** までを実装（自動 BPM 検出は Phase 2）。

#### 設計
- **2系統の役割分担**:
  - **BGM系統**: 拍同期の基準。フォルダ内の音楽をランダム順でループ再生。
  - **音声・セリフ系統**: 拍同期なし、独立再生。効果音・ナレーション用。
- **追加依存なし**: `PySide6.QtMultimedia`（PySide6 同梱）の `QMediaPlayer` + `QAudioOutput` を 2 系統分使用。
- **HAS_MULTIMEDIA フォールバック**: QtMultimedia が import できない環境では音楽機能を無効化し、ビューア本体は動く。設定タブ・ツールバーの 🎵 ボタンはグレーアウト＋警告表示。

#### 実装
- **import 部** [png_viewer.py:190-198](file:///N:/マイドライブ/png_viwer/png_viewer.py#L190-L198): `PySide6.QtMultimedia` を try/except で import、`HAS_MULTIMEDIA` フラグ設定。
- **SettingsDialog タブ6「🎵 音楽」** [png_viewer.py:6170-6240](file:///N:/マイドライブ/png_viwer/png_viewer.py#L6170-L6240):
  - BGMグループ: フォルダ選択 / ON-OFF / 音量スライダー (0-100) / 再生速度スピン (0.50-2.00) / BPM スピン (0-300, 0=無効) / 拍数コンボ (1/2/4/8)。
  - 音声グループ: フォルダ選択 / ON-OFF / 音量 / 再生速度。
  - `HAS_MULTIMEDIA=False` 時は警告のみ表示。
  - `_browse_ss_bgm` / `_browse_ss_voice` でフォルダ選択ダイアログ。
  - `_save()` で cfg に `ss_bgm_folder / ss_bgm_enabled / ss_bgm_volume / ss_bgm_speed / ss_bgm_bpm / ss_bgm_beat / ss_voice_folder / ss_voice_enabled / ss_voice_volume / ss_voice_speed` を保存。
- **FullViewer 音楽エンジン** [png_viewer.py:5264-5410](file:///N:/マイドライブ/png_viwer/png_viewer.py#L5264-L5410):
  - `_init_audio_state()` — cfg からフォルダをスキャンしてランダムキュー構築。
  - `_scan_audio_folder()` — `.mp3/.wav/.flac/.ogg/.m4a/.aac/.opus` を列挙、`random.shuffle`。
  - `_ensure_audio_players()` — 初回 ON 時に QMediaPlayer ×2 + QAudioOutput ×2 を遅延作成、音量・速度適用。
  - `_on_bgm_status` / `_on_voice_status` — `EndOfMedia` で次曲へ自動送り（キュー末尾から先頭へループ）。
  - `_toggle_music()` — 🎵 ボタンハンドラ。設定変更後にキュー再スキャン。有効な系統にファイルが無い場合は警告。
  - `_stop_music()` — 両系統停止。
  - `_beat_interval_ms()` — 拍同期間隔計算: `(60000 / BPM) × 拍数 / 再生速度` ms。BGM OFF / BPM=0 / 音楽 OFF なら None 返し、呼び出し側が固定間隔にフォールバック。
- **ツールバー 🎵 ボタン** [png_viewer.py:4725-4729](file:///N:/マイドライブ/png_viwer/png_viewer.py#L4725-L4729): SS ボタンの隣に配置。`HAS_MULTIMEDIA=False` 時は `setEnabled(False)`。
- **拍同期と既存のフレーム数スピン併存** [png_viewer.py:5414-5424](file:///N:/マイドライブ/png_viwer/png_viewer.py#L5414-L5424): `_start_ss` で `_beat_interval_ms()` が非 None なら拍モード、None ならフレーム数固定間隔にフォールバック。
- **closeEvent で音楽停止** [png_viewer.py:5187-5191](file:///N:/マイドライブ/png_viwer/png_viewer.py#L5187-L5191): ビュア閉じ時に両系統を確実に停止。
- **i18n**: 21 キー追加、テーブル 585 → **606 キー**。en/zh/ko 翻訳込み。
- `APP_VERSION` を `v2.28` → `v2.29` へバンプ。

### 換算早見（拍同期、BGM 再生速度 1.0 倍時）
| BPM | 1拍 | 2拍 | 4拍 | 8拍 |
|---|---|---|---|---|
| 60 | 1000ms | 2000ms | 4000ms | 8000ms |
| 90 | 667ms | 1333ms | 2667ms | 5333ms |
| 120 | 500ms | 1000ms | 2000ms | 4000ms |
| 128 | 469ms | 938ms | 1875ms | 3750ms |
| 140 | 429ms | 857ms | 1714ms | 3429ms |

### Phase 2（次回予定）
- **BPM 自動検出**: numpy + soundfile で onsets + autocorrelation、検出値を「推奨 BPM」として UI 表示、手動で上書き可能。解析結果キャッシュ。

### 検証
- `python -m py_compile` OK。`ast` で `_I18N_TABLE` 606 キー・新規 21 キーの存在を確認。
- offscreen で `SettingsDialog` 構築 → 7 タブ、音楽タブ index 6、BPM 140 / 拍数 4 が `_save()` で cfg に反映されることを確認。
- offscreen で `FullViewer._init_audio_state` → 一時フォルダの .mp3 3件 / .wav 2件 / .txt 1件をスキャンし、.txt を除外して BGM=3 / 音声=2 のキューを構築。
- `_beat_interval_ms()` の換算を境界値で検証: 120bpm×4拍=2000ms / rate=0.5 で 4000ms / bpm=0 で None（固定間隔フォールバック）。
- `HAS_MULTIMEDIA=False` 経路は import 部 try/except でカバー、設定タブとツールバーの無効化ロジックで UX 担保。

---

## [v2.28] 2026-11-21

### 機能変更
- **スライドショー送り間隔を「秒」から「フレーム数」に変更**。ユーザー指定「送り時間は1秒でも長い可能性があるのでフレーム数にして」に対応。
  - 基準 fps は **60fps 固定**（1F ≈ 16.67ms）。UI 上はフレーム数のみを表示し、ツールチップに換算式を記載。
  - スピン範囲: `1〜30 秒` → **`1〜1800 フレーム`**（60fps で 30 秒相当）。デフォルト値: `3` → **`180`**（=3 秒、従来動作を踏襲）。
  - `_start_ss()` の msec 換算: `max(500, value * 1000)` → **`max(16, int(value * 1000 / 60))`**。下限 16ms は 1F 相当で、Qt タイマーの現実的な最小間隔。
  - ツールバーの単位ラベル `秒` → **`F`**（i18n キー `'F'` を追加、en/zh/ko も `F` のまま）。
  - スピンにツールチップ追加: 「送り間隔 (フレーム, 60fps 基準) / 1F≈16.67ms / 60F=1秒 / 180F=3秒」（i18n キー追加、en/zh/ko 翻訳あり）。
  - 4 桁値「1800」が lineEdit に収まるようスピン幅を `72 → 80 px` に拡大（`lineEdit().width()` 53px、テキスト幅 48px で余白確保）。
- `APP_VERSION` を `v2.27` → `v2.28` へバンプ。
- i18n テーブル: 583 → **585 キー**（`F` と送り間隔ツールチップを追加）。

### 換算早見（60fps 基準）
| F | msec | 備考 |
|---|---|---|
| 1 | 16 | 約 60fps（タイマー下限） |
| 15 | 250 | 0.25 秒 |
| 30 | 500 | 0.5 秒 |
| 60 | 1000 | 1 秒 |
| 180 | 3000 | 3 秒（デフォルト） |
| 1800 | 30000 | 30 秒（上限） |

### 検証
- `python -m py_compile` OK。`ast` で `_I18N_TABLE` 585 キー・`F` と送り間隔キーの存在を確認。
- offscreen でスピン幅 80px・値 1800 における lineEdit 幅 53px / テキスト幅 48px を実測し、4 桁が重ならず収まることを確認。
- 換算式を全境界値（1F/15F/60F/180F/1800F）で検証、意図通り msec 化されることを確認。
- `_start_ss` 以外のスライドショー起動経路（`_toggle_ss`、`QTimer.singleShot(350, self._start_ss)` の slideshow 引数）は変更なし。

---

## [v2.27] 2026-11-21

### 不具合修正
- **ビューアのスライドショー間隔スピンの数字とアップ/ダウン矢印が重なる** 問題を修正。
  全画面ビューア上部ツールバーの `▶ SS` 横の秒数スピンが `setFixedWidth(48)` + 既定サブコントロール配置だったため、Windows 上の Qt スタイルで右側に寄る up/down ボタンと `3` などの数字が重なって見えていた。
  修正内容（[png_viewer.py:4703-4718](file:///absolute/path/N:/マイドライブ/png_viwer/png_viewer.py#L4703-L4718)）:
  - 幅を `48 → 72 px` に拡大し、数字側余白とボタン側余白を確保。
  - `QSpinBox::up-button` / `QSpinBox::down-button` を `subcontrol-origin:border` + `subcontrol-position: top right / bottom right` で明示配置（幅 16px、上下それぞれ 13px 段積み）。
  - 本体側に `padding-left:8px`、ボタン側に `border-left` と hover 背景を指定し、数字領域と操作矢印領域を視覚的に分離。
  - `setFixedHeight(28)` と `border-radius:4px` 以外の既存スタイルは維持（ツールバーの高さ 38px に対する収まりは変更なし）。
- `APP_VERSION` を `v2.26` → `v2.27` へバンプ。

### セキュリティガード拡張
- `security_check.py` に **行内 allow ディレクティブ** `# security-check: allow <理由>` を追加。正当な既存コードがルールに誤検知されたとき、その行だけ検出をスキップできる（理由の記載を必須にし、ファイル全体の無効化は不可）。
- 旧メタキャッシュ移行処理 [png_viewer.py:2714](file:///absolute/path/N:/マイドライブ/png_viwer/png_viewer.py#L2714) の `pickle.load(f)` に同ディレクティブを付与（自アプリが書き出した旧キャッシュのみ、読了後即 unlink）。
- `SECURITY.md` の整合性ガード節に追記。

### 検証
- `ast.parse` + `python -m py_compile` で構文検査 OK（png_viewer.py / security_check.py 双方）。
- offscreen で修正後 QSpinBox をレンダリング、`lineEdit().geometry()` が `x=9 width=45`（右端 54px）となり、ボタン領域 x=56〜72 と重ならないことを確認。
- UI 構造・シグナル接続・スライドショー起動ロジック（`_start_ss` / `_toggle_ss` / `_ss_job`）は変更していない。

---

## [v2.26] 2026-11-11

ユーザー指定「UIの多言語化。英語・中国語・韓国語あたりで設定から言語を選べるようにしてほしい」に対応。

### 追加: UIの多言語対応（i18n）
- 表示言語を **日本語 / English / 中文（简体）/ 한국어** の4つから選択できるようにした。
  `環境設定`（Ctrl+,）の **「言語 / Language」タブ** で切り替える。設定は `settings.json` の `language` キーに保存され、`_save()` から `set_ui_language()` で反映（以降に作られるダイアログから新言語になる）。
  完全な反映（メイン画面・ステータスバー・ツールチップ等）は **アプリ再起動後** 。
- UI文字列に `T("...")`（通常文字列）と f-string 内の `Te("...")`（マーカー方式の `{T()}` 相当）を適用した。
- 翻訳テーブル `_I18N_TABLE` は **583キー**。英語・中国語（簡体字）・韓国語を収録。未登録の文字列は **日本語のまま** 表示されるフォールバック付き。
- `"""docstring"""`（開発者向けドキュメント）とコメント・正規表現パターン等は翻訳対象外。
- バックアップを取得（`.tmp_backup_png_viewer_20261111-125313_i18n.py` と Google ドライブ直下の `png_viewer_backup_20261111-125313.py`）。UI・挙動・見た目は変更していない。
- `APP_VERSION` を `v2.25` → `v2.26` へバンプ。

### 検証
- rewrite後のソースを `ast.parse` + `py_compile` で構文検査。
- 書き換え済みファイルの全 `T(...)` / `Te(...)` 呼び出し（507 / 202 か所）の引数が `_I18N_TABLE` のキーに解決されることを AST で確認。未翻訳UI文字列の残存 **0 件**。
- offscreen で `SettingsDialog` を構築し、6タブ・言語タブが index 0・コンボボックスが4言語・`_save()` で `cfg["language"]` が更新されることを確認。

---

## [v2.25] 2026-11-11

- 全画面から最大化へ戻す途中で、一度小さな通常ウィンドウになってから拡大する動きを修正。QtのWindows実装が通常位置を経由する際の移動・縮小命令だけを、復帰処理中に抑制する。
- 全画面化前のWindowsウィンドウ配置を保存・復元し、その後に最大化を解除しても元の通常位置・サイズへ戻るようにした。Windows全体のアニメーション設定は変更しない。
- Windows実ウィンドウで **25テスト成功**。起動時最大化・手動最大化の両方で途中のサイズ変化を監視し、一時的な縮小がないことと、その後の通常表示復元を確認。
- offscreenは **24テスト成功・Windows固有の中間サイズ監視1件は対象外**。Space／F11／Escの往復、倍率、操作バー、既存の検索・サムネイル動作も維持。

---

## [v2.24] 2026-11-11

- 画像をダブルクリックで開き、Spaceで全画面化・解除すると画像が極端に小さくなる問題を修正。Windowsで最大化へ戻した後に操作バーを再表示すると、画像エリアが最小サイズへ縮んでいたため、操作バーを先に復元する順序へ変更。
- 通常ウィンドウの位置・サイズを全画面化の直前に毎回保存し、手動で移動・リサイズした後も正しく復元する。
- `.tmp/v223_regression.py` に全画面往復の回帰テストを追加。通常・最大化・リサイズ＋ズームの3条件で、Space／F11／Escによる解除後の位置・サイズ・倍率・表示モード・操作バーを確認。
- Windows実ウィンドウと offscreen の両方で **24テスト成功**。修正前に再現した2条件の失敗が解消し、全画面化前後のスクリーンショットも確認済み。

---

## [v2.23] 2026-11-11

### 不具合修正
- 保存済みレイアウト B/C が起動時の旧ID変換で A に戻る問題を修正。
- メタデータキャッシュから外れた画像が検索・評価フィルタから漏れる問題を修正。メタデータが必要な検索と解像度順ではバックグラウンドで不足分を取得する。
- 更新された画像のメタデータを先読みし直さない問題、モデル一覧更新で検索条件が解除されても結果が更新されない問題を修正。
- 破損・旧形式のサムネイルキャッシュを同じ読み込み内で再生成。RAWX の寸法・チャンネル・行幅・データ長を検証し、グレースケール画像の保存形式も修正。
- サイズ変更前などの古いサムネイル結果が最新の表示を上書きする問題と、ウィンドウの高さだけを広げた際の描画漏れを修正。
- フッターのサイズバーが余分に横へ伸びる問題と、モデル検索ボタンの未接続信号を切断する警告を修正。

### 最適化
- キャンセル済みワーカーを終了後に解放し、保持された検索スナップショットの蓄積を防止。
- キャッシュ保存を画像読み込みタスク内のアトミック書き込みへ変更し、低優先度の保存ジョブと画像バッファの滞留を解消。
- ファイル名・評価のみの絞り込みでは不要なメタデータ読み込みを回避。同じサムネイルサイズの再設定でもグリッドを再構築しない。
- 拡大ビューアの初期サイズ計算を画像寸法の取得だけで行い、GUIスレッドでの重複した画像デコードを削減。

### 検証
- `.tmp/v223_regression.py`: Windows実ウィンドウと offscreen の両方で **23テスト成功、失敗・未処理例外なし**。
- 検索・選択・拡大表示・画像送り、パネル開閉、サイズ変更、レイアウト切り替え、キャッシュ破損・RGB/RGBA往復・更新競合を確認。
- テスト用画像・設定・キャッシュは一時ディレクトリへ分離。既存の画像や通常利用の設定は変更していない。
- ソースを構文検査し、Windows実画面のスクリーンショットも確認。実画像コレクション全体での速度計測は未実施。

---

## [v2.04]〜[v2.22] のセクションは欠落しています

> v2.04〜v2.22（約19バージョン分）は、本作業中の誤った上書きでローカルから失われました。ローカルの git・GitHub（`rock000914/png_viwer` の CHANGELOG は **v2.03 が最終コミット**で以降は未更新）・`.tmp/` のいずれにも v2.04 以降の本文はありません。**v2.23〜v2.26 は今回のセッション記録から原文どおり復元済み**です。
>
> 取り戻し方: Google ドライブで `png_viwer` フォルダ → `CHANGELOG.md` を右クリック →「バージョン履歴」→ 上書き前（v2.22 まで揃った版）を開いて復元 → 私に渡してください。v2.23〜v2.26 と重複しないよう統合します。

---

## [v2.03] 2026-04-07 (Claude Sonnet 4.6)

### 新機能
- **タイトルロゴ オーバーレイ機能**を実装 (`OverlayManager` クラスを新設)。設定は `overlay_settings.json` に永続化。
  - グリッド右クリックメニュー「🎨 ロゴオーバーレイ」サブメニューを追加:
    - 「ロゴを設定...」: PNG を選択して1件または複数件に一括適用
    - 「位置・サイズ調整...」: X/Y位置(0.0〜1.0)・スケールをスピンボックスで調整
    - 「ロゴを削除」: 選択件数分のオーバーレイ設定を削除
    - 「合成画像を出力...」: 元ファイルを変更せず、合成済みPNGを別途保存
  - ツールメニュー「🎨 ロゴ/タグ連動設定」を追加（タグ→ロゴの紐付けテーブル管理）
  - `_compose_overlay()` メソッド: QPixmap をオフスクリーンで合成し全プレビュー表示に反映
  - `_sel_image()` でタグ連動 auto_apply を実行

---

## [v2.02] 2026-04-07 (Claude Sonnet 4.6)

### 新機能
- **メタキャッシュの永続化**: `MetaCache` に `save_to_disk` / `load_from_disk` メソッドを追加。pickle形式で `%LOCALAPPDATA%/png_viewer/meta_cache.pkl` に保存。起動時に自動ロード、closeEvent で自動保存。`META.get()` に mtime チェックを追加し外部更新ファイルのキャッシュを自動無効化。ツールメニューに「🗑 メタキャッシュを初期化」を追加。
- **全サムネ事前生成**: `_prebuild_all_thumbs` メソッドを追加。ツールメニュー「🖼 全サムネイルを一括生成」から実行可能。キャッシュ済みをスキップしてディスクキャッシュがないものだけ ThumbLoader に一括投入。

### 高速化
- **PreloadWorker のキャッシュ済みスキップ最適化**: ScanWorker が収集した mtime と MetaCache の mtime を照合し、一致するファイルはメタ解析をスキップ。2回目以降のフォルダスキャンで未変更ファイルが即完了するようになる。

---

## [v2.01] 2026-04-07 (Claude Sonnet 4.6)

### UI修正
- **LoRA分析ウィンドウの検索欄の幅を倍に拡大**: `setFixedWidth(240)` → `setFixedWidth(480)` に変更。より長いLoRA名でも切れずに表示・入力できるようになった。

---

## [v2.00] 2026-04-07 (Claude Sonnet 4.6)

### 重大バグ修正
- **サブフォルダ読み込み時にファイル数が多いと絞り込みが機能しない問題**を修正:
  - 根本原因①: `MetaCache._MAX_META_ENTRIES = 5000` が小さすぎた → `50000` に拡大
  - 根本原因②: `FilterWorker` / `TagCountWorker` が未キャッシュファイルを同期パースしていた → 同期パース廃止、未キャッシュはスキップ
  - 根本原因③: `_apply_filter()` が PreloadWorker 完了前に走るタイミング問題 → `_on_preload_progress()` に500件ごとのフィルター再適用ロジックを追加
  - `_on_preload_done()` でもフィルター条件が設定されていれば再適用（全件確定後の最終反映）

---

## [v1.99] 2026-04-07 (Claude Sonnet 4.6)

### 新機能
- **モデル名クリックでツールバーのモデル検索コンボに反映**: 右パネルの「モデル:」欄をクリックすると `model_cb` にセットされ即座にフィルター適用。`MetaPanel` に `model_search_requested` シグナルを新設。フェードトースト「🔍 モデルで絞り込み: <名前>」を表示。

---

## [v1.98] 2026-04-07 (Claude Sonnet 4.6)

### 新機能
- **Positive/Negativeヘッダーに重複タグ数バッジと2種コピーボタンを追加**:
  - 重複タグが1件以上ある場合、ヘッダー右端に「重複N」バッジ（黄色）を表示
  - 📋✨ (dedup): 重複除去済みテキストをコピー / 📋 (plain): 元テキストをそのままコピー

### UI修正
- **並び替えコンボボックス（QComboBox）の高さ**をボタンに揃える: `setFixedHeight(26)` を追加。

---

## [v1.97] 2026-04-06 (Claude Sonnet 4.6)

### バグ修正
- **EXE関連付けで画像をクリックしてもビュアーが画像を開いた状態にならない問題**を修正: `_pending_open` フラグを新設。`_on_scan_done` 内の `_do_select()` 完了後100ms待ってから `_open_full()` を呼ぶ。

---

## [v1.94] 2026-04-06 (Claude Sonnet 4.6)

### 新機能
- **アプリアイコンをBase64埋め込みで設定**: `PNGViewer.ico` (16/32/48/64/128/256px) をソース内にBase64で埋め込み。外部icoファイル不要。exe化後もアイコンが正しく表示される。

---

## [v1.93] 2026-04-06 (Claude Sonnet 4.6)

### UI整理
- **ツールバーとコントロールバーを1本に統合**: 「📂ファイル▼」「🔧ツール▼」「⚙設定▼」「📊LoRA分析」の4つのドロップダウン/ボタンを削除。コントロールバー行を廃止し画面の行数が1行減少。

### 新機能
- **パス表示ラベルを編集可能なアドレスバー（QLineEdit）に変更**: フォルダパスを直接入力してEnterで移動できる。存在しないパスを入力すると赤くハイライトしてエラーをステータスバーに表示。

---

## [v1.92] 2026-04-06 (Claude Sonnet 4.6)

### バグ修正
- **レイアウトボタンを2回クリックするとハイライトが消える問題**を修正: `setCheckable(False)` に変更しQtの自動トグルを排除。ハイライトは `setProperty("active","true/false")` + QSS で管理。

### 改善
- レイアウトボタンのクリック反応を高速化: `clicked` → `pressed` シグナルに変更。

---

## [v1.91] 2026-04-06 (Claude Sonnet 4.6)

### 設計改善・再発防止
- **レイアウトボタンのハイライトが毎回元に戻る根本原因**を構造レベルで修正: `_switch_layout()` を2つに分離（`_switch_layout()`: ユーザー操作用、`_apply_layout_by_id()`: 処理実行用）。

### バグ修正
- `setChecked` 後にQSSの `:checked` 擬似クラスが反映されない問題を修正。
- コントロールバーの「🔍タグ:」ラベルの絵文字がWindowsで豆腐（□）になる問題を修正: `QLabel("タグ:")` に変更。

---

## [v1.90] 2026-04-06 (Claude Sonnet 4.6)

### 改善
- **コピー通知トーストの表示位置**をウィンドウのど真ん中（縦横中央）に変更。

---

## [v1.89] 2026-04-06 (Claude Sonnet 4.6)

### 改善
- コピー通知トーストの表示位置をステータスバーからウィンドウ上部中央に変更。スタイルも強調（border-radius:6px, font-size:10pt, accent カラーボーダー）。

---

## [v1.88] 2026-04-06 (Claude Sonnet 4.6)

### バグ修正
- **フェードアニメーションが依然動作しない問題**を根本修正: `self._anim_in` / `self._anim_out` を一度だけ生成し、`.start()` / `.stop()` のみ呼ぶ設計に変更（毎回newするとGCに破棄されていた）。

---

## [v1.87] 2026-04-06 (Claude Sonnet 4.6)

### バグ修正
- **フェードイン・フェードアウトが動作しない問題**を修正: `_init_fade_message()` を新設しQLabel/QGraphicsOpacityEffect/QTimerを初回のみ生成して使い回す設計に変更。

---

## [v1.86] 2026-04-06 (Claude Sonnet 4.6)

### 重大バグ修正
- **サムネイルが壊れてノイズ画像になる問題**を修正: RAWXキャッシュのヘッダーに `bytesPerLine` (2バイト, uint16 LE) を追加し、stride不一致によるピクセルずれを解消。

### 自動修復
- 起動時に旧フォーマットのキャッシュを自動削除: `_CACHE_FORMAT_VERSION=2` を追加し、バージョン変更時に `thumbs/*.raw` を全削除してクリーンビルドを強制。

---

## [v1.85] 2026-04-06 (Claude Sonnet 4.6)

### 新機能
- **コピー後のステータスメッセージをフェードイン・フェードアウト表示に変更**: フェードイン300ms → 2秒表示 → フェードアウト500ms。`QGraphicsOpacityEffect` + `QPropertyAnimation` を使用。

---

## [v1.84] 2026-04-06 (Claude Sonnet 4.6)

### 根本修正
- **クリックしてもメタデータが表示されない問題**を修正: バックグラウンドスレッドからの `QTimer.singleShot` が動作しない問題を解消するため、`_meta_loaded = Signal(str, dict)` を追加しシグナル経由でメインスレッドに通知。

---

## [v1.83] 2026-04-06 (Claude Sonnet 4.6)

### バグ修正
- **シングルクリックでメタデータが表示されない問題**を修正 (v1.82の修正が逆効果だったため): `mousePressEvent` で直接 `clicked` を emit する元の動作に戻す。

---

## [v1.82] 2026-04-06 (Claude Sonnet 4.6)

### バグ修正
- **画像を2回クリックしないとメタデータが表示されない問題**を修正: `clicked` の emit を `mousePressEvent` → `mouseReleaseEvent` に移動。`mouseDoubleClickEvent` では `_drag_start_pos` を無効値に設定して直前の emit を抑制。

---

## [v1.81] 2026-04-06 (Claude Sonnet 4.6)

### 根本修正・高速化
- **クリック後にメタデータが切り替わらない・5秒以上かかる問題**を修正:
  - `_read_png_text_chunks()` を新設し、PNGファイルを直接バイナリ読み取りして高速化（Pillowを不使用）
  - 画像サイズ取得を `QImageReader.size()` に変更（ヘッダーのみ読む）
  - PNG は完全にPillow不使用、JPG/WebP のみPillowフォールバック
- キャッシュミス時に `MetaPanel.show_loading()` でファイル名を即表示するよう改善

---

## [v1.80] 2026-04-05 (Claude Sonnet 4.6)

### 根本修正・高速化
- **クリック・画像切り替え時のUIフリーズを完全排除**: `PreviewLoader` (QRunnable) を新設し非同期プレビューロードを実装。
- `load_preview_pixmap` を PillowからQImageReaderベースに刷新。`setScaledSize()` でデコード段階で縮小。
- リサイズ時のディスクI/Oを完全排除: メモリキャッシュ(`_mem`)のみ参照して `scaled()` するよう変更。
- `dedup_tags` の正規表現を廃止し純粋な文字列操作に置換。

---

## [v1.79] 2026-04-05 (Claude Sonnet 4.6)

### バグ修正
- **コピー操作でステータスバーにメッセージが表示されない問題**を修正: `MetaPanel` に `status_message = Signal(str)` を追加し `_set_status` に接続。

---

## [v1.78] 2026-04-05 (Claude Sonnet 4.6)

### 根本修正
- **メタデータ表示が5秒遅い問題**を修正: `_init_meta_rows()` で固定行ウィジェットを初回のみ生成してキャッシュし、`_update_meta_rows()` で値ラベルの `setText()` だけを更新する方式に変更。

### バグ修正
- Positive/Negativeヘッダーのコピーボタン(📋)が機能しない問題を修正。
- メタ情報のSeed行クリック時にマウスカーソルが指に変わらない問題を修正。

---

## [v1.77] 2026-04-05 (Claude Sonnet 4.6)

### 高速化
- `ScanWorker` で拡張子判定を `endswith` に変更 + `stat` を同時取得。`ScanWorker.finished` シグナルを `Signal(list, dict)` に変更。
- `FilterWorker` のソートで `os.stat()` 呼び出しをゼロに: ScanWorker が取得済みの stat 辞書を参照するだけで即時ソート完了。
- `imgsize` ソートの `Image.open()` フォールバックを廃止。

### 最適化
- `PreloadWorker` のスレッド数を最大16→最大4に制限。
- `VirtualGrid` の `_live_loaders` を `list` → `dict` (path→loader) に変更し、スクロールアウト時に即 `cancel()` 可能に。

---

## [v1.76] 2026-04-05 (Claude Sonnet 4.6)

### バグ修正
- `_live_loaders` リストの肥大化によるサムネイル表示遅延を修正: ロード完了時に自動削除するよう変更。
- `_pixmap_cache`（VirtualGrid ローカル）の無制限肥大化を修正: 800件超で古い200件を削除する上限チェックを追加。

---

## [v1.75] 2026-04-05 (Claude Sonnet 4.6)

### 新機能
- **画像の「既定のプログラム」として起動対応**: コマンドライン引数にファイルパス/フォルダパスを渡すとそのフォルダを開き、ファイルの場合は選択・プレビュー状態で起動。

---

## [v1.74] 2026-04-04 (Claude Sonnet 4.6)

### 根本修正
- **「エクスプローラーで選択」でファイルが選択されない問題**を修正: `Popen(f'explorer /select,"{norm}"', shell=False)` に変更。

### 新機能
- 複数選択時に全ファイルを選択状態でエクスプローラーを開く: `SHOpenFolderAndSelectItems` を ctypes 経由で呼び出し。

---

## [v1.73] 2026-04-04 (Claude Sonnet 4.6)

### 高速化
- `ScanWorker` を `os.scandir` に全面置き換え。
- `PreloadWorker` をスレッドプール並列化 (`concurrent.futures`): max_workers = min(16, cpu_count * 2)。
- `load_thumb` を `QImageReader` ネイティブ縮小に刷新。Pillow非対応形式は自動フォールバック。サムネ正方形パディング廃止。

---

## [v1.72] 2026-04-04 (Claude Sonnet 4.6)

### 根本修正
- **Ctrl+Left / Ctrl+Right によるフォルダ移動が効かない問題**を修正: `_build_menubar` と `_setup_shortcuts` での同一キー二重登録（Ambiguous Shortcut）を解消。

---

## [v1.71] 2026-04-04 (Claude Sonnet 4.6)

### バグ修正
- `_mem` キャッシュのスレッド競合を解消: `_mem_lock` (threading.Lock) を新設。
- `MetaCache` LRUキャッシュ上限を追加: `_MAX_META_ENTRIES = 5000` で無制限肥大化を防止。

### 改善
- `tag_list` 早期リターンで正規表現エンジンの無駄呼び出しを削減。
- `_normalize_path` のUNCパス処理を強化。
- JPEG/WebP サムネイル生成を `Image.draft()` で高速化。

---

## [v1.70] 2026-04-04 (Claude Sonnet 4.6)

### 改善
- 画像読み込み速度向上: ThumbLoader 並列スレッド数 4→8、`_SCROLL_DEBOUNCE` 60→40ms、`_ROW_BUFFER` 2→3行、`_mem` キャッシュ上限 600→1200エントリ。

### 新機能
- LoRA分析ダイアログに最大化ボタン追加。
- コピーボタン付きエラーダイアログ (`_ErrorDialog`) を追加。
- **アクティビティログパネル (`_ConsolePanel`)**: ツールバー「📋 ログ」ボタンでトグル表示。タイムスタンプ + レベル(INFO/OK/WARN/ERR) + 処理時間(ms) を記録。

---

## [v1.69] 2026-04-04 (Claude Sonnet 4.6)

### バグ修正
- **LoRA分析が全く動かなくなっていた問題**を修正: `META._lock` 保持中に `parse_meta` (ファイルI/O) を呼ぶデッドロックを解消。`META` にスレッドセーフな `snapshot()` メソッドを追加。
- Ctrl+Left/Right でフォルダ移動できない問題を修正。

### 新機能
- プロパティダイアログ / ショートカットダイアログに「📋 コピー」ボタンを追加。

### 改善
- `_render_visible` の `to_remove` 判定を `O(n)` → `O(1)` の set 判定に変更。
- `_sel_image` でキャッシュミス時に非同期でメタを取得しUIをブロックしない方式に変更。

---

## [v1.68] 2026-04-04 (Claude Sonnet 4.6)

### バグ修正
- サムネイル二重表示: `DecorationRole` を廃止し `UserRole+1` でピクスマップを渡す方式に変更。

### 新機能
- LoRA使用率分析に「作成日」列 (COL_CDATE=5) を追加。
- 最大化ボタンを追加。
- 列ヘッダークリックでソート: 使用枚数/使用率/作成日/最終使用日の4列をクリックで昇順↔降順切り替え。

---

## [v1.67] 2026-04-03 (Claude Sonnet 4.6)

### 根本修正
- **LoRA使用率分析のサムネイルが表示されない問題**を完全修正: `QTimer.singleShot()` をバックグラウンドスレッドから呼び出していた（スレッドにQtイベントループがなく発火しない）問題を解消。`_thumb_loaded = Signal(str, object)` を追加しシグナル経由でUIスレッドにサムネを渡す。

---

## [v1.65] 2026-04-03 (Claude Sonnet 4.6)

### バグ修正
- サムネイル列が幅ゼロで表示されない問題を修正: `_build_ui` 末尾に `QTimer.singleShot(0, _fix_columns)` を追加。バックグラウンドスレッドでのQPixmap生成（Qt規約違反）を修正。

---

## [v1.64] 2026-04-03 (Claude Sonnet 4.6)

### 改善
- LoRA使用率分析のサムネイル表示を大きく改善: 行の高さを 56px → 120px に拡大。

---

## [v1.63] 2026-04-03 (Claude Sonnet 4.6)

### 新機能
- **LoRA使用率分析に削除機能を追加**: 右クリックメニュー「🗑 削除」と、チェックボックス一括削除機能を追加。
- **LoRA使用率分析にサムネイル列を追加**: 同stem の `.preview.jpeg` 等を非同期ロードして表示。

---

## [v1.62] 2026-04-03 (Claude Sonnet 4.6)

### バグ修正
- 「エクスプローラーで開く」でファイルが選択されない問題を修正: `subprocess.Popen(['explorer', '/select,', ファイルパス])` に変更。

---

## [v1.61] 2026-04-03 (Claude Sonnet 4.6)

### バグ修正
- **メタ情報のLoRAをクリックするとタグ一覧タブが開いてしまう問題**を修正: `_set_tag_search` に `switch_tab: bool = False` 引数を追加。

---

## [v1.60] 2026-04-03 (Claude Sonnet 4.6)

### 改善
- MetaPanel のスプリッターを縮めてもヘッダー行が消えないよう変更: `_HDR_H = 24px` を定義し `setMinimumHeight(_HDR_H)` を設定。

---

## [v1.59] 2026-04-03 (Claude Sonnet 4.6)

### 新機能
- **MetaPanel のセクションヘッダーをダブルクリックで「全コンテンツ表示」高さに瞬時展開**: 既に展開済みの場合は均等分割に戻すトグル動作。

---

## [v1.58] 2026-04-03 (Claude Sonnet 4.6)

### 改善
- 右パネルのタブ廃止: MetaPanel の「ファイル情報・メタ」ペインに 📋 メタ情報 / 🏷 タグ一覧 タブを内包。右パネルの `QTabWidget` を削除。
- お気に入り追加ボタンを "+" → "➕" 絵文字に変更 + フォント指定。
- サブフォルダ QCheckBox にチェックマーク表示を追加。

---

## [v1.57] 2026-04-03 (Claude Sonnet 4.6)

### バグ修正
- 左ペインお気に入りヘッダーの⭐アイコンが表示されない問題を修正: QLabel 個別にスタイルを設定（Segoe UI Emoji を明示指定）。

---

## [v1.56] 2026-04-03 (Claude Sonnet 4.6)

### バグ修正
- 絵文字アイコンが表示されない問題を修正: グローバルQSSの `font-family` に `"Segoe UI Emoji"` を先頭に追加。

---

## [v1.55] 2026-04-03 (Claude Sonnet 4.6)

### 改善
- Positive/Negativeのコピーボタンをヘッダー右端に移動: `_section_header` に `copy_fn` 引数を追加。

---

## [v1.54] 2026-04-03 (Claude Sonnet 4.6)

### 改善
- 右ペイン内の折りたたみ機能を廃止（バグの温床のため）。ツールバーの「メタ ▶」ボタンによる右パネル全体の折りたたみは引き続き動作。

---

## [v1.53] 2026-04-02 (Claude Sonnet 4.6)

### バグ修正
- レイアウトボタン再クリック時のサムネ点滅・スプリッターリセットを修正。

### 改善
- レイアウトA・C・Dのインラインプレビュースプリッター位置を永続化。
- 右パネル内を QSplitter(Vertical) で「プレビュー」と「メタ情報/タブ」に分割。MetaPanel 内部も QSplitter(Vertical) で3セクションに分割。

---

## [v1.52] 2026-04-02 (Claude Sonnet 4.6)

### 改善
- レイアウトBのみメタデータプレビュー（`_preview_label`）を表示。A/C/D/E レイアウト時は非表示。

### 新機能
- レイアウトA・Cで同じレイアウトボタンを再クリックするとサムネイルとプレビューの位置を逆転できるフリップ機能。

---

## [v1.50] 2026-04-02 (Claude Sonnet 4.6)

### 根本修正
- **同じレイアウトボタンを2回クリックするとサムネイルが消える問題**を根本修正: `_grid._resize_timer.stop()` + `_grid._render_timer.stop()` でタイマーを停止してから操作し、完了後に `_grid._full_rebuild()` を即時呼び出し。

---

## [v1.49] 2026-04-01 (Claude Sonnet 4.6)

### 改善
- LoRAフォルダ未設定→設定→保存後に自動でLoRA分析を再実行。

---

## [v1.46] 2026-04-01 (Claude Sonnet 4.6)

### 改善
- LoRAフォルダ未設定時のダイアログを改善: キーボード操作対応ダイアログに変更。設定画面を開きLoRAタブを直接フォーカスして表示。

---

## [v1.45] 2026-04-01 (Claude Sonnet 4.6)

### 改善
- **削除確認ダイアログにキーボード操作**を追加: ← / → / Tab でフォーカス移動、Enterで決定。

### 新機能
- **ファイルのドラッグ＆ドロップ送り出し対応**: サムネイルセルからブラウザ・エクスプローラー等へファイルをドラッグしてコピー/移動できる。

---

## [v1.44] 2026-04-01 (Claude Sonnet 4.6)

### 改善
- 削除確認ダイアログをキーボード操作に最適化: `QMessageBox` を廃止し `_DeleteConfirmDialog`（カスタムQDialog）を新設。Y/N/Enter/Esc を直接処理。`_delete_path` / `_delete_multi` / `_delete_current`（FullViewer）の3箇所すべてに適用。

---

## [v1.43] 2026-04-01 (Claude Sonnet 4.6)

### 設計変更
- レイアウトC/EでフォルダツリーはそのまW維持、右パネル（メタ）のみ非表示にするよう設計変更。

---

## [v1.42] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- 起動時にレイアウトC/Eを復元するとフォルダツリーが消える問題を修正: レイアウト復元タイマーを `_build_ui()` 内から `__init__` 末尾に移動。

---

## [v1.41] 2026-04-01 (Claude Sonnet 4.6)

### 根本修正
- **レイアウト切り替え時にユーザーが閉じたパネルが勝手に復元される問題**を完全修正: `_user_collapsed_right` フラグを新設し左パネルと完全対称化。

---

## [v1.40] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- FullViewer: ズーム中の矢印キーがナビゲーションとスクロールのどちらに働くか不明瞭だった問題を修正。ズーム時はパン、通常時は前後ナビゲーションに分岐。
- MainWindow: OSリサイズ/最大化解除でパネル幅が誤った値で上書き保存される問題を修正: `changeEvent` で300ms デバウンス後に更新。
- `splitterMoved` デバウンス保存を実装。

---

## [v1.39] 2026-04-01 (Claude Sonnet 4.6)

### 改善
- `parse_meta` のメタデータ解析強化: JPG/WebP/HEICのEXIF UserComment・XMP・IFD0からSD生成パラメータを抽出するよう拡張。

### 新機能
- **FullViewer非同期デコード導入** (`_PixmapLoader` / `_load_pixmap_async`): 巨大画像でのGUIフリーズを防止。
- 削除ダイアログの誤操作防止: デフォルトボタンをNoに変更。
- MetaCache スレッドセーフ化: `threading.Lock` を追加。
- `_normalize_path` 堅牢化: Windows MAX_PATH対応、UNCパス正規化改善。

---

## [v1.34] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- 全ペインのフォルダツリー表示/非表示の挙動を統一: 全ペイン（C・E含む）で `_left_panel_widget.hide()/show()` に統一。

---

## [v1.33] 2026-04-01 (Claude Sonnet 4.6)

### 根本修正
- レイアウト「全(E)」から他レイアウトへの切り替え時にフォルダツリーが表示されない問題を根本修正: `_user_collapsed_left` フラグを新設。

---

## [v1.32] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- フォルダツリーの幅がペイン操作で変わってしまう問題を修正: `splitter.splitterMoved` シグナルで随時更新。
- パネル非表示時に他パネルの幅計算がズレる問題を修正。
- レイアウト「全(E)」から他レイアウトに戻るとフォルダツリーが消える問題を修正。

---

## [v1.31] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- レイアウトE（全）: ≡ボタンを押すとフォルダツリーエリアが残る問題を修正: `_left_panel_widget.hide()` で左パネルwidgetごと非表示に変更。
- レイアウトE: メタ情報オーバーレイを完全削除（右パネルに同等機能があるため）。レイアウトEの定義を「グリッド全画面」に変更（オーバーレイなし）。

---

## [v1.30] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- FullViewer: fitボタンを押すと勝手に最大化する問題を修正。
- FullViewer: `_apply_view` をウィンドウ状態から完全に切り離し。
- FullViewer: ボタンラベルを「⊞ fit」「⊟ 原寸」に統一。

---

## [v1.29] 2026-04-01 (Claude Sonnet 4.6)

### バグ修正
- FullViewer: 小ウィンドウで開いた時に画像が原寸大にならない問題を修正。
- FullViewer: 最大化→通常ウィンドウに戻すと画像が縮小される問題を修正。

---

## [v1.28] 2026-04-01 (Claude Sonnet 4.6)

### 新機能
- FullViewer: ウィンドウの最大化状態を記憶して次回に引き継ぐ機能を追加。

---

## [v1.27] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- 起動時CMDウィンドウが残る問題を修正: `pythonw.exe` へ DETACHED_PROCESS で再起動するよう変更。
- FullViewer: SpaceでSSボタンが反応する問題を修正。
- FullViewer: 全画面→最大化→全画面で元の大きさに戻らない問題を修正。
- FullViewer: ウィンドウモード時のサイズが毎回バラバラな問題を修正: `_init_window()` を新設し画像サイズに合わせたウィンドウサイズを計算。

---

## [v1.26] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- FullViewer: 画像画質が悪い問題を修正: `SmoothTransformation` を追加。
- FullViewer: キー入力が効かない問題を修正: ツールバーボタンに `setFocusPolicy(NoFocus)` を設定。

---

## [v2.0f-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- FullViewer: ウィンドウが開いても画像が表示されない問題を修正: `QTimer.singleShot(50ms)` で `_show_image` を遅延呼び出し。
- 起動時のCMDコンソールウィンドウを非表示に。
- 左パネル開閉の ≡ ボタンを拡大（18×48px → 22×80px）。

---

## [v2.0e-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- FullViewer: ダブルクリックで開いた画像の画質が悪い問題を修正: `_load_pixmap` を新設し `QPixmap(path)` でQtネイティブデコード。
- FullViewer: キー入力が効かない問題を修正: `QGraphicsView` に `setFocusPolicy(NoFocus)` / `setInteractive(False)` を設定。
- FullViewer: 全画面化してもツールバーが見える問題を修正: `showFullScreen()` / `showMaximized()` を `_toggle_immersive` で切り替え。

---

## [v2.0d-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- レイアウトC/E → A/B/D に切り替えたとき左パネル（ツリー）が復元されない問題を修正。

---

## [v2.0c-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- ≡縦バーボタンを押すとボタンごと消える問題を修正: `_left_panel_widget.hide()` → `_left_main_content.hide()` に変更。

---

## [v2.0b-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- 左ペイン縦バー「≡」ボタンがツールバー「◀ フォルダ」と異なる挙動をする問題を修正: 縦バーボタンの `clicked` → `_toggle_left_panel` を直接呼ぶよう変更。

---

## [v2.0-Qt] 2026-03-31 (Claude Sonnet 4.6)

### 新機能
- **左ペインをWindowsエクスプローラー風に全面刷新**: QFileSystemModel のルートをPC全体に変更。ツリーをスプリッターで「⭐ お気に入り」と「📁 フォルダ」に分割。ツリー右クリックメニューに「⭐ お気に入りに追加」を実装。
- 左ペイン表示/非表示ボタンを左端縦バーとして配置。

---

## [v1.9b-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- `monkey-patch(mousePressEvent代入)` はPySide6では動作しないため `installEventFilter` + `eventFilter` で正しく viewport イベントを捕捉する方式に全面置き換え。
- Ctrl+A が動かない問題を修正: `VirtualGrid.keyPressEvent` 冒頭で明示的に処理。

---

## [v1.9-Qt] 2026-03-31 (Claude Sonnet 4.6)

### 新機能
- 前回開いていたフォルダを起動時に自動復元。
- Ctrl+B でブックマーク（お気に入り）ダイアログを表示: `_show_bookmark_dialog` を新設。
- **マウスドラッグによる範囲選択（ラバーバンド選択）**: ドラッグ矩形と交差するセルを複数選択。`QRubberBand` で描画。
- 右パネルのプレビューと右パネル全体をドラッグでリサイズ可能: `_right_vsplitter` を新設。

---

## [v1.8-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- 複数選択の一括削除バグ修正: `get_selection()` で複数取得し `_delete_multi()` で一括処理。

### 新機能
- PageUp/PageDown でページ送り。
- Windowsエクスプローラー相当のキーボード操作を実装: F2 / Shift+Delete / Ctrl+C / Ctrl+X / Ctrl+V / Ctrl+Shift+N / Ctrl+D / Alt+矢印 / Alt+Enter / Shift+矢印 / Escape。
- 右クリックメニューをエクスプローラー相当に拡張。

---

## [v1.7-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- **複数選択バグ修正**: クリック①→Ctrl+クリック②でクリック①の選択が消える問題を修正。通常クリック時も `_multi_sel.add(path)` を追加するなど3点の原因を修正。

---

## [v1.6-Qt] 2026-03-31 (Claude Sonnet 4.6)

### 改善
- サムネサイズ変更をCtrl+ホイールに一本化: ツールバーのプリセットボタンを廃止。

---

## [v1.5-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- プレビュースケール修正: `load_preview_pixmap` を新設（アスペクト比保持、パディングなし）。

### 新機能
- 右パネルのプレビューを復活: レイアウトBでのみ表示。

---

## [v1.4-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- プレビューサイズ修正: ラベルの実サイズを使いパネルにフィットするよう修正。resizeEvent フックを追加。

### 改善
- サイズスライダー → タブボタン化: 「小(80px)」「中(160px)」「大(260px)」のプリセットタブボタンに置換。
- レイアウトボタンを文字表記に変更（「上下」「右」「左右」「帯」「全」）。

---

## [v1.3-Qt] 2026-03-31 (Claude Sonnet 4.6)

### バグ修正
- ◀フォルダ / メタ▶ ボタンが機能しない・完全に隠れない問題を修正: `setChildrenCollapsible(True)` に変更し `hide()/show()` を組み合わせて確実に完全非表示。`toggled` → `clicked` シグナルに変更（再帰呼び出し防止）。

---

## [v1.2-Qt] 2026-03-31 (Claude Sonnet 4.6)

### 改善
- レイアウトボタンA〜Eを絵文字アイコンに変更（🔼📋↔🎞👁）。
- ◀フォルダ / メタ▶ ボタンをpane_barからツールバーに移動。
- ボタンON/OFFの表示を背景塗りつぶしから下線スタイルに変更。
- バージョン情報・AI向けアーキテクチャ説明をファイル冒頭に追加。

---

## [v1.1-Qt] 2026-03-31 (Claude Sonnet 4.6)

### 新機能
- レイアウト A〜E を全実装: A(上部プレビュー) / B(右パネル) / C(左右分割) / D(フィルムストリップ) / E(オーバーレイ)。
- メタ情報/タグ一覧ボタンをセンターpane_barから右パネルヘッダーバーに移動。
- ボタンスタイル管理を `setObjectName` + QSSに統一。

### バグ修正
- `_switch_layout` の `_bg/_fg` 未定義 NameError を修正。
- f-string内クォート競合 SyntaxError を修正（Python 3.11以下対応）。

---

## [v1.0-Qt] 2026-03-31 (Claude Sonnet 4.6)

### 初版
- tkinter版 v11 を PySide6 で完全書き直し。
- 仮想スクロールグリッド / メタデータ解析 / LoRA分析 / FullViewer 等を移植。

---

## [v2.03a] 2026-04-08 (Codex)

### UI更新
- サムネイル一覧の更新を差分反映寄りにして、削除後にサムネイルが一瞬消えにくいよう調整。
- `VirtualGrid` のスクロール処理でデバウンス待ちの前に描画を走らせ、スクロールバーをドラッグ中の空白を減らすよう修正。
- Ctrl+ホイールでサムネイルサイズ変更時に先頭へ飛んでしまう不具合を修正。スクロール位置を比率で維持するよう調整。
- 削除確認ダイアログを先に描画してから待機に入るようにし、白いまま遅れて表示される体感を改善。
- サムネイルのクリック確定をマウスリリース時へ変更し、ドラッグ開始時に追尾画像の表示や選択ハイライトが遅れやすい問題を軽減。
