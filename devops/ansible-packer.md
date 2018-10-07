# Packer와 Ansible 활용 AMI 생성

packer에서 프로비저닝을 할 때 shell 스크립트 파일을 실행하거나 인라인으로 실행하는 것 이외에도 ansible playbook_file을 통해 provisioining을 해줄 수 있다.

```json
"provisioners": [
    {
        "type": "shell",
        "inline":[
            "ls -al /home/ubuntu",
            "cat /home/ubuntu/welcome.txt"
        ]
    },
    {
      "type": "ansible",
      "playbook_file": "playbook/jdk8.yml"
    }
  ]
```

## Packer 빌드 파일 작성

Ansible로 프로비저닝 하기 전에 packer build파일을 먼저 작성하자.

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_KEY_ID`}}",
        "region": "{{env `AWS_REGION`}}",
        "ami_name": "{{isotime \"060102-1504\"}}-packer-example-ami"
    },
    "builders": [
        {
            "type": "amazon-ebs",
            "access_key": "{{user `aws_access_key`}}",
            "secret_key": "{{user `aws_secret_key`}}",
            "region": "{{user `region`}}",
            "ami_name": "packer-example-{{timestamp}}",
            "source_ami": "ami-0a10b2721688ce9d2",
            "vpc_id": "vpc-40c4ce28",
            "subnet_id": "subnet-ef88fca3",
            "instance_type": "t2.micro",
            "ssh_interface": "public_ip",
            "ssh_username": "ec2-user",
            "ami_description": "Amazon Linux Base AMI",
            "tags": {
                "Name": "{{user `ami_name` | clean_ami_name}}",
                "BaseAMI_Id": "{{ .SourceAMI }}",
                "BaseAMI_Name": "{{ .SourceAMIName }}",
                "TYPE": "EC2.ami"
            }
        }
    ],
    "provisioners": [
        {
            "type": "ansible",
            "playbook_file": "playbook/main.yml"
        }
	]
}
```

설정에 대해 간략하게 살펴보면 위의 **variables는 변수가 정의된 부분**이다. 사용은 아래처럼 할 수 있다.

```json
{{user `aws_access_key`}}
```

builders는 생성될 이미지의 옵션들, 빌더 타입들을 지정하는 부분이다. 다양한 옵션들이 있는데 간략하게 살펴보자.

- type

builder 타입으로 amazon-ebs 이외에도 chroot 등 다양한 설정이 있다. 

- source_ami

베이스가 될 ami, 나는 아래와 같은 amazon-linux 기본 ami를 지정했다.

![image-20181007171013939](/images/devops/image-20181007171013939.png)

- vpc_id

vpc 지정, 만약 vpc가 구성되어 있으면 원하는 vpc를 지정하면 된다.

- subnet_id

서브넷 지정, 원하는 서브넷을 선택하면 된다.

- instance_type

말 그대로 인스턴스 타입

- tags

AMI에 지정될 태그들을 정의할 수 있다.

```
"Name": "{{user `ami_name` | clean_ami_name}}",
```

윗 부분에 대해 잠시 보면, clean_ami_name 함수를 통해 잘못된 문자를 -로 변경하는 부분이고, ami_name의 iso_time은 아래의 양식에 맞춰 년, 월, 일을 표시한 부분이다.

![image-20181007235650756](/images/devops/image-20181007235650756.png)

태그를 지정해주면 AMI에 아래와 같은 태그 네임들이 들어가게 된다.

![image-20181007171238023](/images/devops/image-20181007171238023.png)

**provisioners는 부팅 후 이미지를 설치 및 구성하는 부분**인데 shell 스크립트를 이용해서도 할 수 있고 우리는 ansible을 이용해서 할 거기 때문에 아래처럼 옵션을 주면 된다.

```json
"provisioners": [
        {
            "type": "ansible",
            "playbook_file": "playbook/main.yml"
        }
	]
```

이제 위에서 사용할 ansible playbook_file을 생성하기만 하면 된다.

## Ansible

프로비저닝, 구성 관리 등에 대한 자동화를 제공하는 소프트웨어

프로비저닝이란 사용자의 요구에 맞게 시스템 자원을 할당, 배치, 배포해 두었다가 필요 시 시스템을 즉시 사용할 수 있는 상태로 미리 준비해 두는 것, 서버를 예로 들면 Spring Boot 앱을 올리기 위해서 jdk를 설치해두거나, 앱에서 DB를 사용하기 위해서 Mysql을 설치해두거나 DB를 생성해두거나 하는 작업이 여기에 해당한다.

Ansible은 Docker, Vagrant, EC2등에 대한 provisioning을 위한 설정등을 제공한다. yaml 문법으로 작성한 ansible 파일을 통해서 프로비저닝 설정들을 관리할 수 있다.

### Installation

```bash
$ sudo easy_install pip

$ sudo pip install ansible
```

### Usage

Ansible playbook 파일을 작성하기 전에 Ansible playbook의 구조를 한번 살펴보자.

```bash
packer-example
├── example.json
└── playbook
    ├── main.yml
    └── roles
        ├── common
        │   ├── handlers
        │   │   └── main.yml
        │   └── tasks
        │       └── main.yml
        ├── jdk8
        │   └── tasks
        │       └── main.yml
        ├── nginx
        │   └── tasks
        │       └── main.yml
        └── pinpoint
            ├── files
            │   └── pinpoint-bootstrap-1.8.0.jar
            └── tasks
                └── main.yml
```

playbook 디렉토리 바로 밑의 main.yml에는 roles가 존재하는데 roles에 등록된 각각의 작업들(common, jdk8 등)에 대해 **아래와 같은 role 구조를 기초해서 자동으로 내용을 로드하고 프로비저닝을 진행**한다. 지금 예제의 파일 구조는 단순해서 files와 tasks 정도만 사용하는 상태.

```bash
roles
├── common/
│   └── files/
│   └── templates/
│   └── tasks/
│   └── handlers/
│   └── vars/
│   └── defaults/
│   └── meta/
```

jdk8 role을 등록하면 jdk8과 관련된 작업을 tasks의 yml 파일을 읽어서 프로비저닝을 진행한다. 파일 이름은 상관없다.

- #### playbook main.yml 파일

```yaml
- name: Amazon Linux based Java AMI
  hosts: all
  roles:
   - common
   - jdk8
   - pinpoint
   - nginx
```

위의 roles에 등록된 common, jdk8, pinpoint, nginx라는 단위의 프로비저닝 작업을 각각 진행하게 된다.

- #### common role

```bash
roles
├── common
│   ├── handlers
│   │   └── main.yml
│   └── tasks
│       └── main.yml
```

#### tasks/main.yml

```yaml
- name: update yum packages
  yum: list=updates update_cache=true

- name: set Asia/Seoul timezone
  become: yes
  timezone:
    name: Asia/Seoul

- name: install git
  become: yes
  yum:
    name: git
    state: latest

- name: remove unused yum packages -ntp
  become: yes
  yum:
    name: ntp
    state: absent

- name: install chrony
  become: yes
  yum:
    name: chrony
    state: latest
  notify:
    - start chronyd
    
- name: create /usr/agent directory
  become: yes
  file: path=/usr/agent state=directory
```

yum 업데이트, timezone 세팅, git 설치, 필요한 디렉토리 생성 등의 작업이 포함되어 있다.

Amazon Time Sync Service 사용을 위해서 ntp 제거, chrony 설치, handlers에서 chronyd 데몬을 실행하도록 notify action 설정을 추가했다. notify는 handler의 이름으로 등록하면 핸들러에서 해당 작업을 실행한다.

#### handlers/main.yml

```yaml
- name: start chronyd
  become: yes
  service:
    name: chronyd
    enabled: yes
```

notify 액션을 받아서 chrony 데몬을 실행하는 작업을 수행한다.

- #### Jdk8 role

```bash
roles
├── jdk8
│   └── tasks
│       └── main.yml
```

#### tasks/main.yml

```yaml
- name: install OpenJDK8
  become: yes
  yum:
    name: java-1.8.0-openjdk-devel
    state: latest

- name: select lastest java version
  become: yes
  alternatives:
    name: java
    path: /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/java
```

openjdk 설치, 버전 1.7 -> 1.8로 변경하는 부분이 포함되어 있다.

#### pinpoint role

```bash
roles
├── pinpoint
│   └── files
│       └── pinpoint-bootstrap-1.8.0.jar
│   └── tasks
│       └── main.yml
```

#### tasks/main.yml

```yaml
- name: copy pinpoint-bootstrap jar
  become: yes
  copy:
    src: pinpoint-bootstrap-1.8.0.jar
    dest: /usr/agent
    owner: ec2-user
    group: ec2-user
    mode: 0755
```

pinpoint-bootstrap-1.8.0.jar 파일을 common에서 생성한 /usr/agent 경로로 copy하고 권한은 755를 부여하는 작업을 실행한다. tasks와 같은 depth에 있는 files 경로 아래에 파일이 존재해야 src에 지정한 파일을 복사할 수 있다.

- ### nginx role

```bash
roles
├── nginx
│   └── tasks
│       └── main.yml
```

#### tasks/main.yml

```yaml
- name: install nginx and modules
  become: yes
  yum: pkg={{item}} state=latest
  with_items:
    - nginx
    - nginx-mod-http-image-filter
    - nginx-all-modules
    - nginx-mod-http-xslt-filter
    - nginx-mod-http-geoip
    - nginx-mod-stream
    - nginx-mod-http-perl
    - nginx-mod-mail
```

nginx 관련 yum 패키지들을 설치한다. 여러개를 설치할때는 with_items를 이용해 한번에 설치할 수 있다.

셋팅 후 아래의 명령어로 AMI를 생성한다.

```bash
$ packer build example.json
```

## AMI로 인스턴스 생성

만들어진 AMI를 통해서 인스턴스를 생성하고 프로비저닝을 통해서 원하는 세팅들이 잘 됐는지를 체크해보자.

jdk 버전이나 timezone이 세팅되어 있으면 정상적으로 AMI가 생성이 된거다.

```bash
$ java -version
openjdk version "1.8.0_181"

$ cat /etc/sysconfig/clock
ZONE="Asia/Seoul"
UTC=true

$ nginx -v
nginx version: nginx/1.12.1

$ service chronyd status
chronyd (pid  2570) is running...

$ chronyc sources -v
210 Number of sources = 5

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 169.254.169.123               3   6    17    63    -21us[-1387us] +/- 1968us
^- ntp-1.arkena.net              2   6    17    62  +5237us[+5237us] +/-  168ms
^- 4.144.155.104.bc.googleu>     2   6    17    63   -575us[-1942us] +/-  109ms
^- t1.time.sg3.yahoo.com         2   6    17    62   +106us[ +106us] +/-   58ms
^- server.antechnet.sk           3   6    17    62    +40ms[  +40ms] +/-  196ms
```