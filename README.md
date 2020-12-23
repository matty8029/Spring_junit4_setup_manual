# Springでjunit4の環境構築

現在担当しているプロジェクトでは、今まで入力値テストが手入力で
行われていたのでjunit4を用いたテストを実装し効率化を図る。

junit4環境設定からテストコード作成までの手順をまとめる。<br /><br />

今回のテストコードにて行うテスト内容<br />
・入力フォームのバリデーション<br />
・指定のviewに返却されるか<br /><br />
プロジェクトの構造は大雑把に下記のような配置<br /><br />
project┬src┬main┬controller─UserController.java<br />
       │   │    ├service<br />
       │   │    ├model─UserSearchForm.java<br />
       │   │    ├resource─application.properties<br />
       │   │    └webapp┬view┬userSearch.jsp<br />
       │   │           │    └userSearchResult.jsp<br />
       │   │           └ApplicationContext.xml<br />
       │   └test─contororller─UserControllerTest.java<br />
       └pom.xml<br />
       
pom.xmlの設定<br />
dependenciesに下記を追加<br />
```
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.5</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>3.2.3.RELEASE</version>
  <scope>test</scope>
</dependency>
```
  
