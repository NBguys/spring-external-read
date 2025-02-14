# 외부설정과 프로필2

### 외부 설정 사용 - Environment

다음과 같은 외부 설정들은 스프링이 제공하는 Environment 를 통해서 일관된 방식으로 조회할 수 있다.

#### 외부 설정

* 설정 데이터( application.properties )
* OS 환경변수
* 자바 시스템 속성
* 커맨드 라인 옵션 인수

> > > 참고 - properties 캐밥 표기법
> > > properties 는 자바의 낙타 표기법( maxConnection )이 아니라 소문자와 - (dash)를 사용하는 캐밥
> > > 표기법( max-connection )을 주로 사용한다. 참고로 이곳에 자바의 낙타 표기법을 사용한다고 해서 문제가 되
> > > 는 것은 아니다. 스프링은 properties 에 캐밥 표기법을 권장한다.

#### MyDataSourceEnvConfig.java 참고

* MyDataSource 를 스프링 빈으로 등록하는 자바 설정이다.
* Environment 를 사용하면 외부 설정의 종류와 관계없이 코드 안에서 일관성 있게 외부 설정을 조회할 수 있다.
* Environment.getProperty(key, Type) 를 호출할 때 타입 정보를 주면 해당 타입으로 변환해준다. (스프링 내부 변환기가 작동한다.)
    * env.getProperty("my.datasource.etc.max-connection", Integer.class) : 문자 숫자로 변환
    * env.getProperty("my.datasource.etc.timeout", Duration.class) : 문자 Duration (기간) 변환
    * env.getProperty("my.datasource.etc.options", List.class) : 문자 List 변환 ( A,B [A,B] )

스프링은 다양한 타입들에 대해서 기본 변환 기능을 제공한다.

> > > 속성 변환기 - 스프링 공식 문서
> > > https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config.typesafe-configuration-properties.conversion

### ExternalReadApplication.java 참고

* 설정 정보를 빈으로 등록해서 사용하기 위해 @Import(MyDataSourceEnvConfig.class) 를 추가했다.
* @SpringBootApplication(scanBasePackages = "hello.datasource")
    * 예제에서는 @Import 로 설정 정보를 계속 변경할 예정이므로, 설정 정보를 바꾸면서 사용하기 위해 hello.config 의 위치를 피해서 컴포넌트 스캔 위치를 설정했다.
    * scanBasePackages 설정을 하지 않으면 현재 위치인 hello 패키지부터 그 하위가 모두 컴포넌트 스캔이 된다. 따라서 @Configuration 을 포함하고 있는
      MyDataSourceEnvConfig 이 항상 컴포넌트 스캔의 대상이 된다.

### 외부설정 사용 - @Value

@Value 를 사용하면 외부 설정값을 편리하게 주입받을 수 있다.
참고로 @Value 도 내부에서는 Environment 를 사용한다.

#### MyDataSourceValueConfig.java 참고

* @Value 에 ${} 를 사용해서 외부 설정의 키 값을 주면 원하는 값을 주입 받을 수 있다.
* @Value 는 필드에 사용할 수도 있고, 파라미터에 사용할 수도 있다.
    * myDataSource1() 은 필드에 주입 받은 설정값을 사용한다.
    * myDataSource2() 는 파라미터를 통해서 설정 값을 주입 받는다.

기본값  
만약 키를 찾지 못할 경우 코드에서 기본값을 사용하려면 다음과 같이 : 뒤에 기본값을 적어주면 된다.

* 예) @Value("${my.datasource.etc.max-connection:1}") : key 가 없는 경우 1 을 사용한다.

### 외부설정 사용 - @ConfigurationProperties 시작

#### MyDataSourcePropertiesV1.java 참고

}

* 외부 설정을 주입 받을 객체를 생성한다. 그리고 각 필드를 외부 설정의 키 값에 맞추어 준비한다.
* @ConfigurationProperties 이 있으면 외부 설정을 주입 받는 객체라는 뜻이다. 여기에 외부 설정
  KEY의 묶음 시작점인 my.datasource 를 적어준다.
* 기본 주입 방식은 자바빈 프로퍼티 방식이다. Getter , Setter 가 필요하다. (롬복의 @Data 에 의해 자
  동 생성된다.)

#### MyDataSourceConfigV1.java 참고

* @EnableConfigurationProperties(MyDataSourcePropertiesV1.class)
    * 스프링에게 사용할 @ConfigurationProperties 를 지정해주어야 한다. 이렇게 하면 해당 클래
      스는 스프링 빈으로 등록되고, 필요한 곳에서 주입 받아서 사용할 수 있다.
* private final MyDataSourcePropertiesV1 properties 설정 속성을 생성자를 통해 주입 받
  아서 사용한다

### 외부설정 사용 - @ConfigurationProperties 생성자

@ConfigurationProperties 는 Getter, Setter를 사용하는 자바빈 프로퍼티 방식이 아니라 생성자를 통해서 객
체를 만드는 기능도 지원한다. 다음 코드를 통해서 확인해보자.

#### MyDataSourcePropertiesV2.java 참고

* 생성자를 만들어 두면 생성자를 통해서 설정 정보를 주입한다.
* @Getter 롬복이 자동으로 getter 를 만들어준다.
* @DefaultValue : 해당 값을 찾을 수 없는 경우 기본값을 사용한다.
    * @DefaultValue Etc etc
        * etc 를 찾을 수 없을 경우 Etc 객체를 생성하고 내부에 들어가는 값은 비워둔다. ( null , 0 )
    * @DefaultValue("DEFAULT") List<String> options
        * options 를 찾을 수 없을 경우 DEFAULT 라는 이름의 값을 사용한다.

> > > 참고 @ConstructorBinding
> > > 스프링 부트 3.0 이전에는 생성자 바인딩 시에 @ConstructorBinding 애노테이션을 필수로 사용해야 했다.
> > > 스프링 부트 3.0 부터는 생성자가 하나일 때는 생략할 수 있다. 생성자가 둘 이상인 경우에는 사용할 생성자
> > > 에 @ConstructorBinding 애노테이션을 적용하면 된다.

#### MyDataSourceConfigV2.java 참고

* MyDataSourcePropertiesV2 를 적용하고 빈을 등록한다. 기존 코드와 크게 다르지 않다.

### 외부설정 사용 - @ConfigurationProperties 검증

@ConfigurationProperties 를 통해서 숫자가 들어가야 하는 부분에 문자가 입력되는 문제와 같은 타입이 맞지  
않는 데이터를 입력하는 문제는 예방할 수 있다. 그런데 문제는 숫자의 범위라던가, 문자의 길이 같은 부분은 검증이 어렵다.  
예를 들어서 최대 커넥션 숫자는 최소 1 최대 999 라는 범위를 가져야 한다면 어떻게 검증할 수 있을까? 이메일을 외부  
설정에 입력했는데, 만약 이메일 형식에 맞지 않는다면 어떻게 검증할 수 있을까?  
개발자가 직접 하나하나 검증 코드를 작성해도 되지만, 자바에는 자바 빈 검증기(java bean validation)이라는 훌륭한  
표준 검증기가 제공된다.

자바 빈 검증기를 사용하려면 spring-boot-starter-validation 이 필요하다. build.gradle 에 다음 코드
를 추가하자.
build.gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-validation' //추가
```

#### MyDataSourcePropertiesV3.java 참고

* @NotEmpty url , username , password 는 항상 값이 있어야 한다. 필수 값이 된다.
* @Min(1) @Max(999) maxConnection : 최소 1 , 최대 999 의 값을 허용한다.
* @DurationMin(seconds = 1) @DurationMax(seconds = 60) : 최소 1, 최대 60초를 허용한다.

jakarta.validation.constraints.Max  
패키지 이름에 jakarta.validation 으로 시작하는 것은 자바 표준 검증기에서 지원하는 기능이다.  
org.hibernate.validator.constraints.time.DurationMax  
패키지 이름에 org.hibernate.validator 로 시작하는 것은 자바 표준 검증기에서 아직 표준화 된 기능은 아니  
고, 하이버네이트 검증기라는 표준 검증기의 구현체에서 직접 제공하는 기능이다. 대부분 하이버네이트 검증기를 사용  
하므로 이 부분이 크게 문제가 되지는 않는다

#### MyDataSourceConfigV3.java 참고

* MyDataSourceConfigV3 은 기존 코드와 크게 다르지 않다

#### ConfigurationProperties 장점

* 외부 설정을 객체로 편리하게 변환해서 사용할 수 있다.
* 외부 설정의 계층을 객체로 편리하게 표현할 수 있다.
* 외부 설정을 타입 안전하게 사용할 수 있다.
* 검증기를 적용할 수 있다.

### YAML

스프링은 설정 데이터를 사용할 때 application.properties 뿐만 아니라 application.yml 이라는 형식도 지원한다.  
YAML(YAML Ain't Markup Language)은 사람이 읽기 좋은 데이터 구조를 목표로 한다. 확장자는 yaml , yml 이다. 주로 yml 을 사용한다.

application.properties 예시

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

application.yml 예시

```yaml
environments:
  dev:
    url: "https://dev.example.com"
    name: "Developer Setup"
  prod:
    url: "https://another.example.com"
    name: "My Cool App" 
```

* YAML의 가장 큰 특징은 사람이 읽기 좋게 계층 구조를 이룬다는 점이다.
* YAML은 space (공백)로 계층 구조를 만든다. space 는 1칸을 사용해도 되는데, 보통 2칸을 사용한다.
  일관성있게 사용하지 않으면 읽기 어렵거나 구조가 깨질 수 있다.
* 구분 기호로 : 를 사용한다. 만약 값이 있다면 이렇게 key: value : 이후에 공백을 하나 넣고 값을 넣어 주면 된다.

application.properties 를 사용하지 않도록 파일 이름을 변경하자.
application.properties application_backup.properties

```properties
my.datasource.url=local.db.com
my.datasource.username=local_user
my.datasource.password=local_pw
my.datasource.etc.max-connection=1
my.datasource.etc.timeout=3500ms
my.datasource.etc.options=CACHE,ADMIN 
```

src/main/resources/application.yml 를 생성하자.

```yaml
my:
  datasource:
    url: local.db.com
    username: local_user
    password: local_pw
    etc:
      max-connection: 1
      timeout: 60s
      options: LOCAL, CACHE
        ...
```

#### application.yml 참고

* yml 은 --- dash( - ) 3개를 사용해서 논리 파일을 구분한다.
* spring.config.active.on-profile 을 사용해서 프로필을 적용할 수 있다.
* 나머지는 application.properties 와 동일하다.

### @Profile

프로필과 외부 설정을 사용해서 각 환경마다 설정값을 다르게 적용하는 것은 이해했다.  
그런데 설정값이 다른 정도가 아니라 각 환경마다 서로 다른 빈을 등록해야 한다면 어떻게 해야할까?  
예를 들어서 결제 기능을 붙여야 하는데, 로컬 개발 환경에서는 실제 결제가 발생하면 문제가 되니 가짜 결제 기능이 있  
는 스프링 빈을 등록하고, 운영 환경에서는 실제 결제 기능을 제공하는 스프링 빈을 등록한다고 가정해보자.

#### PayClient.java 참고

* DI를 적극 활용하기 위해 인터페이스를 사용한다.

#### LocalPayClient.java 참고

* 로컬 개발 환경에서는 실제 결제를 하지 않는다.

#### ProdPayClient.java 참고

* 운영 환경에서는 실제 결제를 시도한다

#### OrderService.java 참고

* PayClient 를 사용하는 부분이다. 상황에 따라서 LocalPayClient 또는 ProdPayClient 를 주입받는다.

#### PayConfig.java 참고

* @Profile 애노테이션을 사용하면 해당 프로필이 활성화된 경우에만 빈을 등록한다.
    * default 프로필(기본값)이 활성화 되어 있으면 LocalPayClient 를 빈으로 등록한다.
    * prod 프로필이 활성화 되어 있으면 ProdPayClient 를 빈으로 등록한다.

#### RunOrder.java 참고

ApplicationRunner 인터페이스를 사용하면 스프링은 빈 초기화가 모두 끝나고 애플리케이션 로딩이 완료되는 시
점에 run(args) 메서드를 호출해준다.

#### ExternalReadApplication.java 변경
* 실행하기 전에 컴포넌트 스캔 부분에 hello.pay 패키지를 추가하자.

#### @Profile의 정체 
```java
package org.springframework.context.annotation;
...
@Conditional(ProfileCondition.class)
public @interface Profile {
String[] value();
}
```
@Profile 은 특정 조건에 따라서 해당 빈을 등록할지 말지 선택한다. 어디서 많이 본 것 같지 않은가? 바로 @Conditional 이다.  
코드를 보면 @Conditional(ProfileCondition.class) 를 확인할 수 있다.  
스프링은 @Conditional 기능을 활용해서 개발자가 더 편리하게 사용할 수 있는 @Profile 기능을 제공하는 것이다.