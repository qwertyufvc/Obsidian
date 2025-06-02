## 🔍 ローカルの変更を確認するコマンド

### ✅ 1. 未ステージの変更（`git add` 前）を見る

作業ディレクトリの変更内容を確認できます。
`git diff`

---

### ✅ 2. ステージ済みの変更（`git add` 済み）を見る

コミット予定の差分を確認できます。
`git diff --cached`

---

### ✅ 3. 変更されたファイルの一覧（ステージ済・未ステージ含む）を見る

`git status`

---

## 🚀 Git Log の見やすい表示と履歴確認

### ✅ コミット履歴を簡潔にグラフィカルに表示

`git log --oneline --graph --decorate`

---

### ✅ 特定のコミットの詳細を見る

`git show <コミットハッシュ>`

---

### ✅ 特定のコミットで変更のあったファイル一覧だけを見る

`git show --name-only <コミットハッシュ>`

---

## 🧹 ローカルの変更を捨ててリモートの master に強制的に合わせる

※全てのローカル変更が失われます。注意してください。

`git fetch origin git reset --hard origin/master`

---

## 🔄 リモートブランチの取得と切り替え

### ✅ 1. 既にローカルにブランチがある場合

`git checkout ブランチ名 git pull origin ブランチ名`

---

### ✅ 2. ローカルにまだブランチがない場合（リモートから取得して作成）

`git fetch origin git checkout -b ブランチ名 origin/ブランチ名`

---

## 🔧 リモートリポジトリを削除して再登録する

`git remote remove auth-env-filtered`

---
### ✅ 1. **cherry-pick を完了する**

コンフリクト（衝突）などを解消して、以下のようにして完了させます。
`git add <修正したファイル> git cherry-pick --continue`

---

### ✅ 2. **cherry-pick をやめる（中止する）**

やり直したい、あるいは途中でキャンセルしたい場合はこちら：
`git cherry-pick --abort`

## 🧼 まとめ：全部削除したいときのセット

```
# リモート設定を削除
git remote remove auth-filtered

# リモート追跡ブランチを削除
git branch -dr auth-filtered/master

# ローカルブランチを削除（必要な場合）
git branch -d auth-filtered_master  # または -D
```