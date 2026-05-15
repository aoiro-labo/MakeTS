# readme-mod

このファイルはフォーク版における変更・改造メモです。

---

## 変更履歴

### 外部ツール（ffmpeg / TSDuck）の自動検索ロジック追加

**対象ファイル:** `MakeTS.py`

**変更内容:**

`find_exe(name)` 関数を追加し、以下の優先順位で実行ファイルを探索するように変更。

1. **環境変数（PATH）** — `shutil.which` で検索
2. **インストール済みパス** — TSDuck は `%ProgramFiles%\TSDuck\bin\tsp.exe` 等を確認
3. **`exe_files/` ディレクトリ** — アプリと同階層の `exe_files/` を確認
4. **見つからない場合** — エンコード開始前にダウンロード先を案内するダイアログを表示（未実装・実装予定）

**変更前の挙動:**  
`exe_files/ffmpeg.exe` / `exe_files/tsp.exe` のみを参照。インストール済みや PATH に通っていても無視。

**変更した関数:**
- `default_exe_dir()` — コメント整理のみ
- `_tsduck_install_dirs()` ← 新規追加
- `find_exe(name)` ← 新規追加
- `default_ffmpeg_path()` — `find_exe` ベースに変更
- `default_tsp_path()` — `find_exe` ベースに変更

---

### run.bat 追加

**対象ファイル:** `run.bat` ← 新規追加

`python MakeTS.py` を実行するだけのバッチファイル。ダブルクリックで起動でき、エラー時はコンソールが閉じずに確認できる（`pause` による）。Python が PATH に通っていれば追加インストール不要。

---

**TODO:**
- `get_ffprobe_path()` を `find_exe("ffprobe")` ベースに更新
- エンコード開始前の「ツール未検出 → ダウンロード案内」ダイアログ実装

---

### UI再編・モダン化

**対象ファイル:** `MakeTS.py`, `requirements.txt`, `lang/*.json`

**タブ構成変更（設定種別 → サービス別）:**

| 旧タブ | 新タブ |
|---|---|
| Video（映像設定 + OneSeg映像） | **Main**（映像 + 主副音声 + EIT） |
| Audio（主副音声 + OneSeg音声） | **OneSeg 1**（映像 + 音声 + EIT） |
| TS Info（NIT/SDT/CAT） | **OneSeg 2**（映像 + 音声 + EIT） |
| Program（EIT + TOT） | **Station**（NIT/SDT/CAT/TOT） |
| Log | **Log** |

OneSeg 1 の設定が Video/Audio/Program の3タブに分散していたのを1タブにまとめた。

**モダンスタイル:**
- `sv-ttk` ライブラリ（Sun Valleyテーマ）を採用、`requirements.txt` に追加
- sv-ttk 未インストール時は `clam` テーマへ自動フォールバック
- Log タブをダークターミナルスタイルに変更（`#1e1e1e` 背景、Consolas フォント）

**run.bat の事前セットアップ:**
初回のみ `pip install -r requirements.txt` を実行して sv-ttk を導入すること。

---

### UI細部修正

**対象ファイル:** `MakeTS.py`, `lang/*.json`

**リサイズ時の黒フラッシュ対策:**
- `configure_style()` 末尾でウィンドウ背景色をテーマ色に同期（`self.configure(background=bg)`）
- `build_ui()` の旧フレーム破棄後に `update_idletasks()` を呼び、再描画ちらつきを低減

**ラベル文字の被り修正:**
- `combo_row` / `entry_row` / `file_entry_row` / `text_row` から `width=18` を削除
- グリッドの列幅はコンテンツに合わせて自動決定されるため、長い翻訳テキストでも被らない

**出力先の配置変更:**
- 旧：上部「Input / Output」フレームに "Save as"
- 新：Files セクション（ファイルリスト下部）に "Output" として配置
- 上部ヘッダーは言語選択のみのコンパクトな行に変更

**設定ダイアログ追加:**
- 下部バーに「設定」ボタンを追加（右端配置）
- ffmpeg パス・tsp.exe パスを設定ダイアログに集約（モーダル Toplevel）
- Station タブから tsp.exe 行を削除してすっきり

---

### 設定ダイアログ機能追加

**対象ファイル:** `MakeTS.py`

**環境情報コピー:**
- 設定ダイアログの「環境情報をコピー」ボタンでクリップボードにコピー
- 内容: MakeTS バージョン / Python / OS / ffmpeg / tsp / sv-ttk

**アップデート確認:**
- `GITHUB_REPO = "owner/repo"` を設定すると有効化（空の場合はボタンが無効）
- GitHub Releases API を使って最新バージョンを取得、現在の `APP_VERSION` と比較
- バックグラウンドスレッドで取得するため UI がブロックしない

**バージョン管理:**
- `APP_VERSION` 定数をファイル先頭に定義（現在 `"0.1.0"`）
- 設定ダイアログ下部に現在バージョンを表示
