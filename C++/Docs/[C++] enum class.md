# 구문

> scoped enum:
> `enum` `[class|struct]` `[variable-name]` `[:type]` `{` `[enum-list]` `}` `;`

* `[class|struct]` - 열거형의 범위(scope)를 지정
* `[variable-name]` - 열거형에 지정된 형식 이름
* `[:type]` (옵션, defalut: `int`) - 열거형의 기본 타입, 모든 열거자는 동일한 기본 형식을 갖고 모든 정수 계열 형식이 가능하다.
* `enum-list` - 열거형의 목록

# 설명
C++11 이전의 `enum`은 다음과 같은 3가지 문제점을 보여줍니다.

1. `enum`은 암시적으로 `int`로 형변환이 되어, 정수로써 작동되기 원하지 않을 때, 오류를 발생시킨다.
2. 허용 범위(scope) 밖에서도 열거자 이름들을 사용할 수 있기 때문에 이름 충돌이 발생한다.
3. 전진 선언(forward declaration)이 불가능하다.
    
`enum class`는 이러한 문제점을 보완하기 위해 추가된 기능입니다. 전통적인 `enum`의 특징(값을 가진 이름들)과 클래스의 특징(범위를 가진 멤버, 암시적 변환의 부재)을 함쳐 기존의 문제를 해결한 키워드이며 다음과 같은 특징을 지닙니다.

* `enum class`를 `int`로 형변환하고 싶다면 <u>**명시적**</u>으로 형변환을 해야 한다.
* 범위가 존재하며, 범위 밖의 열거자들의 이름 중복이 가능하다.
* 전진 선언이 가능하다.

```cpp
// classic enum
// 이름 충돌 발생
enum AlertWindows { Green, Yellow, Election, Red };
//enum AlertLinux {  Green, Yellow, Election, Red };

// C++11 enum
enum class Color { Red, Blue, Green };
enum class TrafficLight : short { Red, Yellow, Green };
enum class HexColorCode : int;

int main()
{
	AlertWindows alertWindow = 1;				// 오류: 모든 C++ 버전에서 불가능

	int data1 = Red;							// 암시적으로 int로 형변환 가능
	int data2 = AlertWindows::Red;				// 오류: C++11 이전 버전에서는 불가능
	int data3 = Blue;							// 오류: 사용할 수 있는 범위를 밖이기 때문에 사용할 수 없음
	int data4 = Color::Blue;					// 오류: enum class는 암시적으로 형변환 할 수 없음
	int data5 = (int)Color::Blue;				// 명시적으로 int로 형변환 가능
	int redHexCode = (int)HexColorCode::Red;	// C++11 부터 전방 선언 가능
}

enum class HexColorCode : int { Red, Blue, Green };
```

# 참고 자료
[cppreference](https://en.cppreference.com/w/cpp/language/enum)  
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/enumerations-cpp?view=msvc-170)  
[BlockDMask](https://blockdmask.tistory.com/405)  
[소년코딩](https://boycoding.tistory.com/179)  