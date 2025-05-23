
### log
見やすい
`git log --oneline --graph --decorate`

`git show <コミットハッシュ>`

ローカルの変更を捨てて、リモートの master に強制的に合わせる
`git fetch origin`
`git reset --hard origin/master`


1. 既にローカルにブランチがある場合
```
git checkout ブランチ名
git pull origin ブランチ名
```
2. ローカルにまだブランチがない場合（リモート追跡ブランチを作る）
```
git fetch origin
git checkout -b ブランチ名 origin/ブランチ名
```

