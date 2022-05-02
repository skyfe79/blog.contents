---
title: "Firebase Functions 환경 변수 정의 및 사용하기"
date: 2022-05-02T02:28:04Z
author: "skyfe79"
draft: false
tags: ["tips"]
---


Node 에서 `process.env.some_secret_key` 와 같이 환경 변수를 사용하는 경우가 많다. Firebase fuctions를 사용할 때, 이런 부류의 환경 변수가 필요한 경우가 있다. 그럴 때는 아래와 같이 functions의 config에 환경 변수를 정의할 수 있다.

```
$ firebase functions:config:set aws.access_key_id="your_key"
$ firebase functions:config:set aws.secret_access_key="your_key"
```

그리고 코드에서는 아래와 같이 사용한다.

```javascript
AWS.config.update({
  accessKeyId: functions.config().aws.access_key_id,
  secretAccessKey: functions.config().aws.secret_access_key,
});
```

`주의` functions 환경 변수는 functions 런타임 환경에서만 접근 가능하다. 만약 functions.https 에 연결된 REST API를 구현하고 해당 API에서 functions 환경 변수에 접근한다고 가정해 보자. 로컬 머신에서 REST 서버를 띄워 REST 서버에 직접 요청을 보내 테스트를 하면 `functions.config()...` 환경 변수 값을 읽을 수 없다. functions 런타임 환경이 구성되지 않았기 때문이다. 대신 functions 쉘을 실행하여 functions를 실행하면 된다. 

```
$ firebase functions:shell
...
⚠  functions: The following emulators are not running, calls to these services will affect production: firestore, database, pubsub, storage
firebase > 
firebase > someFunction()
```

