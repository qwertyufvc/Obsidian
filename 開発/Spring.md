
### Spring Bootとは何か？

**Spring Boot** は Spring Framework を簡単に使えるように作られたフレームワークで、「Spring ベースのアプリケーションを迅速に開発する」ことを目的としています。

**`spring-boot-dependencies`** と **`spring-boot-gradle-plugin`** のバージョンを統一するのがベストプラクティスです。そして、他のSpring Bootに関連する依存関係（`spring-boot-starter` や `spring-boot-starter-web` など）も、**同じバージョン**に統一するのが望ましい


### ストプラクティスのポイント

1. **`spring-boot-starter` 系の依存関係は自動で解決される**  
    Spring Boot では `spring-boot-gradle-plugin` または `spring-boot-dependencies` を指定すると、他の依存関係のバージョンも自動的にそれに合わせたバージョンで確定されます。つまり、個別にバージョンを指定する必要がなく、統一されます。
    
    例: 次のように、`spring-boot-starter-web` や `spring-boot-starter-data-jpa` を同じバージョンにしたい場合、`spring-boot-dependencies` を通じて同じバージョンが自動的に決定されます。
    
2. **`spring-boot-starter-parent`（Mavenで利用する場合）の利用**  
    Maven を利用する場合、`spring-boot-starter-parent` を使用することで依存関係のバージョンを一元管理できます。