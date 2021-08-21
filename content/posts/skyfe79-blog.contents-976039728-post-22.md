---
title: "Swift로 스크립트 작성하기"
date: 2021-08-21T04:31:43Z
draft: false
tags: ["swift"]
---


`참고:` 이 글은 [Using Swift for scripting](https://rderik.com/blog/using-swift-for-scripting/)를 간략 번역한 것입니다.

## Hello World

Swif로 스크립트를 작성할 수 있다. 방법은 간단하다. `#!`(hashbang)을 사용하여 스크립트를 해석할 프로그램을 swift로 지정하면 된다. hello.swift 파일을 만들고 아래 내용을 작성한다.

```swift
#!/usr/bin/env swift

print("Hello World!")
```

작성한 내용을 저장하고 스크립트를 실행해 보자.

```
$ swift hello.swift 
Hello World!
```

물론 실행 권한을 주어 바로 실행할 수도 있다.

```
$ chmod +x hello.swift
$ ./hello.swift
Hello World!
```




## 스크립트로 다른 명령어 실행하기

스크립트 내에서 다른 명령어를 실행할 수 있다. 명령어 실행을 위해서는 Process 인스턴스를 사용해야 한다. `ls` 명령어를 실행해 보자.

```swift
#!/usr/bin/env swift

import Foundation

let ls = Process()
ls.executableURL = URL(fileURLWithPath: "/usr/bin/env")
ls.arguments = ["ls", "-al"]

do {
  try ls.run()
} catch {
  print(error.localizedDescription)
}
```

실행 결과는 아래와 같다.

```
$ ./run_ls.swift 
total 72                                                                                                                                                                          
drwxr-xr-x  11 burt  staff  352  8 21 13:17 .
drwxr-xr-x   9 burt  staff  288  8 21 12:34 ..
-rwxr-xr-x   1 burt  staff  442  8 21 12:56 capture_output.swift
-rw-r--r--   1 burt  staff   29  8 21 13:16 error.log
-rwxr-xr-x   1 burt  staff   43  8 21 12:34 hello.swift
-rwxr-xr-x   1 burt  staff  230  8 21 13:20 input.swift
-rwxr-xr-x   1 burt  staff  219  8 21 13:06 list.swift
-rwxr-xr-x   1 burt  staff  264  8 21 13:16 list2.swift
-rw-r--r--   1 burt  staff  605  8 21 13:00 pipe.swift
-rw-r--r--   1 burt  staff  651  8 21 13:02 pipe_with_wait.swift
-rwxr-xr-x   1 burt  staff  213  8 21 12:39 run_ls.swift
```

## 명령어 출력 캡쳐하기

모든 프로세스는 표준 출력(stdout), 표준 오류(stderr), 표준 입력(stdin)을 가지고 있다. stdout과 stderr는 데이터를 출력하고 stdin은 데이터를 입력 받는다. 프로세스 실행 결과를 캡쳐하기 위해서는 stdout을 파이프(pipe)로 리디렉션하고 결과를 파이프에서 읽을 수 있다. `bash`에서 파이프 연산자 `|`를 사용하는 것과 동일하다.

```swift
#!/usr/bin/env swift

import Foundation

let ls = Process()
ls.executableURL = URL(fileURLWithPath: "/bin/ls")
ls.arguments = ["-al"]

var pipe = Pipe()

ls.standardOutput = pipe

do {
  try ls.run()
  let data = pipe.fileHandleForReading.readDataToEndOfFile()
  if let output = String(data: data, encoding: String.Encoding.utf8) {
    print("=> This is captured output.")
    print(output)
  }
} catch {
  print(error.localizedDescription)
}
```

위 코드를 보면 ls 프로세스의 표준 출력을 Pipe 인스턴스로 설정하고 나중에 결과를 pipe에서 읽는 것을 확인할 수 있다. 실행 결과는 아래와 같다.

```
$ ./capture_output.swift 
=> This is captured output.
total 72
drwxr-xr-x  11 burt  staff  352  8 21 13:17 .
drwxr-xr-x   9 burt  staff  288  8 21 12:34 ..
-rwxr-xr-x   1 burt  staff  442  8 21 12:56 capture_output.swift
-rw-r--r--   1 burt  staff   29  8 21 13:16 error.log
-rwxr-xr-x   1 burt  staff   43  8 21 12:34 hello.swift
-rwxr-xr-x   1 burt  staff  230  8 21 13:20 input.swift
-rwxr-xr-x   1 burt  staff  219  8 21 13:06 list.swift
-rwxr-xr-x   1 burt  staff  264  8 21 13:16 list2.swift
-rw-r--r--   1 burt  staff  605  8 21 13:00 pipe.swift
-rw-r--r--   1 burt  staff  651  8 21 13:02 pipe_with_wait.swift
-rwxr-xr-x   1 burt  staff  213  8 21 12:39 run_ls.swift
```

## 여러 개의 명령어 파이프로 연결하여 실행하기

`bash`에서 명령어를 `|`로 조합하여 사용하는 것은 아주 흔한 작업이다. 만약 `ls`와 `sort`를 파이프로 연결하여 실행한다면 ls의 표준 출력을 파이프에 연결하고 파이프를 sort의 표준 입력에 연결하면 된다. 아래는 이를 코드로 구현한 내용이다.

```swift
#!/usr/bin/env swift

import Foundation

let ls = Process()
ls.executableURL = URL(fileURLWithPath: "/bin/ls")
ls.arguments = ["-al"]

var pipe = Pipe()

ls.standardOutput = pipe

let sort = Process()
let completePipe = Pipe()

sort.executableURL = URL(fileURLWithPath: "/usr/bin/env")
sort.arguments = ["sort"]
sort.standardInput = pipe
sort.standardOutput = completePipe

do {
  try ls.run()
  try sort.run()
  let data = completePipe.fileHandleForReading.readDataToEndOfFile()
  if let output = String(data: data, encoding: .utf8) {
    print(output)
  }
} catch {
  print(error.localizedDescription)
}
```

실행해 보자.

```
$ ./pipe.swift 
-rw-r--r--   1 burt  staff   29  8 21 13:16 error.log
-rw-r--r--   1 burt  staff  651  8 21 13:02 pipe_with_wait.swift
-rwxr-xr-x   1 burt  staff   43  8 21 12:34 hello.swift
-rwxr-xr-x   1 burt  staff  213  8 21 12:39 run_ls.swift
-rwxr-xr-x   1 burt  staff  219  8 21 13:06 list.swift
-rwxr-xr-x   1 burt  staff  230  8 21 13:20 input.swift
-rwxr-xr-x   1 burt  staff  264  8 21 13:16 list2.swift
-rwxr-xr-x   1 burt  staff  442  8 21 12:56 capture_output.swift
-rwxr-xr-x   1 burt  staff  605  8 21 13:00 pipe.swift
drwxr-xr-x   9 burt  staff  288  8 21 12:34 ..
drwxr-xr-x  11 burt  staff  352  8 21 13:17 .
total 72
```

`ls`와 `sort` 실행이 빨리 끝났지만 매우 오래 걸리는 프로세스라면 `waitUntExit`를 사용해 끝날 때까지 기다리는 것이 안전하다. 

```swift
#!/usr/bin/env swift

import Foundation

let ls = Process()
ls.executableURL = URL(fileURLWithPath: "/bin/ls")
ls.arguments = ["-al"]

var pipe = Pipe()

ls.standardOutput = pipe

let sort = Process()
let completePipe = Pipe()

sort.executableURL = URL(fileURLWithPath: "/usr/bin/env")
sort.arguments = ["sort"]
sort.standardInput = pipe
sort.standardOutput = completePipe

do {
  try ls.run()
  try sort.run()

  ls.waitUntilExit()
  sort.waitUntilExit()

  let data = completePipe.fileHandleForReading.readDataToEndOfFile()
  if let output = String(data: data, encoding: .utf8) {
    print(output)
  }
} catch {
  print(error.localizedDescription)
}
```



## 오률을 옳게 처리하기

사용자 목록을 출력하는 스크립트 `user.swift`를 구현해 보자.

```swift
#!/usr/bin/env swift
import Foundation

var users = ["zoe", "joe", "albert", "james"]

for user in users {
  print(user)
}
```

그리고 파이프를 사용하여 sort와 함께 실행해 보자.

```
$ ./list.swift | sort
```

```
$ ./list.swift | sort

albert
james
joe
zoe
```

만약 빈문자열 이름일 경우에는 오류를 출력한다고 가정해 보자.

```swift
#!/usr/bin/env swift
import Foundation

var users = ["zoe", "joe", "albert", "james", ""]

if users.contains(""){
    print("Error: there is an empty name")
} else {
  for user in users {
    print(user.capitalized)
  }
}
```

위 코드를 `&&` 연자로 아래와 같이 실행하면 결과는 어떻게 될까? 참고로 `&&` 연산자는 앞의 명령어가 오류 없이 실행되면 뒤의 명령어를 실행하는 연산자다.

```
$ ./list.swift && echo "Users validated successfully." 
Error: there is an empty name
Users validated successfully.
```

오류가 발생했지만 `&&` 는 echo를 실행했다. 그 이유는 `list.swift`가 정상적으로 종료되었기 때문이다. 명령어의 종료 코드를 알 수 있는 방법이 있다. `$?`를 출력해 보면 된다.

```
$ ./list.swift
Error: there is an empty name
$ echo $?
0
```

오류가 있어 비정상 종료 되었다고 알리기 위해서는 0이 아닌 오류코드를 반환하면 된다. 이 때, `exit()`함수를 사용할 수 있다. 아래와 같이 해보자.

```swift
#!/usr/bin/env swift

import Foundation

var users = ["zoe", "joe", "albert", "james", ""]

if users.contains("") {
  print("Error: there is an empty name")
  exit(1)
} else {
  for user in users {
    print(user)
  }
}
```

이제 다시 아래 명령어를 실행하면 오류문구만 출력되는 것을 확인할 수 있다.

```
$ ./list.swift && echo "Users validated successfully." 
Error: there is an empty name
```



## STDOUT과 STDERR

표준 출력을 파일로 리디렉션하여 저장하려고 할 때, `>`를 사용한다. 여기에 파일 디스크립터 번호를 붙이면 표준 출력(1) 또는 표준 오류(2)를 지정하여 파일로 저장할 수 있다.

```
$ ls > list.txt
$ cat list.txt
cat list.txt 
capture_output.swift
error.log
hello.swift
input.swift
list.swift
list.txt
list2.swift
pipe.swift
pipe_with_wait.swift
run_ls.swift
```

```
$ ls 1> list.txt
$ cat list.txt
capture_output.swift
error.log
hello.swift
input.swift
list.swift
list.txt
list2.swift
pipe.swift
pipe_with_wait.swift
run_ls.swift
```

```
$ ls 2> error.txt
capture_output.swift error.txt            hello.swift          list.swift           list2.swift          pipe_with_wait.swift
error.log            error2.txt           input.swift          list.txt             pipe.swift           run_ls.swift
$ cat error.txt
```

`ls 2> error.txt` 의 경우는 표준 오류에 값이 없기 때문에 ls의 표준 출력이 화면에 표시되고 `error.txt` 는 아무 내용이 없는 `0Byte` 크기의 파일이 된다.




없는 파일을 ls로 실행하면 어떻게 될까? 표준 오류로 오류가 출력됨을 확인할 수 있다.

```
$ ls  non_existant_file.txt 2> error.log
$ cat error.log
ls: non_existant_file.txt: No such file or directory
```

빈문자열 이름이 있을 경우에 오류를 출력하는 `list.swift`로 테스트 해 보자.

```
$ ./list.swift 2> error.log
Error: there is an empty name
$ cat error.log
```

표준 출력으로 오류가 출력되고 표준 오류 내용을 저장한 error.log에는 아무런 내용도 담기지 않는다. 그 이유는 `print()`함수가 표준 출력으로 문자열을 출력하기 때문이다.

표준 오류로 오류 내용을 출력하기 위해서는 직접 표준 오류에 써 주어야 한다.

```swift
#!/usr/bin/env swift

import Foundation

var users = ["zoe", "joe", "albert", "james", ""]

if users.contains("") {
  FileHandle.standardError.write("Error: there is an empty name".data(using: .utf8)!)
  exit(1)
} else {
  for user in users {
    print(user)
  }
}
```

이제 다시 실행해 보자.

```
$ ./list2.swift 2> error.log
$ cat error.log
Error: there is an empty name
```

원하는 대로 오류 문구가 표준 오류로 출력되었다. 이제 처음에 `&&` 연산자를 다시 실행해 보자.

```
$ ./list2.swift && echo "Users validated successfully." 
Error: there is an empty name
```

실행이 올바르게 됨을 확인할 수 있다.

## 입력 받기

실행 인자는 `CommandLine.arguments`을 통해 얻을 수 있다. 실행 인자의 갯수는 `CommandLine.argc`로 알 수 있다.

```swift
#!/usr/bin/env swift

import Foundation

print(CommandLine.arguments)
print("Number of arguments: \(CommandLine.argc)")
```

```
$ ./input.swift these are the arguments
["./input.swift", "these", "are", "the", "arguments"]
Number of arguments: 5
```

사용자 입력은 `readLine`함수로 받을 수 있다.

```swift
#!/usr/bin/env swift

import Foundation

print(CommandLine.arguments)
print("Number of arguments: \(CommandLine.argc)")

print("Enter your name: ")
var name = readLine(strippingNewline: true)
print("Hello \(name ?? "anonymouse")")
```

```
$ ./input.swift these are the arguments
["./input.swift", "these", "are", "the", "arguments"]
Number of arguments: 5
Enter your name: 
Burt.K
Hello Burt.K
```

`readLine()` 함수는 표준 입력에서 값을 읽으므로 아래와 같이 파이프를 사용하여 실행할 수도 있다.

```
$ echo "Burt.K" | ./input.swift
["./input.swift"]
Number of arguments: 1
Enter your name: 
Hello Burt.K
```

Swift 스크립트는 쉽고 작성하고 빠르게 실행해 볼 수 있어 매우 유용하다. 그러나 오픈소스 패키지를 쉽게 사용할 수 없디는 점이 아쉽다. 