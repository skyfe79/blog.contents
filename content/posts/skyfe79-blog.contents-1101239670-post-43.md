---
title: "Google Sheet API 인증"
date: 2022-01-13T05:53:58Z
author: "skyfe79"
draft: false
tags: ["tips"]
---

https://developers.google.com/sheets/api 를 사용하여 비공개(private) Google Sheet 에 접근해야할 때, 인증부분에서 겪은 삽질을 공유한다.

## API KEY

먼저 알아야 할 것은 API KEY로 접근하는 것은 공개(public) sheet 만 read-only 권한으로 가능하다. 그 이외의 접근에는 API KEY를 사용할 수 없다.

## Service Account

비공개 sheet에 접근할 때는 [Google Cloud Platform](https://console.cloud.google.com/) 에서 발급할 수 있는 서비스 계정(Service Account)이 필요하다.

Google Sheet를 접근하고자 하는 프로젝트로 이동하여 `API 및 서비스` > `라이브러리` 메뉴를 선택한다.

<img width="403" alt="스크린샷 2022-01-13 오후 2 50 06" src="https://user-images.githubusercontent.com/309935/149297944-1ab8fe4c-b877-44a9-8845-3ee55f0361ee.png">



Google Sheet API를 검색한 후 `사용설정`을 한다. 그 후 `관리` 버튼을 눌러 서비스 계정을 생성한다.

<img width="730" alt="스크린샷 2022-01-13 오후 2 50 28" src="https://user-images.githubusercontent.com/309935/149298130-d1d590b9-49ff-493b-8601-d2085b84f40e.png">



`사용자 인증 정보` 를 선택한다.

<img width="755" alt="스크린샷 2022-01-13 오후 2 50 53" src="https://user-images.githubusercontent.com/309935/149298190-aaa5ef42-67f9-4e81-84f9-6a25d06ca245.png">



그 후 `+ 사용자 인증 정보 만들기` 를 선택하고 `서비스 계정`을 선택한다.

<img width="603" alt="스크린샷 2022-01-13 오후 2 51 03" src="https://user-images.githubusercontent.com/309935/149298291-2277bcb1-d638-432f-8436-0144a2f01d70.png">

`서비스 계정 이름`과 `서비스 계정 ID`를 입력하고 `완료` 버튼을 누른다. 서비스 계정 ID가 중요하다. 나중에 문서 공유를 설정할 때, 여기에서 생성한 이메일이 필요하기 때문이다.

<img width="578" alt="스크린샷 2022-01-13 오후 2 51 33" src="https://user-images.githubusercontent.com/309935/149298480-c2471788-2d2e-42e1-8b75-90a1b858c373.png">

서비스 계정 생성이 완료되면 해당 서비스 계정을 선택하여 상세 화면으로 진입한다. 그리고 `키` 메뉴를 선택하고 `키 추가` > `새 키 만들기` 를 선택한다.

<img width="573" alt="스크린샷 2022-01-13 오후 2 51 59" src="https://user-images.githubusercontent.com/309935/149298673-3a89d4ca-b98c-40a9-bc9b-d770cad2f13a.png">

이어 나오는 모달 화면에서 `JSON` 을 선택하면 서비스 계정 키가 JSON 형식으로 생성되고 로컬 컴퓨터로 다운로드 된다.

<img width="617" alt="스크린샷 2022-01-13 오후 2 52 07" src="https://user-images.githubusercontent.com/309935/149298833-9eaefa28-1ef9-42f6-a25f-62f931857b4a.png">

해당 파일에는 아래와 같은 정보가 포함되어 있다.

```json
{
  "type": "service_account",
  "project_id": "...",
  "private_key_id": "...",
  "private_key": "...",
  "client_email": "my-google-sheet-bot@youtext-dashboard.iam.gserviceaccount.com",
  "client_id": "...",
  "auth_uri": "...",
  "token_uri": "...",
  "auth_provider_x509_cert_url": "...",
  "client_x509_cert_url": "..."
}
```

위 정보를 바탕으로 [google-spreadsheet
](https://www.npmjs.com/package/google-spreadsheet) 라이브러를 사용하여 문서에 접근해 보자.

```javascript
const creds = require('./my-project-service-account-key.json');
const { GoogleSpreadsheet } = require('google-spreadsheet');

(async () => {
  try {
    // Initialize the sheet - doc ID is the long id in the sheets URL
    const doc = new GoogleSpreadsheet('Google Sheet ID');
    await doc.useServiceAccountAuth(creds);
    await doc.loadInfo();
    console.log(doc.title);
  } catch (error) {
    console.log(error.message);
  }
})();
```

실행해 보면 `Google API error - [403] The caller does not have permission` 오류가 발생한다.

그 이유는 Google Sheet 에 접근하기 위해서는 위 서비스 계정과 문서를 공유해야 하기 때문이다. API 접근하려는 Google Sheet로 이동한다.

문서에서 `공유` 버튼을 누른다.

<img width="1190" alt="스크린샷 2022-01-13 오후 6 10 23" src="https://user-images.githubusercontent.com/309935/149307871-ec61fcb7-f8fc-4ea4-92a2-982a6df5fcb8.png">

`사용자 및 그룹 추가` 란에 서비스 계정의 이메일을 추가한다.

<img width="692" alt="스크린샷 2022-01-13 오후 6 11 03" src="https://user-images.githubusercontent.com/309935/149307944-009cb48c-18de-4081-aa8d-289cbc87112d.png">



`보내기` 버튼을 눌러 공유를 완료한다.

<img width="694" alt="스크린샷 2022-01-13 오후 6 11 25" src="https://user-images.githubusercontent.com/309935/149308115-bb5e7b0c-55ef-4fb1-8fca-ed945aacd306.png">

`사용자 및 그룹과 공유`에 방금 추가한 서비스 계정이 추가되었음을 알 수 있다. 필요에 따라 공유 타입을 `뷰어`, `댓글 작성자`, `편집자` 중에 하나로 변경한다.

<img width="678" alt="스크린샷 2022-01-13 오후 6 11 46" src="https://user-images.githubusercontent.com/309935/149308252-58b362a7-71ab-4ce6-84f5-4abe6c0d222b.png">

이제 다시 아래 코드를 실행해 보자.

```js
const creds = require('./my-project-service-account-key.json');
const { GoogleSpreadsheet } = require('google-spreadsheet');

(async () => {
  try {
    // Initialize the sheet - doc ID is the long id in the sheets URL
    const doc = new GoogleSpreadsheet('Google Sheet ID');
    await doc.useServiceAccountAuth(creds);
    await doc.loadInfo();
    console.log(doc.title);
  } catch (error) {
    console.log(error.message);
  }
})();
```

```
$  node index.js
테스트용
```

문서의 제목이 출력됨을 확인할 수 있다.
