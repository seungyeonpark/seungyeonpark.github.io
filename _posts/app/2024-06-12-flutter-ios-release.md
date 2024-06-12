---
title: "Flutter iOS 앱스토어 배포 절차"
excerpt: "Flutter iOS 앱스토어 배포 절차에 대해 알아보자"

categories:
  - App
tags:
  - [app]

permalink: /app/flutter-ios-release

toc: true
toc_sticky: true

date: 2024-06-12
last_modified_at: 2024-06-12
---

1. 플러터 프로젝트 내 ios 프로젝트 xcode로 오픈
2. Runner > Targets > Runner > General에서 아래 두 항목 점검 
   - Display Name: 디스플레이 이름
   - Bundle Identifier: 다른 앱과 구분 짓는 유니크한 앱 아이디
3. Runner > Targets > Runner > Signing&Capabilities
   - Automatically manage signing이 활성화되어 있지 않다면 클릭해서 활성화 
   - Team에 Apple Developer에 가입된 팀 선택
4. Bundle ID 등록 (Apple Developer 사이트 혹은 Xcode에서 등록할 수 있다. 여기서는 Xcode로 추가하는 방법 사용)
   - target > runner > signing&capabilities
   - 하단 왼쪽 위 +Capability 클릭
   - In-App Purchase 검색 후 항목 찾아서 더블 클릭하면 signing&capabilities 하단 항목으로 추가됨
     - 이렇게 추가하는 과정에서 Xcode가 자동으로 Apple Developer 사이트에 Bundle ID를 자동으로 등록한다
     - https://developer.apple.com.account/resources/identifiers/list에서 XC로 시작하는 Bundle ID가 등록된 걸 확인할 수 있다
5. App Store Connect에서 어플리케이션 생성하기
   - Add Apps 클릭하면 New App 정보 입력란이 뜬다
     - Name: 사용자가 앱스토어에서 보는 앱 이름
     - SKU: 앱스토어 이외 다른 곳에서 쓰이는 아이디(Bundle ID 뒤에 .ios를 붙여서 생성하기도 한다)
     - User Access: Full Access(특정 대상이 아닌 모든 사람에게 앱을 배포)
6. 앱 아이콘 이미지 준비
   - 앱 아이콘을 만들기 위해 사용할 이미지의 사이즈는 적어도 192*192 px, 배경이 없는 PNG 파일 권장
   - 프로젝트 root 경로에 assets > icon 폴더 생성 후 이미지 저장 (원하는 경로에 위치 시켜도 상관 없음)
7. flutter_launcher_icons
   - pubspec.yaml 파일에서 아래 코드 붙여넣기 (dependencies, dev_dependencies 하위 같은 계층에 위치)
     ```yaml
     flutter_launcher_icons:
         android: "launcher_icon"
         ios: true
         image_path: "assets/icons/logo.png"
         min_sdk_android: 21
         web:
             generate: true
             iamge_path: "assets/icons/logo.png"
             background_color: "#hexcode"
             theme_color: "#hexcode"
         windows:
             generate: true
             image_path: "assets/icons/logo.png"
             icon_size: 48
         macos:
             generate: true
             image_path: "assets/icons/logo.png"
     ```
   - 터미널에서 `flutter pub get`으로 flutter_launcher_icons를 include
   - 터미널에서 `flutter pub run flutter_launcher_icons` 실행해서 아이콘 생성
   - 잘 생성 됐는지 확인하려면 Xcode에서 Runner > Runner > Assets에서 AppIcon을 선택하면 확인할 수 있다
8. Launch Screen 설정
   - Xcode에서 Runner > Runner > LaunchScreen로 들어가면 스토리보드 파일 확인 가능
   - Runner > Runner > Assets에서 LaunchImage 등록
     - LaunchImage 화면에서 사이즈 1부터 3까지 Launch Screen에 들어갈 이미지를 넣어준다
     - background color는 오른쪽 View 탭에서 변경 가능
9. 버전 확인하기
   - Runner > Targets > Runner > Info
     - Bundle version: 앱을 스토어에 등록 혹은 업데이트 할 때마다 기존 번들 버전보다 큰 숫자로 지정해야 업데이트 가능
     - Bundle version string (short): 기록을 위한 버전으로 유저가 볼 수 있다
10. API 및 아카이브 파일 빌드
    - 터미널에서 프로젝트 루트 경로 위치 후 `flutter build ipa` 실행
    - build > ios > archive/Runner.xcarchive와 build > ios > ipa > *.ipa 두 파일이 확인되면 빌드가 잘 된 것
11. 앱 유효성 검사 및 업로드
    - build > ios > archive/Runner.xcarchive를 Xcode에서 open
    - 오른쪽에 Validate App 클릭한 뒤 유효성 검사 후 done
    - 다시 오른쪽에 Distribute App 클릭한 뒤 App Store Connect > Upload 이후 설정값 그대로 진행한 수 done
    - 업로드 결과 확인
      - App Store Connect에서 My Apps > 해당 앱 클릭
      - Build 섹션에서 앱이 보여야 함 (보이지 않는다면 30분 후에 다시 확인)
12. App Store Connect에서 앱 정보 작성
    - App Store Connect에서 My Apps > 해당 앱 > General > App Information
      - Name: 사용자가 앱스토어에서 보는 이름
      - Subtitle: 사용자가 보는 설명란
      - General Information에서 앱 정보 마저 입력한 후 우측 상단의 save 클릭
    - App Store Connect에서 My Apps > 해당 앱 > General > App Privacy
      - get started 클릭 후 정보 정책 입력
      - Contact Info에서 앱을 개발한 개발팀 정보 입력