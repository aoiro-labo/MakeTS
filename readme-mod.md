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
- 各言語ファイル（`lang/*.json`）へ新規翻訳キー追加
