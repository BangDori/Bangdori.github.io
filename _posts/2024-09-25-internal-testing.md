---
title: Internal Testing으로 팀원들과 앱 공유하기
description: 현재 개발 중인 앱을 내 휴대폰에서 확인할 수 있다고?
date: 2024-09-25 15:53:00 +/-TTTT
categories: [Frontend, React Native]
tags: [app, react native, deployment, internal testing]
image:
  path: assets/img/thumbnail/internal_testing.png
sitemap: 
    changefreq : daily
    priority : 0.8
---

## 1. 개요

팀원들과 그리고 사용자들에게 서비스 개발을 빠르게 전달해 줄 수 있었던 이전 프로젝트와는 달리 React Native 기반의 프로젝트에서는 가상 기기로 프로젝트를 진행되고 있었기에 팀원들에게 서비스의 가치를 시각적으로 전달해 주기 어려웠다. 그렇다 보니 창업 초기 단계에 있는 우리 팀원들이 앱 개발의 진척도를 가시적으로 보여주기 어렵다는 문제점이 발생했다.

이러한 문제점을 해소하고 팀원들에게 서비스의 가치를 전달해 주며 팀의 사기를 돋우기 위해 Internal Testing을 적용하기로 하였다.

## 2. Internal Testing

Internal Testing이란 내부 테스트로 팀 내에서 내부적으로 팀원들과 서비스를 공유할 수 있는 기능을 제공한다.

> 참고 링크
> - [Google Play Console Internal Testing](https://play.google.com/console/about/internal-testing/)
> - [App Store TestFlight](https://developer.apple.com/testflight/)
{: .prompt-tip }

내부 테스트는 각각 Android, iOS 개발자 계정만 있다면 심사 없이 간편하게 등록할 수 있으며 팀원들의 이메일을 내부 테스트 앱에 등록만 해주면 테스트 앱을 설치하고 직접 확인할 수 있게 된다.

### 2-1. Internal Testing 언제 하는 게 가장 좋을까?

현재 React Native 기반의 앱 개발이 처음인 나는 앱 생태계를 전혀 모르는 상태로 진행하였기 때문에 사용자 인증 시스템을 모두 구축하고 난 이후에 Internal Testing을 도입했다. 하지만 개인적으로 생각하기에는 Internal Testing은 프로젝트의 시작과 함께 설정을 해두고 팀원들과의 조율하여 내부 테스트 배포 시점을 설정하는 게 좋을 것 같다. 내부 테스트를 등록하고 난 이후 팀원들이 조기에 발견해 준 에러 로그 목록은 다음과 같다.

> - 안드로이드 디바이스 내부에서는 중첩 `ScrollView`시 하위 `ScrollView`는 동작하지 않음
> - `Pressable` 영역이 좁아서 클릭이 잘되지 않는 문제
> - 현재 App의 배경 색상이 Figma에 나온 색상과 다른 문제
> - Progress Bar와의 간격이 Prod에 적용되지 않는 문제
{: .prompt-warning }

위 외에도 수많은 에러가 발생하였지만, 이런 에러를 조기에 발견하고 해결할 수 있었던 것은 바로 내부 테스트의 힘이였다고 생각한다. 배경 색상은 너무 사소하지 않냐고 생각할 수 있겠지만 팀 내 프론트엔드 개발자가 혼자라서 꼼꼼하게 확인하더라도 사소한 부분을 놓치는 경우가 가끔 있었다.

내부 테스트 중요성에 관해서는 이야기를 나눴다. 하지만 내부 테스트를 진행하기 위한 과정은 너무나 복잡하다. 우선 안드로이드와 iOS의 배포 과정을 간략하게 알아보자.

### 2-2. Android Deploy Process

![Android Deploy Process](assets/img/writing/1/android_deploy_process.png){: width="640" }
_Android Deploy Process_

1. Create KeyStore and Setup Release signing
2. Generating the release AAB
3. Upload AAB to Play Console
4. Download App on Google Play

안드로이드 배포 프로세스는 위와 같다. 1번과 4번 과정은 최초 1회만 수행하면 된다고 생각하더라도 2번과 3번은 매번 수동으로 진행해 줘야 한다. 그렇다면 iOS는 어떨까?

### 2-3. iOS Deploy Process

![iOS Deploy Process](assets/img/writing/1/ios_deploy_process.png){: width="640" }
_iOS Deploy Process_

1. Connect AppStore and Signing
2. Generate an Application Archive File
3. Upload TestFlight
4. Download App on TestFlight

현재 생략된 부분이 있을 수도 있지만 iOS의 배포 프로세스도 안드로이드와 마찬가지로 서명, 스토어 연결과 같은 단계는 한 번만 진행해 주면 되겠지만, 배포용 파일을 생성하고 스토어 올린 후 확인해야 하는 이러한 단계는 배포할 때마다 **수동으로 진행**해줘야 한다.

근데 이 지루한 과정을 매번 수동으로 하는 건 시간과 리소스가 부족한 우리 팀에게 매우 비효율적인 과정이다. 그렇다면 이러한 비효율적인 과정을 자동화해 주는 도구에는 어떤 게 있을까?

## 3. 자동화 도구를 찾아서..

우선 내가 찾아본 결과 Internal Testing 과정을 자동화해 주기 위한 도구 3가지 정도를 추출해 보았다.

> 자동화 도구
> - Github Actions
> - Fastlane
> - Bitrise
{: .prompt-tip }

Github Actions는 웹 프로젝트를 진행할 때 CI/CD 파이프라인을 구현하면서 어느 정도 익숙했지만 Fastlane과 Bitrise는 낯선 도구였기에 다음과 같이 비교분석해보았다.

|  Tools  |  주요 특징  |  장점  |  단점  |
| :--: | :--: | :--: | :--: |
| **GitHub Actions** | Github 통합 CI/CD | 워크플로, 무료 시간 제공, Github 완벽 통합 | 코드 서명 작업 수동 설정 |
| **Fastlane** | 배포 자동화 도구, 오픈 소스 | 코드 서명, 릴리스 등 다양한 서비스 자동화, 무료 | 복잡한 초기 설정, CI/CD 툴과 함께 사용 필요 |
| **Bitrise** | 모바일 특화 CI/CD 플랫폼 | 간편한 GUI 기반 설정, 코드 서명 자동화 | 무료 사용 시간제한, 추가 비용 발생 |

우선 나는 결론부터 말하자면 Github Actions + Fastlane을 선택했다. 2가지의 이유를 설명해 보자면 

1. Bitrise의 장점은 매우 유혹적이지만 우리 팀에게 추가 비용 발생은 치명적이다. => **Fastlane**
2. 우선 Fastlane은 배포 자동화 도구로 CI/CD의 모든 과정을 담당하지는 못한다.
  - 지속적 통합을 Github Actions가 지속적 배포를 Fastlane이 담당하면 되지 않을까?
  - 추후 Github에 완벽 통합하여 release 브랜치에 merge 되었을 때 자동 배포가 적용되도록 한다면 Github에서 완벽하게 제어할 수 있지 않을까? => 🎉 **Github Actions + Fastlane**

## 참고

- [Publishing to Google Play Store](https://reactnative.dev/docs/signed-apk-android)
- [TestFlight Made Simple: How To Upload & Manage An App On TestFlight! 🚀](https://www.youtube.com/watch?v=3aautA1kclE&t=567s)
- [Bitrise - iOS를 위한 CI/CD 사용해보기 (with CocoaPods, Carthage cache)](https://void0306.tistory.com/3)