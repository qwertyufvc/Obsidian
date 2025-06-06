[[Gitミラーリング検討]]
- `git filter-repo` を使って、Aリポジトリから `src/AAA/B` を抽出しつつ、`src/BBB` にパスを変更する
- その上で Bリポジトリにインポート（マージ）できる
- **履歴を保持したままパスだけ変更できる**
---
## 🛠️ 仮手順

### ① Aリポジトリの対象フォルダを抽出しつつ、パスを変更

```
git clone https://example.com/A.git A-filtered
cd A-filtered

git filter-repo \
  --path src/AAA/B \
  --path-rename src/AAA/B/:src/BBB/

```

📌 このコマンドの意味：

- `--path`：抽出対象のパス（`src/AAA/B`）
- `--path-rename`：新しいパスに変更（`src/BBB`）
👉 つまり「`src/AAA/B` の中身を `src/BBB` に変更し、履歴付きで保持する」

---

### ② Bリポジトリに取り込む

```
cd ..
git clone https://example.com/B.git B
cd B

# Aから抽出したリポジトリをリモートとして追加
git remote add A-filtered ../A-filtered
git fetch A-filtered

# Aの履歴付きコミットをマージ（競合があれば手動対応）
git merge A-filtered/main --allow-unrelated-histories

```

---

## ✍️ 注意点

- `--allow-unrelated-histories` が必要（別リポジトリをマージするため）
- 競合は手動解決が必要（特に `build.gradle` など）
- `git filter-repo` は `filter-branch` より高速＆安定です

---

## 🧪補足：動作確認後、Bリポジトリをpush
`git push origin main`

---

## ✍️ 後で書き足す注意点

- 異なる改行コードでコミットされていたりするのでgithub上で全体差分で表示されているケースあり。全体差分になってるものは仕方ないのでwinmergeなどで差分とって何が追加されたかを確認してからマージする
- 差分的なものなら取り込めばテストも動くと思うが、ファイル追加だとライブラリをjakartaに直したりなど必要なのでテストケースが動かないorビルド通らないことがあると思う
- AppConfig.javaは設定を移動しているので森さんに確認？→エクセルにも記載
- 
---

# ✅ 実際の移行内容まとめ

## 「ミラーリングリポジトリ→更改のmaster」
### 📁 `app-service`

| 状態      | パス                                     |
| ------- | -------------------------------------- |
| **移行前** |                                        |
| **移行後** | `application/app-service/app-service/` |
### ① Aリポジトリの対象フォルダを抽出しつつ、パスを変更
```
# filter_repo用（ディレクトリ構造いじる）にclone
git clone https://github.com/Soba-Noodles/sc-nttd-app-service.git appservice-filtered

cd appservice-filtered

# 現行リポジトリを更改リポジトリのフォルダ構成に合わせる
git filter-repo --path-rename :application/app-service/app-service/
```


### ② Bリポジトリに取り込む

```
# clone済みの場合は下記コマンドは不要
git clone https://github.com/Soba-Noodles/repo-merge-test.git B

# Bリポジトリに移動
cd ..
cd B

# masterからmaster_mergeブランチを作成して切り替え
git checkout -b master_merge origin/master

# Aから抽出したリポジトリをリモートとして追加(パスは作業フォルダに応じて要変更)
git remote add appservice-filtered ../appservice-filtered
git fetch appservice-filtered
（# 絶対パスの場合。下記は例）
git remote add appservice-filtered file:///C:/Users/fujishiroh/Desktop/Git_test/appservice-filtered
git fetch appservice-filtered


# filter-repoした取り込みブランチのログを確認して取り込み対象のID取得
git log appservice-filtered/master --oneline

# cherry-pickコマンドで指定したIDのコミットをマージ
# `-m 1` → 親1（マージ先ブランチ側）を基準に cherry-pick
git cherry-pick -m 1 commitID

# （連続するコミットをまとめて指定する場合）
git cherry-pick <最初のコミットID>^..<最後のコミットID>

git push --set-upstream origin master_merge
```

　上記を各コンテキスト分繰り返してmaster_mergeブランチに変更を取り込みプッシュ。
　master_merge→masterへのプルリクエストを作成。

- 現行の修正が優先なのでコンフリクト解消などは出ない想定。
  →上記手順は自動化できそう

---

## 「現行の変更→更改のdevelop」

 1. app-service→app-service & app-service-env→app-service-env対応
 2. app-service-env→app-service(envから移動したファイルのマージ対応)
### 📁 `app-service`

| 状態                                                                                   | パス  |
| ------------------------------------------------------------------------------------ | --- |
| **移行                                                                                 |     |
| `application/app-service/app-service/` or `application/app-service/app-service-env/` |     |

### 1. app-service→app-service & app-service-env→app-service-env対応
#### ① Aリポジトリの対象フォルダを抽出しつつ、パスを変更
```
# filter_repo用（ディレクトリ構造いじる）にclone
git clone https://github.com/Soba-Noodles/sc-nttd-app-service.git appservice-filtered

cd appservice-filtered

# 現行リポジトリを更改リポジトリのフォルダ構成に合わせる
git filter-repo --path-rename :application/app-service/app-service/
```


#### ② Bリポジトリ（develop_merge）に取り込む

```
# clone済みの場合は下記コマンドは不要
git clone https://github.com/Soba-Noodles/repo-merge-test.git B

# Bリポジトリに移動
cd ..
cd B

# masterからdevelop_mergeブランチを作成して切り替え
git checkout -b develop_merge origin/develop

# Aから抽出したリポジトリをリモートとして追加(パスは作業フォルダに応じて要変更)
# すでに実施済みの場合は下記remote、fetchコマンドは不要
git remote add appservice-filtered ../appservice-filtered
git fetch appservice-filtered
（# 絶対パスの場合。下記は例）
git remote add appservice-filtered file:///C:/Users/fujishiroh/Desktop/Git_test/appservice-filtered
git fetch appservice-filtered

# filter-repoした取り込みブランチのログを確認して取り込み対象のID取得
git log appservice-filtered/master --oneline

# cherry-pickコマンドで指定したIDのコミットをマージ
# `-m 1` → 親1（マージ先ブランチ側）を基準に cherry-pick
git cherry-pick -m 1 commitID

# コンフリクトの手順は省略
git status
コンフリクト解消
～～
git add .
git cherry-pick --continue
（git cherry-pick --abort を使うと、チェリーピックを中断して元の状態に戻せます）

# 変更完了したらリモートブランチを新しく作ってプッシュ
git push --set-upstream origin develop_merge

```
上記①、②手順をapp-service-envにも対応

### 2.app-service-env→app-service

app-service-env→app-serviceにファイルパスを変更したファイルは変更確認し手動取り込みしてマージ
envに変更ないか確認→あればエクセルで格納先フォルダ確認して変更取り込み。

対象は下記ファイル
- build.gradle
- javancss.gradle
- application-env.properties
- application.properties
- env-config.xml
- logback.xml


```
1. app-service→app-service & app-service-env→app-service-env対応
2. app-service-env→app-service
```
上記を各コンテキスト分繰り返してdevelop_mergeブランチに変更を取り込みプッシュ。
develop_merge→developへのプルリクエストを作成。
