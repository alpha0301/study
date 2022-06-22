# 개요

배포를 위한 빌드 및 도커 이미지 생성 시 용량 및 속도를 최적화하기 위해 필요한 설정입니다.

## 요약

- 빌드 설정: gradle 설정을 통한 layered spring boot jar 생성
- 패키징 설정: docker 이미지 생성 시 layered jar 압축 해제 후 COPY 명령어를 통해 소스 변경 layer 외에는 캐시를 활용하도록 설정
- 빌드, 패키징, 배포 적용: gitlab ci pipeline

# 어플리케이션 빌드 설정

gradle 설정을 통해 spring boot jar 파일을 layered 구조로 생성할 수 있습니다.

```gradle
jar {
    enabled = false
}

bootJar {
    layered {
        enabled = true
    }
    archiveFileName = 'app.jar'
}
```

gradle 7.4 기준으로 실행이 불가능한 jar 파일을 생성하는 jar 설정을 비활성화합니다.
bootJar.layered 설정을 활성화하여 실행 가능한 jar 파일을 생성할 때 layered 구조로 생성합니다.
아래는 spring boot 에서 지원하는 layered 구조입니다.

```
dependencies
snapshot-dependencies
spring-boot-loader
application
```

만일 신규 배포 버전이 의존성 변경 없이 소스만 수정할 경우, application 레이어만 변경되므로 도커 이미지 용량 및 빌드 속도, 실행 속도를 줄일 수 있습니다.

# 패키징 설정

```Dockerfile
# alias를 builder으로 등록합니다.
FROM amazoncorretto:17-al2-jdk as builder

# user, group 추가를 위한 명령어를 쓰기 위해 설치해줍니다.
RUN yum install -y yum-utils
RUN yum-config-manager --enable epel && yum update -y && yum -y install shadow-utils && yum clean all

RUN groupadd -g 999 spring
RUN useradd -r -u 999 -g spring spring
RUN mkdir /spring
RUN chown spring:spring /spring

USER spring
WORKDIR /spring
# gradle 빌드를 통해 생성한 boot jar 파일을 docker 환경으로 복사하고 layered 구조로 압축을 해제합니다.
COPY build/libs/*.jar .
RUN java -Djarmode=layertools -jar app.jar extract

# 실행 환경을 개별로 불러옵니다.
FROM amazoncorretto:17-al2-jdk

RUN yum install -y yum-utils
RUN yum-config-manager --enable epel && yum update -y && yum -y install shadow-utils xmlstarlet saxon augeas bsdtar unzip && yum clean all

RUN groupadd -g 999 spring
RUN useradd -r -u 999 -g spring spring
RUN mkdir /spring
RUN chown spring:spring /spring

USER spring
WORKDIR /spring
# 각 레이어 별로 COPY를 수행합니다. 변경되지 않은 레이어는 캐싱 처리됩니다.
COPY --from=builder spring/dependencies/ ./
COPY --from=builder spring/snapshot-dependencies/ ./
COPY --from=builder spring/spring-boot-loader/ ./
COPY --from=builder spring/application/ ./
EXPOSE 9060

CMD java org.springframework.boot.loader.JarLauncher
```
# CI pipeline 수행

```yml
stages:
  - build
  - test
  - package
  - deploy

build:
  image: amazoncorretto:17.0.3-alpine3.15
  stage: build
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 week
  only:
    - /^.*ci-test.*$/
    - main
    - tags
  script:
    - chmod +x gradlew
    - ./gradlew clean build -g $GRADLE_USER_HOME

docker-build:
  stage: package
  only:
    - main
    - tags
  script:
    - docker build -t $CI_HARBOR_URL/$CI_HARBOR_PROJECT/$CI_PROJECT_NAME:latest .

docker-deploy:
  stage: deploy
  only:
    - main
    - tags
  script:
    - docker login -u $CI_DOCKER_USER -p $CI_DOCKER_PASSWORD $CI_HARBOR_URL
    - docker push $CI_HARBOR_URL/$CI_HARBOR_PROJECT/$CI_PROJECT_NAME:latest
```
