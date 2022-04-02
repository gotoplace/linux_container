## docker 강의 내용 정리 : [따배도 - 이성미 강사님](https://www.youtube.com/watch?v=NLUugLQ8unM&list=PLApuRlvrZKogb78kKq1wRvrjg1VMwYrvi)
* Youtube 강의 내용을 개인적으로 정리한 내용이므로 오타 또는 오류가 있을 수 있습니다.
* 아래 작성 내용에는 container에 대한 상세 설명이 포함되지 않았으므로, Youtube 강의 내용을 참고하여 직접 따라해보는 것을 추천합니다.
* 아래 // 주석으로 표시한 설명은 dockerfile이나 작성 파일 내용에는 포함되지 않아야 합니다.

docker 설치 (ubuntu, centos)
-----

> 공식 사이트 doc 참고
* [docs.docker.com](https://docs.docker.com/)
* [Ubuntu 20.04](https://docs.docker.com/engine/install/ubuntu/)
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc

$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

$ sudo docker version
// client, server 정보가 보여야 함

// docker 실행 권한 부여
# usermod -a -G docker chris
// logout / login하여 확인 가능
```

* [CentOS 7](https://docs.docker.com/engine/install/centos/)
* sudo 사용하지 않고, root로 직접 진행
```
# yum install -y yum-utils
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# yum install docker-ce docker-ce-cli containerd.io -y

// CentOS는 service 시작 및 등록을 직접 해야 함
# systemctl start docker
# systemctl enable docker

# usermod -a -G docker chris
// logout / login하여 확인 가능
```

*****

docker commands
-----

> image 찾기
* docker search nginx : 예) nginx image 찾기

> image 받기 (hub.docker.com에서...)
* docker pull nginx:latest : 예) nginx latest로 받아오기
  * image 저장 경로 : /var/lib/docker/overlay2
  * 동영상에서는 latest 붙이지 않았으며 레이어는 5개라고 설명되었지만, 2022.03.24 기준으로 레이어는 6개로 확인됨
```
// 일반 계정으로 실행 권장
$ docker pull nginx
```

> image 리스트 보기
* docker images 또는
* docker image ls

> image 실행
* docker run -d --name web -p 80:80 nginx:latest
  * 동영상에서 실행시 "docker run --name web -d -p 80:80 nginx" 사용
  * 일반 계정으로 실행 권장, 실행 후 container ID가 보임
```
$ docker run -d --name web -p 80:80 nginx

f91dcbf92b53ede8dcadfb49c4dcd0aaddee1f6a983454fba775ce13aa195d7a

// 아래와 같이 실행하면 html 파일 내용을 보여줘야 함
$ curl localhost:80
```

> 실행 중인 container 관리
* 실행 중인 container 확인
```
$ docker ps
```
   
* container 실행 중단
```
$ docker stop containername
```
   
* container 삭제 (engine에서만 내리는 것)
```
$ docker rm containername
```
   
* container image 삭제
```
$ docker rmi containername
```
* docker rm image containername 동작 안함
* 삭제 되었는지 /var/lib/docker/overlay2 확인해 볼 것

*****

docker image 생성 및 배포
-----

> dockerfile
* 쉽고, 간단, 명확한 구문을 가진 text file로 Top-down 해석
* 컨테이너 이미지를 생성할 수 있는 고유의 지시어 (Instruction)를 가짐
* 대소문자 구분하지 않으나 가독성을 위해 사용함
* 동영상 예제에서 사용된 hello.js
```js
const http = require('http');
const os = require('os');
console.log("Test server starting...");

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("Container Hostname: " + os.hostname() + "\n");
};

var www = http.createServer(handler);
www.listen(8080);
```
* docker image 생성
```
$ mkdir build
$ cd build
$ vi dockerfile
FROM node:12
COPY hello.js /
CMD ["node", "/hello.js"]

$ docker build -t imagename:tag .
```

> dockerfile 문법 (자주 사용되는 것)
* \# : comment
* FROM : 컨테이너의 BASE IMAGE (운영환경)
* MAINTAINER : 이미지를 생성한 사람의 이름 및 정보
* LABEL : 컨테이너 이미지에 컨테이너의 정보를 저장
* RUN : 컨테이너 빌드를 위해 base image에서 실행할 commands
* COPY : 컨테이너 빌드시 호스트의 파일을 컨테이너로 복사
* ADD : 컨테이너 빌드시 호스트의 파일 (tar, url 포함)을 컨테이너로 복사
* WORKDIR : 컨테이너 빌드시 명령이 실행될 작업 디렉토리 설정
* ENV : 환경변수 지정
* USER : 명령 및 컨테이너 실행시 적용할 유저 설정
  * 보안 관점에서 root가 아닌 system 계정을 따로 두고 운영하는 것이 좋음
* VOLUME : 파일 또는 디렉토리를 컨테이너의 디렉토리로 마운트
* EXPOSE : 컨테이너 동작 시 외부에서 사용할 포트 지정
* CMD : 컨테이너 동작 시 자동으로 실행할 서비스나 스크립트 지정
* ENTRYPOINT : CMD와 함께 사용하면서 command 지정 시 사용

> docker image 배포
* public hub, private hub 모두 배포 가능

*****

실습 - nodejs 기반 hellojs 컨테이너
-----

> dockerfile 생성 및 실행 command
* dockerfile 내용 : hello_js라는 디렉토리를 하나 생성하고 진행
```
$ vi dockerfile
```
```
FROM node:12
COPY hello.js /
CMD ["node", "/hello.js"]
```

* 컨테이너 생성 command
  * base가되는 node:12를 받아오고
  * hello.js를 이미지에 포함한다.
  * latest는 안 붙여도 자동으로 latest가 된다.
```
$ docker build -t hellojs:latest .
```

* 컨테이너 실행
  * latest tag는 붙이지 않아도 되지만, latest가 아닌 tag는 필히 붙여야 한다.
```
$ docker run -d -p 8080:8080 --name web hellojs
```

* 서비스 동작 여부 확인
```
$ docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
74c9047cb909   hellojs   "docker-entrypoint.s…"   3 seconds ago   Up 3 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   web

$ curl localhost:8080
Container Hostname: 74c9047cb909
```

* 컨테이너 삭제
```
$ docker rm -f web
```

*****

실습 - 우분투 기반의 웹 서버 컨테이너 만들기
-----

> dockerfile 생성 및 실행 command
* dockerfile 내용
```
FROM ubuntu:18.04
LABEL maintainer="Chris Choe"
# install apache
RUN apt-get update \
    && apt-get install -y apache2
RUN echo "TEST WEB" > /var/www/html/index.html
EXPOSE 80
CMD ["/usr/sbin/apache2ctl", "-DFOREGROUND"]
```

* 컨테이너 생성 command
```
$ docker build -t webserver:v1 .
```

* 컨테이너 실행
  * latest tag는 붙이지 않아도 되지만, latest가 아닌 tag는 필히 붙여야 한다.
```
$ docker run -d -p 80:80 --name web webserver:v1
```

* 서비스 동작 여부 확인
```
$ docker ps
```

* localhost에서 접근 하는 예제
```
$ curl localhost:80

// 결과
TEST WEB
```

* 현재 게스트 OS가 아닌 다른 게스트 OS에서 접근 하는 예제
```
$ curl vm-ubuntu-s:80

// 결과
TEST WEB
```

* 컨테이너 삭제
```
$ docker rm -f web
```

*****

컨테이너 배포하기
-----

> docker ID 생성, 로그인
* 각자 알아서 생성하기
* docker login

> tag를 붙여주고, push 해야함
* docker images로 확인
* tag 붙이기
```
$ docker tag webserver:v1 flatsixna/webserver:v1
$ docker tag hellojs:latest flatsixna/hellojs:latest

$ docker images

// IMAGE ID가 같으므로 이미지는 동일하다
REPOSITORY            TAG       IMAGE ID       CREATED             SIZE
hellojs               latest    0c18476c6472   29 minutes ago      918MB
flatsixna/hellojs     latest    0c18476c6472   29 minutes ago      918MB
flatsixna/webserver   v1        bca1dc508dca   About an hour ago   200MB
webserver             v1        bca1dc508dca   About an hour ago   200MB
node                  12        5b6c488f14dd   5 days ago          918MB
ubuntu                18.04     b67d6ac264e4   6 days ago          63.2MB

```
* push 하기
  * [repositories](https://hub.docker.com/repositories) 에서 확인 가능
```
$ docker push flatsixna/webserver:v1
// latest 생략 가능
$ docker push flatsixna/hellojs
```

*****

컨테이너 보관 창고 (Registry)
-----

> Registry
* docker hub
  * [hub.docker.com](https://hub.docker.com/)
  * Official image 업로드, 다운로드 가능
  * 계정 필요
  * cli docker search 로 검색 가능
* private registry
  * 사내에서 private하게 운영할 수 있는 registry
  * [docker registry 받아서 실행하면 private registry 사용 가능](https://hub.docker.com/_/registry)

* 사용 예제, localhost에서 private registry를 운영할 수 있음
```
$ docker run -d -p 5000:5000 --restart always --name registry registry:2
$ docker pull ubuntu
$ docker tag ubuntu localhost:5000/ubuntu
$ docker push localhost:5000/ubuntu
```

> public registry에서 받고 계정에 연결된 registry에 업로드
* 아래 순서로 진행
  *  flatsixna는 개인 계정이므로 각자 생성한 계정을 사용한다.
```
$ docker login
$ docker pull httpd:latest
$ docker tag httpd:latest flatsixna/httpd:latest
$ docker push flatsixna/httpd:latest
// hub.docker.com에서 확인
```

> private registry 운영
* 아래 순서로 진행
  * registry 이미지가 없으면 자동으로 다운 받고 실행
```
$ docker run -d -p 5000:5000 --restart always --name registry registry:2

// private registry에 올릴 때 이름이 중요, localhost 또는 domain, IP address가 될 수 있음
$ docker tag httpd:latest localhost:5000/httpd:latest
$ docker push localhost:5000/httpd:latest

// 아래 경로에 registry가 운영되고있고, 하위 디렉토리에 업로드 됨.
/var/lib/docker/volumes/6ef9e49da167930d2dd2611a0309e703781c4b7cc2268d8839c2f2cad12c91be/_data/docker/registry/v2/repositories/httpd
```
* TODO
  * ubuntu 20.04 base에 vi 및 zsh, 자주 사용하는 tool 환경 구성하여 이미지 생성 및 push
  * CentOS 7 base에 vi 및 zsh, 자주 사용하는 tool 환경 구성하여 이미지 생성 및 push

****

컨테이너 이미지 사용
-----

> 컨테이너 이미지 사용 방법
* 이미지 검색
```
$ docker search [option] <imagename:tag>
```

* 이미지 다운로드
```
$ docker pull [option] <imagename:tag>
```

* 다운 받은 이미지 목록
```
$ docker images
```

* 다운 받은 이미지 상세보기
```
$ docker inspect [option] <imagename:tag>
```

* 이미지 삭제
```
$ docker rmi [option] <imagename:tag>
```
*****

컨테이너 실행 / 종료
-----
> 컨테이너 생성, 실행, 종료
* 컨테이너 생성
  * running 중 상태는 아님 : nginx이미지를 webserver라는 이름의 컨테이너를 생성
```
$ docker create --name webserver nginx:1.14
```
* 컨테이너 실행
  * 생성할 때 지정한 이름
```
$ docker start webserver
```

* 컨테이너 생성과 실행
  * pull -> create -> start 포함한 command
```
$ docker run --name webserver -d nginx:1.14
```

* 컨테이너 실행 상태 확인
```
$ docker ps
```

* 컨테이너 실행 상태 상세 확인
```
$ docker inspect webserver
```

* 컨테이너 종료
```
$ docker stop webserver
```

* 컨테이너 삭제
```
$ docker rm webserver [option] containername
```
*****

동작중인 컨테이너 관리
----

> 동작중인 컨테이너 process 상태 확인
* 컨테이너 상태
  * 실행 중인 webser 컨테이너의 상태 확인
```
$ docker ps
$ docker top webserver
$ docker logs webserver
```

* foreground 실행 중인 컨테이너에 연결
```
docker attach [option] containername
```
* 컨테이너에 추가 실행
  * webserver 컨테이너에 bash process 실행하여 접근
```
docker exec webserver /bin/bash
```

*****

컨테이너 사용하기 실습
-----

> 검색 및 다운로드, 실행, 확인, 종료
* 검색
  * 보안 측면에서 official image만 다운로드 받을 것
```
$ docker search nginx
```
* 다운로드
```
$ docker pull nginx:1.14

$ docker pull mysql
// latest IMAGE ID와 같다면 추가로 다운로드 받지 않는다.
$ docker pull mysql:8
```

* 리스트 확인
  * IMAGE ID full name 표시
```
$ docker images --no-trunc
```

* 실행
  * create는 기본적으로 bg로 시작함, status : created
```
$ docker create --name webserver nginx:1.14

// status : UP ...
$ docker start webserver
```

* 컨테이너 확인
  * 할당 받은 IP address와 같은 정보 확인 가능
```
$ docker inspect webserver

...
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",

// 일부만 확인하고자 할 때, 자주사용하므로 alias로 등록해서 사용하자.
$ docker inspect --format '{{.NetworkSettings.IPAddress}}' webserver

// html 확인
$ curl 172.17.0.3

// log 확인
$ docker logs webserver

// process 상태 확인
$ docker top webserver
```

* 컨테이너에 shell하나 실행하여 접근
```
// 현재 실행 중인 webserver container에는 nginx daemon밖에 없음, 오픈되어있는 shell이 없으므로 하나 실행
// 이 후, 내부에 존재하는 파일 접근 및 수정 가능
// -it : interactive, terminal의 약자
$ docker exec -it webserver /bin/bash

// shell prompt가 하나 뜸
root@c8dcf2a668ab:/#
root@c8dcf2a668ab:/# cd /usr/share/nginx/html/
// 원래 파일은 백업해두고, 아래와 같이 변경
root@c8dcf2a668ab:/# echo "This page has been modified by chris." > index.html
root@c8dcf2a668ab:/# exit

// 변경 파일 확인
$ curl 172.17.0.3
This page has been modified by chris.

```

* 종료
```
$ docker stop webserver
```

* 컨테이너 삭제
```
// 컨테이너 삭제시, 변경된 파일도 모두 사라지므로, 삭제는 신중하게 해야 함
$ docker rm webserver

// 동작 중인 컨테이너 강제로 삭제
$ docker rm -f webserver

// alias 등록하면 편리함
$ alias dkrm='docker rm -f $(docker ps -aq)'
```

> 컨테이너 실행 실습 (apache 서버)
```
$ docker search httpd
$ docker pull httpd

$ docker run --name web -d httpd
$ docker inspect web
$ docker inspect --format '{{.NetworkSettings.IPAddress}}' web

$ curl 172.17.0.3
$ docker logs web
$ docker top web
```

*****

컨테이너 리소스 관리
-----

> 컨테이너 HW 리소스 제한
* CPU, Memory, Disk (block IO) 제한 가능
* Memory 리소스 제한
  * 제한 단위 : b, k, m, g   

옵션 | 의미
---|---
--memory, -m | 컨테이너가 사용할 최대 메모리 양을 지정
--memory-swap | 컨테이너가 사용할 스왑 메모리 영역에 대한 설정</br>메모리 + 스왑. 생략시 메모리의 2배가 설정됨
--memory-reservation | --memory 값보다 적은 값으로 구성하는 소프트 제한 값 설정
--oom-kill-disable | OOM Killer가 프로세스 kill하지 못 하도록 보호

* 예제 : 512MB까지 사용가능, over되면 스스로 kill됨
```
$ docker run -d -m 512m nginx:1.14
```

* 예제 : 최대 1GB, 500MB는 항상 보장 받음
```
$ docker run -d -m 1g --memory-reservation 500m nginx:1.14
```

* 예제 : 200MB까지 사용가능, swap은 300MB에 200MB가 포함된 의미이므로, 100MB만 추가로 사용가능함
```
$ docker run -d -m 200m --memory-swap 300m nginx:1.14
```

* 커널이 out of memory killer 동작시 kill하지 못 하도록 설정
```
$ docker run -d -m 200m --oom-kill-disable nginx:1.14
```

* CPU 리소스 제한

옵션 | 의미
---|---
--cpus | 컨테이너에 할당할 CPU core수를 지정.</br>--cpus="1.5" 컨테이너가 최대 1.5개의 CPU 파워 사용 가능
--cpuset-cpus | 컨테이너가 사용할 수 있는 CPU나 core를 할당.</br>cpu index는 0부터. --cpuset-cpus=0-4
--cpu-share | 컨테이너가 사용하는 CPU 비중을 1024 값을 기반으로 설정.</br>--cpu-share 2048 기본 값 보다 두 배 많은 CPU 자원 할당

* 예제 : CPU core 1개의 50%만 사용 가능
```
$ docker run -d --cpus=".5" ubuntu:18.04
```

* 예제 : CPU core 범위 설정
```
$ docker run -d --cpuset-cpus 0-3 ubuntu:18.04
```

* 예제 : 상대적인 비중을 설정, default로 1024인데 2048로 하면 다른 container보다 많은 비중으로 점유 가능
```
$ docker run -d --cpu-share 2048 ubuntu:18.04
```

* Block IO 제한

옵션 | 의미
---|---
--blkio-weight</br>--blkio-weight-device | Block IO의 Quota를 설정할 수 있으며 100 ~ 1000까지 선택, default 500
--device-read-bps</br>--device-write-bps | 특정 디바이스에 대한 읽기와 쓰기 작업의 초당 제한을 kb, mb, gb 단위로 설정
--device-read-iops</br>--device-write-iops | 컨테이너의 read/write 속도의 쿼터를 설정한다.</br>초당 quota를 제한해서 I/O를 발생시킴.</br>0이상의 정수로 표기</br>초당 데이터 전송량 = IOPS * 블럭크기 (단위 데이터 용량)

  * [참고 : [Docker] Resource : Block I/O 제한하기](https://m.blog.naver.com/alice_k106/220899310289)


* 예제 : 상대적 가중치, 다른 blkio보다 적게 할당하려할 때
```
// --rm : 1회성으로 실행할 때 사용됨, 컨테이너가 종료될 때 컨테이너와 관련된 리소스(파일 시스템, 볼륨)까지 깨끗이 제거해줌
$ docker run -it --rm --blkio-weight 100 ubuntu:latest /bin/bash
```

* 예제 : 초당 1MByte 쓰기 제한
```
docker run -it --rm --device-write-bps /dev/vda:1mb ubuntu:lateat /bin/bash
docker run -it --rm --device-write-bps /dev/vda:10mb ubuntu:lateat /bin/bash
```

* 예제 : 쓰기 속도 지정
```
$ docker run -it --rm --device-write-iops /dev/vda:10 ubuntu:lateat /bin/bash
$ docker run -it --rm --device-write-iops /dev/vda:100 ubuntu:lateat /bin/bash
```

> 컨테이너 사용 리소스를 확인하는 모니터링 툴
* docker monitoring commands
  * docker stats : 실행중인 컨테이너의 런타임 통계를 확인
    * docker stats [option] \[container...]
  * docker event : 도커 호스트의 실시간 event 정보를 수집해서 출력
    * docker events -f container=\<NAME>
    * docker image -f container=\<NAME>
* cAdvisor : [cAdvisor link](https://github.com/google/cadvisor)


> 컨테이너 리소스 관리 실습
* stress container 생성
  * 컨테이너 빌드 : 부하 테스트 프로그램 stress를 설치하고 동작시키는 컨테이너 빌드
  * CPU 부하 테스트 : 2개 CPU core를 100% 사용하도록 부하 발생
    * **stress --cpu 2**
  * 메모리 부하 테스트 : 프로세스 수 2개와 사용할 메모리만큼 부하 발생
    * **stress --vm 2 --vm-bytes <사용할 크기>**
  * dockerfile 작성
```
FROM debian
# <id@email.com> maybe added if this field is required
MAINTAINER Chris Choe
RUN apt-get update; apt-get install stress -y
CMD ["/bin/sh", "-c", "stress -c 2"]
```
* 빌드 및 실행
```
$ docker build -t stress .
$ docker run -d --name c1 --cpus="1.0" stress:latest stress --cpu 1
```

* 메모리 리소스 제한
```
// swap 메모리 용량 제한이 실제 메모리 제한과 어떤 관련성이 있는지 확인해 본다.

// MAX 100MBytes, swap 100M이면 실제 swap 사용 불가하다는 의미, 90M 로드를 발생 시킴
$ docker run -m 100m --memory-swap 100m stress:latest stress --vm 1 --vm-bytes 90m -t 5s

// swap 100M이므로 150M는 실행 불가, 실행되면서 바로 kill됨
$ docker run -m 100m --memory-swap 100m stress:latest stress --vm 1 --vm-bytes 150m -t 5s

stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (415) <-- worker 8 got signal 9
stress: WARN: [1] (417) now reaping child worker processes
stress: FAIL: [1] (421) kill error: No such process
stress: FAIL: [1] (451) failed run completed in 0s

// swap 사이즈 생략하면, default로 메모리 * 2, 실행 가능
$ docker run -m 100m stress:latest stress --vm 1 --vm-bytes 150m -t 5s
```

* OOM-Killer로부터 보호
```
$ docker run -d -m 100M --name m4 --oom-kill-disable=true nginx

// 확인 방법 1 : inspect로 확인
docker inspect m4
"Memory": 104857600,
...
"MemorySwap": 209715200,
...
"OomKillDisable": true,

// 확인 방법 2 : 아래 경로에서 확인, 리소스 제한은 cgroup에서 걸어 줌
$ docker ps -a
459314c8f74c   nginx     "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   80/tcp    m4

cat /sys/fs/cgroup/memory/docker/459314c8f74cec662e769f2afc9ea28fd37e2f4a077fbc358ceef681abd5f4d9/memory.oom_control
oom_kill_disable 1
under_oom 0
oom_kill 0
```

* CPU 리소스 제한
```
// CPU 개수를 제한하여 컨테이너를 실행한다.
// CPU #1
$ docker run --cpuset-cpus 1 --name c1 -d stress:latest stress --cpu 1
// htop -d 10 으로 확인

$ docker run --cpuset-cpus 0-1 --name c2 -d stress:latest stress --cpu 1
// htop -d 10 으로 확인

$ docker rm c1

// 컨테이너별로 CPU 상대적 가중치를 할당하여 실행되도록 구성한다.
// 컨테이너 모두 제거 후에 실행
$ docker run -c 2048 --name cload1 -d stress:latest
$ docker run --name cload2 -d stress:latest
$ docker run -c 512 --name cload3 -d stress:latest
$ docker run -c 512 --name cload4 -d stress:latest

// 컨테이너 리소스 사용량 모니터링
$ docker stats cload1 cload2 cload3 cload4
```

* Block IO 제한
```
// 컨테이너에서 --device-write-iops를 적용해서 write속도의 초당 quota를 제한해서 IO write를 제한한다.
// /dev/sda : lsblk로 확인
$ docker run -it --rm --device-write-iops /dev/sda:10 ubuntu:latest /bin/bash
$ dd if=/dev/zero of=file1 bs=1M count=10 oflag=direct

10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 1.80381 s, 5.8 MB/s

10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 1.80358 s, 5.8 MB/s

// 다음 write quota를 100으로 변경 후 같은 작업을 반복한다.
$ docker run -it --rm --device-write-iops /dev/sda:100 ubuntu:latest /bin/bash
dd if=/dev/zero of=file1 bs=1M count=10 oflag=direct

10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.112516 s, 93.2 MB/s

10+0 records in
10+0 records out
10485760 bytes (10 MB, 10 MiB) copied, 0.10667 s, 98.3 MB/s

// bg로 memory까지 설정
$ docker run -it --rm --device-write-iops /dev/sda:100 -m 500m --name c1 -d ubuntu:latest /bin/bash
```

* cAdvisor : [cAdvisor link](https://github.com/google/cadvisor), 쿠버네티스에 포함됨
  * Quick Start: Running cAdvisor in a Docker Container 내용 copy해서 그대로 실행
  * 호스트 전용 어댑터 사용시 해당 IP로 접근 가능 : 예) http://192.168.56.101:8080/
  * CPU, Memory, Swap할당
  * 현재 사용량 등

> 추가 실습
* db라는 이름을 가지는 mysql 컨테이너를 다음의 조건으로 실행하기
  * MYSQL_ROOT_PASSWORD : pass
  * 물리 메모리 200M
  * swap 메모리 300M
  * 할당 CPU core 수 : 1
```
// 실행시 error 발생, docker stats로 확인하면 이미 종료된 상태
$ docker run -e MYSQL_ROOT_PASSWORD=pass -p 3306:3306 -m 200m --memory-swap 300m --cpuset-cpus 1 --name db -d mysql:latest

// swap을 2배로 늘리면 정상 동작함
$ docker run -e MYSQL_ROOT_PASSWORD=pass -p 3306:3306 -m 200m --memory-swap 400m --cpuset-cpus 1 --name db -d mysql:latest
```
* db 컨테이너 리소스 사용량을 docker stats 명령을 확인하기
```
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O       PIDS
12f1230ae861   db        7.82%     181.1MiB / 200MiB   90.55%    806B / 0B   115MB / 230MB   8
```
*****

컨테이너 storage (Volume)
-----

> 컨테이너 볼륨
* 컨테이너 이미지는 read only
* 컨테이너에 추가되는 data들은 별도의 rw 레이어에 저장
  * docker run으로 실행하면 RW 레이어를 생성하게 됨
  * UFS(union file system) : 개념상 image에 포함된 ro layer위에 최종적으로 rw layer를 추가하여 마치 하나인 것 처럼 관리하는 기법
    * [참고 : 도커 컨테이너 까보기(2) – Container Size, UFS](http://cloudrain21.com/examination-of-docker-containersize-ufs)
  * base가되는 image의 특정 파일 변경이 필요한 경우, COW 기법을 이용하여 변경이 필요한 파일의 복사본이 생성되고 이 파일이 수정됨.
    * COW (Copy-on-write)는 유닉스/리눅스에서 fork()할 때 사용하는 기법
    * [참고 : linux system programming - fork()](https://github.com/gotoplace/linux_system_programming/tree/main/chapter_5#%EC%83%88%EB%A1%9C%EC%9A%B4-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0)
    * [참고 : The Copy-on-Write Mechanism](https://www.cloudbees.com/blog/docker-storage-introduction)
  * docker rm으로 삭제하면 rw 레이어에 저장된 data도 같이 삭제되어 복원할 수 없음

> 데이터를 영구적으로 보존하기
* -v /path:/var/lib/xxx 와 같은 형태의 옵션 필요
```
// mysql db를 host의 특정 경로에 volume mount하여 보존하는 예제. host의 /dbdata는 자동 생성되거나 기존 경로 사용
$ docker run -d --name db -v /dbdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass mysql:latest
```
* volume 옵션
```
-v <host path>:<container mount path>
-v <host path>:<container mount path>:<read write mode>
-v <container mount path>

// 기본적으로 rw mode로 mount됨. 보안 측면에서 꼭 필요한 경우가 아니라면 ro mode를 사용하는 것이 안전함.
$ docker run -d --name db -v /dbdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass mysql:latest

// ro mode를 사용
$ docker run -d -v /webdata:/var/www/html:ro httpd:latest

// container path만 걸어주면, 자동으로 /var/lib/docker/volumes/uuid/ 하위에 data directory 영구적으로 생성
$ docker run -d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass mysql:latest
```

> 컨테이너끼리 데이터 공유하기
* host의 특정 경로를 공유
```
// host의 /webdata에 파일을 저장
$ docker run -v /webdata:/webdata -d --name df smlinux/df:latest

// df에서 저장한 파일을 nginx에서 ro로 접근 가능
$ docker run -d -v /webdata:/usr/share/nginx/html:ro -d ubuntu:latest
```

> 컨테이너 storage 실습
* Mysql DB data 영구 보존하기 - host 경로 지정
```
// 컨테이너 실행
$ docker run -d --name db -v /dbdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass mysql:latest

// 컨테이너 내부에 shell하나 오픈
$ docker exec -it db /bin/bash

// mysql 접속
$ mysql -u root -ppass
```
```
// db 보기
show databases;

// 테스트용으로 database 하나 생성
create database chris;

// 종료
exit
```
```
// host에서 확인
$ ls -alh /dbdata
drwxr-x---  2 systemd-coredump systemd-coredump 4.0K Mar 27 00:17  chris

// 컨테이너 삭제 후, host에서 재확인
$ docker rm -f db
$ ls -alh /dbdata
```

* Mysql DB data 영구 보존하기 - host 경로 미지정
```
// host 경로 미지정하여 실행
$ docker run -d --name db -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=pass mysql:latest

// 상세 정보 확인, Source, Destination
// Source 경로에 이동하여 확인해보면, mysql db 파일들 저장되어 있음
$ docker inspect db

        "Mounts": [
            {
                "Type": "volume",
                "Name": "c9465149b2a053c72adb3829bb00929eece62ead75b6f793a7a8d417ba9c8bdc",
                "Source": "/var/lib/docker/volumes/c9465149b2a053c72adb3829bb00929eece62ead75b6f793a7a8d417ba9c8bdc/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
```

* volume 관리 command
```
$ docker volume ls

$ docker volume rm UUID
```

* 웹데이터 readonly 서비스로 컨테이너에 지원하기
  * /webdata/index.html 작성
```
$ mkdir /webdata
cd /webdata

$ echo "<h1>I'm now learning docker with Youtube</h1>" > index.html
$ docker run -d --name web -v /webdata:/usr/share/nginx/html:ro -p 80:80 -d nginx:latest
$ docker ps

// host의 web browser로 guest OS 접근시 확인 가능
// index.html 파일 수정 후 host의 browser로 재확인
```

* 컨테이너간 데이터 공유하기
  * 컨테이너 이미지 하나 생성
  * df.sh 내용
```
#!/bin/bash
mkdir -p /webdata
while true
do
  df -h > /webdata/index.html
  sleep 10
done
```
* dockerfile
```
FROM ubuntu:18.04
ADD df.sh /bin/df.sh
RUN chmod +x /bin/df.sh
ENTRYPOINT ["/bin/df.sh"]
```

```
// 이미지 생성 및 업로드
$ docker build -t flatsixna/df:latest .

// df를 주기적으로 실행하는 컨테이너 실행
$ docker run -d --name df -v /webdata:/webdata flatsixna/df:latest
$ docker run -d --name web -v /webdata:/usr/share/nginx/html:ro -p 80:80 -d nginx:latest

// host의 web browser로 guest OS 접근시 확인 가능
```

*****

컨테이너간 통신
-----

> 컨테이너의 통신 원리
* docker0
  * virtual ethernet bridge : 172.17.0.0/16
  * L2 기반 통신
  * container 생성시 veth 인터페이스 생성 (sandbox) : veth0, veth1 ...
  * 모든 컨테이너는 외부 통신을 docker0를 통해 진행
  * 컨테이너 실행시 172.17.X.Y로 IP 주소 할당
  * iptables로 NAT 및 rule 정책 설정

> 컨테이너 포트를 외부로 노출하기
* port forwarding
  * container port를 외부로 노출시켜 외부로부터 연결 허용
  * iptables rule을 통한 포트 노출
    * -p hostPort:containerPort
    * -p containerPort : host의 random port:containerPort 맵핑을 의미함
    * -P : 대문자 P, host의 random port:dockerfile-expose에 정의된 containerPort 맵핑을 의미함
```
$ docker run --name web -d -p 80:80 nginx:1.14
$ iptables -t nat -L -n -v
```

> 컨테이너 네트워크 추가하기
* user-defined bridge network 생성
  * docker0 network 대역은 변경이 가능하지만, static IP할당이 제한됨
  * 이러한 사유로 user-defined network 생성하여 사용
```
// --driver default는 bridge이므로 생략 가능
// --subnet default는 172.18.x.x 대역
// --gateway default는 해당 network의 1
$ docker network create --driver bridge --subnet 192.168.100.0/24 --gateway 192.168.100.254 mynet
$ docker network ls

$ docker run -d --name web -p 80:80 nginx:1.14
$ curl localhost

$ docker run -d --name appjs --net mynet --ip 192.168.100.100 -p 8080:8080 flatsixna/appjs
$ curl localhost:8080
```

> 컨테이너간 통신 방법
* 컨테이너를 이용한 server & client 서비스 운영
```
// mysql server, backend
$ docker run -d --name mysql -v /dbdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=wordpress -e MYSQL_PASSWORD=wordpress mysql:5.7

// wordpress web server (client of  mysql), frontend
// --link <컨테이너 이름>:<별칭>
$ docker run -d --name wordpress --link mysql:mysql -e WORDPRESS_DB_PASSWORD=wordpress -p 80:80 wordpress:4
```

> 컨테이너간 통신 실습
* 컨테이너 네트워크 사용하기 & 컨테이너 포트 외부로 노출하기
```
// brctl은 bridge-utils package 설치 필요
// docker0 bridge network
$ ip addr
$ brctl show

// terminal #1
$ docker run --name c1 -it busybox
$ docker inspect c1

        "NetworkSettings": {
            "Bridge": "",
            // container c1의 network 환경을 생성해줌
            "SandboxID": "aa23a3be5fcf67a33122c4b7f8124d86540ee3e079159590e1090f5dd8e2031d",
...
            // container c1의 network interface veth0
            "EndpointID": "251921b6aa59196934173507fdd648fd1d93b718210e80257dc92e29f3142ddc",
            "Gateway": "172.17.0.1",
...
            "IPAddress": "172.17.0.2",

// NAT 확인
$ iptables -t nat -L -v

// 172.17.0.0 -> anywhere로 가는 packet은 host의 IP로 전환하여 masquerade하겠다는 의미
Chain POSTROUTING (policy ACCEPT 6 packets, 969 bytes)
 pkts bytes target     prot opt in     out     source               destination
    1    84 MASQUERADE  all  --  any    !docker0  172.17.0.0/16        anywhere
    0     0 MASQUERADE  tcp  --  any    any     172.17.0.4           172.17.0.4           tcp dpt:http

// terminal #2
$ docker run --name c2 -it busybox
$ docker inspect c2

// terminal #3
$ docker run -d -p 80:80 --name web1 nginx

// from CentOS
$ curl vm-ubuntu-s
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

```
// port-forwarding
$ docker run --name web1 -d -p 80:80 nginx

// 다른 terminal 또는 vm centos, host의 brower에서도 접근 가능
$ curl vm-ubuntu-s

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

// host에서 사용하지 않는 port를 random하게 선택하여 맵핑. 할당된 port로 접근 가능
$ docker run --name web2 -d -p 80 nginx

// dockerfile - EXPOSE 80
$ docker run --name web -d -P nginx


$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS                                     NAMES
0fc5a32f7639   nginx     "/docker-entrypoint.…"   4 seconds ago    Up 2 seconds   0.0.0.0:49154->80/tcp, :::49154->80/tcp   web
d59e01ccdbad   nginx     "/docker-entrypoint.…"   4 minutes ago    Up 4 minutes   0.0.0.0:49153->80/tcp, :::49153->80/tcp   web2
abf516d6865b   nginx     "/docker-entrypoint.…"   10 minutes ago   Up 9 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp         web1
```

* 컨테이너 네트워크 추가하기
```
$ docker network create --driver bridge --subnet 192.168.100.0/24 --gateway 192.168.100.254 mynet
$ docker network ls

NETWORK ID     NAME      DRIVER    SCOPE
365121fd5be3   bridge    bridge    local
d91bc19c32c7   host      host      local
dc1b8bf87742   mynet     bridge    local **
480c09338f73   none      null      local

$ docker network inspect mynet

        "Name": "mynet",
        "Id": "dc1b8bf877421567d58bb5d97cfcf7bf9acda84bff9c9f9ca1a34ae65f078ed5",

            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.254"
                }

// 192.168.100.x 할당됨 또는 static IP
$ docker run -it --name c1 --net mynet --ip 192.168.100.100 busybox

$ docker run -d --name web -p 80:80 nginx
$ curl localhost

// appjs 구성한 경우 실습
$ docker run -d --name appjs --net mynet --ip 192.168.100.100 -p 8080:8080 flatsixna/appjs
$ curl localhost:8080
```

* 컨테이너를 이용한 server & client 서비스 운영
```
// mysql server, backend
// error 발생시, /dbdata/* 삭제 후 다시 실행
$ docker run -d --name mysql -v /dbdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=wordpress -e MYSQL_PASSWORD=wordpress mysql:5.7

// wordpress web server (client of  mysql), frontend
// --link <컨테이너 이름>:<별칭>
// host의 browser에서 호스트 전용 어댑터에 할당된 IP로 wordpress 사이트 접근 가능
$ docker run -d --name wordpress --link mysql:mysql -e WORDPRESS_DB_PASSWORD=wordpress -p 80:80 wordpress:4
```

* 추가 실습 - container 빌드
  * genid에서 생성한 index.html은 volume을 통해 nginx의 웹 컨텐츠로 공유 되어야 함
  * nginx 80 port를 통해 genid가 생성한 html 파일을 서비스
  * genid.sh 파일 내용
```
#!/bin/bash
mkdir -p /webdata
while true
do
  /usr/bin/rig | /usr/bin/boxes -d boy > /webdata/index.html
  sleep 5
done
```
* dockerfile내용
```
FROM ubuntu:18.04
RUN apt-get update; apt-get -y install rig boxes
ADD genid.sh /bin/genid.sh
RUN chmod +x /bin/genid.sh
ENTRYPOINT ["/bin/genid.sh"]
```
* 이미지 생성 및 실행
```
$ docker build -t genid .

// host의 /webdata에 파일을 저장
$ docker run -d --name genid -v /webdata:/webdata genid:latest

// genid에서 저장한 파일을 nginx에서 ro로 접근 가능
$ docker run -d --name web -v /webdata:/usr/share/nginx/html:ro -p 80:80 -d nginx:latest
```

*****

빌드에서 운영까지 (using Docker Compose)
-----

> 도커 컴포즈
* 여러 컨테이너를 일괄적으로 정의하고 실행할 수 있는 툴
  * 하나의 서비스를 운영하기 위해서는 여러 개의 애플리케이션이 동작해야 함
  * 컨테이너화 된 애플리케이션들을 통합 관리할 수 있음
  * YAML 사용

> 도커 컴포즈로 컨테이너 실행하기
* YAML 문법
  * [참고 : Docker Compose - sample wordpress](https://docs.docker.com/samples/wordpress/)
  * docker-compose.yml 예제
```
# version : compose 버전, 버전에 따라 지원 문법에 차이가 있음.
version: "2"

# services : 컴포즈를 이용해서 실행할 컨테이너 옵션을 정의
service:
  webserver:
    image: nginx
  db:
    image: redis

# build : 컨테이너 빌드
webapp:
  build: .

# image : compose를 통해 실행할 이미지를 지정
webapp:
  image: centos:7

# command : 컨테이너에서 실행될 명령어 지정
app:
  image: node:12-alpine
  command: sh -c "yarn install && yarn run dev"

# port : 컨테이너가 공개하는 포트를 나열
webapp:
  image: httpd:latest
  port:
    - 80
    - 8443:443

# link : 다른 컨테이너와 연계할 때 연계할 컨테이너 지정
webserver:
  image: wordpress:latest
  link:
    db:mysql

# expose : port를 link로 연계된 컨테이너에게만 공개할 port. 호스트OS에 포트를 공개하지 않음.
expose:
  - "8080"

# volumes : 컨테이너에 볼륨 마운트
webapp:
  image: httpd
  volumes:
    - /var/www/html

# environment : 컨테이너에 적요할 환경변수를 정의
database:
  image: mysql:5.7
  environment:
    MYSQL_ROOT_PASSWORD: pass

# restart : 컨테이너가 종료될 때 적용할 restart 정책
#no: 재시작하지 않음
#always: 컨테이너를 수동으로 끄기 전까지 항상 재시작
#on-failure: 오류가 있을 시에 재시작
database:
  image: mysql:5.7
  restart: always

# depends_on : 컨테이너 간의 종속성을 정의. 정의한 컨테이너가 먼저 동작되어야 한다.
services:
  web:
    image: wordpress:latest
    depends_on:
      - db
  db:
    image: mysql
```

```
# sample for wordpress

version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

* docker compose로 동작시키는 웹서버 예제
```
// 1단계 : 서비스 디렉토리 생성
mkdir webserver
cd webserver

// 2단계 : docker-compose.yml 생성
version: "3"
services:
  web:
    image: httpd:latest
    ports:
      - "80:80"
    links:
      - mysql:db
    command: apachectl -DFOREGROUND
  mysql:
    image: mysql:latest
    command: mysqld
    environment:
      MYSQL_ROOT_PASSWORD: pass

// 3단계 : docker-compose 명령어
$ docker-compose up -d
$ docker-compose ps
$ docker-compose scale mysql=2
$ docker-compose ps
$ docker-compose down
```

* docker-compose 명령어

command | descriptions
---|---
up | 컨테이너 생성 / 시작
ps | 컨테이너 목록 표시
logs | 컨테이너 로그 출력
run | 컨테이너 실행
start | 컨테이너 시작
stop | 컨테이너 정지
restart | 컨테이너 재시작
pause | 컨테이너 일시 정지
unpause | 컨테이너 재개
port | 공개 포트 번호 표시
config | 구성 확인
kill | 실행 중인 컨테이너 강제 정지
rm | 컨테이너 삭제
down | 리소스 삭제

```
$ docker-compose config
$ docker-compose up
$ docker-compose up -d

// 다른 디렉토리의 yaml 실행
$ docker-compose -f /other-dir/docker-compose.yml

$ docker-compose ps
$ docker-compose scale 서비스이름=개수
$ docker-compose run 서비스이름 실행 명령어
$ docker-compose logs 서비스이름

$ docker-compose stop
$ docker-compose start
$ docker-compose down
```

> 빌드에서 운영까지
* 빌드와 운영
  * 방문 횟수를 카운트하는 python 컨테이너 빌드와 운영 예제
  * [참고 : Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)


> docker-compose 실습
* docker-compose 설치하기
  * [참고 : Install Docker Compose](https://docs.docker.com/compose/install/)
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose

// after logout and login...
$ docker-compose --version
```

* 컨테이너 빌드에서 운영까지
  * 방문 횟수를 카운트하는 python 컨테이너 빌드와 운영 예제
  * [참고 : Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)

```
// 1단계 : 서비스 디렉토리 생성, 필요한 파일 생성
$ mkdir composetest
$ cd composetest
```

```
// app.py 파일 내용

import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

```
// requirements.txt 파일 내용

flask
redis
```

```
// 2단계 : 빌드를 위한 dockerfile 생성

# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
# Flask : micro web framework
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
# required libs, pip install libs
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

```
// 3단계 : docker-compose.yml 생성
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

```
// 4단계 : docker-compose 명령어
$ docker-compose up -d
```
   
```
// docker-compose 추가 확인
$ docker-compose ps
$ docker-compose logs

// docker-compose port
$ docker-compose port web 5000
0.0.0.0:8000

$ docker-compose config
```

```
// 중단 및 리소스 제거
$ docker-compose down
```

```
// 5단계 : docker-compose.yml - bind mount 추가. 현재 디렉토리를 /code로 mount했기 때문에 app.py 파일을 수정하면 바로 적용됨.
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"

// app.py 파일을 일부 수정 & 저장한 후 host의 brower에서 바로 확인
'Hello Docker!!! I have been seen {} times.\n'.format(count)

// 서비스 개수 scale out
$ docker-compose scale redis=3
WARNING: The scale command is deprecated. Use the up command with the --scale flag instead.

Creating composetest_redis_2 ... done
Creating composetest_redis_3 ... done

// 서비스 개수 scale in
$ docker-compose scale redis=3

WARNING: The scale command is deprecated. Use the up command with the --scale flag instead.
Stopping and removing composetest_redis_2 ... done
Stopping and removing composetest_redis_3 ... done

// WARNING: The scale command is deprecated. Use the up command with the --scale flag instead.
// 아래 command로 변경하여 사용
$ docker-compose up -d --scale redis=3
$ docker-compose up -d --scale redis=1
```
   
```
// docker exec와 유사한 command. web 이라는 서비스에 env를 전달.
$ docker-compose run web env

// 서비스 중인 web의 log를 보고자 할때
$ docker-compose logs web

// 서비스 중지 only
$ docker-compose stop

// 완전히 종료, network, volume까지 제거
$ docker-compose down --volumes
```

* mysql database를 사용하는 wordpress 운영하기
  * [참고 : Quickstart: Compose and WordPress](https://docs.docker.com/samples/wordpress/)

```
// 디렉토리 생성 및 필요한 파일 생성
$ mkdir my_wordpress
$ cd my_wordpress
```

```
// docker-compose.yml
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

```
// docker-compose 실행
$ docker-compose up -d


// host에 실제 저장되는 경로
/var/lib/docker/volumes/my_wordpress_db_data/_data
/var/lib/docker/volumes/my_wordpress_wordpress_data/_data


// host의 brower로 8000 port로 접근


// shutdown and cleanup
// removes the containers and default network, but preserves your WordPress database
$ docker-compose down

// removes the containers, default network, and the WordPress database.
$ docker-compose down --volumes
```

*****
