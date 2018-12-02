# Docker

리눅스 컨테이너 기술을 이용해 어플리케이션 패키징, 배포를 지원하는 경량의 가상화 오픈소스 프로젝트

- **Container 기반 가상화 / Hypervisor 기반 가상화**

![image-20181027141453532](/images/devops/docker/image-20181027141453532.png)

Hypervisor(Vmware)기반 vm은 Host OS 위에 Hypervisor, vm 하나하나 모두에 OS가 올라가고 그 위에 어플리케이션이 올라간다.

Hypervisor와 컨테이너는 Guest OS가 있고 없고의 차이가 존재하는데 컨테이너는 각각 이미지에 Guest OS가 올라가지 않는다. Guest OS가 각각 컨테이너에에 존재하지 않아서 가볍기 때문에 vm보다 훨씬 빠르게 구동된다.

컨테이너는 Host OS의 내용을 그대로 사용한다. Host OS와 Container의 OS가 다르면 다른 부분만큼만 패키징되며 컨테이너 OS에 대한 isolation은 Linux namespace를 통해 수행한다.

- **Linux Container(LXC)**

  ![lxc](/images/devops/docker/lxc.png)

단일 리눅스 호스트상에서 여러 격리된 리눅스 시스템을 실행하기 위한 OS 수준의 가상화 방법

Libvirt라는 라이브러를 통해 가상 호스트(단일 리눅스 상의 여러 리눅스 호스트들을 관리해준다)

## Docker 구조

![image-20181031222340683](/images/devops/docker/image-20181031222340683.png)

Docker는 Client와 Server(Docker Host)의 구조를 가지고 있다.

Docker Client에서 명령어를 보내면 Docker Host로 명령을 보내고 Docker daemon은 Docker Hub의 Registry에서 이미지를 가져온 뒤 Docker daemon이 바라보는 위치에 이미지들이 저장되며 이 이미지를 통해 컨테이너가 실행된다.

- **Client**

`docker run` 같은 명령을 입력하면 클라이언트가 이 명령을 전송하여 Docker daemon에서 명령을 수행하게 된다. Docker 클라이언트는 둘 이상의 데몬과 통신 할 수 있다.

- **Docker Host의 Dokcer daemon**

Docker API 요청을 수신하고 이미지, 컨테이너, 네트워크 및 볼륨과 같은 Docker 객체를 관리한다. 데몬은 Docker 서비스를 관리하기 위해 다른 데몬과 통신 할 수도 있다.

Client의 명령을 받은 뒤 Docker Hub의 Registry에서 이미지를 가져오고 이미지를 저장하고 이 저장된 이미지를 통해 컨테이너를 실행한다.

- **Dokcer Registry**

Docker Registry는 Docker 이미지를 저장한다. Docker Hub 및 Docker Cloud는 누구나 사용할 수있는 공용 레지스트리

> Docker Deamon을 실행하는 사용자는 **root와 동일한 수준의 권한이 필요**함  Linux kenel을 쉐어링하기 때문
>
> Docker client와 Docker Host는 UNIX 소켓 또는 네트워크 인터페이스를 통해 REST API를 사용하여 통신한다.
>
> 기본적으로 Unix 소켓을 통해 통신하며, REST API를 사용하고 싶으면 TCP 포트를 열어야 한다.

### Docker의 핵심 기반 기술

나중에 읽어보자. [Docker NameSpaces and Cgroups](https://medium.com/@kasunmaduraeng/docker-namespace-and-cgroups-dece27c209c7)

- **NameSpaces**

리눅스 커널의 기능 중 하나, 컨테이너간의 격리를 위해 다양한 종류의 네임스페이스들을 사용한다.

- **Cgroups**

특정 컨테이너가 지정한 만큼만 리소스를 쓰도록 제어하는 기술.

## Installation

64비트 리눅스 커널 버전 3.10이상 필요.

```bash
$ sudo yum install docker
```

설치 후 docker 서버, 클라이언트 버전 확인(통신, 서버와 클라이언트 간 버전 차이 여부 확인)

```bash
$ docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:21:31 2018
 OS/Arch:           darwin/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:29:02 2018
  OS/Arch:          linux/amd64
  Experimental:     true
```

## Docker Image

컨테이너를 만들기 위한 베이스가 되는 것. 이미지를 통해 동일한 형태의 컨테이너를 만든다.

![img](/images/devops/docker/docker-layer.png)

도커는 위 그림처럼 layer 구조의 파일 시스템으로 구성되어 있다.

도커 이미지는 변경된 내용을 계속 수정하는 구조가 아니라 변경된 내용을 계속 위에 덧붙이는 구조를 가진다.

도커는 부트 파일 시스템 위에 Read only layer를 층층히 쌓고, 도커 컨테이너가 이미지를 통해서 실행될 때 Read write layer를 얹어 하나의 파일시스템으로 보이는 구조를 가진다.

***Docker Read-Only Layer : 이미지 영역 / Docker Read-Write Layer : 컨테이너 영역***

위 그림에서 Base Image는 이미지의 부모가 되는 이미지이며 ubuntu, centos의 베이스 이미지는 scratch(베이스 이미지)다.

### Docker 이미지 검색

DockerHub에서 해당 이름의 이미지를 검색한다.

```bash
$ docker search name
```

### Docker 이미지 가져오기

이미지를 가져와서 docker daemon이 관리하는 공간에 저장

```bash
$ docker pull docker/whalesay
```

### Docker 이미지 리스트 보기

나오는 사이즈는 논리적 사이즈

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               5.6                 a46c2a2722b9        6 days ago          256MB
nginx               latest              dbfc48660aeb        2 weeks ago         109MB
alpine              latest              196d12cf6ab1        7 weeks ago         4.41MB
```

### Docker 이미지 삭제

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               5.6                 a46c2a2722b9        6 days ago          256MB
nginx               latest              dbfc48660aeb        2 weeks ago         109MB

$ docker rmi nginx
$ docker rmi dbfc48660aeb # id로도 삭제 가능. 특정만 되면 앞자리 3-4자리만 적어도 삭제 가능
```

이미지를 이용해 띄운 컨테이너가 있는 경우 지울 수 없다. 이 이유는 아마 레이어 구조의 파일 시스템의 최상위에 Read write 영역인 컨테이너 영역이 위치하기 때문에 아래의 이미지만 지울 수 없는것 때문일 듯..

## Docker Container

도커 이미지를 이용해서 컨테이너를 구동시킬 수 있다. 컨테이너는 읽기 쓰기가 가능하다.

### Docker 이미지를 이용해 컨테이너 실행하기

컨테이너가 종료 **되거나** 데몬이 종료 될 때 컨테이너가 제거된다.

```bash
$ docker run 이미지 이름
```

컨테이너 실행 관련 옵션 [Docker run reference](https://docs.docker.com/engine/reference/run/#operator-exclusive-options)

```bash
$ docker run -d # detached mode 컨테이너가 백그라운드로 실행된다.
$ docker run -i # interactive, 표준 입력 활성화, 보통 t 옵션과 같이 쓴다. -t Allocate a pseudo-tty
$ docker run -p 80:80 # port 지정 호스트 포트:컨테이너 포트
$ docker run -P # -P 컨테이너의 모든 포트를 외부에 노출.
$ docker run -e MYSQL_ROOT_PASSWORD='Password' # 환경 변수 지정
$ docker run -v /home/docker/config:/etc/config # 볼륨 지정, 호스트 디렉터리:컨테이너 디렉터리
$ docker run --name=hello # 컨테이너 이름 지정
$ docker run --link db:db # 컨테이너 간 연결 컨테이너 이름:별칭
$ docker run --rm # 컨테이너 종료 시 컨테이너 삭제. 배치 잡이나 테스트용으로 사용한 경우에 이 옵션을 사용하자
```

### Docker 컨테이너 리스트 보기

```bash
$ docker ps # 살아있는 컨테이너
$ docker ps -a # 모든 컨테이너
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
7d15cd802213        nginx               "nginx -g 'daemon of…"   22 seconds ago      Exited (0) 2 seconds ago                       friendly_mcnulty
f601c3d384f6        nginx               "-p 80:80"               33 seconds ago      Created                    80/tcp              quirky_wozniak
```

### Docker 컨테이너 삭제

```bash
$ docker rm 컨테이너 id
$ docker rm -f id, name # f 옵션, 컨테이너가 데몬으로 떠 있어도 삭제.
```

### Docker 컨테이너 정지

```bash
$ docker stop id,name
```

### Docker 컨테이너 로그

```bash
$ docker logs myweb # -f 옵션을 붙일 시 로그를 연속해서 보여주게 된다.
```

### Dokcer 컨테이너 체크

```bash
$ docker inspect nginx 
```

### Docker 호스트와 컨테이너 간 파일 전달

```bash
$ docker cp (호스트 파일 경로) (컨테이너(id, name)):(컨테이너 경로) # 호스트에서 컨테이너로 파일 전달
$ docker cp (컨테이너(id, name)):(컨테이너 경로) (호스트 파일 경로) # 컨테이너의 파일 호스트로 전달
```

### Dokcer 단일 호스트에서 여러 컨테이너간 연결하기

단일 Docker 호스트에 여러 컨테이너를 띄우는 경우에 여러 컨테이너간 연결이 필요한 경우

```bash
$ docker run -d -p 8080:8080 5000:5000 --name customjenkins jenkins
$ docker run -p 80:80 --link customjenkins:jenkins -d nginx
```

위처럼 link 연결을 해주고, nginx 컨테이너 내부에 nginx 설정에 아래처럼 Jenkins:8080을 지정하면 해당 이름의 jenkins와 연결한다.

ping Jenkins 할 때 해당 서버로 ping을 줄 수 있는 것 처럼 nginx 컨테이너 안에서 jenkins라는 alias를 설정하는 것, nginx.config에 아래처럼 proxy_pass로 http://jenkins:8080을 주면 설정된 alias를 통해 접근이 가능하다.

```bash
http {
	server {
		listen 80;
        location / {
          proxy_set_header        Host $host;
          proxy_set_header        X-Real-IP $remote_addr;
          proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header        X-Forwarded-Proto $scheme;
          # Fix the "It appears that your reverse proxy set up is broken" error.
          proxy_pass          http://jenkins:8080;
          proxy_read_timeout  90;
        }
    }
}

```

### Docker 데이터 컨테이너 생성하고 연결하기

여러 컨테이너 간 디렉토리를 공유하는 데이터 컨테이너를 만들어보자.

이 컨테이너는 데몬으로 실행할 필요 없이 생성만 되어있어도 정상적으로 데이터 공유가 이루어진다.

```bash
$ docker run --name mydata -v /data busybox true
```

테스트 할 컨테이너 하나를 실행해보자.

데이터 컨테이너 연결은 --volumes-from 명령어를 통해 이루어진다.

rm 옵션은 빠져나오면서 생성된 컨테이너가 삭제된다.(배치 잡 돌리고 삭제하거나 하는 경우 사용)

```bash
$ docker run --rm -it --volumes-from mydata ubuntu
$ cd /data
$ touch example
```

그리고 한대를 더 띄워서 정상적으로 sharing이 됐는지 체크해보자.

```bash
$ docker run -it --volumes-from mydata ubuntu
$ cd /data
$ ls
```
