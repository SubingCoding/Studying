# Android App Test

앱 테스트는 앱 개발 과정에서 핵심적인 부분입니다. 앱 테스트를 일관되게 실행하여 앱을 공개적으로 출시하기 전에 앱의 정확성, 기능 동작 및 사용성을 확인할 수 있습니다.

테스트는 다음과 같은 이점도 제공합니다.

* 장애에 관한 신속한 피드백
* 개발 주기에서 조기 장애 감지
* 더 안전한 코드 리팩터링을 통해 회귀 걱정 없이 코드를 최적화할 수 있습니다.
* 기술적 문제를 최소화하는 안정적인 개발 속도



## 1. What to test in Android

### 1. 테스트 디렉터리 구성

Android 스튜디오의 일반적인 프로젝트에는 실행 환경에 따라 테스트를 보유하는 디렉터리가 두 개 포함되어 있습니다. 설명된 대로 다음 디렉터리에 테스트를 구성합니다.

- androidTest 디렉터리에는 실제 또는 가상 기기에서 실행되는 테스트가 포함되어야 합니다. 이러한 테스트에는 통합 테스트, 엔드 투 엔드 테스트, JVM만으로 앱 기능의 유효성을 검사할 수 없는 기타 테스트가 포함됩니다.
- test 디렉터리에는 로컬 시스템에서 실행되는 테스트(예: 단위 테스트)가 포함되어야 합니다. 위와 달리 로컬 JVM에서 실행되는 테스트일 수 있습니다.

### 2. 필수 단위 테스트

권장사항을 따를 때는 다음과 같은 경우에 단위 테스트를 사용해야 합니다.

- ViewModel 또는 프레젠터의 단위 테스트
- data 레이어, 특히 저장소의 단위 테스트 대부분의 데이터 레이어는 플랫폼과 독립적이어야 합니다. 이렇게 하면 테스트 더블이 테스트에서 데이터베이스 모듈과 원격 데이터 소스를 대체할 수 있습니다.
- 사용 사례 및 상호작용자와 마찬가지로 도메인 레이어와 같은 기타 플랫폼과 상관없는 레이어의 단위 테스트
- 문자열 조작 및 수학과 같은 유틸리티 클래스의 단위 테스트

### 3. 특이한 케이스 테스트

단위 테스트는 일반 사례와 특이 사례 모두에 집중해야 합니다.  
특이 사례는 인간 테스터와 대규모 테스트에서 포착할 가능성이 낮은 드문 시나리오입니다. 예를 들면 다음과 같습니다.

- 음수, 0, 경계 조건을 사용하는 수학 연산
- 발생할 수 있는 모든 네트워크 연결 오류입니다.
- 잘못된 형식의 JSON과 같은 손상된 데이터입니다.
- 파일에 저장할 때 전체 저장용량 시뮬레이션하기
- 프로세스 도중에 다시 생성된 객체 (예: 기기 회전 시의 활동)

## 2. Unit Test
### 1. Setup
1. location : /com/example/unittestexample(test)
2. gradle : 필수 추가(기본적으로는 이미 추가되어있음.)
```
testImplementation 'junit:junit:4.13.2'
androidTestImplementation 'androidx.test.ext:junit:1.1.5'
androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
```
### 2. Junit 메소드
- @Before : @Test 메소드가 시작하기 전에 항상 호출되는 메소드
- @After : @Test 메소드가 종료되면 호출되는 메소드, 주로 메모리 release
- @Test : @Before가 완료되면 실제 코드 테스트 진행
- @Rule : 해당 Test클래스에서 사용하게 될 ActivityTestRule과 ServiceTestRule 정의
- @BeforeClass, @AfterClass : public static method로 정의해야 하며, 전체 테스트 클래스에서 테스트 실행 시 시작할 때와 끝날 때 한 번 실행되도록 하기 위한 어노테이션
- @Test(Timeout = ) : @Test 에 대한 timeout을 지정하게 되어 timeout안에 테스트 완료될 수 있도록 합니다. 타임 초과시 실패
- @RequiresDevice : 에뮬레이터를 사용하지 않고 기기만 사용 가능
- @SdkSupress : minSdkVersion을 이용
- @SmallTest, @MediumTest, @LargeTest : 테스트 성격 구분하여 테스트

### 3. assertThat
assertThat은 Junit4.4부터 추가되었으며, 여러 matcher를 사용할 수 있어 기존 assertXXX 메서드보다 더 많은 유연성을 제공한다  
- 가독성
  expected와 actual의 위치가 명확히 보이기 때문에 가독성이 향상된다.
  ```
  assertEquals(expected, actual);
  assertThat(actual, is(expected));    // is() 사용
  ```
  또한, 같지 않을 경우를 나타내야 할 때도 가독성이 향상된다.
  ```
  assertFalse(expected.equals(actual));
  asserThat(actual, is(not(expected)));    // is(not()) 사용
  ```
- Failure 메시지
  expected 값과 actual 값 모두 에러 메시지에 반환된다.  
  원인을 찾기 위해 별도의 디버깅 필요없이 에러 메시지 만으로 잘못된 부분을 바로 파악할 수 있다.
- Type 안정성
  assertEquals는 Object를 인자로 받고 있기 때문에 Type에 대한 안정성을 보장하지 않는다.
  ```
  static public void assertEquals(Object expected, Object actual) {
    assertEquals(null, expected, actual);
  }

  assertEquals("abc", 123); // 컴파일에는 성공하지만, 실행시 실패한다.
  ```
  반면, assertThat은 위와같은 경우 Compile Error가 발생한다.
  Generic을 사용하고있어 Type을 확인하기 때문이다
  ```
  public static <T> void assertThat(T actual, Matcher<? super T> matcher) {
    assertThat("", actual, matcher);
  }
  ```
- 유연성
   anyof와 allof와 같은 matcher를 제공한다.
  ```
  assertThat("test", allOf(is("test2"), containsString("te")));
  ```
- Custom Matchers
  custom matchers를 생성할 수있다. TypeSafeMatcher<>를 상속받는다.
  // TODO: 예제 추가하기.  

## 3. Example
```
// 기본 생성 Unit Test Example
public class ExampleUnitTest {

    @Test
    public void addition_isCorrect() {
        assertEquals(4, 2 + 2);
    }

}
```

```
...

import static org.hamcrest.Matchers.*;
import static org.hamcrest.MatcherAssert.assertThat;

import org.junit.Test;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class VersionUnitTest {

    /**
     * firmware version을 Integer array list 형태로 반환한다.
     * @return index 0 : Major, 1 : Minor, 2 : Revision, 3 ~ n : etc
     *         변환 후 Major, Minor, Revision 각 index가 비어있을 경우 0을 채워서 반환한다.
     *         ex1) [] -> [0,0,0]
     *         ex2) [0,1] -> [0,1,0]
     */
    public static ArrayList<Integer> getFirmwareVersion(String version) {
        String[] versions = version.split("\\.");
        ArrayList<Integer> version_list = new ArrayList<>();
        int end_index;

        for (String ver : versions) {
            for (end_index = 0; end_index < ver.length(); end_index++) {
                if (ver.charAt(end_index) < '0' || ver.charAt(end_index) > '9') break;
            }

            if (end_index > 0) {
                version_list.add(Integer.parseInt(ver.substring(0, end_index)));
            }
        }

        if(version_list.size() < 3) {
            for (int i = version_list.size(); i < 3; i++){
                version_list.add(i, 0);
            }
        }

        return version_list;
    }

    private void testFirmwareVersion(String actualVersion, List<Integer> expectedVersion) {
        ArrayList<Integer> actualResult = getFirmwareVersion(actualVersion);
        assertThat(actualResult, is(equalTo(expectedVersion)));
    }

    @Test
    public void getFirmwareVersion_isCorrect() {
        testFirmwareVersion("0.0.1", Arrays.asList(0,0,1));
        testFirmwareVersion("0.1", Arrays.asList(0,1,0));
        testFirmwareVersion("0.0.1_test", Arrays.asList(0,0,1));
        testFirmwareVersion("", Arrays.asList(0,0,0));
        testFirmwareVersion("TestString", Arrays.asList(0,0,0));
    }
}
```
-----
### Reference
[[Android Developer] Test App on Android](https://developer.android.com/training/testing)  
[Unit Test에서 AssertThat을 사용하자](https://jongmin92.github.io/2020/03/31/Java/use-assertthat/)
