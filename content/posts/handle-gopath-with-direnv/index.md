---
title: "direvn로 $GOPATH 다루기"
date: 2019-11-10T14:10:12+09:00
draft: false
tags: ["Go", "GOPATH", "direnv", "아마추어 번역"]
---

Go언어를 아직 많이 사용해 본 것은 아니지만 go modules를 사용할 때 로컬 패키지 처리가 번거러운 것 같아서 `$GOPATH`를 다루는 쉬운 방법이 없을까 찾아 보았습니다. 그러던 중 `direnv`를 발견하여 관련 내용을 정리해 봅니다.

## direnv

[direnv](https://direnv.net/)는 쉘의 환경 변수를 재정의하는 유틸리티입니다. Go언어는 `$GOPATH`를 워크스페이스마다 달리 설정해야 합니다. 하지만 매번 `.bash_profile`이나 `.zshrc` 같은 환경 파일을 편집해서 $GOPATH를 변경하는건 뭐랄까... 음... 

파이썬에는 `virtualenv`같은 가상의 개발 환경을 만들어주는 도구가 존재하지만 Go언어는 그렇지 않습니다. 워크스페이스 폴더를 `$GOPATH`로 설정해 주어야 할 때 `direnv`를 사용하면 아주 편리합니다.

### direnv 설치

이 글은 독자가 MacOSX 및 zsh을 사용하고 있다는 전제하에 작성하였습니다. direnv는 brew를 통해서 쉽게 설치할 수 있습니다.

```
$ brew update
$ brew install direnv
$ echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
```

설치와 설정을 마친 후 터미널을 종료했다가 다시 실행합니다. 다시 실행하지 않으면 direnv가 동작하지 않을 경우가 많더군요.

### 테스트

`testgo`라는 폴더를 생성합니다. 

```
$ mkdir testgo
$ cd testgo
$ echo 'export GOPATH=$(PWD):$GOPATH' >> .envrc
$ echo 'export PATH=$(PWD)/bin:$PATH' >> .envrc 
$ direnv allow
```

실행 도중에 `direnv: error .envrc is blocked. Run `direnv allow` to approve its content.` 오류를 만나도 무시하면 됩니다. `direnv allow`를 실행하지 않아서 발생하는 문제입니다. `direnv allow`를 실행하면 아래와 같이 `.envrc`환경 변수를 읽어서 `$PATH`에 `$GOPATH`를 추가한 것을 알 수 있습니다.

```
direnv: loading .envrc                                                                                                                                                          
direnv: export +GOPATH ~PATH
```

`$GOPATH`를 확인해 봅시다.

```
$ echo $GOPATH
/Users/burt/testgo
```

`$GOPATH`가 잘 변경된 것을 확인할 수 있습니다.

## mkgoproject 만들기

`direnv`를 사용하더라더 매번 `.envrc`파일을 만드는 과정이 귀찮을 수 있습니다. zsh 에서 실행할 수 있는 쉘함수를 만들어 사용하면 귀차니즘을 쉽게 해결할 수 있습니다.

```bash
function mkgoproject {
  TRAPINT() {
    print "Caught SIGINT, aborting."
    return $(( 128 + $1 ))
  }
  echo 'Creating new Go project:'
  if [ -n "$1" ]; then
    project=$1
  else
    while [[ -z "$project" ]]; do 
      vared -p 'what is your project name: ' -c project; 
    done
  fi
  namespace='github.com/skyfe79'
  while true; do 
    vared -p 'what is your project namespace: ' -c namespace 
    if [ -n "$namespace" ] ; then 
       break
    fi
  done
  mkdir -p $project/src/$namespace/$project
  git init -q $project/src/$namespace/$project
  main=$project/src/$namespace/$project/main.go
  echo 'export GOPATH=$(PWD):$GOPATH' >> $project/.envrc
  echo 'export PATH=$(PWD)/bin:$PATH' >> $project/.envrc
  echo 'package main' >> $main 
  echo 'import "fmt"' >> $main
  echo 'func main() {' >> $main
  echo '    fmt.Println("hello world")' >> $main 
  echo '}' >> $main
  direnv allow $project
  echo "cd $project/src/$namespace/$project #to start coding"
}
```

위 함수를 `.zshrc`에 정의한 다음, 터미널을 재시작 후 아래처럼 프로젝트를 생성합니다.

```
$ mkgoproject hellogo
Creating new Go project:
what is your project namespace: github.com/skyfe79
엔터
cd hellogo/src/github.com/skyfe79/hellogo #to start coding
```

그러면 위처럼 네임스페이스를 묻게 되고 설정 후 엔터를 누르면 아래와 같은 폴더 구조로 프로젝트를 생성합니다.

```
hellogo
└── src
    └── github.com
        └── skyfe79
            └── hellogo
                └── main.go

4 directories, 1 file
```

`알림` 이 글은 [Handling Go workspace with direnv](http://rachbelaid.com/handling-go-workspace-with-direnv/)을 편역한 글입니다.