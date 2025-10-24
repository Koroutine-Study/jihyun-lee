# Ch.2 코루틴 개발 환경 설정

> ### 2장에서 다루는 내용
> 
> - 첫 코루틴 실행해보기
> - 코루틴 디버깅을 위해 JVM 옵션 설정하기
> - CoroutineName 사용해 코루틴에 이름 설정하기


- 코틀린은 언어 수준에서 코루틴을 지원하지만 저수준 API만을 제공 → 실제 애플리케이션에 사용하는 데는 한계
- 보통 젯브레인스에서 만든 코루틴 라이브러리(kotlinx.coroutines) 사용
    - async, await 같은 고수준 API 제공

## 1. 코루틴 라이브러리 추가하기

- build.gradle.kts 파일의 dependencies 블록에 `org.jetbrains.kotlinx:kotlinx-coroutines-core` 의존성 추가
- jvmToolchain이 사용하는 jdkVersion을 17로 변경
    
    ```kotlin
    kotlin {
    		jvmToolchain(17) // jvmToolchain의 jdkVersion 17로 변경
    }
    ```
    
- 테스트 라이브러리에 대한 의존성은 제거

## 2. 첫 코루틴 실행하기

```kotlin
fun main() = runBlocking {  // this: CoroutineScope
    println("Hello Coroutines")
}
```

- `runBlocking` 함수: 해당 함수를 호출한 스레드를 사용해 실행되는 코루틴을 만들어 냄
    - main 함수에서 실행 → `runBlocking` 함수는 메인 스레드를 점유하는 코루틴을 만듦
    - `runBlocking` 인자로 들어온 람다식을 실행
    - 람다식 내부의 모든 코드가 실행 완료될 때까지 코루틴은 종료되지 않음
    - `runBlocking` 함수로 생성된 코루틴이 실행 완료될 때까지 이 코루틴과 관련 없는 다른 작업이 스레드를 점유하지 못하게 막음

## 3. 코루틴 디버깅 환경 설정하기

### 3.1 실행 중인 스레드 출력하기

- 코루틴은 일시 중단이 가능하지만 일시 중단 후 작업 재개 시 실행 스레드가 바뀔 수 있으므로 어떤 코루틴이 어떤 스레드에서 실행되고 있는지를 알아야 디버깅 가능
- `Thread.currentThread().name` 은 현재 실행 중인 스레드 출력
    
    ```kotlin
    fun main() = runBlocking<Unit> {  // this: CoroutineScope
        println("[${Thread.currentThread().name}] 실행")
    }
    ```
    

### 3.2 실행 중인 코루틴 이름 출력하기

- JVM의 VM options에 옵션을 추가하면 됨
1. Edit Configurations 클릭
2. Run/Debug Configurations 창의 VM options에 `-Dkotlinx.coroutines.debug` 입력

### 3.3 launch 사용해 코루틴 추가로 실행하기

- `runBlocking` 함수의 람다식에서는 수신 객체인 CoroutineScope에 접근할 수 있음
- CoroutineScope 객체의 확장 함수로 정의된 launch 함수를 사용하면 코루틴을 추가로 생성할 수 있음

```kotlin
fun main() = runBlocking<Unit> {  // this: CoroutineScope
    println("[${Thread.currentThread().name}] 실행")
    launch {
        println("[${Thread.currentThread().name}] 실행")
    }
    launch {
        println("[${Thread.currentThread().name}] 실행")
    }
}
```

- 총 3개의 코루틴 생성
    - `runBlocking` 함수를 통해 하나의 코루틴 생성
    - `launch` 가 두 번 호출돼 각각 하나의 코루틴 생성

### 3.4 CoroutineName 사용해 코루틴에 이름 추가하기

- CoroutineName 객체는 코루틴의 이름을 구분하는 객체
- CoroutineName 객체 생성 함수에 코루틴에 설정될 이름을 인자로 넘김으로서 객체 생성
    - ex. `CoroutineName("Main")`

```kotlin
fun main() = runBlocking<Unit>(context = CoroutineName("Main")) {  // this: CoroutineScope
    println("[${Thread.currentThread().name}] 실행")
    launch(context = CoroutineName("Coroutine1")) {
        println("[${Thread.currentThread().name}] 실행")
    }
    launch(context = CoroutineName("Coroutine2")) {
        println("[${Thread.currentThread().name}] 실행")
    }
}
```

## 4. 요약

1. 코틀린은 언어 레벨에서 코루틴을 지원하지만 저수준 API 만을 지원한다.
2. 실제 개발에 필요한 고수준 API는 코루틴 라이브러리를 통해 제공받을 수 있다.
3. runBlocking 함수나 launch 함수를 사용해 코루틴을 실행할 수 있다.
4. 현재 실행 중인 스레드의 이름은 `Thread.currentThread().name` 을 통해 출력할 수 있다.
5. JVM의 VM optiosn에 `-Dkotllinx.coroutines.debug` 를 추가하면 스레드의 이름을 출력할 때 코루틴의 이름을 추가로 출력할 수 있다.
6. CoroutineName 객체를 통해 코루틴의 이름을 지정할 수 있다.