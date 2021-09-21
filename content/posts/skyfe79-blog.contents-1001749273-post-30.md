---
title: "Github Actions Runner의 설치된 소프트웨어 패키지 확인하기."
date: 2021-09-21T03:09:49Z
draft: false
tags: ["tips","github","actions"]
---

Github Actions는 Workflow를 실행하는 가상 머신을 Runner라고 명칭한다. Runner에는 두 가지 종류가 있다.

- Github 이 호스팅하는 Github Hosted Runner
- 사용자가 개인의 머신을 사용할 수 있는 Self Hosted runner

## Github Hosted runner

가끔 Gihub Hosted Runner의 하드웨어 사양과 설치된 소프트웨어 패키지 정보가 필요할 때가 있다. 예를 들어, iOS 앱 빌드에 필요한 Xcode 가 앱스토어에 새로운 버전이 출시 되었을 때 GitHub Actions Runner가 해당 버전을 지원하는지 확인할 필요가 있다.


## GitHub Actions Runner 정보

Github 이 제공하는 Runner의 정보는 [https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources) 에서 확인할 수 있다.

### Hardware specification for Windows and Linux virtual machines:

- 2-core CPU
- 7 GB of RAM memory
- 14 GB of SSD disk space

### Hardware specification for macOS virtual machines:

- 3-core CPU
- 14 GB of RAM memory
- 14 GB of SSD disk space

지원하는 운영체제의 버전도 위 문서에서 확인할 수 있다.

### 설치된 소프트웨어 버전 확인하기

설치된 소프트웨어는 지원하는 OS 버전에 따라 다를 수 있다. 

- [Ubuntu 20.04 LTS](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu2004-README.md)
- [Ubuntu 18.04 LTS](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu1804-README.md)
- [Windows Server 2022](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2022-Readme.md)
- [Windows Server 2019](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2019-Readme.md)
- [Windows Server 2016](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2016-Readme.md)
- [macOS 11](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-11-Readme.md)
- [macOS 10.15](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-10.15-Readme.md)

위 정보는 모두 [https://github.com/actions/virtual-environments](https://github.com/actions/virtual-environments) 에서 관리하고 있다.

금일 Async/Await를 지원하는 Swift 5.5가 릴리즈 되었다. Swift 5.5를 사용하기 위해서 Xcode 13도 필요하여 Github Actions에서 Xcode 13을 지원하는지 확인해 보았다.

### [Xcode](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-11-Readme.md#xcode)
| Version          | Build    | Path                              |
| ---------------- | -------- | --------------------------------- |
| 13.0 (beta)      | 13A5212g | /Applications/Xcode_13.0_beta.app |
| 13.0             | 13A233   | /Applications/Xcode_13.0.app      |
| 12.5.1 (default) | 12E507   | /Applications/Xcode_12.5.1.app    |
| 12.5             | 12E262   | /Applications/Xcode_12.5.app      |
| 12.4             | 12D4e    | /Applications/Xcode_12.4.app      |
| 11.7             | 11E801a  | /Applications/Xcode_11.7.app      |

다행히도 지원하고 있다. [Installed SDK](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-11-Readme.md#installed-sdks)를 보면 iOS 15.0 도 설치되어 있다.

