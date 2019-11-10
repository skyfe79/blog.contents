---
title: "Jetpack Compose 첫느낌"
date: 2019-11-10T22:11:47+09:00
draft: false
tags: ["Android", "Jetpack", "Compose"]
---

Android Studio 4.0 Canary 1이 배포된 이후로 Jetpack Compose에 대한 내용이 하나둘씩 채워지고 있다. Jetpack Compose에 대한 공식 튜토리얼이 나와서 따라해 보았다.

* [Jetpack Compose 공식 튜토리얼](https://developer.android.com/jetpack/compose/tutorial)

우선은 SwiftUI와 비슷하게 Compose로 구성한 앱을 시뮬레이터나 디바이스에 직접 심지 않고 미리 보기를 통해 레이아웃을 확인할 수 있다. 미리 보기를 위해서 `@Preview` 어노테이션을 제공한다. 단, `@Preview`로 설정한 함수는 매개변수를 받을 수 없다. 매개변수가 필요한 `@Compose`함수라면 임의의 매개변수를 제공하여 미리 보기 함수를 구성해야 한다.

```kotlin
@Preview
@Composable
fun PreviewGreeting() {
    Greeting("Android")
}
```

dp, px 등과 같은 단위가 Number 타입의 확장으로 제공되어 매우 편리하고 가독성이 높아졌다. 기존에 dp2px, px2dp 같은 함수를 이제 만들지 않아도 된다.

```kotlin
@Composable
fun NewsStory() {
    Column(
        crossAxisSize = LayoutSize.Expand,
        modifier=Spacing(16.dp)
    ) {
        Text("A day in Shark Fin Cove")
        Text("Davenport, California")
        Text("December 2018")
    }
}
```

모서리가 둥근 이미지를 화면에 그리고자 할 때, 코드의 SwiftUI 보다 depth가 깊다. 레이아웃과 컨테이너 그리고 기타 등등을 표현하려면 코드의 들여쓰기가 상당히 깊어지지 않을까... 이럴 경우에는 해당 부분을 `@Composable`함수로 만들어 사용해야 한다.

```kotlin
@Composable
fun NewsStory() {
    val image = +imageResource(R.drawable.header)

    MaterialTheme {
        Column(
            crossAxisSize = LayoutSize.Expand,
            modifier=Spacing(16.dp)
        ) {
            Container(expanded = true, height = 180.dp) {
                Clip(shape = RoundedCornerShape(8.dp)) {
                    DrawImage(image)
                }
            }

            HeightSpacer(16.dp)

            Text("A day in Shark Fin Cove")
            Text("Davenport, California")
            Text("December 2018")
        }
    }
}
```

하지만 코드는 매우 쉽게 읽힌다. 안드로이드의 기존 UI 작업에 비교하면 정말 감사할 따름이다 :) 

공식 튜토리얼에서 List와 Navigation 부분이 없어서 약간 아쉽다. 이제 슬슬 Jetpack Compose를 다뤄봐야겠다.

흥미로운 글은 코멘트로 계속해서...