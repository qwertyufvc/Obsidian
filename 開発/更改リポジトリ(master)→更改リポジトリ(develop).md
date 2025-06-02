

- [ ] 接続設定（コンテキスト名-env）のプロジェクトとメインのプロジェクトで設定ファイルなどを廃止or別ファイルに移し替えた物を整理
    - 下記まとめ。saison_connect_sdk_aws\application\app-serviceは省略

| 移行元                                                      | 移行先                                                                                           | file                                                   | 変更内容                                          | マージ要否                 |
| -------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------- | --------------------- |
| \app-service\build-envs                                  | ー                                                                                             | 全て                                                     | 廃止                                            | 不要                    |
| \app-service\gradle                                      | ー                                                                                             | 全て                                                     | 廃止                                            | 不要                    |
| \app-service\                                            | saison_connect_sdk_aws\application                                                            | build.gradle                                           | 一部はapplicationのbuild.gradleへ移動。一部はこちらで管理      | 要（バージョン管理や一部ライブラリなど？） |
|                                                          | saison_connect_sdk_aws\application                                                            | gradlew<br>gradlew.bat<br>settings.gradle              | applicationで管理のため不要                           | 不要                    |
| app-service-env\src\Production\resources\META-INF\spring | saison_connect_sdk_aws\application\app-service\app-service\src\main\resources\META-INF\spring | application-env.properties                             | envプロジェクトから移動<br>application.propertiesにまとまる？ | 要（接続先追加など）            |
| これは変化してない？                                               |                                                                                               | application.properties                                 | envプロジェクトの環境差分を移動                             | 要（環境別差異？）             |
|                                                          |                                                                                               | env-config.xml                                         | envプロジェクトから移動                                 | 要（環境別差異？）             |
| app-service-env\src\Production\resources                 | saison_connect_sdk_aws\application\app-service\app-service\src\main\resources\META-INF        | logback.xml                                            | envプロジェクトから移動                                 | 要                     |
| sc-nttd-auth-pc-env\src\Production\resources             | saison_connect_sdk_aws\application\auth-pc\auth-pc\src\main\resources\META-INF                | cryptkeys.properties<br>diff.properties<br>logback.xml | envプロジェクトから移動                                 | 要                     |
|                                                          |                                                                                               |                                                        |                                               |                       |
|                                                          |                                                                                               |                                                        |                                               |                       |



| プロジェクト          | ディレクトリ                                                                                                                                                                                                                                                | ファイル名                             | 変更内容                      |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | ------------------------- |
| ra-web-common   | C:\Users\fujishiroh\Desktop\Git_test\sc-nttd-ra-web-common\src\main\java\jp\co\nttdata\cp\saison\infra\fw\security\CtcCrypter.javaC:\Users\fujishiroh\Desktop\Git_test\sc-nttd-ra-web-common\src\main\java\jp\co\nttdata\cp\saison\infra\fw\security\ | CtcCrypter.java                   | 削除                        |
|                 | C:\Users\fujishiroh\Desktop\Git_test\sc-nttd-ra-web-common\src\main\java\jp\co\nttdata\cp\saison\infra\fw\spring\context\support\                                                                                                                     | DirectDataSourceConfig.java       | 削除                        |
|                 | C:\Users\fujishiroh\Desktop\Git_test\sc-nttd-ra-web-common\src\test\groovy\jp\co\nttdata\cp\saison\infra\fw\security\                                                                                                                                 | CtcCrypterTest.groovy             | 削除                        |
|                 | C:\Users\fujishiroh\Desktop\Git_test\sc-nttd-ra-web-common\src\test\groovy\jp\co\nttdata\cp\saison\infra\fw\spring\context\support\                                                                                                                   | DirectDataSourceConfigTest.groovy | 削除                        |
| auth-web-common | src/mainとsrc/test配下のgitsフォルダがgsbuにフォルダ名変更されてる                                                                                                                                                                                                         |                                   | 変更多数なのでようかくにん。変更も入らないと思うし |
| sc-admin-common | 差異なし                                                                                                                                                                                                                                                  |                                   |                           |


- 各コンテキストごとにマージを行う。
→各コンテキストのsrc配下ごとにdevelopへマージ

1. `gitignore` にいらないもの記入??
   - master→developでは srcフォルダとbuild.gradle?だけマージすればOK（他ファイルフォルダは不要？）
   - 無視ファイル設定は必要なくなるかも？
1. `gitattributes` を使ってファイルごとのマージ戦略を制御。　
   

2. 対象のコンテキストのフォルダのみをマージする。
   - src、build.gradle、javancss.gradleをまずマージ
　build.gradleなどのプロジェクト構成変更などに伴うファイルは手動マージしたほうが早いか？
   #TODO
   - envから移動したものをマージ
     - application-env.properties 。どのフォルダのものをマージするか確認。production？　#TODO
     - env-config.xml 。どのフォルダのものをマージするか確認。production？　#TODO
　
　- filter-repoでsrcをまずマージ
　- そのあと下記ファイルをそれぞれマージ（パスが変わってるファイルはこのほうが早そう）
　  - build.gradle
　  - javancss.gradle
　  - application-env.properties
　  - application.properties
　  - env-config.xml
　  - logback.xml
　  
```
# app-serviceの場合　
# マージ先のブランチに移動
git checkout develop

# 対象のディレクトリ、ファイルをマージ
git log master -- application\app-service\app-service\
git apply --stat --allow-empty patch.diff

# コンフリクトを解消
~~
git add コンフリクトファイル
git commit -m "masterブランチのappserviceフォルダ内容をdevelopブランチに取り込み"

git push
```

envから移動した下記ファイルをマージ(差分を作って `git apply` で取り込む＝差分マージ)
- application-env.properties
- application.properties
- env-config.xml
- logback.xml
```
# マージ先のブランチに移動
git checkout develop

# masterブランチの対象ファイルと、developの対象ファイルの差分を出す(下記はapplication-env.properties)
git diff master:application\app-service\app-service-env\src\Production\resources\META-INF\spring/application-env.properties develop:application\app-service\app-service\src\main\resources\META-INF\spring/application-env.properties > patch.diff

# developブランチにいて、W/Xファイルにパッチを当てる
git apply patch.diff

# コンフリクトを解消
~~
git add コンフリクトファイル
git commit -m "masterブランチのappserviceフォルダ内容をdevelopブランチに取り込み"

git push
```

1. コンフリクトを解消してコミット。次回以降は`git rerere`を用いて以前のコンフリクト解消を適用させる
   - build.gradleは確実に差分がでる。ここは解消が必要
   →rerereで前回のバージョン部分の書き換えを反映できるか？
   　→rerereだとほかの競合も意図せず勝手に競合解消マージしちゃうかも？ 