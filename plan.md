以下の分析をもとに、「現状時刻を README に追記する」タスクの具体的な TODO プランをまとめました。

---

# TODO プラン

## 1. README.md への挿入位置を決める
- **目的**：どこに “最終更新時刻” を表示するかレイアウトを確定する  
- **作業内容**：  
  - `README.md` のヘッダー直下 or フッター など挿入箇所を検討  
  - Markdown の書式例を決定  

```markdown
<!-- 例：ヘッダー直下に追記 -->
# プロジェクト名

**最終更新**: 2023-08-01 12:34:56

（以下略）
```
【F:README.md†L1-L5】

---

## 2. スクリプトを作成・設置する
- **目的**：実行すると現在時刻を取得し、README.md の所定位置を更新する  
- **設置場所**：`bin/update_readme.dart` を新規作成  
- **実装イメージ**：
  1. `README.md` を読み込む  
  2. “最終更新” 行の有無を判定  
     - 既存行があれば置換  
     - なければ所定位置に挿入  
  3. ファイルを書き戻す  
- **参照例**：既存の CLI スクリプト `bin/my_tool.dart` の構成を参考にする  
```dart
#!/usr/bin/env dart
import 'dart:io';

void main() {
  final path = 'README.md';
  final now = DateTime.now().toString().split('.').first;
  final lines = File(path).readAsLinesSync();

  // "最終更新" 行を置換または挿入するロジック
  // （ここに実装）
  
  File(path).writeAsStringSync(lines.join('\n'));
  print('README.md を更新しました: $now');
}
```
【F:bin/my_tool.dart†L1-L50】

---

## 3. 実行手順のドキュメント化
- **目的**：誰でもスクリプトを使えるように、README.md に実行方法を追記  
- **作業内容**：
  - `bin/update_readme.dart` を実行するコマンド例を README.md に追加  
  - （必要なら）GitHub Actions や他 CI で定期実行する設定例を追記  
- **追記例**：
```markdown
## 更新手順

```bash
dart pub get
dart run bin/update_readme.dart
```

<!-- GitHub Actions 例 -->
```yaml
# .github/workflows/update-readme.yml
name: Update README
on:
  schedule:
    - cron: '0 * * * *'  # 毎時実行
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: dart-lang/setup-dart@v1
      - run: dart pub get
      - run: dart run bin/update_readme.dart
      - run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add README.md
          git commit -m "chore: update README timestamp"
          git push
```
```

---

## 4. 単体テストの追加
- **目的**：時刻挿入ロジックが意図どおり動作することを自動で検証  
- **設置場所**：`test/update_readme_test.dart` を新規作成  
- **テストケース例**：
  1. README に「最終更新」行がない場合、所定位置に新規挿入される  
  2. 既に「最終更新」行がある場合、古い行が置換される  
- **dart のテストライブラリ**：`package:test/test.dart` を利用  

---

## 5. CI／ワークフローへの組み込み（任意）
- **目的**：マニュアル不要で自動的に README を最新化  
- **作業内容**：
  - （手順 3 で追記した）GitHub Actions ワークフローをリポジトリに追加  
  - 動作確認  

---

# 次のステップ
1. README.md の「最終更新」挿入位置（ヘッダ下 or フッタなど）を確定  
2. 上記プランに沿って `bin/update_readme.dart` の雛形を作成  
3. 進捗を共有いただければレビューします

---

以上、ご確認ください。