---
title: "Swift로 Command Line 앱 만들기 #3"
date: 2021-08-23T11:51:51Z
draft: false
tags: ["swift"]
---

앞서 살펴본 [Swift로 Command Line 앱 만들기 #1](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976285028-post-23/) 과 [Swift로 Command Line 앱 만들기 #2](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976291146-post-24/) 글은 모두 동기식으로 명령어를 처리하여 별 어려움이 없었다.

하지만 앱을 개발할 때는, 비동기식으로 명령어를 처리해야 할 경우가 많다. 대표적인 예가 네트워크 API 요청일 것이다. 그래서 3편에서는 네트워크 API를 호출하여 앱스토어에 출신된 앱의 정보를 조회하는 커맨드라인 앱을 만들어 볼 것이다.

## 프로젝트 생성

`appinfo` 이름으로 프로젝트를 생성한다. 

```
$ mkdir appinfo && cd appinfo
$ swift package init --type executable
```



## 디펜던시 추가

`appinfo` 프로젝트는  `ArgumentParser` 와 `Alamofire`를 사용할 것이다. `Package.swift` 파일을 아래와 같이 입력한다.

```swift
// swift-tools-version:5.3
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "appinfo",
    platforms: [
        .macOS(.v10_15)
    ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        .package(url: "https://github.com/apple/swift-argument-parser", from: "0.4.4"),
        .package(url: "https://github.com/Alamofire/Alamofire.git", .upToNextMajor(from: "5.4.0")),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "appinfo",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
                .product(name: "Alamofire", package: "Alamofire")
            ]),
        .testTarget(
            name: "appinfoTests",
            dependencies: ["appinfo"]),
    ]
)
```

외부 패키지 설치를 위해 빌드를 한다.

```
$ swift build
Fetching https://github.com/Alamofire/Alamofire.git from cache
Fetching https://github.com/apple/swift-argument-parser from cache
Cloning https://github.com/apple/swift-argument-parser
Resolving https://github.com/apple/swift-argument-parser at 0.4.4
Cloning https://github.com/Alamofire/Alamofire.git
Resolving https://github.com/Alamofire/Alamofire.git at 5.4.3
```


## 비동기 처리

`main.swift` 파일에 적은 함수 실행문은 메인스레드에서 순차적으로 동기로 실행된다. 만약 비동기로 실행하기 위해서는 비동기 실행을 한 다음 비동기 실행이 끝날 때까지 메인스레드를 잠시 멈추어야 한다. 그래야 비동기 실행이 마무리 될 때까지 프로그램이 종료되지 않고 대기할 수 있다.

Swift에서는 이러한 내용을 [DispatchGroup](https://developer.apple.com/documentation/dispatch/dispatchgroup)을 사용하여 구현할 수 있다. 일련의 비동기 작업을 하나의 DispatchGroup에 담아 실행하고 그룹에 담긴 모든 비동기 작업이 끝나면 DipatchGroup은 completion handler를 호출한다. [dispatchMain()](https://developer.apple.com/documentation/dispatch/1452860-dispatchmain/) 함수는 메인스레드의 작업 실행을 블럭한다. 

따라서 `DispatchGroup`에 비동기 작업을 담아 실행하고 메인스레드는 `dispatchMain()`함수로 블럭해 놓는다. 그리고 그룹에 담긴 비동기 작업이 모두 끝나면 completion handler에서 `exit()` 함수를 호출해 프로그램을 종료한다.

비동기 실행을 요하는 `ArgumentParser` 프로그램의 모습은 대략 아래와 같다.

```swift
func asyncTask(group: DispatchGroup) {
  group.enter()
  doSomeAsyncTask(completed: {
    group.leave()
  })
}

struct AsyncCommand: ParsableCommand {
  func run() throws {
    let group = DispatchGroup()
    asyncTask(group: group)
    group.notify(queue: .main, execute: {
      AsyncCommand.exit()
    })
    dispatchMain()
  }
}
```

## 실제 구현

이제 위 내용을 바탕으로 번들ID와 앱ID로 앱 정보를 조회하는 커맨드라인 앱을 만들어 보자. 전체 코드 구조는 [Swift로 Command Line 앱 만들기 #2](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976291146-post-24/) 와 크게 다르지 않기 때문에 자세한 설명은 생략한다.

### 비동기 작업을 담아둔 AppInfoModel 구현

```swift
import Foundation
import ArgumentParser
import Combine
import Alamofire

@available(macOS 10.15, *)
class AppInfoModel {
  var cancellables = Set<AnyCancellable>()

  static let baseUrl = "https://itunes.apple.com/lookup"

  func fetchAppInfoBy(bundleId: String) -> Future<String, Never> {
    return Future<String, Never> { promise in
      AF.request("\(AppInfoModel.baseUrl)?bundleId=\(bundleId)").responseJSON { response in 
        if let value = response.value {
          promise(.success("\(value)"))
        } else {
          promise(.success("\(bundleId) 앱을 찾을 수 없습니다."))
        }
      }
    }
  }

  func fetchAppInfoBy(appId: String) -> Future<String, Never> {
    return Future<String, Never> { promise in
      AF.request("\(AppInfoModel.baseUrl)?id=\(appId)").responseJSON { response in 
        if let value = response.value {
          promise(.success("\(value)"))
        } else {
          promise(.success("\(appId) 앱을 찾을 수 없습니다."))
        }
      }
    }
  }
}
```

### appinfo 명령어 구현

```swift
@available(macOS 10.15, *)
struct AppInfo: ParsableCommand {
  static let configuration = CommandConfiguration(commandName: "appinfo", abstract: "앱스토어에 출시한 앱 정보를 조회합니다.", subcommands: [
    BundleId.self,
    AppId.self
  ])
}
```

### bundleId 명령어 구현

```swift
extension AppInfo {
  struct BundleId: ParsableCommand {
    static let configuration = CommandConfiguration(commandName: "bundleId", abstract: "번들ID로 앱 정보를 조회합니다.")

    @Argument(help: "앱의 번들ID")
    var bundleId: String

    func validate() throws {
      guard !bundleId.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else { 
        throw ValidationError("번들ID를 입력해 주세요")
      }
    }

    func run() throws {
      let model = AppInfoModel()
      let group = DispatchGroup()
      group.enter()
      model.fetchAppInfoBy(bundleId: bundleId)
        .receive(on: DispatchQueue.main)
        .sink(receiveCompletion: { _ in 
          group.leave()
        }, receiveValue: { value in 
          print(value)
        })
        .store(in: &model.cancellables)
      group.notify(queue: .main, execute: {
        AppInfo.exit()
      })
      dispatchMain()
    }
  }
}
```

### appId 명령어 구현

```swift
extension AppInfo {
  struct AppId: ParsableCommand {
    static let configuration = CommandConfiguration(commandName: "appId", abstract: "앱ID로 앱 정보를 조회합니다.")

    @Argument(help: "앱ID")
    var appId: String

    func validate() throws {
      guard !appId.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else { 
        throw ValidationError("앱ID를 입력해 주세요")
      }
    }

    func run() throws {
      let model = AppInfoModel()
      let group = DispatchGroup()
      group.enter()
      model.fetchAppInfoBy(appId: appId)
        .receive(on: DispatchQueue.main)
        .sink(receiveCompletion: { _ in 
          group.leave()
        }, receiveValue: { value in 
          print(value)
        })
        .store(in: &model.cancellables)
      group.notify(queue: .main, execute: {
        AppInfo.exit()
      })
      dispatchMain()
    }
  }
}
```

## 실행

우선 도움말을 확인해 보자.

```
$ swift run appinfo --help
OVERVIEW: 앱스토어에 출시한 앱 정보를 조회합니다.

USAGE: appinfo <subcommand>

OPTIONS:
  -h, --help              Show help information.

SUBCOMMANDS:
  bundleId                번들ID로 앱 정보를 조회합니다.
  appId                   앱ID로 앱 정보를 조회합니다.

  See 'appinfo help <subcommand>' for detailed help.
```

### bundleId로 앱 정보 조회

필자가 만든 앱 중 하나인 [Jikon](https://apps.apple.com/kr/app/jikon-cam/id1315263599) 앱 정보를 번들ID를 사용하여 조회해 보자.

```
$ swift run appinfo bundleId com.github.skyfe79.jikon
{
    resultCount = 1;
    results =     (
                {
            advisories =             (
            );
            appletvScreenshotUrls =             (
            );
            artistId = 388959489;
            artistName = "SUNG CHEOL KIM";
            artistViewUrl = "https://apps.apple.com/us/developer/sung-cheol-kim/id388959489?uo=4";
            artworkUrl100 = "https://is2-ssl.mzstatic.com/image/thumb/Purple118/v4/62/67/81/626781d5-2ea6-8901-510c-b48852439bdc/source/100x100bb.jpg";
            artworkUrl512 = "https://is2-ssl.mzstatic.com/image/thumb/Purple118/v4/62/67/81/626781d5-2ea6-8901-510c-b48852439bdc/source/512x512bb.jpg";
            artworkUrl60 = "https://is2-ssl.mzstatic.com/image/thumb/Purple118/v4/62/67/81/626781d5-2ea6-8901-510c-b48852439bdc/source/60x60bb.jpg";
            averageUserRating = "4.2000000000000001776356839400250464677";
            averageUserRatingForCurrentVersion = "4.2000000000000001776356839400250464677";
            bundleId = "com.github.skyfe79.jikon";
            contentAdvisoryRating = "4+";
            currency = USD;
            currentVersionReleaseDate = "2017-12-05T22:21:09Z";
            description = "Jikon is an emotional film camera app that gives you the pleasure of developing and printing with your negative film.\nThe pictures you take are automatically developed as negative film, and you apply filters on negative film to print as real photos.\nWe choose only the filters that match film photos, giving you a total of 68 filters for pretty printing.\nWe recommend this to you!\n\n+ If you want to feel the joy of printing your pictures, use JiKon!\n+ If you want to take pictures during the day and printing the films during the night to organize it, use JiKon!\n+ If you want to print your photos with someone you love after taking pictures, use JiKon!\n+ If you just want to take a pretty picture without any annoying, use JiKon!\n+ If you want simple UI/UX, use JiKon!\n+ If you want to give toy camera app to your kids, use JiKon!\n\n\nJiKon shoots the pictures in 3:4 ratio. You can post the picture to the instagram directly.\nJikon is called 'now' in Japanese. Take a picture with JiKon now!\n\n+ Take JiKon when you travel!\n+ Take pretty landscape pictures with JiKon!\n+ Turn on FILM button, the film effect is applied to the picture!\n+ Turn on DATE button, the shooting date is stamped on the picture!\n+ Turn on GRAIN button, the grain ffect is applied to the picture!\n+ Turn on LEAK button, the lightleak ffect is applied to the picture!\n+ Swipe slider to control exposure!\n+ Swipe slider to control focus!\n+ Swipe slider to control zoom-in & out!\n+ Swipe Up/Down to flip camera position!\n+ Take pictures with volume keys!";
            features =             (
            );
            fileSizeBytes = 91347968;
            formattedPrice = Free;
            genreIds =             (
                6008,
                6012
            );
            genres =             (
                "Photo & Video",
                Lifestyle
            );
            ipadScreenshotUrls =             (
            );
            isGameCenterEnabled = 0;
            isVppDeviceBasedLicensingEnabled = 1;
            kind = software;
            languageCodesISO2A =             (
                EN,
                KO
            );
            minimumOsVersion = "9.0";
            price = 0;
            primaryGenreId = 6008;
            primaryGenreName = "Photo & Video";
            releaseDate = "2017-11-28T05:06:44Z";
            releaseNotes = "This app has been updated by Apple to use the latest Apple signing certificate.\n\n- Fixed minor bugs.\n- Adjusted the size and position of the date stamp.";
            screenshotUrls =             (
                "https://is4-ssl.mzstatic.com/image/thumb/Purple118/v4/7d/01/0a/7d010ae2-9110-de21-8106-6f0bf2faf72a/pr_source.png/392x696bb.png",
                "https://is5-ssl.mzstatic.com/image/thumb/Purple118/v4/fa/a3/82/faa3823f-a240-2aa7-c70c-4716f6946e57/pr_source.png/392x696bb.png",
                "https://is3-ssl.mzstatic.com/image/thumb/Purple118/v4/ef/da/a5/efdaa528-2b2b-c2cc-95a2-4b7b32631f40/pr_source.png/392x696bb.png",
                "https://is5-ssl.mzstatic.com/image/thumb/Purple118/v4/ad/27/c3/ad27c3b6-0963-3fd8-3eb0-93086a17d185/pr_source.png/392x696bb.png",
                "https://is1-ssl.mzstatic.com/image/thumb/Purple128/v4/bb/ff/98/bbff9875-e02a-89d6-9fea-735dc7c53c6b/pr_source.png/392x696bb.png"
            );
            sellerName = "SUNG CHEOL KIM";
            supportedDevices =             (
                "iPad2Wifi-iPad2Wifi",
                "iPad23G-iPad23G",
                "iPhone4S-iPhone4S",
                "iPadThirdGen-iPadThirdGen",
                "iPadThirdGen4G-iPadThirdGen4G",
                "iPhone5-iPhone5",
                "iPodTouchFifthGen-iPodTouchFifthGen",
                "iPadFourthGen-iPadFourthGen",
                "iPadFourthGen4G-iPadFourthGen4G",
                "iPadMini-iPadMini",
                "iPadMini4G-iPadMini4G",
                "iPhone5c-iPhone5c",
                "iPhone5s-iPhone5s",
                "iPadAir-iPadAir",
                "iPadAirCellular-iPadAirCellular",
                "iPadMiniRetina-iPadMiniRetina",
                "iPadMiniRetinaCellular-iPadMiniRetinaCellular",
                "iPhone6-iPhone6",
                "iPhone6Plus-iPhone6Plus",
                "iPadAir2-iPadAir2",
                "iPadAir2Cellular-iPadAir2Cellular",
                "iPadMini3-iPadMini3",
                "iPadMini3Cellular-iPadMini3Cellular",
                "iPodTouchSixthGen-iPodTouchSixthGen",
                "iPhone6s-iPhone6s",
                "iPhone6sPlus-iPhone6sPlus",
                "iPadMini4-iPadMini4",
                "iPadMini4Cellular-iPadMini4Cellular",
                "iPadPro-iPadPro",
                "iPadProCellular-iPadProCellular",
                "iPadPro97-iPadPro97",
                "iPadPro97Cellular-iPadPro97Cellular",
                "iPhoneSE-iPhoneSE",
                "iPhone7-iPhone7",
                "iPhone7Plus-iPhone7Plus",
                "iPad611-iPad611",
                "iPad612-iPad612",
                "iPad71-iPad71",
                "iPad72-iPad72",
                "iPad73-iPad73",
                "iPad74-iPad74",
                "iPhone8-iPhone8",
                "iPhone8Plus-iPhone8Plus",
                "iPhoneX-iPhoneX",
                "iPad75-iPad75",
                "iPad76-iPad76",
                "iPhoneXS-iPhoneXS",
                "iPhoneXSMax-iPhoneXSMax",
                "iPhoneXR-iPhoneXR",
                "iPad812-iPad812",
                "iPad834-iPad834",
                "iPad856-iPad856",
                "iPad878-iPad878",
                "iPadMini5-iPadMini5",
                "iPadMini5Cellular-iPadMini5Cellular",
                "iPadAir3-iPadAir3",
                "iPadAir3Cellular-iPadAir3Cellular",
                "iPodTouchSeventhGen-iPodTouchSeventhGen",
                "iPhone11-iPhone11",
                "iPhone11Pro-iPhone11Pro",
                "iPadSeventhGen-iPadSeventhGen",
                "iPadSeventhGenCellular-iPadSeventhGenCellular",
                "iPhone11ProMax-iPhone11ProMax",
                "iPhoneSESecondGen-iPhoneSESecondGen",
                "iPadProSecondGen-iPadProSecondGen",
                "iPadProSecondGenCellular-iPadProSecondGenCellular",
                "iPadProFourthGen-iPadProFourthGen",
                "iPadProFourthGenCellular-iPadProFourthGenCellular",
                "iPhone12Mini-iPhone12Mini",
                "iPhone12-iPhone12",
                "iPhone12Pro-iPhone12Pro",
                "iPhone12ProMax-iPhone12ProMax",
                "iPadAir4-iPadAir4",
                "iPadAir4Cellular-iPadAir4Cellular",
                "iPadEighthGen-iPadEighthGen",
                "iPadEighthGenCellular-iPadEighthGenCellular",
                "iPadProThirdGen-iPadProThirdGen",
                "iPadProThirdGenCellular-iPadProThirdGenCellular",
                "iPadProFifthGen-iPadProFifthGen",
                "iPadProFifthGenCellular-iPadProFifthGenCellular"
            );
            trackCensoredName = "JiKon Cam";
            trackContentRating = "4+";
            trackId = 1315263599;
            trackName = "JiKon Cam";
            trackViewUrl = "https://apps.apple.com/us/app/jikon-cam/id1315263599?uo=4";
            userRatingCount = 5;
            userRatingCountForCurrentVersion = 5;
            version = "1.0.1";
            wrapperType = software;
        }
    );
}
```

### 앱ID로 조회

이번에는 앱ID로 조회해 보자.

```
$ swift run appinfo appId 1315263599
{
    resultCount = 1;
    results =     (
                {
            advisories =             (
            );
            appletvScreenshotUrls =             (
            );
            artistId = 388959489;
            artistName = "SUNG CHEOL KIM";
            artistViewUrl = "https://apps.apple.com/us/developer/sung-cheol-kim/id388959489?uo=4";
            artworkUrl100 = "https://is2-ssl.mzstatic.com/image/thumb/Purple118/v4/62/67/81/626781d5-2ea6-8901-510c-b48852439bdc/source/100x100bb.jpg";
            artworkUrl512 = "https://is2-ssl.mzstatic.com/image/thumb/Purple118/v4/62/67/81/626781d5-2ea6-8901-510c-b48852439bdc/source/512x512bb.jpg";
            artworkUrl60 = "https://is2-ssl.mzstatic.com/image/thumb/Purple118/v4/62/67/81/626781d5-2ea6-8901-510c-b48852439bdc/source/60x60bb.jpg";
            averageUserRating = "4.2000000000000001776356839400250464677";
            averageUserRatingForCurrentVersion = "4.2000000000000001776356839400250464677";
            bundleId = "com.github.skyfe79.jikon";
            contentAdvisoryRating = "4+";
            currency = USD;
            currentVersionReleaseDate = "2017-12-05T22:21:09Z";
            description = "Jikon is an emotional film camera app that gives you the pleasure of developing and printing with your negative film.\nThe pictures you take are automatically developed as negative film, and you apply filters on negative film to print as real photos.\nWe choose only the filters that match film photos, giving you a total of 68 filters for pretty printing.\nWe recommend this to you!\n\n+ If you want to feel the joy of printing your pictures, use JiKon!\n+ If you want to take pictures during the day and printing the films during the night to organize it, use JiKon!\n+ If you want to print your photos with someone you love after taking pictures, use JiKon!\n+ If you just want to take a pretty picture without any annoying, use JiKon!\n+ If you want simple UI/UX, use JiKon!\n+ If you want to give toy camera app to your kids, use JiKon!\n\n\nJiKon shoots the pictures in 3:4 ratio. You can post the picture to the instagram directly.\nJikon is called 'now' in Japanese. Take a picture with JiKon now!\n\n+ Take JiKon when you travel!\n+ Take pretty landscape pictures with JiKon!\n+ Turn on FILM button, the film effect is applied to the picture!\n+ Turn on DATE button, the shooting date is stamped on the picture!\n+ Turn on GRAIN button, the grain ffect is applied to the picture!\n+ Turn on LEAK button, the lightleak ffect is applied to the picture!\n+ Swipe slider to control exposure!\n+ Swipe slider to control focus!\n+ Swipe slider to control zoom-in & out!\n+ Swipe Up/Down to flip camera position!\n+ Take pictures with volume keys!";
            features =             (
            );
            fileSizeBytes = 91347968;
            formattedPrice = Free;
            genreIds =             (
                6008,
                6012
            );
            genres =             (
                "Photo & Video",
                Lifestyle
            );
            ipadScreenshotUrls =             (
            );
            isGameCenterEnabled = 0;
            isVppDeviceBasedLicensingEnabled = 1;
            kind = software;
            languageCodesISO2A =             (
                EN,
                KO
            );
            minimumOsVersion = "9.0";
            price = 0;
            primaryGenreId = 6008;
            primaryGenreName = "Photo & Video";
            releaseDate = "2017-11-28T05:06:44Z";
            releaseNotes = "This app has been updated by Apple to use the latest Apple signing certificate.\n\n- Fixed minor bugs.\n- Adjusted the size and position of the date stamp.";
            screenshotUrls =             (
                "https://is4-ssl.mzstatic.com/image/thumb/Purple118/v4/7d/01/0a/7d010ae2-9110-de21-8106-6f0bf2faf72a/pr_source.png/392x696bb.png",
                "https://is5-ssl.mzstatic.com/image/thumb/Purple118/v4/fa/a3/82/faa3823f-a240-2aa7-c70c-4716f6946e57/pr_source.png/392x696bb.png",
                "https://is3-ssl.mzstatic.com/image/thumb/Purple118/v4/ef/da/a5/efdaa528-2b2b-c2cc-95a2-4b7b32631f40/pr_source.png/392x696bb.png",
                "https://is5-ssl.mzstatic.com/image/thumb/Purple118/v4/ad/27/c3/ad27c3b6-0963-3fd8-3eb0-93086a17d185/pr_source.png/392x696bb.png",
                "https://is1-ssl.mzstatic.com/image/thumb/Purple128/v4/bb/ff/98/bbff9875-e02a-89d6-9fea-735dc7c53c6b/pr_source.png/392x696bb.png"
            );
            sellerName = "SUNG CHEOL KIM";
            supportedDevices =             (
                "iPad2Wifi-iPad2Wifi",
                "iPad23G-iPad23G",
                "iPhone4S-iPhone4S",
                "iPadThirdGen-iPadThirdGen",
                "iPadThirdGen4G-iPadThirdGen4G",
                "iPhone5-iPhone5",
                "iPodTouchFifthGen-iPodTouchFifthGen",
                "iPadFourthGen-iPadFourthGen",
                "iPadFourthGen4G-iPadFourthGen4G",
                "iPadMini-iPadMini",
                "iPadMini4G-iPadMini4G",
                "iPhone5c-iPhone5c",
                "iPhone5s-iPhone5s",
                "iPadAir-iPadAir",
                "iPadAirCellular-iPadAirCellular",
                "iPadMiniRetina-iPadMiniRetina",
                "iPadMiniRetinaCellular-iPadMiniRetinaCellular",
                "iPhone6-iPhone6",
                "iPhone6Plus-iPhone6Plus",
                "iPadAir2-iPadAir2",
                "iPadAir2Cellular-iPadAir2Cellular",
                "iPadMini3-iPadMini3",
                "iPadMini3Cellular-iPadMini3Cellular",
                "iPodTouchSixthGen-iPodTouchSixthGen",
                "iPhone6s-iPhone6s",
                "iPhone6sPlus-iPhone6sPlus",
                "iPadMini4-iPadMini4",
                "iPadMini4Cellular-iPadMini4Cellular",
                "iPadPro-iPadPro",
                "iPadProCellular-iPadProCellular",
                "iPadPro97-iPadPro97",
                "iPadPro97Cellular-iPadPro97Cellular",
                "iPhoneSE-iPhoneSE",
                "iPhone7-iPhone7",
                "iPhone7Plus-iPhone7Plus",
                "iPad611-iPad611",
                "iPad612-iPad612",
                "iPad71-iPad71",
                "iPad72-iPad72",
                "iPad73-iPad73",
                "iPad74-iPad74",
                "iPhone8-iPhone8",
                "iPhone8Plus-iPhone8Plus",
                "iPhoneX-iPhoneX",
                "iPad75-iPad75",
                "iPad76-iPad76",
                "iPhoneXS-iPhoneXS",
                "iPhoneXSMax-iPhoneXSMax",
                "iPhoneXR-iPhoneXR",
                "iPad812-iPad812",
                "iPad834-iPad834",
                "iPad856-iPad856",
                "iPad878-iPad878",
                "iPadMini5-iPadMini5",
                "iPadMini5Cellular-iPadMini5Cellular",
                "iPadAir3-iPadAir3",
                "iPadAir3Cellular-iPadAir3Cellular",
                "iPodTouchSeventhGen-iPodTouchSeventhGen",
                "iPhone11-iPhone11",
                "iPhone11Pro-iPhone11Pro",
                "iPadSeventhGen-iPadSeventhGen",
                "iPadSeventhGenCellular-iPadSeventhGenCellular",
                "iPhone11ProMax-iPhone11ProMax",
                "iPhoneSESecondGen-iPhoneSESecondGen",
                "iPadProSecondGen-iPadProSecondGen",
                "iPadProSecondGenCellular-iPadProSecondGenCellular",
                "iPadProFourthGen-iPadProFourthGen",
                "iPadProFourthGenCellular-iPadProFourthGenCellular",
                "iPhone12Mini-iPhone12Mini",
                "iPhone12-iPhone12",
                "iPhone12Pro-iPhone12Pro",
                "iPhone12ProMax-iPhone12ProMax",
                "iPadAir4-iPadAir4",
                "iPadAir4Cellular-iPadAir4Cellular",
                "iPadEighthGen-iPadEighthGen",
                "iPadEighthGenCellular-iPadEighthGenCellular",
                "iPadProThirdGen-iPadProThirdGen",
                "iPadProThirdGenCellular-iPadProThirdGenCellular",
                "iPadProFifthGen-iPadProFifthGen",
                "iPadProFifthGenCellular-iPadProFifthGenCellular"
            );
            trackCensoredName = "JiKon Cam";
            trackContentRating = "4+";
            trackId = 1315263599;
            trackName = "JiKon Cam";
            trackViewUrl = "https://apps.apple.com/us/app/jikon-cam/id1315263599?uo=4";
            userRatingCount = 5;
            userRatingCountForCurrentVersion = 5;
            version = "1.0.1";
            wrapperType = software;
        }
    );
}
```

## 예제 코드

[Swift로 Command Line 앱 만들기 #1](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976285028-post-23/) 과 [Swift로 Command Line 앱 만들기 #2](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976291146-post-24/) 을 포함한 예제 코드는 
[https://github.com/my-swift-lab/learning-swift-cli](https://github.com/my-swift-lab/learning-swift-cli)에서 확인할 수 있다.