---
title: React Native에서 네이버 지도 연동하기
description: 네이버 지도 연동이 이렇게 어려운거 였나요
date: 2024-09-30 21:07:00 +/-TTTT
categories: [React Native]
tags: [app, react native, naver map]
image:
  path: assets/img/writing/android_build_error_github.png
---

<!-- > 세 줄 요약
> 1. `@gorhom/bottom-sheet` iOS, Android에서 bottom-sheet의 UI가 동일하게 구성되지 않음
> 2. 높이가 동적으로 적용되어 있기 때문에 발생한 문제였음!
> 3. `enableDynamicSizing`을 `false`로 설정하기!
{: .prompt-tip } -->

## 1. 개요

React Native에서 네이버 지도를 렌더링하기 위해 [react-native-naver-map](https://github.com/mym0404/react-native-naver-map) 라이브러리를 적용하려고 하였는데 적용하는 과정에서 어려움을 느껴 이를 해결하고 나와 같은 어려움을 느끼는 사람들에게 공유하고자 게시물을 작성하게 되었다.

## 2. 적용하기

우선 현재 프로젝트에서 사용하고 있는 React Native의 버전은 `0.72.6`이고 Bridge-base Architecture를 적용하고 있으므로 `v1.5.9`를 설치하였다. 문제의 Android를 뒤에 하고 iOS 설정부터 해보자.

### 2-1. iOS

#### 1. Set Naver SDK key to `info.plist`

`info.plist`로 이동한 다음 NMFClientId 키에 네이버 클라우드에서 발급받은 클라이언트 아이디를 추가한다.

```md
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>NMFClientId</key>
  <string>YOUR_CLIENT_ID_HERE</string>
<dict>
<plist>
```

#### 2. (Optional) Set location permission usage description to `info.plist`

단순 로컬에서 테스트하기 위한 목적이라면 `info.plist`에 권한 요청에 대한 설명을 작성해 줄 필요가 없지만, 앱 스토어의 심사를 받고 정식 출시를 해야 한다면 해당 권한을 사용하는 목적을 반드시 기재해야 한다.

```md
<plist version="1.0">
<dict>
  <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
  <string>{{usage description}}</string>
  <key>NSLocationTemporaryUsageDescriptionDictionary</key>
  <dict>
    <key>{{your purpose key}}</key>
    <string>{{usage description}}</string>
  </dict>
  <key>NSLocationWhenInUseUsageDescription</key>
  <string>{{usage description}}</string>
</dict>
</plist>
```

![iOS Naver Map](assets/img/writing/ios_naver_map.png){: width="240" }
_iOS Naver Map_

이렇게 2번의 단계만 거치면 iOS는 네이버 지도가 화면에 잘 표시된다. 그럼, 이제 문제의 Android 알아보자!

### 2-2. Android

Android도 iOS와 마찬가지로 간단한 설정만 거치면 된다.

#### 1. Maven repository import

```gradle
allprojects {
    repositories {
        maven {
            url "https://repository.map.naver.com/archive/maven"
        }
    }
}
```

우선 루트 경로에 있는 `android/build.gradle`에 위와 같이 설정을 해준다. 여기서 주의해야 할 점이 두 가지가 있다. 간혹 네이버 지도 저장소를 `https://naver.jfrog.io/artifactory/maven/`로 설정하는 경우가 있는데 이는 이전 버전의 저장소이기 때문에 해당 저장소를 가져오게 되면 네이버 지도를 정상적으로 가져오지 못하므로 위 코드에 나오는 경로를 기입해 주자.

그리고 위에 나온 이미지의 네이버 지도 저장소 저장소는 `buildscript`에 추가하는 게 아니라 `buildscript`와 분리하여 생성해야 한다!

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext {
        buildToolsVersion = "34.0.0"
        minSdkVersion = 23
        compileSdkVersion = 34
        targetSdkVersion = 34

        // We use NDK 23 which has both M1 support and is the side-by-side NDK version from AGP.
        ndkVersion = "23.1.7779620"
    }
    repositories {
      // ...
    }
    dependencies {
      // ...
    }
}

allprojects {
    repositories {
        maven {
            url "https://repository.map.naver.com/archive/maven"
        }
    }
}
```

#### 2. Add Naver SDK key to AndroidManifest.xml

```xml
<manifest>
    <application>
        <meta-data
            android:name="com.naver.maps.map.CLIENT_ID"
            android:value="YOUR_CLIENT_ID_HERE" />
    </application>
</manifest>
```

앞서 iOS 1단계에서 가져온 클라이언트 아이디를 위 `android:value`에 기입해 주면 된다.

#### 3. (Optional) Request location permission to AndroidManifest.xml

그리고 마지막 단계로 현재 위치를 사용자의 현재 위치를 표시하기 위해 위치 권한을 요청하는 설정을 추가해 주면 된다.

```xml
<manifest>
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  # optional for background location
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
</manifest>
```

![Android Naver Map](assets/img/writing/android_naver_map.png){: width="240" }
_Android Naver Map_

Android도 생각보다 되게 간단하다!

## 3. 에러 로그

### 3-1. Could not find com.naver.maps:map-sdk:3.18.0.

해당 에러는 com.naver.maps.map-sdk를 찾을 수 없어서 발생한 이슈로 `android/build.gradle`을 다음과 같이 구성하여서 발생한 이슈이다.

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext {
        buildToolsVersion = "34.0.0"
        minSdkVersion = 23
        compileSdkVersion = 34
        targetSdkVersion = 34
        ndkVersion = "23.1.7779620"
    }
    repositories {
      // ...
      maven { url 'https://repository.map.naver.com/archive/maven' } // 🚨 Could not find com.naver.maps:map-sdk:3.18.0
    }
    dependencies {
      // ...
    }
}
```

해당 이슈를 해결하기 위해서는 `android/app/build.gradle`에 SDK에 대한 의존성을 추가하는 것이 아닌 **buildscript에 작성한 `maven { url 'https://repository.map.naver.com/archive/maven' }`을 다음과 같이 분리**해야 한다.

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext {
        buildToolsVersion = "34.0.0"
        minSdkVersion = 23
        compileSdkVersion = 34
        targetSdkVersion = 34

        // We use NDK 23 which has both M1 support and is the side-by-side NDK version from AGP.
        ndkVersion = "23.1.7779620"
    }
    repositories {
      // ...
    }
    dependencies {
      // ...
    }
}

// ✅ 네이버 지도 SDK 저장소 분리
allprojects {
    repositories {
        maven {
            url "https://repository.map.naver.com/archive/maven"
        }
    }
}
```

해당 이슈를 해결하고 나와 동일한 문제를 겪고 있는 사람을 [이슈](https://github.com/mym0404/react-native-naver-map/issues/104)에서 마주해서 댓글로 도움을 드렸다!

![Android Build Error](assets/img/writing/android_build_error_github.png)

뿌듯 😄

### 3-2. Android Build Error - Execution failed for task ':app:mergeExtDexDebug'.

Android로 빌드하는 도중 `Execution failed for task ':app:mergeExtDexDebug'.` 에러가 발생한다면, 개발 빌드가 완료되었을 때 발생하는 에러로 `minSdkVersion`을 업그레이드하여 해결할 수 있다.

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext {
        buildToolsVersion = "34.0.0"
        minSdkVersion = 24 // ✅ 23 -> 24 업그레이드
        compileSdkVersion = 34
        targetSdkVersion = 34 

        // We use NDK 23 which has both M1 support and is the side-by-side NDK version from AGP.
        ndkVersion = "23.1.7779620"
    }
    repositories {
      // ...
    }
    dependencies {
      // ...
    }
}

allprojects {
    repositories {
        maven {
            url "https://repository.map.naver.com/archive/maven"
        }
    }
}
```

## 참고

- [react-native-naver-map](https://github.com/mym0404/react-native-naver-map)
- [Execution failed for task ':app:mergeExtDexDebug'](https://github.com/invertase/react-native-firebase/discussions/7489)
- [Map SDK 라이브러리 저장소 변경 안내](https://www.ncloud-forums.com/topic/284/)