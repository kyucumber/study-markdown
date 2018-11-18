# GoCD 설치 및 사용


Jenkins와 유사한 CI/CD 툴.

![image-20181016194305511](/images/devops/gocd/image-20181020231757846.png)

대략적인 구조는 위의 그림처럼 GoCD서버와 GoCD 에이전트로 이루어져있다. 서버에서 파이프라인을 추가할 수 있는 인터페이스를 제공하고 등록된 파이프라인이 트리거되는 경우 해당 작업 실행을 에이전트에 할당한다. 에이전트에서 구성된 작업을 수행하기 때문에 최소 하나의 이상의 에이전트가 구성되어야 한다. 

간단하게 구성할때는 GoCD서버와 같은 서버에 GoCD 에이전트를 설치하면 된다.


## Installation

- GoCD Server (RPM based distributions (ie RedHat/CentOS/Fedora))

```bash
$ sudo curl https://download.gocd.org/gocd.repo -o /etc/yum.repos.d/gocd.repo
$ sudo yum install -y java-1.8.0-openjdk-devel.x86_64 # 자바 기반이라 자바 설치

$ sudo /usr/sbin/alternatives --config java # 자바 버전 1.8 변경

$ sudo yum remove java-1.7.0-openjdk # 기존 1.7 자바 삭제

$ sudo yum install -y go-server # go-server 설치

$ sudo /etc/init.d/go-server start # goCD server start
```

> Amazon Linux에서 설치 후 go-server start 할 때 hostname을 제대로 인
>
> 식하지 못하는 에러가 있어서 /etc/hosts에 현재 hostname을 직접 넣어줘서 구동시켰다. 아래와 같은 에러가 나오면 hosts에 현재 호스트네임을 넣어주자.

관련 로그는 /var/log/go-server/ 에서 확인할 수 있음

```bash
Error starting Go Server.

$ hostname #출력되는 hostname을 받아서 hosts에 기록.

$ vim /etc/hosts
```

hosts 파일에 hostname을 넣고 go-server 구동

```bash
127.0.0.1   your-host-name
```

- GoCD Agent (RPM based distributions (ie RedHat/CentOS/Fedora))

```bash
$ sudo yum install -y go-agent

$ sudo /etc/init.d/go-agent start
```

## Usage

해당 서버의 8153번 포트로 들어가면 젠킨스 같은 관리 페이지가 나오고 여기서 파이프라인을 만들 수 있다.

![image-20181017233250714](/images/devops/gocd/image-20181017233250714.png)

### Pipeline

Pipeline은 여러개의 Stage로 구성된다. Stage1이 실패하면 2로 넘어가지 않는다. 간단하게 파이프라인 하위에 여러 스테이지가 존재하고 스테이지 밑에는 각각의 독립적인 JOB이 존재하고 JOB 하위에는 여러 Task가 존재하는 구조이다.

![image-20181017233451935](/images/devops/gocd/image-20181017233451935.png)

### Stage

여러 Job을 구성한다. Job들은 Stage 내에서 독립적(병렬 수행)으로 실행된다. Job은 독립적으로 수행되지만 하나의 Job이 실패하는 경우 그 Stage는 실패한걸로 간주된다.

### Job

여러 Task를 구성하며 Task들은 순서대로 실행된다.

### Task

Pipeline의 최하위 작업 개념. shell script를 실행하거나 하는 작업들.



### Material

Pipeline이 트리거되기 위한 조건, Git, Subversion… 등등 Pipeline이 끝난 뒤 Pipeline이 트리거되도록 설정할 수도 있다.

![image-20181017234159541](/images/devops/gocd/image-20181017234159541.png)



- **Git(private repository) 키 등록하기**

서버에서 go 사용자로 로그인 후 ssh 키 발급 후 github 계정에 등록.

~go/.ssh 경로에 접근 권한이 go 유저에 없어서 추가해주어야 함.

.ssh/ 경로에는 권한 700이 있어야하며 .ssh /id_rsa.pub, .ssh/config에는 600 권한이 있어야 한다.

```bash
$ sudo -su go
$ ssh-keygen

chmod 700 .ssh
chmod 600 config
```

config 추가(ssh fingerprint 무조건 저장)

```bash
StrictHostKeyChecking no
UserKnownHostsFile /dev/null
```

### Authentication

GoCD 서버를 처음 구동하면 인증 없이 모두가 접근 할 수 있는데 인증을 추가할 수 있다.

- **Password File 기반 인증 추가**

```bash
$ sudo yum install httpd

$ sudo -su go
$ cd /etc/go
$ htpasswd -c -s password user
```

htpasswd 명령어를 통해서 아래 형태의 password 저장 파일 생성. 

```bash
user:{SHA}qUqP5cyxm6YcTAhz05H=
```

생성한 파일을 기반으로 유저를 추가할 수 있다.

![image-20181019232317650](/images/devops/gocd/image-20181019232317650.png)

위 버튼 선택 후 오른쪽 위의 Add 버튼 클릭

![image-20181019232651377](/images/devops/gocd/image-20181019232651377.png)

생성한 경로의 파일을 지정하고 id를 입력하고 저장하면 로그인 정보가 추가된다.
