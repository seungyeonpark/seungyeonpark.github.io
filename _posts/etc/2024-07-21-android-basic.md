---
title: "Android 앱 개발의 기본"
excerpt: ""

categories:
  - ETC
tags:
  - [Android]

permalink: /etc/android-basic

toc: true
toc_sticky: true

date: 2024-07-21
last_modified_at: 2024-07-21
---

# 프로젝트 기본 구조

```
app/
├── src/
│   ├── main/
│   │   ├── java/com/example/myapp/
│   │   │   ├── MainActivity.kt
│   │   │   ├── ui/
│   │   │   │   ├── theme/
│   │   │   │   │   ├── Color.kt
│   │   │   │   │   ├── Theme.kt
│   │   │   │   ├── Greeting.kt
│   │   ├── res/
│   │   │   ├── drawable/
│   │   │   ├── layout/
│   │   │   ├── values/
│   │   ├── AndroidManifest.xml
├── build.gradle
├── proguard-rules.pro
├── settings.gradle
```

- app: 프로젝트의 메인 애플리케이션 모듈
- src/main
  - java/ 또는 kotlin/: 애플리케이션의 소스 코드가 위치
    - MainActivity.kt
      - 앱의 주요 진입점
      - Compose 프로젝트에서는 setContent를 사용해 Composable 함수를 정의한다
      - @Composable
        - UI를 구성하는 기본 단위가 되는 함수
    - ui/
      - UI 관련 컴포넌트를 정의하는 파일들이 위치한다.
      - 각 화면이나 UI 구성 요소를 Composable 함수로 정의한다.
      - themes/
        - Compose 프로젝트에서는 테마와 스타일을 Kotlin 코드로 정의한다.
        - Color.kt: 앱에서 사용하는 색상을 정의한다.
        - Theme.kt: 앱의 테마를 정의하며, 다크 모드와 라이트 모드 등 다양한 스타일을 설정할 수 있다.
  - res/: 애플리케이션의 리소스 파일(레이아웃, 문자열, 이미지 등)이 위치
  - AndroidManifest.xml: 애플리케이션의 구성 정보를 포함하는 파일로, 액티비티/서비스/권한 등을 정의

<br>

# Activity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyApp()
        }
    }
}
```

- 안드로이드 애플리케이션의 주요 구성 요소 중 하나로 단일 화면을 나타낸다.
- 사용자 인터페이스(UI)를 제공하고, 사용자가 앱과 상호작용할 수 있도록 한다.
- 안드로이드 앱은 여러 개의 액티비티를 가질 수 있으며, 각 액티비티는 독립적인 화면을 구성한다.
- 위 코드에서 setContent 함수는 액티비티나 프래그먼트에서 호출되어 전달된 람다(Composable 함수)를 실행하여 화면에 UI를 표시한다.

<br>

# Fragment

```kotlin
class MainFragment : Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return ComposeView(requireContext()).apply {
            setContent {
                MainFragmentContent()
            }
        }
    }

    @Composable
    fun MainFragmentContent() {
        MaterialTheme {
            Greeting("Jetpack Compose in Fragment")
        }
    }

    @Composable
    fun Greeting(name: String) {
        Text(text = "Hello, $name!")
    }
}
```

- 프래그먼트는 액티비티 내에서 화면의 일부를 구성하며, 액티비티보다 더 유연하게 화면을 나눌 수 있다.

<br>

# 액티비티 생명주기
- onCreate(): 액티비티가 생성될 때 호출된다. 초기화 작업을 수행한다.
- onStart(): 액티비티가 사용자에게 보이기 시작할 때 호출된다.
- onResume(): 액티비티가 사용자와 상호작용하기 시작할 때 호출된다.
- onPause(): 다른 액티비티가 시작되면 호출된다. 일반적으로 데이터를 저장하거나 애니메이션을 중기하는 등의 작업을 수행한다.
- onStop(): 액티비티가 사용자에게 더 이상 보이지 않을 때 호출된다.
- onDestroy(): 액티비티가 완전히 종료될 때 호출된다.

<br>

# 프래그먼트 생명주기
- onAttach(): 프래그먼트가 액티비티에 첨부될 때 호출된다.
- onCreateView(): 프래그먼트의 UI를 생성할 때 호출된다.
- onViewCreated(): 프래그먼트의 UI가 생성된 후 호출된다.
- onStart(), onResume(), onPause(), onStop(), onDestroyView(), onDestroy(), onDetach()

<br>

# Composable
- Composable 함수는 Jetpack Compose에서 UI를 선언하기 위해 사용되는 함수이다.
- `@Composable` 애노테이션을 사용하여 정의되며, 다른 Composable 함수를 호출하거나 UI 컴포넌트를 구성하는 데 사용된다.
- 모든 Composable 함수는 주로 화면에 UI 요소를 그리기 위해 사용되며, `Unit`을 반환한다.

<br>

# Composable 함수와 상태 관리
- Jetpack Compose은 선언적 UI 프로그래밍을 지원한다. 이는 UI가 상태에 따라 자동으로 갱신되는 것을 의미한다.
- Composable 함수는 상태를 가질 수 있으며, 상태가 변경되면 Compose는 해당 상태를 사용하는 Composable 함수를 자동으로 다시 호출(recompose)하여 UI를 업데이트한다. 이를 통해 UI와 데이터 상태가 일관되게 유지되도록 한다.

```kotlin
@Composable
fun Counter() {
	var count by remeber { mutableStateOf(0) }

	Button(onClick = { count++ }) {
		Text("Clicked $count times")
	}
}
```

- 위 예제에서는 count 상태가 변경될 때마다 Compose는 Counter함수를 다시 호출하여 UI를 업데이트한다. 즉, 버튼을 클릭할 때마다 버튼에 표시된 클릭 횟수가 증가한다.
- remember 함수는 mutableStateOf(0)을 호출하여 초기 값이 0인 MutableState 객체를 생성한다.
- by 키워드는 count 프로퍼티를 MutableState 객체가 관리하도록 권한을 위임한다.
- Button 함수는 첫 번째 인자로 onClick = { count++ }, 두 번째 인자로 { Text("Clicked $count times") }를 전달받고 있다.
  - `()` 안의 파라미터는 Column 컴포저블 자체의 속성을 설정
  - `{}` 안의 파라미터는 Column 내에 포함될 자식 컴포저블들을 정의

<br>

# MVVM 패턴의 구성 요소
- Model: 데이터와 비즈니스 로직을 처리하는 부분이다. 데이터베이스나 네트워크 호출 등의 데이터 소스를 관리한다.
- View: 사용자 인터페이스(UI)를 담당한다. 사용자의 입력을 받아 ViewModel에 전달하고, ViewModel의 데이터를 관찰하여 UI를 업데이트한다.
- ViewModel: Model과 View 사이의 중간 역할을 한다. Model에서 데이터를 가져와서 가공하고, View에 제공하며, 사용자의 입력을 처리한다.

<br>

# MVVM 패턴의 동작 흐름
1. View는 사용자 입력을 받는다.
2. View는 사용자 입력을 ViewModel에 전달한다.
3. ViewModel은 필요한 데이터를 Model에서 가져오고, 이를 가공하여 View에 제공할 데이터를 준비한다.
4. View는 ViewModel을 관찰하여 데이터가 변경되면 UI를 업데이트한다.

<br>

# SharedPreference
- SharedPreference는 XML 파일 형식으로 데이터를 저장
- 앱이 종료되거나 디바이스가 재부팅되어도 데이터를 유지할 수 있도록 한다.
- SharedPreference 파일은 앱의 내부 저장소(Internal Storage)에 위치한 디렉토리에 저장된다.
- 일반적으로 /data/data/<your_app_package>/shared_prefs/ 경로에 XML 파일로 저장된다.