下記は、これまでのリポジトリ調査を踏まえ、「現状時刻を README.md に追記する」タスクを確実に進めるための TODO プランです。

---

## 1. README.md に時刻挿入用マーカーを設置

まずは README.md の末尾等、所定の場所にプレースホルダー（マーカー）を追加します。

```diff
@@ README.md
- <!-- ここに現状時刻を追記したい -->
+ <!-- TIME_MARK -->
```
【F:README.md†L6-L7】

---

## 2. シェルスクリプトの動作確認・改善

既存の `scripts/update_readme_time.sh` をベースに、以下をチェック・修正します。

| 作業内容                                      | 理由・詳細                                                 |
|---------------------------------------------|------------------------------------------------------------|
| 2.1 sed オプションのクロスプラットフォーム対応 | macOS (`-i ''`)／Linux (`-i`) の差異を吸収                  |
| 2.2 プレースホルダー文字列の正確一致確認       | `<!-- TIME_MARK -->` を確実に置換できているか              |
| 2.3 実行権限の付与                             | スクリプトに `chmod +x` が必要か                           |

```bash
#!/usr/bin/env bash
set -euo pipefail

# 現在時刻を取得（YYYY-MM-DD HH:MM:SS 形式）
now=$(date '+%Y-%m-%d %H:%M:%S')

# sed の -i オプションをプラットフォームに合わせて切り替え
# GNU/Linux: sed -i, macOS: sed -i ''
if sed --version >/dev/null 2>&1; then
  SED_OPTS=(-i)
else
  SED_OPTS=(-i '')
fi

# プレースホルダーを置換
sed "${SED_OPTS[@]}" \
  -e "s/<!-- TIME_MARK -->/${now}/" \
  README.md

echo "README.md を ${now} で更新しました。"
```
【F:scripts/update_readme_time.sh†L1-L10】

---

## 3. ローカル実行手順のドキュメント化

README.md や CONTRIBUTING.md 等に下記手順を追記し、誰でも簡単に時刻を更新できるようにします。

```markdown
### 時刻の更新方法

```bash
# (1) スクリプトに実行権限を付与（初回のみ）
chmod +x scripts/update_readme_time.sh

# (2) README.md の時刻を更新
./scripts/update_readme_time.sh

# (3) 変更をコミット＆プッシュ
git add README.md
git commit -m "Update README timestamp"
git push
```
```
【F:README.md†L15-L23】

---

## 4. CI（GitHub Actions 等）への組み込み検討

手動だけでなく、定期／プッシュ時に自動で実行させたい場合は CI ワークフローを追加します。例として以下のようなジョブを用意。

```yaml
# .github/workflows/update-readme-time.yml
name: Update README Time

on:
  schedule:
    - cron: '0 * * * *'    # 毎時 0 分に実行
  workflow_dispatch:      # 手動トリガーも可能

jobs:
  update-readme-time:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Update README timestamp
        run: |
          chmod +x scripts/update_readme_time.sh
          ./scripts/update_readme_time.sh
      - name: Commit & Push if changed
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: "ci: update README timestamp"
```
【F:.github/workflows/update-readme-time.yml†L1-L22】

*(※ 上記ファイルは新規追加になります。)*

---

## 5. 動作検証 & マージ

1. **ローカル検証**  
   - プレースホルダーが正しく置換されるか  
   - sed のオプション周りにエラーがないか  
2. **PR 作成 → レビュー → マージ**  
   - リリースノート等への影響チェック  
   - CI がグリーンになることを確認

---

### まとめ

| ステップ | タスク                                                                                  |
|:--------:|:----------------------------------------------------------------------------------------|
| 1        | README.md に `<!-- TIME_MARK -->` を追加                                               |
| 2        | `scripts/update_readme_time.sh` の sed オプション対応・実行権限付与・エラーハンドリング |
| 3        | 実行手順をドキュメント化（README.md 等に追記）                                         |
| 4        | CI（GitHub Actions）への定期／手動実行ワークフローを作成                                |
| 5        | ローカル検証 → PR → レビュー → マージ                                                   |

この TODO プランに沿って実装を進めれば、「現状時刻を README.md に自動追記する」仕組みが安定して運用できるようになります。  
ご確認のうえ、次の実装フェーズに進めてよろしければお知らせください。