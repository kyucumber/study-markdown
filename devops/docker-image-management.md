## Docker 이미지 관리

도커 이미지를 생성하고, 로컬에서만 관리하면 유실 될 위험이 있다.

원격 저장소에서 도커 이미지를 관리해보자.

## Docker Hub에서 이미지 관리

DockerHub에 공개적인 이미지를 올리고 관리할 수 있다.

private repository도 지원하는데, 무료의 경우 1개의 repository를 지원한다.

- **Docker Hub에 이미지 업로드**

이미지를 올릴 Repository를 먼저 생성하자.

![image-20181202173051504](/images/devops/docker/image-20181202173051504.png)

현재 가지고 있는 이미지를 허브에 올리기 전에 태그를 붙여서 식별할 수 있는 이름을 부여

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              140454e1db24        4 seconds ago       275MB
$ docker tag 140454e1db24 tramyu/test:lastest
```

도커 허브에 로그인 후, 이미지 푸시.

```bash
$ docker login
$ docker push tramyu/test:lastest
```

- **Docker Hub에서 이미지 받아오기**

위에서 푸시한 도커 이미지를 받아오자.

```bash
$ docker pull tramyu/test
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tramyu/test         latest              c588174bbfef        5 days ago          275MB
```

## Private Registry에서 이미지 관리

직접 Registry를 구축하고 그 서버에서 이미지를 관리해줄 수 있다. 로컬에서 Registry를 도커 컨테이너로 띄워보자.

Registry는 볼륨 바인딩으로 로컬에서 이미지 데이터를 관리해줄수도 있고, S3나 클라우드 스토리지를 활용해 관리할수도 있다.

지금은 간단한 예제라 인증이 구축되어있지 않은데, 인증을 통한 권한 관리도 직접 해줘야 한다.

- **Registry를 Docker Container로 올리기**

```bash
$ docker run -d -p 5000:5000 --name kyunamregistry registry:2
```

레지스트리의 저장소는 볼륨 바인딩을 통해 관리할 수도 있고 외부 클라우드 저장소(S3) 등을 이용할 수 있다.

- **Registry에 이미지 푸시하기**

이미지 태깅 후 Registry로 푸시

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
registry            2                   2e2f252f3c88        2 months ago        33.3MB
$ docker tag 2e2f252f3c88 localhost:5000/docker-registry
$ docker push localhost:5000/docker-registry
```

- **Registry에서 이미지 받아오기**

```bash
$ docker rmi -f 해당 이미지 # 삭제 후 받아와지는지 테스트
$ docker pull localhost:5000/docker-registry
$ docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
registry                         2                   2e2f252f3c88        2 months ago        33.3MB
localhost:5000/docker-registry   latest              2e2f252f3c88        2 months ago        33.3MB
```

## AWS ECR(Elastic Container Service)을 이용한 이미지 관리

AWS에서 제공하는 ECR을 이용한 이미지 관리

ECR에서 pull, push하는 권한을 자동으로 가져오기 위해서 AWS ECR Docker Credential Helper를 이용하자.

- **Amazon ECR Docker Credential Helper 설정하기**

```bash
$ go get -u github.com/awslabs/amazon-ecr-credential-helper/ecr-login/cli/docker-credential-ecr-logi

# 생성된 바이너리 파일을 /usr/local/bin에 cp
$ cd /home/ec2-user/go/bin
$ cp docker-credential-ecr-login /usr/local/bin
```

config.json 생성할 디렉토리 생성

```bash
$ mkdir ~/.docker
$ vim ~/.docker/config.json
```

아래 내용 입력 후 저장

```json
{
	"credsStore": "ecr-login"
}
```

- **ECR에 이미지 올리기**

태깅 후 푸시 권한이 없으면 에러가 발생하니 적절한 role을 설정하자.

테스트용이라 AmazonEC2ContainerRegistryFullAccess policy 부여

```bash
$ docker tag cd14cecfdb3a 603921118307.dkr.ecr.ap-northeast-2.amazonaws.com/ecr-test-kyunam:v1
$ docker push 603921118307.dkr.ecr.ap-northeast-2.amazonaws.com/ecr-test-kyunam:v1
```

- **ECR에서 이미지 받아오기**

Repository에서 아래 Image URI를 복사하고.

`docker pull (Image URI)` 명령어를 통해 가져올 수 있다.

![image-20181208164241927](/images/devops/docker/image-20181208164241927.png)

```bash
$ docker pull {image-uri}
```

권한이 없으면 아래의 에러가 발생한다.

ecr:GetAuthorizationToken role을 해당 서버에 등록해주자.

```
Error retrieving credentials
error="ecr: Failed to get authorization token: AccessDeniedException: User: arn:aws:sts::314737209462:assumed-role/role-name is not authorized to perform: ecr:GetAuthorizationToken on resource: *\\n\\tstatus code: 400, request id: c0fb1a2b-fabb-11e8-a233-37d5e10ab5f0"
Error response from daemon: Get <https://314737209462~~>
no basic auth credentials
```
