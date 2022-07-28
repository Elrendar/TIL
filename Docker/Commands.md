# 자주 사용하는 도커 명령어

[목록으로 돌아가기](/README.md)

## 목록

* ### `docker -version`

    도커의 버전을 출력한다.

* ### `docker pull {image name}`

    도커 리포지토리로부터 image를 가져온다.

* ### `docker run {옵션명} {image name}`

    도커 이미지로부터 컨테이너를 생성한다.

    예시)

    `docker run -dp 8080:8080 spring-hw2`

* ### `docker build {path to dockerfile}`

    경로의 도커파일로부터 이미지를 빌드한다.

* ### `docker ps`

    현재 실행중인 도커 컨테이너 목록을 출력한다.

* ### `docker ps -a`

    실행중이지 않은 도커 컨테이너까지 포함한 모든 컨테이너 목록을 출력한다.

* ### `docker images`

    로컬에 있는 모든 도커 이미지 목록을 출력한다.

* ### `docker rmi {image id}`

    도커 이미지를 삭제한다.

* ### `docker exec {옵션명} {container id} {명령어}`

    현재 실행중인 도커 컨테이너에 접속해 명령을 실행한다.

    예시)

* `docker exec -it {container id} bash`

* ### `docker stop {container id}`

    실행 중인 컨테이너를 정지한다.

* ### `docker kill {container id}`

    실행 중인 컨테이너를 강제종료한다.

* ### `docker rm {container id}`

    실행 중이 아닌 컨테이너를 삭제한다.

* ### `docker login`

    도커 허브에 로그인한다.

* ### `docker commit {conatainer id} {username/imagename}`

    로컬에 새로운 이미지를 생성한다.

* ### `docker push {username/imagename}`

    이미지를 도커 허브 리포지토리에 푸쉬한다.
