# 目次
- [🔲実現したいこと](#🔲実現したいこと)
- [✅ 実際の手順](#-実際の手順)
  - [「ミラーリングリポジトリ→更改のmaster」](#「ミラーリングリポジトリ→更改のmaster」)
    - [① Aリポジトリの対象フォルダを抽出しつつ、パスを変更](#① Aリポジトリの対象フォルダを抽出しつつ、パスを変更)
    - [② Bリポジトリに取り込む](#② Bリポジトリに取り込む)
      - [取り込み対象のログについて](#取り込み対象のログについて)
  - [「現行の変更→更改のdevelop」](#「現行の変更→更改のdevelop」)



# 🔲 実現したいこと

- ミラーリングリポジトリ（以後 A）から開発リポジトリ（以後 B）へ自動マージしたい  
  - A から B(master) へのマージ（基本 A 優先）  
  - B の master から develop へのマージ
- B(master) について、リポジトリ構成が違うので自動で吸収できるようにしたい
- `build.gradle` など大幅にいじってるところは自動マージは難しいので別管理で吸収するフローに乗せたい

---

# ✅ 実際の手順

## 「ミラーリングリポジトリ → 更改の master」

`git filter-repo` を使って、A リポジトリを `application/app-service/app-service/` にパス変更する。  
以降は app-service プロジェクトを例に記載。

| 状態      | パス                                     |
|-----------|------------------------------------------|
| **移行前** | ルートから                               |
| **移行後** | `application/app-service/app-service/`   |

### ① A リポジトリの対象フォルダを抽出しつつ、パスを変更
```
# Aリポジトリをfilter_repo用（ディレクトリ構造いじる）にclone
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

# filter-repoした取り込みブランチのログを確認して取り込み対象のID取得
git log appservice-filtered/master --oneline

# cherry-pickコマンドで指定したIDのコミットをマージ（取り込み対象については下記の説明参照）
# `-m 1` → 親1（マージ先ブランチ側）を基準に cherry-pick
git cherry-pick -m 1 commitID

# コンフリクト解消や追加ファイルを解消
～～
git add .

# 解消したらcherry-pickを終了
git cherry-pick --continue

# ブランチをプッシュ
git push --set-upstream origin master_merge
```

　上記を各コンテキスト分繰り返してmaster_mergeブランチに変更を取り込みプッシュ。
　master_merge→masterへのプルリクエストを作成。

#### 取り込み対象のログについて
下記のようなログの場合、現行開発ではリリース後にmasterブランチへ開発ブランチをマージコミットしている。
そのため **個別の変更コミットではなく、マージコミット（Merge pull request ...）のみを cherry-pick する**ことで対応可能（コミット履歴も見やすい）
```
6c90b2a3c (tag: v3.40.1_2250516, auth-filtered/master, auth-filtered/HEAD) Merge pull request #2407 from nk-auth-api/release-3.40.0-20250515
f6110eb9e バージョン3.40.1を修正　　←取り込み不要
17bb63dd1 Merge pull request #2405 from nk-auth-api/develop-3.40.0-20250512
4c53abf5a Merge pull request #2402 from nk-auth-api/fix-3.40.0-20250512_S0515
4970de4d7 2025年05月12日リリース_MH-103_【セゾンコネクト】STOREE SAISON Open会員-セゾンID連携　　←取り込み不要
03c99f4ad 2025年05月12日リリース_MH-103_【セゾンコネクト】STOREE SAISON Open会員-セゾンID連携　　←取り込み不要
a4fc3c523 Merge pull request #2401 from nk-auth-api/fix-3.40.0-20250512_S0515
284a06da8 2025年05月12日リリース_MH-103_【セゾンコネクト】STOREE SAISON Open会員-セゾンID連携　　←取り込み不要
eedfa4c8a (tag: v3.40.0_20250515) Merge pull request #2400 from nk-auth-api/release-3.40.0-20250515
```

上記の場合は下記のようなコマンドとなる。
（古い順からコミット）
```
git cherry-pick -m 1 eedfa4c8a
git cherry-pick -m 1 a4fc3c523
git cherry-pick -m 1 4c53abf5a
git cherry-pick -m 1 17bb63dd1
git cherry-pick -m 1 6c90b2a3c

（git cherry-pick --abort を使うと、チェリーピックを中断して元の状態に戻せます）
```

---

## 「現行の変更→更改のdevelop」

 1. app-service→app-service & app-service-env→app-service-env対応
 2. app-service-env→app-service(envから移動したファイルのマージ対応)

1の手順は基本的には「ミラーリングリポジトリ→更改のmaster」の手順と同じ。
マージする際にdevelopはファイルの修正を多数対応しているため都度コンフリクト解消が必要となる。

2はファイルごとにfilter-repoで対応すると手順が多く煩雑になるためファイルごとに手動取り込みを行う。