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
       │   │           └applicationContext.xml
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
次にテストを行うコントローラーは下記(今回テストは検索機能のテスト)
UserController.java
```java
@Controller
public class UserController {
 /*      ~~~省略~~~   */
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
/*              ~~~入力値にあった検索を行う処理~~~  */
              //検索結果のページへ
              return "userSearchResult";
       }
/*       ~~~省略~~~     */
}
```
次にユーザ検索のフォームは下記
UserSearchForm.java
```java
public class UserSearchForm{
       //未入力とパターンのバリデーション
       @NotEmpty(groups = group1.class)
       @Pattern(regexp = "^\\d{3}-\\d{4}$", groups = group2.class)
       private String zipCode;
       
       //未入力と長さのバリデーション
       @NotEmpty(groups = group1.class)
       @Size(max = 15, groups = group2.class)
       private String firstName;
       
       //未入力と長さのバリデーション
       @NotEmpty(groups = group1.class)
       @Size(max = 15, groups = group2.class)
       private String lastName;
       
       public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getZipCode() {
		return zipCode;
	}

	public void setZipCode(String zipCode) {
		this.zipCode = zipCode;
	}
}
```
最後にテストコードは下記
UserControllerTest.java
```java
```
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@ContextConfiguration(locations = { "file:src/main/webapp/applicationContext.xml"})
public class UserControllerTest{
	@Autowired
	private UserController userController;
	
	MockMvc mockMvc;

	Faker faker = new Faker(new Locale("ja_JP"));
	
	//viewでファイル拡張子をjspで作成しているためInternalResourceViewResolverで定義しcontrollerでjspが呼び出されるようにする。
	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setSuffix(".jsp");
		return resolver;
	}
	
	@Before 
	public void setup() {
		mockMvc = MockMvcBuilders.standaloneSetup(userController).setViewResolvers(viewResolver()).build();
	}
	
	//該当URLで適切なjspに遷移するかテスト
	//遷移できたかどうかはhttpレスポンスステータス200になったかで確認を行う
	@Test
	public void testURL遷移テスト()throws Exception{
		//検索画面への確認		
		mockMvc.perform(get("/userSearch")).andExpect(view().name("userSearch")).andExpect(status().isOk());
		//適切な入力だったら検索結果画面へ
		mockMvc.perform(post("/userSearch").param("zipCode", "222-3333").param("lastName", "テスト").param("firstName", "太郎")).andExpect(view().name("userSearchResult")).andExpect(status().isOk());
		
	}
}
