# Golang

Golang

## Installation

for mac

```bash
$ brew install go
```

for linux

```bash
$ wget https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.11.2.linux-amd64.tar.gz


$ cd ~
$ vim .profile

# 아래 내용 추가
export PATH=$PATH:/usr/local/go/bin

$ source .profile
```

## GOPATH, GOROOT

go에서 사용되는 주요 환경 변수

- **GOROOT**

go가 설치되는 위치 linux에 설치하고 나니 /usr/local/go 경로에 설치됐다.

- **GOPATH**

사용자의 workspace가 설정되는 위치 linux에서 설치하고 나니 $HOME/go가 기본으로 잡혔다.

만약 위의 두 경로를 직접 설정하고 싶으면 직접 GOROOT, GOPATH 환경변수를 설정해주면 된다.

나는 direnv를 이용해서 .envrc 파일에 아래 내용을 입력하고 사용해서 GOPATH를 동적으로 잡아주게 설정했다.

[direnv](https://direnv.net/) 

해당 워크스페이스에 layout go를 적은 .envrc 파일을 두고 GOPATH를 자동으로 설정하게 해서 사용하는 중

## Go Import와 command

```go
import "fmt"
import "github.com/golang/example/stringutil"

func main() {
    fmt.Println("Hello world")
}
```

위와 같은 코드가 있을 때 github.com/golang/example/stringutil 의존성을 찾지 못해서 빌드가 실패한다.

```bash
$ go build hello.go
cannot find package "github.com/golang/example/stringutil" in any of:
	/usr/local/Cellar/go/1.11.2/libexec/src/github.com/golang/example/stringutil
```

go get 명령어를 통해 필요한 의존성을 받아올 수 있다. 해당 src/프로젝트 경로에서 go get을 실행하고, build를 실행하면 잘 동작하는 것을 확인할 수 있다. 소스 코드 저장은 **$GOPATH/src/package-name** 에 저장된다.

```bash
$ go get
$ go build hello.go
```

- **go get**

필요한 의존성들을 $GOPATH/src/import-path에 배치하고 

go get <저장소 주소>로 하는 경우 바로 패키지를 받아 $GOPATH/pkg에 배치하고, go get을 하는 경우 해당 소스코드 내에 필요한 의존성을 받아 src 아래에 배치한다.

- **go list**

현재 디렉토리, 하위 디렉토리가 가진 import path를 나열한다.

- **go test**

현재 디렉토리, 하위 디렉토리가 포함하는 패키지를 테스트한다.

- **go install**

현재 디렉토리의 소스를 빌드해서 bin 아래로 install 시켜준다.

```bash
$ cd src/go-example
$ go install
$ which go-example
/Users/kimkyunam/go/bin/go-example
```

## Go 예제 프로젝트 생성하기

```bash
$ cd $GOPATH/src
$ mkdir -p github.com/tramyu/example

└── src
    └── github.com
        └── tramyu
            ├── example
            │   └── main.go
            └── stringutil
                ├── stringutil.go
                └── stringutil_test.go
```

- **main.go**

```go
package main

import "fmt"
import "github.com/tramyu/stringutil"

func main() {
	fmt.Println("Hello world")
    fmt.Println(stringutil.Concat("1234", "5678"))
}
```

example에서 사용할 라이브러리를 직접 만들어서 사용해보자. 문자열을 이어붙이는 Concat을 만들어보자.

- **stringutil.go**

```go
package stringutil

func Concat(s1 string, s2 string) string {
	return string(s1 + s2)
}
```

install을 통해서 패키지 오브젝트 파일을 pkg에 위치시키자.

```bash
$ cd /src/github.com/tramyu/stringutil
$ go install
# 혹은 바로 아래처럼
$ go install github.com/tramyu/stringutil
```

그리고 example을 빌드하고 실행해보자.

```bash
$ go install github.com/tramyu/example
$ example
Hello world
12345678
```

### 테스트 코드 작성해보기

기존에 만든 Concat 함수의 테스트코드를 작성해보자. go에서 기본으로 지원해주는 testing 프레임워크를 이용해서 테스트를 할 수 있다.

_test.go로 끝나는 이름의 파일을 만들고 그 안에 함수를 만들어서 테스트 할 수 있다.
t.Error이나 t.Fail와 같은 함수들을 호출할 시 테스트 실패로 간주된다.

- **stringutil_test.go**

```go
package stringutil

import "testing"

type TestCaseString struct {
	s1   string
	s2   string
	want string
}

var testStrings = []TestCaseString{
	{"Hello", "World", "HelloWorld"},
	{"1234", "5678", "12345678"},
	{"", "", ""},
}

func TestConcat(t *testing.T) {
	for _, c := range testStrings {
		got := Concat(c.s1, c.s2)
		if got != c.want {
			t.Errorf("Concat(%q, %q) == %q, want %q", c.s1, c.s2, got, c.want)
		}
	}
}
```

아래 명령어로 테스트 실행

```bash
$ go test github.com/tramyu/stringutil
ok      github.com/tramyu/stringutil    0.004s
```

