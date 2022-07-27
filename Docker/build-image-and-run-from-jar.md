# .jar 파일로부터 Docker 이미지 빌드 및 실행하기

[목록으로 돌아가기](/README.md)

## Dockerfile

먼저 도커 이미지를 빌드하려면 Dockerfile이 필요 하다.

Dockerfile은 확장자가 없는 텍스트 파일이다.

```dockerfile
# 빌드 및 실행에 필요한 jdk를 명시한다.
# 로컬에 없으면 도커가 알아서 해당 이미지를 도커 허브에서 다운 받아 사용한다.
FROM azul/zulu-openjdk:17
# jar 파일의 상대경로(Dockerfile 기준)를 적는다. 스프링 프로젝트 루트에 있다면 /build/lib/*.jar 같은 식이다.
ARG JAR_FILE=*.jar
# 구체적인 파일이름을 적는다.
COPY ${JAR_FILE} Homework2-0.0.1-SNAPSHOT.jar
# 터미널에서 자바 코드를 실행할 때 쓰는 명령어이다.
ENTRYPOINT ["java","-jar","/Homework2-0.0.1-SNAPSHOT.jar"]
```

## 도커 이미지 빌드하기

Dockerfile의 작성이 끝났다면, 아래의 명령어로 도커 이미지를 빌드할 수 있다.

```docker
docker build -t {임의의 이름:태그} {경로}
```

예시:

```docker
docker build -t spring-hw2:ver1 .
```

(`:` 뒤에 태그를 입력하지 않을 경우, 자동으로 lastest가 들어가는 것 같다.)

## 빌드된 이미지 확인하기

```docker
docker images

REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
spring-hw2          ver1    c141faab584b   25 minutes ago   440MB
azul/zulu-openjdk   17        d5910bcf06da   2 hours ago      398MB
```

이 명령어를 사용하면 현재 로컬에 존재하는 이미지 목록을 확인할 수 있다. 여기에는 내가 빌드한 이미지와, 도커가 허브에서 다운로드 한 이미지들이 포함되어 있다.

위의 목록에서 `azul/zulu-openjdk`가 바로 `Dockerfile`을 작성할 때 맨 첫 줄에 적은 그것이다.

## 이미지를 컨테이너에 넣어 실행하기

아래와 같은 명령어로 컨테이너를 실행할 수 있다.

```docker
docker run {옵션} {이미지 식별자} {명령어} {인자}
```

예시:

```docker
docker run -dp 8080:8080 spring-hw2
```

`-d` 옵션을 넣으면 컨테이너가 detached 모드에서 실행되며, 실행 결과로 컨테이너 ID가 출력된다.

이렇게 실행된 컨테이너는 터미널이 종료되도 계속 백그라운드에 남아서 실행된다.

`-p` 옵션은 호스트와 컨테이너 간의 포트(port) 배포(publish)/바인드(bind)를 위해서 사용된다고 한다.

대충 이 옵션이 있어야 `주소:8080`으로 접속할 수 있다는 것은 알겠는데, 아직 포트에 대해 공부가 부족해서 이 부분은 추후 보충할 예정이다.
