# application.yaml 환경 분리

> `application.yaml`은 애플리케이션 설정을 관리하는 파일이며, 환경별로 설정 파일을 분리해서 사용할 수 있다.

개발 환경과 운영 환경은 사용하는 데이터베이스, 외부 API 주소, 로그 레벨 등이 다를 수 있다.
이런 값들을 하나의 설정 파일에 모두 넣으면 관리가 어려워진다.

그래서 Spring Boot에서는 프로필을 사용해 환경별 설정 파일을 나눌 수 있다.

## 환경별 설정 파일

`application.yaml` 파일은 보통 `src/main/resources` 아래에 둔다.
환경별로 설정을 나누고 싶다면 다음과 같이 파일을 분리할 수 있다.

- 로컬 환경 : `application-local.yaml`
- 개발 환경 : `application-dev.yaml`
- 운영 환경 : `application-prod.yaml`

예를 들어 로컬 환경에서는 H2 DB와 로컬 API 주소를 사용하고, 운영 환경에서는 운영 DB와 운영 API 주소를 사용하도록 설정할 수 있다.

### 분리 예시

- 로컬 설정 : 별도의 DB 설치 없이 실행할 수 있도록 H2 DB를 사용하고, 기본 데이터를 넣기 위해 `seed-data.sql`을 실행한다.
- 운영 설정 : 운영 DB와 외부 API 주소를 환경 변수로 주입하고, 초기 SQL이 자동 실행되지 않도록 설정한다.

`application-local.yaml`은 다음과 같이 작성할 수 있다.

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:local
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
  sql:
    init:
      mode: always
      data-locations:
        - optional:file:./seed-data.sql

external:
  api:
    base-url: http://localhost:8081
```

`application-prod.yaml`은 다음과 같이 작성할 수 있다.

```yaml
spring:
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT:5432}/${DB_NAME}
    driver-class-name: org.postgresql.Driver
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  sql:
    init:
      mode: never

external:
  api:
    base-url: ${EXTERNAL_API_BASE_URL}
```

## 프로필 실행 방법

환경별 설정 파일을 사용하려면 실행할 때 활성화할 프로필을 지정해야 한다.

```bash
SPRING_PROFILES_ACTIVE=local ./gradlew bootRun
```

위 명령어는 `local` 프로필을 활성화한다.
이 경우 Spring Boot는 기본 설정인 `application.yaml`과 함께 `application-local.yaml`을 읽는다.

IDE에서 서버를 실행한다면 실행 설정에서 환경 변수나 VM 옵션으로 프로필을 지정할 수 있다.

## 테스트 환경 설정

테스트 전용 설정이 필요하다면 `src/test/resources` 아래에 `application.yaml`을 둘 수 있다.

```text
src/test/resources/application.yaml
```

테스트 실행 시에는 `src/test/resources`의 설정 파일이 우선 사용된다.
테스트 전용 설정 파일이 없다면 `src/main/resources`의 설정이 사용될 수 있다.
