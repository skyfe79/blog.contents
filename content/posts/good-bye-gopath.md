---
title: "Hello Go module, Goodbye GOPATH"
date: 2019-11-02T20:14:00+09:00
draft: false
tags: ["go", "gopath", "gomodule"]
---

`Go`언어를 접하면서 `$GOPATH`를 설정하는 점이 약간 부담스러웠다. 그러나 1.13 이후 버전부터는 `go module`기능을 사용하여 `$GOPATH`이외의 위치에서 프로젝트를 생성할 수 있다. 

## 시작하기

우선 프로젝트를 만들 폴더를 만든다. 예제로 AwesomeBin 폴더를 만들고 보자.

```
$ mkdir AwesomeBin
$ cd AwesomeBin
```

AwesomeBin 폴더에 진입한 다음에 우리가 만들 모듈의 이름. main package가 있는 곳이면 실행 파일의 이름으로 모듈을 초기화한다. awesome 실행파일이라고 가정해 보자.

```
$ go mod init awesome
go: creating new go.mod: module awesome
```

그러면 `mod.go` 파일이 생성된다.

```
$ l
drwxr-xr-x  3 burt  staff    96B 11  2 20:21 .
drwxr-xr-x  4 burt  staff   128B 11  2 20:21 ..
-rw-r--r--  1 burt  staff    24B 11  2 20:21 go.mod
```

go.mod 파일의 내용은 아래와 같다.

```go
module awesome

go 1.13
```

여기에 디펜던시를 추가하려면 아래와 같이 `require`문을 사용하여 선언해준다.

```go
module awesome

go 1.13

require github.com/cosiner/argv v0.0.1
```

중요한 점은 버전을 꼭 명시해 주어야 한다. 버전 없이 기입하면 아래와 같이 `go build`를 할 때 오류가 발생한다.

```
usage: require module/path v1.2.3
```

이제 main.go 파일을 만들고 아래와 같이 작성해 보자. 테스트로 작성하는 것이어서 구현 내용은 무시하자. :)

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/cosiner/argv"
)

func main() {
	fmt.Println("Testing Go module")
	args, err := argv.Argv([]rune(" ls   `echo /`   |  wc  -l "), argv.ParseEnv(os.Environ()), argv.Run)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(args)
}
```

main.go 파일을 저장한다.

```
$ l
drwxr-xr-x  4 burt  staff   128B 11  2 20:26 .
drwxr-xr-x  4 burt  staff   128B 11  2 20:21 ..
-rw-r--r--  1 burt  staff    57B 11  2 20:25 go.mod
-rw-r--r--  1 burt  staff   282B 11  2 20:26 main.go
```

`go build` 문으로 빌드를 한다. 

```
$ go build
$
```

빌드가 끝나고 확인해 보면 `go mod init`으로 준 이름으로 실행파일이 생성된다.

```
$ l
drwxr-xr-x  6 burt  staff   192B 11  2 20:28 .
drwxr-xr-x  4 burt  staff   128B 11  2 20:21 ..
-rwxr-xr-x  1 burt  staff   2.3M 11  2 20:28 awesome
-rw-r--r--  1 burt  staff    64B 11  2 20:28 go.mod
-rw-r--r--  1 burt  staff   165B 11  2 20:28 go.sum
-rw-r--r--  1 burt  staff   282B 11  2 20:26 main.go
```

실행해 보면 아래와 같이 출력된다.

```
$ ./awesome
Testing Go module
[[ls /] [wc -l]]
```

`$GOPATH`에서 해방되었다!!!

## 다른 예제

[Go Modules 살펴보기](https://velog.io/@kimmachinegun/Go-Go-Modules-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-7cjn4soifk) 글에서 나온 예제도 만들어 보자.

`go mod init myecho`로 모듈을 초기화한다.

```go
module myecho

go 1.13

require github.com/labstack/echo/v4 v4.1.11
```

`main.go`파일을 만들어 아래 코드를 입력한다.

```go
package main

import (
	"net/http"

	"github.com/labstack/echo/v4"
)

func main() {
	e := echo.New()
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello, World")
	})

	e.Logger.Fatal(e.Start(":1323"))
}
```

`main.go` 파일을 저장하고 `go build`로 빌드한다.

```
$ go build
$ l
drwxr-xr-x  6 burt  staff   192B 11  2 20:36 .
drwxr-xr-x  5 burt  staff   160B 11  2 20:31 ..
-rw-r--r--  1 burt  staff    68B 11  2 20:34 go.mod
-rw-r--r--  1 burt  staff   2.8K 11  2 20:35 go.sum
-rw-r--r--  1 burt  staff   232B 11  2 20:36 main.go
-rwxr-xr-x  1 burt  staff   8.1M 11  2 20:36 myecho
```

`myecho`를 실행한다.

```
$ ./myecho

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v4.1.11
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:1323
```

브라우저에서 localhost:1323 으로 접속하면 결과가 출력된 화면을 볼 수 있다.

```
Hello, World
```

좀 더 다양한 얘기는 다음 글을 참고한다.

* [Using Go Modules](https://blog.golang.org/using-go-modules)
* [Go Modules 살펴보기](https://velog.io/@kimmachinegun/Go-Go-Modules-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-7cjn4soifk)
* [Go modules 시작하기](https://aidanbae.github.io/code/golang/modules/)





