# 가상 이미지 생성을 위한 Packer

Hashicorp 에서 만든 가상 머신 이미지를 만드는 오픈소스.

Aws의 AMI, Azure Image, Google Cloud Image등을 스크립트 파일을 이용해서 생성할 수 있다.

json 파일을 정해진 형식대로 작성하고, 그 json파일을 읽어서 그 데이터를 토대로 AWS나 Azure, Google Cloud의 api를 호출해서 이미지를 만드는 구조인 것 같다.

## Install

OS X라서 HomeBrew를 이용해 설치

```bash
$ brew install packer
```

## Command

packer의 명령어들

```bash
Available commands are:
    build       build image(s) from template
    fix         fixes templates from old versions of packer
    inspect     see components of a template
    validate    check that a template is valid
    version     Prints the Packer version
```

기본적으로 build 명령어를 통해 template를 읽어서 이미지를 생성한다.

- 이미지 생성

```bash
$ packer build example.json
```

## Templates

만들 이미지의 설정들을 json 형태로 저장하고 읽어서 이미지를 생성하는 방식이다.  AWS를 기준으로 하면 아래와 같은 형태의 json 파일 구조를 가지게 된다.

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
        "region":         "us-east-1"
    },
    "builders": [
        {
            "access_key": "{{user `aws_access_key`}}",
            "ami_name": "packer-linux-aws-demo-{{timestamp}}",
            "instance_type": "t2.micro",
            "region": "{{user `region`}}",
            "secret_key": "{{user `aws_secret_key`}}",
            "source_ami_filter": {
              "filters": {
              "virtualization-type": "hvm",
              "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
              "root-device-type": "ebs"
              },
              "owners": ["099720109477"],
              "most_recent": true
            },
            "ssh_username": "ubuntu",
            "type": "amazon-ebs"
        }
    ],
    "provisioners": [
        {
            "type": "file",
            "source": "./welcome.txt",
            "destination": "/home/ubuntu/"
        },
        {
            "type": "shell",
            "inline":[
                "ls -al /home/ubuntu",
                "cat /home/ubuntu/welcome.txt"
            ]
        },
        {
            "type": "shell",
            "script": "./example.sh"
        }
    ],
    "post-processors": [
        {
          "type": "compress",
          "format": "tar.gz"
        }
    ]
}
```

- variables

변수 정의 부분, 메인 템플릿 안에 적어도 되고 변수가 지정된 파일이나 CLI 옵션으로 변수값 전달이 가능하다고 한다.

- builders

이미지 설정에 대한 부분 정의

- provisioners

이미지 설정 후 소프트웨어 설치 등에 대한 부분 정의, inline으로 명령어 실행도 가능하고 미리 만들어둔 shell script를 실행하는 것도 가능하다.

### Run on Specific Builds

```json
{
  "type": "shell",
  "script": "script.sh",
  "only": ["virtualbox-iso"]
}
```

### Build-Specific Overrides

```json
{
  "type": "shell",
  "script": "script.sh",
  "override": {
    "vmware-iso": {
      "execute_command": "echo 'password' | sudo -S bash {{.Path}}"
    }
  }
}
```

- post-processors

이미지 생성 후 실행되는 부분 파일 압축이나 업로드 등이 포함된다. 위의 경우에는 tar.gz 형식으로 압축하는 작업을 진행한다.

```json
#simple definition
{
  "post-processors": ["compress"]
}

#detailed definition
{
  "post-processors": [
    {
      "type": "compress",
      "format": "tar.gz"
    }
  ]
}

#sequence definition (여러 post-processors 연결, 순서대로 실행)
{
  "post-processors": [
    [
      "compress",
      { "type": "upload", "endpoint": "http://example.com" }
    ]
  ]
}
```

### Run on Specific Builds

특정 빌드에서만 실행

```json
"post-processors": [
    {
      "type": "vagrant",
      "only": ["virtualbox-iso"]
      #only or except
    }
]
```

The values within `only` or `except` are *build names*

## Create AWS AMIs

Packer는 Amazon AMI 생성을 위해 전략에 따른 Amazon AMI Builder를 제공한다. 아래 4가지 종류가 있음.

### Amazon AMI Builders

- [amazon-ebs](https://www.packer.io/docs/builders/amazon-ebs.html) - EBS-backed AMI를 생성 후, Source AMI를 시작한 뒤 provisioning 작업 이후 새 AMI로 다시 패키징한다. 가장 쉬운 방법
- [amazon-instance](https://www.packer.io/docs/builders/amazon-instance.html) - Instance-store backed AMI를 생성하고 Source AMI를 시작한 뒤 provisioning 작업 후 S3에 업로드해서 AMI 생성
- [amazon-chroot](https://www.packer.io/docs/builders/amazon-chroot.html) - Chroot 환경을 이용해서 EBS-backed AMI를 생성. EC2 인스턴스를 시작 할 필요가 없어서 가장 빠르지만 고급 빌더라서 조금 어려운 듯.
- [amazon-ebssurrogate](https://www.packer.io/docs/builders/amazon-ebssurrogate.html) - 처음부터 EBS-backed AMI를 생성한다. chroot 빌더와 비슷하게 작동하는데 AWS에서 실행 될 필요가 없다고 함. 고급 빌더라서 마찬가지로 어렵다고 함.

>  Amazon Elastic Block Store, Instance store
>
> Instance store 기반 EC2는 데이터는 인스턴스 중지, 종료 또는 하드웨어 장애에 대한 영구성을 제공하지 않음. 데이터 장기 보관 등을 위해선 EBS 지원 EC2 생성 필요

### Create AWS AMI

AMI를 생성하기 전에 IAM에서 AMI 생성을 위한 유저를 생성하고 권한을 부여하자.

어떤 Role이 필요할 지 자세히는 모르겠지만 감으로 EC2FullAccess 권한을 부여했다. 생성이 되는 걸 보면 저 권한으로 충분한 것 같고 나중에는 정말 필요한 권한들만 따로 부여해서 사용해야 할 것 같다.

![image-20181003160513204](/images/devops//image-20181003160513204.png)

저 유저에 대한 AccessKey, SecreyKey를 받아온 다음에 아래의 파일에 입력하고 테스트.

```json
{
    "variables": {
        "aws_access_key": "your_access_key",
        "aws_secret_key": "yout_secrey_key",
        "region":         "ap-northeast-2"
    },
    "builders": [
        {
            "access_key": "{{user `aws_access_key`}}",
            "ami_name": "packer-linux-aws-demo-{{timestamp}}",
            "instance_type": "t2.micro",
            "region": "{{user `region`}}",
            "secret_key": "{{user `aws_secret_key`}}",
            "source_ami_filter": {
              "filters": {
              "virtualization-type": "hvm",
              "name": "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*",
              "root-device-type": "ebs"
              },
              "owners": ["099720109477"],
              "most_recent": true
            },
            "ssh_username": "ubuntu",
            "type": "amazon-ebs"
```

파일에 모든 정보를 입력한 후 아래 명령어 입력을 통해 AMI 생성

```json
$ packer build filename.json
```

![image-20181003160820941](/images/devops/image-20181003160820941.png)

위의 과정을 통해서 AMI가 생성되고 아래처럼 AMIs 리스트에서 확인할 수 있다.

![image-20181003161631585](/images/devops/image-20181003161631585.png)

## 참고

- [VM 이미지 생성을 위한 오픈소스 Packer](http://bcho.tistory.com/1225)
- [Packer](https://www.packer.io/intro/getting-started/install.html)