OAuth_20
========

Samples for OAuth 2.0 Provider &amp; Client 

OAuth2 Provider & Consumer Sample입니다.
https://tools.ietf.org/html/rfc6749  를 근거하여 작성하였습니다.
이 코드는 샘플 코드입니다. 상업적인 목적으로 사용하지 말아주세요. 스터디 용도로만 사용해주세요.
환경설정에 대한 질문, 코드 자체에 대한 질문은 받지 않겠습니다.
OAuth2.0에 대한 질문도 위의 RFC 문서를 직접 읽어보세요.

상업적인 목적이 아니라면 이용가능한 코드입니다.
상업적인 목적으로는 사용하지 말아주세요.

이 코드의 저작권은 stepanowon@hotmail.com 에게 있습니다.

A. environment

  - Oracle 10g Express(H2 Database 사용가능)
  - Java 1.6 + Spring 3.1 + Eclipse(indigo) + Maven + iBatis 2.0 + Tomcat 6.0(HTTP Port 8000)

B. project

  - oauth2provider : 인증서버 & 리소스 서버
  - oauth2client : web server flow 클라이언트
  - oauth2client_agentflow : User Agent flow 클라이언트 

C. configuration

  - table 생성
    - DB는 oracle 10g express에 oauth2/oauth2 계정을 생성하여야 함.
    - oauth2provider의 src 디렉토리에 oauth2provider.sql 파일을 읽어 테이블 설치
      사용자 계정은 t1000, gdhong, arnold 세개의 계정(암호 동일)

  - 상수값 설정
    - net.oauth.v2 패키지의 OAuth2Constant클래스에서 상수값변경
      * USE_REFRESH_TOKEN : refresh token 기능을 사용할지 말지를 결정함.
      * AES_ENCRYPTION_KEY : 내부에서 토큰 생성시 사용할 AES 암호화 키 값
      * EXPIRES_IN_VALUE : refresh token 기능을 사용할 때 토큰의 유효기간(기본값:3600(초))
      
    - net.oauth.v2 패키지의 OAuth2Scope 클래스에서 상수값 변경
      * 조직에 따라 scope 상수값 설정(현재는 6개의 샘플 scope을 설정하였음)
      * resource 엔드포인트 url 별로 권한 설정(scope 지정)
      
D. endpoint
  - login & client app 등록
     * com.multi.oauth2.provider.view.controller.LoginController 클래스 참조
     * com.multi.oauth2.provider.view.controller.ClientController 클래스 참조
     
  - authorization
     * /oauth2provider/oauth2/auth
       - response_type 파라미터가 code일 경우는 web server flow
       - response_type 파라미터가 token일 경우는 user agent flow(Mobile App, Desktop포함)

  -  token  
     * /oauth2provider/oauth2/token
       - grant_type 파라미터가 authorization_code 인 경우는 server flow로 access token 발급
       - grant_type 파라미터가 refresh_token 인 경우는  access token을 갱신하게 됨.

  - protected resource
     * 이 샘플에서의 protected resource는 승인한 사용자의 계정 정보로 가정하였고, 
       endpoint는 /oauth2provider/resource/myinfo.do 이다.
     * 여러 protected resource에 대한 권한을 부여하기 위해 end point별 권한은 net.oauth.v2.OAuth2Scope 클래스의
       scopeUrlMap 필드에 Hashmap으로 작성한다
     * access token 정보 검증, scope 검증은 Interceptor(com.multi.oauth2.provider.util.Oauth2Interceptor)를 
       이용해 Controller 실행 전에 처리한다.
     * 예외 처리는 Controller 상에서 OAuth2Exception 을 발생시키면
       ExceptionResolver가 error 페이지로 이동시켜 OAuth2.0 spec에 따른 에러 코드와 메시지를 응답한다.
     * 클라이언트 앱이 user agent 타입으로 등록되었다면 Protected Resource 접근시 Cross Domain 문제를
       해결해줄 수 있도록 CORS(Cross Origin Resource Sharing)기법을 지원하도록 하였다.
  
  - 인증과정 또는 token 발급 과정에서 CSRF(Cross Site Request Forgery)공격에 대한 대응으로 
    OAuth2.0에서 recommended된 state 파라미터를 사용하였다.

E. 추가/개선할 사항..
  - OAuth 2.0 에서는 에러 발생시 WWW-Authenticate 헤더를 통해 응답하도록 하고 있으나
      Google, Facebook은 다른 방식을 사용하고 있다. 본 샘플은 facebook 스타일(?)로 작성하였다.

  - OAuth2.0 의 처리과정 중 Web Server flow 와 user agent flow만 처리하였다.
     * password credential과 client credential 방식은 작성하지 않았다. 대신 
       com.multi.oauth2.provider.view.controller.OAuth2Controller 클래스의 280번 라인에서 
       주석처리하여 향후 구현해야 함을 명시하였다.
     * refresh token을 사용할 것인지는 OAuth2Constant의 상수값을 변경하면 됨.
     * refresh token을 사용하지 않는 경우는 access token을 생성하지 않고 정해진 규칙에 따라
     	 생성하도록 하였음. --> 랜덤값으로 토큰을 생성하여 DB에 저장하도록 변경가능
  
  - 이 샘플에서는 redirect_uri 값을 비교하는 validation 과정을 거치므로 클라이언트 App을 등록할 때
    반드시 접근가능한 URL을 입력해야 함( localhost 허용)
     

F. 알림사항
   - 이 샘플은 잘 작동하지만 제대로된 설계없이 뚝딱거리면서 만들었음.
   - 따라서 잘 정리된 코드는 아님. 주석도 개발새발임.
   - 디버깅 목적으로 코드 사이사이에 콘솔 출력하는 코드가 많으니 알아서들 제거하고 테스트해야 함.

   - oracle 10g 대신에 다른 DB 쓸거면 maven dependency, applicationContext.xml, oauth2.xml, 
     oauth2provider.sql 파일을 수정하여 쓰면 됨.
   
   
   
   
G. OAuth2.0 Client
  - oauth2Client 
    * Web Server flow로 처리하도록 만든 client임.
    * HTTP 통신을 위해 apache common의 HttpClent 클래스 사용
    * client 각 요소는 jsp로 간단히 작성
    * Settings.java 파일을 찾아 client_id, client_secret, 각각의 endpoint uri를 변경한 후 실행함.
     
  - oauth2client_agentflow
    * User Agent Flow 방식의 Client임.
    * html 파일로 작성
    * jQuery를 사용한 웹앱, webview를 통해 인증하고 access token을 획득하는 모바일앱. 이렇게 두가지의 경우
      이 코드를 참조할 수 있음.
    * index.html과 callback.html의 내부의 client_id, client_secret, 각각의 endpoint를 설정하고 실행함.
   
  - 실행에 앞서 client app 을 인증서버(oauth2provider)에 등록해야 함.
  
  20 ~ oauth

= = = = = = = =



Samples for OAuth 2.0 Provider &client



OAuth2 Provider & Consumer Sample。

tools https:/ / html ietf . org”,并根据rfc6749 /制作。

这个代码是样本代码。不要用于商业目的。请只作为学习用。

不接受关于配置的问题，代码本身的问题。

关于OAuth2.0的问题，也可以直接阅读上面的RFC文档。



如果不是为了商业目的，它是可以被使用的代码。

不要用于商业目的。



此代码的版权在stepanowon@hotmail.com。



因为a。



- Oracle 10g Express(H2 Database可用)

- Java 1.6 + Spring 3.1 + Eclipse(indigo) + Maven + iBatis 2.0 + Tomcat 6.0(HTTP Port 8000)



b。project



- oauth2provider:认证服务器&资源服务器

- oauth2client: web server flow客户端

- oauth2client_agentflow: User Agent flow客户端



c configuration。



-创建table

- DB必须在oracle 10g express中创建oauth2/oauth2帐户。

——oauth2provider的src目录中的oauth2provider。读取sql文件安装表

用户帐号为t1000, gdhong和arnold三个账户(相同密码)



-设定常数值

- net。oauth。从v2包的OAuth2Constant类更改常量

token:决定是否使用refresh token功能

key:内部生成代币时使用的加密密钥值

使用repis_in_value: refresh token功能时，代币的有效期(基本值:3600秒)



- net。oauth。在v2分组OAuth2Scope类中更改常量

*根据组织设定scope常数值(目前设置了6种样品scope)

* resource endent按url设定权限(scope)



d endpoint。

-注册login & client app

* . com provider。oauth2。multi。”controller。view。参考LoginController类

* . com provider。oauth2。multi。”controller。view。参考ClientController类



- authorization

oauth2provider oauth2 / / / * auth

如果type参数是code的话，那么web server flow

如果type参数为token，则user agent flow(包括Mobile App, Desktop)



token -

oauth2provider oauth2 / / / * token

如果是code的话，可以使用server flow发放access token

如果是，grant_type参数为refresh_token的话，可以更新access token。



- protected resource

*这个样品中的protected resource是假定得到了用户的帐号信息，

endpoint resource oauth2provider / / /在myinfo。do。

*各种protected resource权限赋予各point end为了oauth。net权限。v2。oauth2scope class的

在scopeUrlMap网址上用Hashmap填写

* access token scope验证、信息。com(interceptor验证。provider oauth2。multi util。oauth2interceptor)。

利用Controller在运行前进行处理。

*例外处理在Controller上产生OAuth2Exception

ExceptionResolver移动到error页面，回答OAuth2.0 spec的错误代码和信息。

*•布莱恩特app user agent类型,如果登记resource protected domain问题基本上cross

能够解决cors(cross origin resource sharing),并支援技法。



为应对CSRF(Cross Site Request Forgery)攻击，

recommended 0到。oauth2파라미터state的使用。



E.需要追加/改进的事项..

OAuth 2.0中的任何错误都是通过WWW-Authenticate header来回答的，

Google, Facebook有不同的方式。本样品采用facebook样式(?)制作。



- oauth2。0的处理过程中server web flow和user agent flow只处理。

* password credential和client credential没有制作方式。相反

oauth2。com”。provider。multi controller。view。OAuth2Controller的280号，

今后处理主席应明确体现。

* refresh token是否使用的oauth2constant常数的价格随着变更。

* refresh不使用token token access的情况是,并没有生成的,根据规则。

创建了大脑。->可更改为创建随机代币并存储在数据库中



-在样本redirect __ uri价格比较validation过程,所以经过·布莱恩特app登记时

必须输入可访问的URL(允许localhost)





f .通知事项。

-李样本是正确的,但是运作良好,距离宏设计制作。

-所以不是很好的代码。主席也是开发者。

-디버깅目的意气相投视觉上的编码打印索内容都知道很多,所以应该去除测试。



- oracle代替10g maven如果使用其他数据库applicationcontext、dependency。xml、oauth2 xml、。

oauth2provider。可以修改sql文件。









G. OAuth2.0 Client

- oauth2client

* web server flow处理client制作。

*公示板通信apache common的httpclent class使用

* client各要素的jsp制作简单,

* settings。java文件client找到__ id、client __ secret、各endpoint uri变更后实行。



- oauth2client agentflow __

* user agent flow client的方式。

* html拟定文件

* jquery使用的网络应用程序,通过webview token access并认证的移动应用程序。上述两种情况

这一代码可以参考。

* index)。html和callback。html的内部client __ id、client __ secret、各endpoint,设定并实施。



-实行之前client app认证oauth2provider服务器(应)注册。
  



      
    
        

  
