---
layout: post
title: Announcing new SDK versioning in Google Play services and Firebase
description: Google Play Service
categories: [Android]
tags: [Google Play Service]
comments: true
---

원문 : <https://android-developers.googleblog.com/2018/05/announcing-new-sdk-versioning.html?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+blogspot%2FhsDu+%28Android+Developers+Blog%29>

# Announcing new SDK versioning in Google Play services and Firebase
03 May 2018

Posted by Doug Stevensong, Developer Advocate

오늘부터, 구글 플레이 서비스와 파이어베이스를 위한 안드로이드 SDK는 새로운 빌드와 버전 스킴을 사용합니다. 이것은 여러분이 안드로이드 앱을 빌드하는데 조금 변경이 필요하며, 자세한 내용을 이해하기 위해 아래 내용을 자세히 읽어주세요.

이들 SDK의 새로운 내용을 요약 하면 
- 모든 의존성은 이제 시맨틱 버저닝을 사용합니다.
- 각각 의존성은 개별적으로 업데이트 되어, 앱에서 동시에 모두 업그레이드 해야되는 것을 제거 합니다.
- 각각 의존성은 새로운 기능과 버그 픽스를 위해 더 빠른 사이클을 가집니다.

버전 15부터 모든 플레이 서비스와 파이어베이스 라이브러리는 버전 번호는 [시멘틱 버저닝](https://semver.org/) 스킴으로 이뤄 집니다. 여러분도 알다시피 semver 는 소프트웨어 컴포넌트의 버전 관리를 위한 산업 표준 이며, 각각 라이브러리의 버전 변호 변경으로 그 라이브러리가 어느정도의 변경되었는지 예상할 수 있습니다.

이제 더이상 빌드시점이나 런터암 시점에 올바르게 동작하기 위해 `com.google.android.gms:play-services-*` 와 `com.google.firebase:firebase-*`의 메이븐 의존성에 동일한 버전을 맞출 필요가 없습니다. 이제 각각 의존성을 서로 개별적으로 업그레이드할 수 있습니다. 이와 같이, 그래들 빌드에서 공유된 버전을 플레이 서비스와 파이어 베이스에 적용하기 위해서 사용했던 일반 적인 패턴은 더이상 기대한 대로 동작하지 않습니다. 그 패턴 (이제는 안티 패턴)은 아래와 같습니다.
```
buildscript {
    ext {
        play_version = '15.0.0'
    }
}

dependencies {
    // DON'T DO THIS!!
    // The following use of the above buildscript property is no longer valid.
    implementation "com.google.android.gms:play-services-auth:${play_version}"
    implementation "com.google.firebase:firebase-auth:${play_version}"
    implementation "com.google.firebase:firebase-firestore:${play_version}"
}
```

위에서 그래들 환경 설정은 플레이 서비스와 파이어 베이스 SDK 버전을 위한 `play_version` 이라는 프로퍼티를 정의하고 의존성 정의를 위해 사용하고 있습니다. 이 패턴은 기존에 필요로 했던 의존성들의 버전을 한번에 유지하는데 유용 했습니다. 하지만 버전 15부터 이 패턴은 유효하지 않습니다. 각각의 의존성 버전은 이제 서로 다릅니다. 이제 각각 라이브러리 업데이트는 동일 시점에 배포되지 않습니다. 그것들은 이제 독립적으로 업데이트 됩니다.

이 버전 정책 변경을 지원하기 위해 플레이 서비스 그래들 플러그인이 업데이트 되었습니다. 만약 여러분이 이 플러그인을 사용한다면, 여러분 앱 모듈의 `build.gradle` 마지막 줄에 아래처럼 적혀있을 것입니다.
```
apply plugin: 'com.goole.gms.google-services'
```

이 플러그인에서 변경된 점은
- 플레이 서비스와 파이어 베이스 라이브러리의 호환성을 체크 합니다. `failOnVersionConflict()` [ResolutionStrategy](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html)를 활성화 하는 것과 유사합니다.
- 라이센스 정보는 각각 개별 빌드 결과물에 포함 됩니다. 만약 라이센스 요구사항 관리를 위해 [oss-license plugin](https://developers.google.com/android/guides/opensource#add_the_gradle_plugin)을 사용한다면 최신 버전으로 업데이트 해야 합니다.

새로운 버저닝 시스템이 동작하는 첫 플러그인의 버전은 3.3.0 입니다. 새로운 버전의 플레이 서비스와 파이어베이스 라이브러리를 사용하기 위해서 여러분의 빌드 스크립트 클래스패스 의존성에 아래처럼 추가해야 합니다.
```
classpath 'com.google.gms:google-services:3.3.0'
```
만약 플러그인을 사용하지 않고 사용하는 의존성들에 대한 엄격한 버전 체크를 원한다면, 대신 새로운 그래들 플러그인을 적용할 수 있습니다:
```
apply plugin: 'com.google.android.gms.strict-version-matcher-plugin'
```
이 플러그인을 사용하기 위해 빌드스크립트 클래스패스에 구글 메이븐 레파지토리의 아래 내용을 추가해야 합니다:
```
classpath 'com.google.android.gms:strict-version-matcher-plugin:1.0.0'
```

여러분이 앱을 개발하는데 안드로이드 스튜디오 3.1을 사용하지 않는다면, IDE에서 올바른 버전 체크 동작을 위해서 업그레이드 해야 합니다. 안드로이드 스튜디오 새 버전은 [여기](https://developer.android.com/studio/index.html)에서 얻을 수 있습니다.

이런 변화를 통해, 이제 여러분은 모든것을 한번에 업데이트 해야하는 제약사항 없이, 다양한 SDK의 새로운 버전을 자유롭게 적용할 수 있습니다. 개발팀이 더 빠르게 향상되고 버그가 수정된 SDK를 사용할 수 있게 합니다. 앞으로는 제공된 링크를 통해 [플레이 서비스 SDK](https://developers.google.com/android/guides/releases)와 [파이어베이스 SDK](https://firebase.google.com/support/release-notes/android)의 릴리즈를 추적할 수 있습니다. 

