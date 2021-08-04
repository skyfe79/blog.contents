---
title: "Go 로컬 패지키 모듈"
date: 2019-11-10T17:35:24+09:00
draft: false
tags: ["Go", "modules", "local package"]
---

Go modules를 사용하면 `$GOPATH`를 신경쓰지 않고 Go 프로젝트를 시작할 수 있습니다. 이미 오픈되어 있는 Go모듈을 사용할 경우에는 아래처럼 `go.mod`파일에 패키지의 주소와 버전을 기입하면 패키지를 모듈로 사용할 수 있습니다.

```go
module star

go 1.13

require (
	github.com/fogleman/gg v1.3.0
	github.com/golang/freetype v0.0.0-20170609003504-e2365dfdc4a0 // indirect
	golang.org/x/image v0.0.0-20191009234506-e7c1f5e7dbb8 // indirect
)
```

## 로컬패키지

그렇다면 로컬에 존재하는 패키지는 어떻게 `go.mod`파일에 기입할 수 있을까요? 아래와 같은 폴더 구조를 가지는 프로젝트를 생각해 보겠습니다.

```
hellogo
├── constants
└── logics
```

hellogo 폴더 하위에 모듈로 constants와 logics를 가지고 있습니다. 각 디렉토리로 진입하여 go 파일을 생성합니다. 그리고 `go mod init <MODULE NAME>`을 실행합니다.

```
$ cd constants
$ touch constants.go
$ go mod init constants
```

logics폴더로 진입하여 같은 작업을 합니다.

```
$ cd logics
$ touch logics.go
$ go mod init logics
```

지금까지 한 것은 각 모듈을 `go modules`를 사용해 초기화한 것입니다. 이 모듈들을 사용하려면 메인 모듈에서 모듈 설정을 해 주어야 합니다.

`hellogo`폴더에서 `go mod init`을 실행합니다.

```
$ go mod init hellogo
```

그리고 편집기로 `go.mod`파일을 엽니다.

```
$ vi go.mod
```

```go
module hellogo

go 1.13
```

사용할 로컬 패키지를 기입합니다. 로컬 패지지를 입력할 때 `도메인주소/패키지명 버전`형식으로 입력해야 합니다. 도메인주소를 생략하면 go 버전에 따라서 오류가 발생할 수도 있습니다. 아래처럼 임의의 도메인 주소를 붙여 패키지를 기입합니다. 

```go
module hellogo

go 1.13

require (
	hellogo.com/constants v0.0.0
	hellogo.com/logics v0.0.0
)
```

패키지 주소와 버전을 임의로 붙였습니다. 실제 위 주소에 패키지가 없을 것입니다. 그래서 `replace`를 사용해서 패키지 주소를 로컬 패스로 변경해 주어야 합니다.

```go
module hellogo

go 1.13

require (
	hellogo.com/constants v0.0.0
	hellogo.com/logics v0.0.0
)

replace (
	hellogo.com/constants v0.0.0 => ./constants
	hellogo.com/logics v0.0.0 => ./logics
)
```

위처럼 하면 hellogo 를 만들기 위한 모듈 구성이 끝난 것입니다. 이제 각 모듈을 사용해 보기 위해서 go 파일에 코드를 작성해 봅시다. 

## constants.go

```go
package constants

const (
	NAME = "Burt.K"
	AGE  = 29
)
```

## logics.go

```go
package logics

import (
	"fmt"
)

func Greeting(name string, age int) {
	fmt.Println(name, age)
}
```

## main.go

```go
package main

import (
	"hellogo.com/constants"
	"hellogo.com/logics"
)

func main() {
	logics.Greeting(constants.NAME, constants.AGE)
}
```

## go build

이제 터미널에서 빌드를 수행합니다.

```
$ go build
```

결과 실행파일을 실행하면 원하는 결과가 출력됨을 알 수 있습니다.

```
$ ./hellogo
Burt.K 29
```


