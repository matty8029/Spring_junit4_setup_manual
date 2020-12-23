# Springでjunit4の環境構築

現在担当しているプロジェクトでは、今まで入力値テストが手入力で
行われていたのでjunit4を用いたテストを実装し効率化を図る。

junit4環境設定からテストコード作成までの手順をまとめる。<br /><br />

今回のテストコードにて行うテスト内容<br />
・入力フォームのバリデーション<br />
・指定のviewに返却されるか<br /><br />
プロジェクトの構造は大雑把に下記のような配置(テストに不要な要素は記載しておりません)
```
project┬src┬main┬controller─UserController.java
       │   │    ├service
       │   │    ├model─UserSearchForm.java
       │   │    ├resource─application.properties
       │   │    └webapp┬view┬userSearch.jsp
       │   │           │    └userSearchResult.jsp
       │   │           └ApplicationContext.xml
       │   └test─contororller─UserControllerTest.java
       └pom.xml
```    
pom.xmlの設定<br />
dependenciesに下記を追加
```
<!-- junit4 --!>
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

<!--BeanValidationユニットテスト -->
<dependency>
  <groupId>org.glassfish</groupId>
  <artifactId>javax.el</artifactId>
  <version>3.0.0</version>
</dependency>
<dependency>
  <groupId>javax.el</groupId>
  <artifactId>javax.el-api</artifactId>
  <version>2.2.5</version>
</dependency>

<!--入力値のランダム生成用ライブラリ -->
<dependency>
  <groupId>com.github.javafaker</groupId>
  <artifactId>javafaker</artifactId>
  <version>0.16</version>
</dependency>
```
こちらでpom.xmlの設定は完了<br/>
次にテストを行うコントローラーは
UserController.java
```java
@Controller
public class UserController {
       ~~~省略~~~   
       @RequestMapping(value = "/userSearch", method = RequestMethod.GET)
       public String userSearchGet(ModelMap model) {
              model.addAttribute("userSearhForm", new UserSearchForm());
              return "userSearch";
       }
       @RequestMapping(value = "/userSearch", method = RequestMethod.POST)
       public String userSearchPost(ModelMap model,@ModelAttribute @Validated(GroupOrders.GroupOrder.class)UserSearchForm userSearchForm,BindingResult result) {
              //バリデーションエラーなら検索ページへリダイレクト
              if(result.hasError())
                     return "userSearch";
              ~~~入力値にあった検索を行う処理~~~
              //検索結果のページへ
              return "userSearchResult";
       }
       ~~~省略~~~
}
```
