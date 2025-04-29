# アイトラッカー制作 — 開発インストラクション

> **対象読者**: 本プロジェクトを開発・保守する開発者（現状は本人のみ）\
> **最終目的**: 依頼主（脳科学研究者）が実験で使用できる安価な顔トラッキング・コミュニケーションデバイスの **研究用プロトタイプ** (Lv 1–3) を 1 か月で完成させる。

---

## 1. プロジェクト概要

- **プロジェクト名**: **HITOMI‑GO**
- **機能範囲**: 開発指標 10 段階の **Lv 1–3** (YES/NO, 文字入力, 絵文字) を実装
- **ターゲット環境**: **Windows 11** (x64) + 1080p Webカメラ
- **配布対象**: 依頼主の脳科学実験で使用（商用配布なし）

### 1.1 主要機能
1. **リアルタイム視線トラッキング**
   - Webカメラ映像を30FPS以上で処理
   - 両眼の瞳孔中心と注視座標(x,y)を推定
2. **YES/NOインタラクション**
   - 視線の固定時間(700ms)でYES/NOボタンを確定
   - CSVへの記録機能
3. **オンスクリーン文字入力**
   - T9風レイアウトまたは6×5グリッドを視線で選択
   - IME連携・CSV保存
4. **絵文字/シンボル選択**
   - 6～10個のスタンプを選択して感情を伝達
5. **ロギング&リプレイ**
   - 生フレーム、推定gaze座標、選択結果をタイムスタンプ付きで保存

### 1.2 性能要件
- **レイテンシ**: カメラ取得→注視座標出力 ≤ 100ms
- **推定FPS**: 25-30FPS安定
- **トラッキング精度**: 画面対角誤差 ≤ 2°
- **CPU使用率**: ノートPC(i7-13620H)で40%未満
- **ロスト検出復帰時間**: 1秒以内

---

## 2. リポジトリ & 開発環境

| 項目                | 内容                                                                          |
| ----------------- | --------------------------------------------------------------------------- |
| **GitHub リポジトリ名** | `hitomi-go-prototype` *(仮案)*                                                |
| **ブランチ構成**        | `main`: 安定版 / `dev`: 開発用 (CI 対象)                                            |
| **IDE / Editor**  | **VS Code** (最新) + **Cursor** 拡張                                            |
| **VS Code 設定**    | `.vscode/settings.json`, `launch.json`, `tasks.json` を共有                    |
| **CI/CD**         | **GitHub Actions**: commit/push で `pytest`→`flake8`→PyInstaller Win64 build |

---

## 3. .cursorrules (ドラフト)

Cursor AI へ指示する共通ルールをファイルで管理し、自動 PR 生成やコミットメッセージを統一。

```ini
# .cursorrules
[commit]
style = conventional        # feat:, fix:, docs:, chore: ...
scope = core, ui, doc

[prompt]
language   = en
min_lines  = 3
require_io = true           # 入力例と期待出力を必ず記述
```

> **TODO**: 実開発で不足を感じたら更新。

---

## 4. 技術スタック

| レイヤー  | 採用技術                        | 備考                     |
| ----- | --------------------------- | ---------------------- |
| 言語    | **Python 3.12**             | typing を徹底し将来の型安全性を確保  |
| GUI   | **PyQt 6**                  | QtDesigner で `.ui` を管理 |
| 画像処理  | **MediaPipe**, **OpenCV**   | wheel 配布版を vcpkg で管理   |
| モデリング | scikit-learn / transformers | Lv 2 まで軽量で十分           |
| 配布    | **PyInstaller**             | `dist/` に exe 出力       |
| テスト   | **pytest**, **pytest-qt**   | GUI クリックは QtBot で最小限   |

---

## 5. 開発フロー

1. Issue を立てる → `dev` ブランチで実装 → PR → GitHub Actions が pass したら `main` へ merge。
2. コミットメッセージは **Conventional Commits** (.cursorrules 参照)。
3. 週 1 回 `main` をタグ付け (`v0.Y.Z`) して依頼主へ deliver。

---

## 6. 未決定事項

| 項目        | 現状                                 | 決定期限                   |
| --------- | ---------------------------------- | ---------------------- |
| **ライセンス** | MIT, Apache‑2.0, Proprietary のいずれか | ベータ版公開時 (Day 22)       |
| **テスト範囲** | 単体 (モデル・ロジック) は必須、GUI 自動操作は検討      | Sprint 1 レビュー時 (Day 7) |

---

## 7. 参考ドキュメント

- 依存ライブラリやレベル詳細は `docs/アイトラッカー開発.md` を参照。
- リリース手順・ユーザーマニュアルは完成後に作成予定。

---

## .cursorrules (v1.0)

下記ルールは **Cursor（AI コード補完 & エージェント）** に対して厳守させる開発ポリシーです。

```text
# 1. 基本確認フロー
1.1  新しく関数・変数・ファイル・フォルダ・コンポーネント・外部ライブラリを追加する前に、
      1.1-1  既存コード・依存関係に同等の実装がないか必ず調査すること。
      1.1-2  新規追加が既存実装に競合・ビルドエラーを生じさせないか静的解析 / 仮ビルドで確認すること。
1.2  `management_dir.md` を照合し、実際のディレクトリ構成と差分がある場合は報告のうえ、
      修正案 (プロジェクト側を合わせるか md 側を更新するか) を提案すること。
1.3  `management_ver.md` を参照し、指定バージョンとローカル環境が一致するか確認すること。
1.4  ライブラリの再インストール・削除・保存先変更・バージョン変更を行った場合、
      `management_ver.md` に **追記** で新バージョンを記載し、旧情報は削除しないこと。
1.5  インポートエラー発生時は以下フローでデバッグすること。
      (1) 指定パスに対象データが存在するか
      (2) 必要データの実際の配置場所を CLI / VSCode 検索で特定
      (3) データ名・モジュール名にスペルミスがないか
      (4) Path が実在するか (OS 大文字小文字含む)
1.6  コード補完により未定義要素が増えた場合は、ユーザーへ報告し追加定義の可否を確認すること。
1.7  本ルールは **毎回のユーザーとの対話直後に実検証** し、確認したルール番号を読み上げること。

# 2. Git / ブランチ戦略
2.1  ブランチは `main` (保守) と `dev` (統合) を基軸とし、機能ごとに `feature/<topic>` を切る。
2.2  全 PR は GitHub Actions で以下ジョブを必須とする :
      - Lint & Format (ruff / black)
      - Type Check (mypy)
      - Unit Test (pytest)
      - Build (PyInstaller)

# 3. コミット規約
3.1  Conventional Commits (`feat:`, `fix:`, `docs:` など) を使用。
3.2  日本語が混在する場合も英語ラベルを先頭に付ける。

# 4. コード品質
4.1  PEP8 + Black フォーマット、自動整形前提。
4.2  すべての新規・変更関数にタイプヒントと Google‑style docstring を付与。
4.3  例外は `logging` モジュールで記録し、`print` はデバッグ用途でも禁止。

# 5. テスト
5.1  コアロジックは pytest ユニットテストを作成し、**行カバレッジ 80% 以上** を維持。
5.2  GUI は `pytest-qt` で Smoke Test を行い、起動・主要 UI 操作が成功することを確認。

# 6. 依存管理
6.1  依存ライブラリは `requirements.txt` にピン留め (`==`) で記載し、更新時は PR 単位で実施。
6.2  新規 OSS 追加時はライセンスを確認し、MIT / Apache‑2.0 のみ許可。その他は要相談。

# 7. ドキュメント整備
7.1  新モジュール追加時は `docs/` に rst または md で簡易 API 仕様を追記すること。
7.2  README のインストール手順・起動手順の差分も同時更新する。

# 8. セキュリティ / プライバシー
8.1  実験データ (顔画像・動画) は **Git 管理対象外** (`data/` は .gitignore)。
8.2  API キーや個人情報は環境変数経由で読み込み、ハードコーディングしない。

# 9. パフォーマンスガード
9.1  UI 応答遅延 >150 ms となる変更は PR 内で理由と改善計画を記述。

# 10. ユーザーコミュニケーション
10.1  毎セッション終了時に「実装内容・差分・懸念点・次アクション」を 4 行以内で要約し報告。
```

---

---

## 3. ブランチ専用ルールファイル

開発ブランチ (`main`, `release/*` を除く) では **`<branch-name>blunchrules.mdc`** を必ず作成し、以下のテンプレートを満たすこと。ファイルが存在しない場合、CI でビルドを失敗させる。

### 3.1 初期テンプレート

```md
# blunchrules for <branch-name>

## Overview / Purpose
<!-- 機能追加・バグ修正の目的を 2–3 行で -->

## Scope / Out‑of‑Scope
- **In scope**: 
- **Out of scope**: 

## Directory Changes
<!-- 追加・削除・移動するファイル / フォルダと理由 -->

## Dependency Changes
<!-- 新規ライブラリ、バージョンアップなど。management_ver.md 追記必須 -->

## Test Checklist
- [ ] Unit tests
- [ ] GUI tests (Qt / 視線計測 UI)
- [ ] Manual sanity check (カメラ起動・YES/NO 入力確認)

## Migration / Data Considerations
<!-- 既存データへの影響があれば記載 -->

## Rollback Plan
<!-- 失敗時に安全に戻す手順 -->

## Milestone & Deadline
<!-- 例: Lv2 完了までに 2025‑05‑15 -->
```

### 3.2 自動生成フロー

| 層                 | メカニズム                                       | 概要                                 |
| ----------------- | ------------------------------------------- | ---------------------------------- |
| **ローカル**          | `.git/hooks/post-checkout` (PowerShell 版)   | ブランチ作成直後にテンプレートをワークツリーに生成・ステージング。  |
| コミットは開発者の判断でまとめる。 |                                             |                                    |
| **CI**            | `.github/workflows/ensure-branch-rules.yml` | `on: push` & `on: pull_request`:   |

- 非 `main` / `release/*` ブランチで `<branch>blunchrules.mdc` の存在と必須セクション (見出し 9 個) をチェック。
- 欠落時はジョブ失敗し PR をブロック。 |

> **採用理由**: ローカルフックで "即時にテンプレを得る" 体験を確保しつつ、GitHub Actions で "リモート UI で作成したブランチ" や "フック設定漏れ" を防止する二重チェック体制。

本セクション追加に伴い、`.cursorrules` の **Rule 2** として以下を追記済み:

```
2.1  main, release/* を除くすべてのブランチは <branch>blunchrules.mdc をプロジェクト直下に含めること。
2.2  CI が当ファイルを検出できない場合、ブランチは "ブロッキング" 扱いとなる。
```

---

> **次のアクション**
>
> - PowerShell フック & GitHub Actions スクリプトのコードひな形を要望があれば生成します。
> - その他テンプレ項目の追加やフォーマット変更が必要な場合はお知らせください。

---

### 3.2 PowerShell Git Hook (`post-checkout`)

> 保存先: `.git/hooks/post-checkout` (Windows では拡張子なし・実行属性不要)。

```powershell
#!/usr/bin/env pwsh
param(
    [string]$OldHead,
    [string]$NewHead,
    [string]$IsBranchCheckout
)

# ブランチ切り替え時のみ実行
if ($IsBranchCheckout -eq '1') {
    $branch = (git rev-parse --abbrev-ref HEAD)
    if ($branch -notmatch '^(main|release/.*)$') {
        $rulesFile = "$branch" + 'blunchrules.mdc'
        if (-not (Test-Path $rulesFile)) {
            Write-Host '[post-checkout] creating' $rulesFile
@'
# blunchrules for $branch

## Overview / Purpose
<!-- 機能追加・バグ修正の目的を 2–3 行で -->

## Scope / Out‑of‑Scope
- **In scope**:
- **Out of scope**:

## Directory Changes
<!-- 追加・削除・移動するファイル / フォルダと理由 -->

## Dependency Changes
<!-- 新規ライブラリ、バージョンアップなど -->

## Test Checklist
<!-- 完了定義 -->
'@ | Out-File -Encoding UTF8 $rulesFile
            git add $rulesFile
        }
    }
}
```

### 3.3 GitHub Actions Workflow (`.github/workflows/blunchrules.yml`)

```yaml
name: Verify blunchrules

on:
  push:
    branches-ignore:
      - main
      - 'release/**'
  pull_request:
    branches-ignore:
      - main
      - 'release/**'

jobs:
  blunchrules-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check rules file exists
        run: |
          branch="${{ github.head_ref || github.ref_name }}"
          file="${branch}blunchrules.mdc"
          if [ ! -f "$file" ]; then
            echo "::error title=Missing blunchrules file::$file not found"
            exit 1
          fi
```

---

## 4. 管理ファイル仕様

### 4.1 `management_req.md`

```md
# Functional Requirements

| ID | Title | Description | Priority (MoSCoW) | Status |
|----|-------|-------------|--------------------|--------|
| FR-001 | Example | Short description | Must | Open |

# Non‑Functional Requirements

| ID | Metric | Target | Verification |
|----|--------|--------|--------------|
| NFR-001 | Eye‑gaze accuracy | ≤1° | Bench test |
```

### 4.2 `management_ver.md`

```md
# Dependency Versions

| Package | Required | Installed | Notes |
|---------|----------|-----------|-------|
| opencv-python | 4.10.0.82 |  |  |
| mediapipe | 0.10.9 |  |  |
```

*バージョン更新時は新しい行を ****追記**** し、既存行は残すこと。*

### 4.3 `management_dir.md`

```md
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
```

*差分がある場合は **`.cursorrules`** §1.2 に従って報告・対処すること。*

---

### 5. `.cursorrules` 追加セクション

```ini
[requirement_check]
file = docs/management_req.md
strict = true

[version_check]
file = docs/management_ver.md

[dir_check]
file = docs/management_dir.md
```

