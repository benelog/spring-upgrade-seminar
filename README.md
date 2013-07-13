# Spring 3.0 -> 3.1 -> 3.2 따라잡기
- [Helloworld 제 5회 오픈 세미나](http://helloworld.naver.com/helloworld/416268)
- 발표자 소개
	- NHN Business Platform 웹플랫폼개발랩
	- Java 프레임워크 개발, 기술지원, 교육
	- Tech@NHN 시리즈 공저자, Helloworld 기고

- 얻을 수 있는 것
	- Spring 쓰시는 분
		- 실무에서 이슈가 되었던 부분(보안/성능/하위 호환성)
		- 버전업 체크리스트
	- Spring 안 쓰시는 분
		- 공통 모듈/ 프레임워크 개선 사례
		- 인터페이스, 클래스 설계 원칙
		- 성능 개선 기법
- 기대하지 말아야할 것
	- Spring 3.1, 3.2의 신규 기능의 자세한 소개
		- 참고자료로 대신 제공
	
## 버전을 올리면서 조심할 부분
### XSD 스키마 버전
	<beans 
		xmlns="http://www.springframework.org/schema/beans"
		xsi:schemaLocation=
			"http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd">

- jar에 없는 파일은 HTTP로 읽어옴 - 불필요한 성능 저하
- 서버에서 <http://www.springframework.org/schema/beans/>에 접근을 못한다면?

	nested exception is org.xml.sax.SAXParseException: cvc-elt.1: Cannot find the declaration of element 'beans'.

### Dependency 변화
- Optional dependency
	- spring-tx -> spring-core 등
	- 직접 선언하지 않으면 ClassNotFoundException
-  spring-asm 모듈이 spring-core로 통합
	- [[SPR-9669](https://jira.springsource.org/browse/SPR-9669)] Upgrade to ASM 4.0 and CGLIB 3.0 
	- 별도의 dependency 선언이 필요없어졌음.


### @MVC의 변화
- 3.1 부터 MVC기반 클래스 변경
	- <mvc:annotation-driven/>에 의해 등록되는 클래스들
	- HandlerMapping : DefaultAnnotationHandlerMapping -> RequestMappingHandlerMapping
	- HandlerAdaptor : AnnotationMethodHandlerAdapter -> RequestMappingHandlerAdapter
	- HandlerExceptionResolver : AnnotationMethodHandlerExceptionResolver -> ExceptionHandlerExceptionResolver 
	- 이름을 보면
		- Annotation은 이제 당연하므로 생략.
		- @RequestMapping이 연상되도록
- <mvc:annotation-driven/>를 쓸 때 옛날 클래스와 같이 쓰지 않도록 주의
- ArgumentResolver
	- 역할과 책임 분담
- HandlerInterceptor
	- 3번째 파라미터

## 개선 지점 심층 분석 
### ViewResolver의 Cache
- OOM 가능성
- 3.0 이전의 해결책
- 3.1의 해결책
- 3.2
	- OOM 가능성 방어 코드
	- 그래도 Cache효율성을 고려할 필요가 있음

### EL injection 방어
- <https://gist.github.com/benelog/4582041>

##당장 써야 할 신규기능
### 대표적 추가기능
- Spring 3.1
	- Cache absraction : @Cacheable
	- Profile : \<beans profile="dev"\> , @Profile("dev")
	- 환경변수 통합 관리 : -파일, OS환경변수, JNDI 등의 다양한 위치의 환경변수를 통합해서 ${} 표현식으로 표현.
	- javaConfig 확장 :  @~Enable
	- Test프레임워크에서 JavaConfig와 Profile 지원
	- c: namespace - \<bean id="employe eService" c:groupName="webpl".. /\>
- Spring 3.2
	- Servlet 3.0 기반 비동기 요청 처리
	- Spring MVC Test Framework
	- @ControllerAdvice
	- Matrix variables

### MVC Test

### 통합 Resource 관리

## 시사점
### 환경변화에 맞춤
- JavaEE
	- EL 2.2 때문에 생긴 보안 취약점 대처
	- JavaEE 신규 스펙 지원
- Java
	- JDK 1.4를 위한 배려는 그만
		- ViewResolver 캐쉬 개선
		- SimpleJdbcTemplate deprecated

### 완벽한 하위 호환성은 포기한 개선 지점
- 완벽한 하위 호환성을 일부 포기
	- @MVC artitecture : WebArgumentResolver 등
- 완충지대
	- Custom namespace : \<mvc:annotation-driven/\>, 
	- bean 등록 방식으로는 이전 방식도 지원
	- Adaptor : 
- 표준 스펙보다 빠른 개선 싸이클

### 신중한 접근
- 별도의 프로젝트로 검증 후 Spring 주류에 포함
	- Spring MVC test, Java Config

## 정리
# 업그레이드의 이득은?
	- 위험성 대처 : 보안, 버그 혹은 잘못 쓰기 쉬운 기능
	- 성능 개선
	- 신규 기능, 더 짧은 코드
	- 향후 4.0 따라가기 준비 : JDK8 지원
 
## 참고자료
### Spring 3.0 -> 3.1
- 레퍼런스 메뉴얼의 3.1 신규 기능 설명
	- <http://static.springsource.org/spring-framework/docs/3.1.x/spring-framework-reference/html/new-in-3.1.html>
- 스프링소스 블로그 자료
	- <http://blog.springsource.org/2011/02/11/spring-framework-3-1-m1-released/>
	- <http://blog.springsource.org/2011/02/14/spring-3-1-m1-introducing-profile/>
	- <http://blog.springsource.org/2011/02/15/spring-3-1-m1-unified-property-management/>
	- <http://blog.springsource.org/2011/06/09/spring-framework-3-1-m2-released/>
	- <http://blog.springsource.org/2011/06/10/spring-3-1-m2-configuration-enhancements/>
	- <http://blog.springsource.org/2011/06/21/spring-3-1-m2-testing-with-configuration-classes-and-profiles/>
	- <http://blog.springsource.org/2011/10/12/spring-framework-3-1-rc1-released/>
	- <http://blog.springsource.org/2011/12/13/spring-framework-3-1-goes-ga/>
	- <http://blog.springsource.org/2012/04/06/migrating-to-spring-3-1-and-hibernate-4-1/>
- Toby님의 Spring 3.1 분석 자료
	- [스프링 3.1 (1) JavaConfig의 위험성](http://toby.epril.com/?p=1168)
	- [스프링 3.1 (2) HandlerInterceptor의 적용순서](http://toby.epril.com/?p=1172)
	- [스프링 3.1 (3) @Enable~](http://toby.epril.com/?p=1174)
	- [스프링 3.1 (4) Static @Bean 메소드](http://toby.epril.com/?p=1177)
	- [스프링 3.1 (5) @Enable*을 이용한 설정 재활용 기법 세미나 동영상 & 자료](http://toby.epril.com/?p=1179)
	- [스프링 3.1 (6) web.xml의 활성 프로파일 설정](http://toby.epril.com/?p=1188)
	- [스프링 3.1 (7) 프로퍼티 소스 추상화와 PropertySourcePlaceholderConfigurer](http://toby.epril.com/?p=1191)
	- [스프링 3.1 (8) web.xml 없는 스프링 개발](http://toby.epril.com/?p=1205)
	- [스프링 3.1 (9) 애노테이션은 상속되지 않는다?](http://toby.epril.com/?p=1207)
	- [스프링 3.1 (10) 심심풀이 @RequestMapping 요청 조건 퀴즈](http://toby.epril.com/?p=1210)
	- [스프링 3.1 (11) 심심풀이 @RequestMapping 요청 조건 퀴즈 풀이](http://toby.epril.com/?p=1212)
	- 토비의 Spring 3.1 VOl2의 '4.9 스프링 3.1의 @MVC (p633-669)' : @MVC의 구조가 변한 이유를 자세히 설명
- 상세변경 이력
	- Deprecated : <http://static.springsource.org/spring-framework/docs/3.1.x/javadoc-api/deprecated-list.html>
	- 변경 패키지, 클래스 목록 : <http://static.springsource.org/spring-framework/docs/3.0.6.RELEASE_to_3.1.0.BUILD-SNAPSHOT/>
	- Change log : <http://static.springsource.org/spring/docs/3.1.x/changelog.txt>

### Spring 3.1 -> 3.2
- 레퍼런스 메뉴얼의 3.2 신규 기능 설명
	- <http://static.springsource.org/spring-framework/docs/3.2.x/spring-framework-reference/html/new-in-3.2.html>

- 스프링소스 블로그의 자료
	- <http://blog.springsource.org/2012/05/06/spring-mvc-3-2-preview-introducing-servlet-3-async-support/>
    	- <http://blog.springsource.org/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/>
    	- <http://blog.springsource.org/2012/05/10/spring-mvc-3-2-preview-making-a-controller-method-asynchronous/>
    	- <http://blog.springsource.org/2012/05/13/spring-mvc-3-2-preview-adding-long-polling-to-an-existing-web-application/>
    	- <http://blog.springsource.org/2012/11/05/spring-framework-3-2-rc1-released/>
    	- <http://blog.springsource.org/2012/11/12/spring-framework-3-2-rc1-spring-mvc-test-framework/>
    	- <http://blog.springsource.org/2012/12/13/spring-framework-3-2-goes-ga/>
- 상세변경 이력
	- Deprecated : <http://static.springsource.org/spring-framework/docs/3.2.x/javadoc-api/deprecated-list.html >
	- 변경 패키지, 클래스 목록 : <http://static.springsource.org/spring-framework/docs/3.1.3.RELEASE_to_3.2.0.RELEASE/>
	- Change log : <http://static.springsource.org/spring/docs/3.2.x/changelog.txt>

