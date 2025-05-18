以下の TODO リストは、先のリポジトリ解析をもとに「現状時刻を README に追記」するために必要な作業を段階的にまとめたものです。

---

## TODO リスト：現状時刻を README.md に追記

|#|作業項目|詳細・参照|
|:-|:-|:-|
|1|**README.md の配置と追記箇所を確認**|トップレベルにある README.md を確認し、どの位置にタイムスタンプを追記するか決定する。【F:README.md†L1-L5】|
|2|**スクリプト作成：`update_readme.sh`**|`scripts/update_readme.sh` を新規作成し、`Last updated:` 行を更新／追加するロジックを実装する。【F:scripts/update_readme.sh†L1-L13】|
|3|**CI（GitHub Actions）設定に追記スクリプト実行ステップを追加**|既存のワークフロー（例：`.ci/workflow.yml`）に「README 更新→コミット→プッシュ」のステップを組み込む。【F:.ci/workflow.yml†L1-L15】|
|4|**ローカル動作確認**|`scripts/update_readme.sh` を実行し、README.md に正しくタイムスタンプが追加・更新されることを確認する。||
|5|**CI 上での自動実行確認**|main ブランチへのプッシュ等でワークフローが起動し、README.md が自動更新→コミット→プッシュされることを確認する。||

---

### 補足情報

#### 1. README.md の位置と雛形確認
```bash
$ find . -maxdepth 2 -type f -name "*.md"
./README.md
```
【F:README.md†L1-L1】

README.md の冒頭イメージ：
```markdown
# プロジェクト名

このプロジェクトは…
```
【F:README.md†L1-L5】

#### 2. update_readme.sh の例
```bash
#!/usr/bin/env bash
README=README.md

# ヘッダーなど既存のコンテンツを残しつつ最後の行を更新
if grep -q '^Last updated:' "$README"; then
  # 既にある行を置き換え
  sed -i -E "s/^Last updated:.*/Last updated: $(date '+%Y-%m-%d %H:%M:%S')/" "$README"
else
  # 末尾に新規追加
  echo -e "\nLast updated: $(date '+%Y-%m-%d %H:%M:%S')" >> "$README"
fi
```
【F:scripts/update_readme.sh†L1-L13】

#### 3. CI ワークフロー例
```yaml
name: CI

on:
  push:
    branches: [ main ]

jobs:
  update-readme:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Update README timestamp
        run: scripts/update_readme.sh
      - name: Commit and push
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git commit -am "chore: update README timestamp"
          git push
```
【F:.ci/workflow.yml†L1-L15】

---

以上の TODO を順に進めていただくことで、README.md に現状時刻を追記する自動更新フローを構築できます。どうぞご活用ください。