# study-docker
docker를 사용하며 공부한 내용을 정리하는 Repo

# Docker + NextJS 배포

## About Docker

도커는 컨테이너 기반의 오픈소스 가상화 플랫폼입니다.
컨테이너는 격리된 공간에서 프로세스가 동작하는 기술

## https://docs.docker.com/

### 도커가 빠른 이유

- 도커 이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있기 때문에 보통 용량이 수백메가에 이릅니다.
- 처음 이미지를 다운 받을 때는 부담이 안되지만, 기존 이미지 파일에 파일 하나 추가했다고, 수백 메가를 다운 받기에는 부담이 너무 큽니다.

- 이러한 문제를 해결하고자 도커에는 레이어 라는 개념을 사용하고 유니온 파일 시스템을 이용하여 여러개의 레이어를 하나의 파일 시스템으로 사용할 수 있게 해줍니다!!

- 즉 A + B + C 에 D가 하나 추가되면 D 레이어만 다운받으면 되기 때문에 굉장히 효율적으로 이미지를 관리할 수 있습니다.

### 도커 이미지 업데이트

도커에서 컨테이너를 업데이트 하려면 새 버전의 이미지를 다운(pull)받고 기존 컨테이너를 삭제(stop, rm) 한 후 새 이미지를 기반으로 새 컨테이너를 실행(run)하면 됩니다.

**컨테이너를 삭제한다는 건 컨테이너에서 생성된 파일이 사라진다는 뜻입니다. 데이터베이스라면 그동안 쌓였던 데이터가 모두 사라진다는 것이고 웹 어플리케이션이라면 그동안 사용자가 업로드한 이미지가 모두 사라진다는 것입니다.**

도커 빌드는 임시 컨테이너 생성 > 명령어 수행 > 이미지로 저장 > 임시 컨테이너 삭제 > 새로 만든 이미지 기반 임시 컨테이너 생성 > 명령어 수행 > 이미지로 저장 > 임시 컨테이너 삭제 > … 의 과정을 계속해서 반복한다고 볼 수 있습니다. 명령어를 실행할 때마다 이미지 레이어를 저장하고 다시 빌드할 때 Dockerfile이 변경되지 않았다면 기존에 저장된 이미지를 그대로 캐시처럼 사용합니다.

### Docker Compose

지금까지 도커를 커맨드라인에서 명령어로 작업했습니다. 지금은 간단한 작업만 했기 때문에 명령이 길지 않지만 컨테이너 조합이 많아지고 여러가지 설정이 추가되면 명령어가 금방 복잡해집니다.

도커는 복잡한 설정을 쉽게 관리하기 위해 YAML 방식의 설정파일을 이용한 Docker Compose라는 툴을 제공합니다.

<!--  -->

참조
https://wnsgml972.github.io/setting/2020/07/20/docker/

<!--  -->

## 구성요소

### Dockerfile

```
# 환경 이미지 https://hub.docker.com/_/node
# alpine : 리눅스 경량 버전인 "알파인 리눅스" 이미지 설치를 명시한 것
FROM node:lts-alpine

WORKDIR /usr/src/app

# 의존성 목록 파일 복사
COPY package.json ./
COPY yarn.lock ./
# next.js 의존성 설치
RUN yarn
# 필요한 모든 파일을 복사
COPY . .
# next.js 앱 빌드
RUN yarn build

# 컨테이너 포트 설정
EXPOSE 3000

CMD ["yarn", "start"]

# 1. RUN
# 이 중에 RUN 명령어는 확연한 차이가 있습니다.
# RUN 명령어는 도커파일로부터 도커 이미지를 빌드하는 순간에 실행이 되는 명령어입니다.
# 그래서, RUN 명령어는 라이브러리 설치를 하는 부분에서 주로 활용이 됩니다.


# 2. CMD
# CMD 명령어는 RUN 명령어가 이미지를 빌드할 때 실행되는 것과 달리, 이미지로부터 컨테이너를 생성하여 최초로 실행할 때 수행됩니다.
```

참조 https://webruden.tistory.com/1060
참조 https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html

### .dockerignore

프로젝트 복사할 때, 필요 없는 부분 복사 방지

```
.next
.git
node_modules
.gitignore
```

## 사용법(cli)

```
docker build --tag PROJECTNAME . // 도커 빌드
ex)_
docker build --tag test-next .
// Macbook M1을 사용하는 사람들은 linux/arm64로 만들어지기 때문에 터미널로 --platform linux/amd64를 추가해줘야 한다.
docker build --platform linux/amd64 --tag test-next . // 일단 로컬에선 안적어도 문젠 없었음

docker ps -all // 도커 이미지 볼 수 있음(docker desktop app에서도 가능)


https://www.daleseo.com/docker-run/

// [호스트 포트]:[컨테이너 포트]
docker run -d -p 8080:3000 test-next
// name 설정으로 도커 컨테이너 아이디 대신 이름으로 컨트롤 가능
docker run -d --name docker-test -p 3000:3000 test-next
docker kill name or id // 프로세스 종료
docker rm name or id // 컨테이너 삭제

https://yoo11052.tistory.com/97?category=981650

docker run -p 4000:80 -d --rm 83f5acae904b // run 종료시 컨테이너 자동삭제


```
d
## EC2를 이용한 Docker 배포

https://velog.io/@nuri00/EC2%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-Docker-%EB%B0%B0%ED%8F%AC

## docker compose

https://www.daleseo.com/docker-compose/
https://darrengwon.tistory.com/793

## docker 로컬 개발환경 구성

https://www.44bits.io/ko/post/why-should-i-use-docker-container

### volume 사용법

**dockerignore에 등록된 파일 및 폴더여도, 폴더자체를 볼륨으로 가져오면 가져와버림**

https://velog.io/@1-blue/docker-volume-%EC%82%AC%EC%9A%A9%EB%B2%95
https://medium.com/dtevangelist/docker-%EA%B8%B0%EB%B3%B8-5-8-volume%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-data-%EA%B4%80%EB%A6%AC-9a9ac1db978c
