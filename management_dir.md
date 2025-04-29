# Directory Layout (authoritative)

HITOMI-GO/
├─ src/
│  ├─ core/
│  ├─ gui/
│  └─ tests/
├─ models/
├─ resources/
│  └─ images/
└─ docs/

## Sub-directory Details

| Path | 必須 / 任意 | 役割 / 主要ファイル |
|------|-------------|----------------------|
| **src/core/** | 必須 | ビジネスロジック |
| ├─ capture/ | 任意 | `video_capture.py` |
| ├─ detection/ | 必須 | `gaze_detector.py` |
| ├─ pipeline/ | 任意 | `processor.py` |
| └─ utils/ | 必須 | `config.py`, `logging_utils.py` |
| **src/gui/** | 必須 | PyQt6 UI コンポーネント |
| ├─ windows/ | 必須 | `main_window.py` |
| ├─ widgets/ | 任意 | `video_panel.py` |
| └─ styles/ | 任意 | `theme.qss`, `icons/` |
| **src/tests/** | 必須 | pytest テスト |
| ├─ unit/ | 必須 | 単体テスト |
| └─ integration/ | 任意 | GUI テスト |
| **models/** | 任意 | 学習済みモデル |
| **resources/images/** | 任意 | アイコン・ダミー画像 |
| **docs/** | 必須 | ドキュメント & 図 |
| **.github/workflows/** | 必須 | CI 定義 |
| **scripts/** | 任意 | ビルド・補助スクリプト |
| **config/** | 任意 | TOML/INI/YAML 設定 |

---

### managementrules of this file
- **ファイル追加時**: 上表の “必須” 行に該当するフォルダへ置く。別の場所に置くなら `management_dir.md` を更新してから PR。  
- **新フォルダを作る場合**: テーブル末尾に **Path / 必須 or 任意 / 役割** を追加して同時コミット。  
- **ビルド生成物・キャッシュ**: `build/`, `dist/`, `__pycache__/` は追跡しない（`.gitignore` 済み）。  

CI (`branch-rules.yml`) は PR ごとに実ディレクトリをスキャンし、`management_dir.md` との差分があれば失敗させる。
