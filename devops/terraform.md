# Terraform

Terraform은 여러 인프라들에 대한 관리를 제공한다. GUI 환경의 AWS Console을 통해 제어되던 부분들을 코드를 통해 구축, 변경, 버전화할 수 있는 도구다.

[HashiCorp Configuration Language (HCL)](https://github.com/hashicorp/hcl) 문법으로 작성된 tf 파일을 통해서 인프라스트럭쳐에 대한 설정들을 코드로 관리한다. 여러 프로바이더들이 제공되고 대표적인 프로바이더로는 Amazon AWS, MS Azure, Google Cloud등이 있다.

## Installation

- for Mac(brew)

```bash
$ brew install terraform
```

- binary file

```bash
$ wget https://releases.hashicorp.com/terraform/0.11.8/terraform_0.11.8_linux_amd64.zip
$ unzip terraform
$ mv terraform /usr/local/bin/
```

.bash_profile에 아래 내용 추가

```bash
export PATH=$PATH:/terraform-path/
```
```bash
$ source .bash_profile
```

## Usage

tf 파일을 통해 간단한 인스턴스 생성을 해보자. AWS 인스턴스 생성을 위해서 먼저 aws provider를 설정해야 한다.

### AWS Provider

AWS Provider를 통해서 AWS에서 제공하는 리소스를 사용할 수 있다. 먼저 자격 증명을 위한 키 설정이 필요하다.

EC2 서버 상에서 Role이 설정되어 있다면 권한이 있어서 키 설정이 필요없겠지만 로컬에서 테스트 한다면 아래와 같은 provider 설정이 필요하다.

IAM에서 아래처럼 유저를 생성하고 키를 발급하자. EC2 생성만 간단하게 할 거라 EC2FullAccess 권한만 부여했다.

![image-20181009160638494](/images/devops/image-20181009160638494.png)

[Terraform docs AWS Provider ](https://www.terraform.io/docs/providers/aws/index.html)

이제 발급한 키를 통해 .tf파일에서 provider를 설정해주어야 한다.

### Static credentials

직접 키값을 넣어서 하는 방법.

```json
provider "aws" {
  region     = "us-west-2"
  access_key = "anaccesskey"
  secret_key = "asecretkey"
}
```

### Environment variables

환경 변수에 설정하는 방법. 난 direnv를 이용해서 환경 변수에 키 값을 지정해서 사용했다.

```
provider "aws" {}
```

```bash
$ export AWS_ACCESS_KEY_ID="anaccesskey"
$ export AWS_SECRET_ACCESS_KEY="asecretkey"
$ export AWS_DEFAULT_REGION="us-west-2"
```

### Create EC2

example.tf 파일을 생성하고 아래 내용을 작성한다. 아래 파일에서는 환경 변수에 설정된 값을 통해 AWS provder를 설정한다.

```json
provider "aws" {}

resource "aws_instance" "example" {
  ami           = "ami-0a10b2721688ce9d2"
  instance_type = "t2.micro"
}
```

그리고 나서 아래 명령어 입력.

```bash
$ terraform init #Initialize a Terraform working directory

Initializing provider plugins...

Terraform has been successfully initialized!

$ terraform plan #Generate and show an execution plan

$ terraform apply #Builds or changes infrastructure
```

이후 콘솔에 들어가서 확인해보면 EC2가 terraform 설정 파일을 통해 생성된 걸 확인할 수 있다.

![image-20181009161231083](/images/devops/image-20181009161231083.png)