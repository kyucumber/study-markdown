# Terraform으로 Launch Templates 생성

Terraform으로 Auto Scaling Group에서 사용할 Launch Template를 생성해보자.

## Example Usage
아래는 launch_teamplte 생성에 대한 기본 예시인데 여기서 필요한 정보들만 바꾸고 실행시키면 원하는 launch template를 만들 수 있다.

```bash
resource "aws_launch_template" "foo" {
  name = "foo"

  block_device_mappings {
    device_name = "/dev/sda1"
    ebs {
      volume_size = 20
    }
  }

  credit_specification {
    cpu_credits = "standard"
  }

  disable_api_termination = true

  ebs_optimized = true

  elastic_gpu_specifications {
    type = "test"
  }

  iam_instance_profile {
    name = "test"
  }

  image_id = "ami-test"

  instance_initiated_shutdown_behavior = "terminate"

  instance_market_options {
    market_type = "spot"
  }

  instance_type = "t2.micro"

  kernel_id = "test"

  key_name = "test"

  monitoring {
    enabled = true
  }

  network_interfaces {
    associate_public_ip_address = true
  }

  placement {
    availability_zone = "us-west-2a"
  }

  ram_disk_id = "test"

  vpc_security_group_ids = ["sg-12345678"]

  tag_specifications {
    resource_type = "instance"
    tags {
      Name = "test"
    }
  }

  user_data = "${base64encode(...)}"
}
```

##  Argument Reference

Launch Template를 만들기 전에 terraform에서 정의해둔 Launch Template의 설정들을 살펴보자.

- [`name`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#name) 

name은 만들어질 Launch Template의 이름이다. 이름을 주지 않으면 terraform이 유니크한 이름을 만들어준다.

```json
resource "aws_launch_template" "foo" {
  name = "foo"
}
```

- [`name_prefix`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#name_prefix)

해당 이름을 prefix로 가지는 유니크한 이름을 생한다. 위의 name과 충돌한다고 하니 하나만 지정해야 할 듯

```json
resource "aws_launch_template" "foo" {
  name_prefix = "foo"
}
```

- [`description`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#description)

Launch template에 대한 설명

```json
resource "aws_launch_template" "foo" {
  description = "default launch teamplte"
}
```

- [`block_device_mappings`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#block_device_mappings)

AMI에서 지정한 볼륨 이외에 인스턴스에 연결할 볼륨을 지정하는 부분. Launch Teamplte를 통해 새로 실행될 인스턴스에 대한 볼륨을 변경해주는 부분이다. AMI 기본 볼륨 설정 위에 덮어쓰는 느낌으로 생각하면 될 것 같다.

```json
resource "aws_launch_template" "foo" {
  block_device_mappings {
    device_name = "/dev/sda1"
    ebs {
      volume_size = 20
    }
  }
}
```

- [`credit_specification`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#credit_specification) 

T2 계열에서만 존재하는 CPU Credit에 대한 설정인것 같다. unlimited와 standard가 존재하는 듯

```json
resource "aws_launch_template" "foo" {
  credit_specification {
    cpu_credits = "standard"
  }
}
```

- [`disable_api_termination`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#disable_api_termination)

true로 설정하면 아래의 Launch Teamplte Advanced details의 Terminaton protection 설정이 Enable로 설정된다.

![image-20181009194650742](/images/devops/image-20181009194650742.png)

 [EC2 Instance Termination Protection](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingDisableAPITermination)

- [`ebs_optimized`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#ebs_optimized)

EBS-optimized 여부, true로 설정 시 EC2 Instance가 EBS-optimized로 생성된다.

[EBS-optimized](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EBSOptimized.html#ebs-optimization-support)

- [`elastic_gpu_specifications`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#elastic_gpu_specifications) 

Elastic GPU를 인스턴스에 추가하는지 여부

- [`iam_instance_profile`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#iam_instance_profile) 

인스턴스를 시작하는 IAM 프로필

```json
resource "aws_launch_template" "foo" {
  iam_instance_profile {
    name = "test"
  }
}
```

- [`image_id`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#image_id)

Launch Template에 적용될 AMI 아이디

```bash
resource "aws_launch_template" "foo" {
  image_id = "ami-123456789"
}
```

- [`instance_initiated_shutdown_behavior`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#instance_initiated_shutdown_behavior)

인스턴스 종료 동작, 디폴트 값은 stop

```json
resource "aws_launch_template" "foo" {
  instance_initiated_shutdown_behavior = "terminate"
}
```

- [`instance_market_options`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#instance_market_options)

마켓 옵션 뭐지?

- [`instance_type`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#instance_type)

인스턴스 타입

- [`kernel_id`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#kernel_id)

커널 아이디

- [`key_name`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#key_name)

인스턴스에 사용할 key pair 이름(pem)

- [`monitoring`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#monitoring)

인스턴스에 대한 모니터링 옵션. enabled true인 경우 자세한 모니터링 활성화.

- [`network_interfaces`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#network_interfaces)

인스턴스에 연결할 네트워크 인터페이스 사용자 정의

- [`placement`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#placement)

인스턴스의 placement 정보

```json
resource "aws_launch_template" "foo" {
  placement {
    availability_zone = ""
  }
}
```

- [`ram_disk_id`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#ram_disk_id)

ram disk id

- [`security_group_names`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#security_group_names)

연결할 security group 이름들, vpc에서 인스턴스를 만드는 경우 vpc_security_group_ids를 사용할 것

- [`vpc_security_group_ids`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#vpc_security_group_ids)

연결할 security group id들

- [`tag_specifications`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#tag_specifications)

인스턴스 시작 시 리소스에 적용할 태그들

```json
resource "aws_launch_template" "example" {
  tag_specifications {
    resource_type = "instance"
    tags {
      Name = "test"
    }
  }
  tag_specifications {
    resource_type = "volume"
    tags {
      Name = "test"
    }
  }
}
```

- [`tags`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#tags)

Launch Template에 적용할 태그

```json
resource "aws_launch_template" "example" {
  tags {
    Name = "test",
    TEAM = "hello"
  }
}
```

- [`user_data`](https://www.terraform.io/docs/providers/aws/r/launch_template.html#user_data)

인스턴스 시작 시 제공할 Base64로 인코딩 된 사용자 데이터 정보

```json
resource "aws_launch_template" "example" {
  user_data = "${base64encode(...)}"
}
```

## Create launch template

Launch template을 생성할 스크립트 코드 작성

**launch.tf**에 아래 내용 작성

provider설정을 아래처럼 하려면 환경변수에 값을 넣어두어야 한다.

```json
provider "aws" {}

resource "aws_launch_template" "kyunam" {
  name = "launch-template-kyunam-${uuid()}"

  block_device_mappings {
    device_name = "/dev/xvda"
    ebs {
      volume_size = 100
    }
  }

  ebs_optimized = true

  iam_instance_profile {
    name = "arn:aws:iam::1111:instance-profile/example"
  }

  image_id = "ami-0bb01e989e36b8ab5"

  instance_initiated_shutdown_behavior = "terminate"

  instance_type = "t2.micro"

  key_name = "launch-template-test"

  placement {
    availability_zone = "ap-northeast-2"
  }

  vpc_security_group_ids = ["sg-12341234"]

  tag_specifications {
    resource_type = "instance"
    tags {
      Name = "launch-test"
    }
  }

  tags {
    Name = "test",
    ENV = "test"
  }
}
```

해당 디렉토리에서 아래 명령어 실행을 통해 launch template 생성

```bash
$ terraform init
$ terraform plan
$ terraform apply
```
AWS 콘솔에서 아래처럼 생성이 제대로 됐는지 확인해보자.

![image-20181013165630540](/images/devops/image-20181013165630540.png)