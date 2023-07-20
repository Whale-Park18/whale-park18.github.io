# 구문
> `[` `capture` `]` `(` `params` ` )` `[:specifiers]` `[:exception]` `[:trailing return type]` `{` `body` `}`

1. `capture` - 캡처
2. `params` - 매개 변수
3. `[:specifiers]` - 변경 가능한 사양
4. `[:exception]` - 예외 사양
5. `[:trailing return type]` - 후위 반환 형식
6. `body` - 본문

# 설명
lambda expression는 람다(lambda)라고 많이 불리며 함수 객체를 정의하는 편리한 방법입니다. 즉, 익명 함수를 뜻합니다. 람다 식은 일반적으로 알고리즘 또는 비동기 함수에 전달되는 몇 줄의 코드를 캡슐화하는데 사용됩니다.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>

using namespace std;

void absSort(vector<int>& origin)
{
    sort(origin.begin(), origin.end(),
    	// 람다 식을 이용한 커스텀 Compare
        [](int left, int right) {
            return abs(left) < abs(right);
        }
    );
}

int main()
{
    vector<int> values = { -5, 4, 3, -2, 1 };
    absSort(values);
    for (auto value : values)
    {
        cout << value << endl;
    }
}
```

## Capture
람다는 캡처를 이용해, 람다 외부에 정의되어 있는 변수나 상수를 람다 내부에서 사용할 수 있습니다. '매개 변수로 전달해 사용하면 되는 것 아닌가?' 의문이 들 수 있습니다.

외부에 있는 모든 변수와 상수에 대한 매개 변수를 정의할 수 없으며 STL을 사용할 경우, 제약이 발생할 수 있습니다. 때문에 이를 방지하기 위해 람다 내부와 소통할 수 있는 캡처 절을 제공하는 것입니다.

람대 외부에 정의된 변수나 상수를 캡처하는 방식으로 **call-by-vaule**, **call-by-reference** 두 가지가 존재합니다. 즉, 값으로 캡처하는가, 참조로 캡처하는가 차이입니다.

캡처 절에서는 다음과 같은 5가지 형태를 제공합니다(외부 변수 x, y, z가 존재).
1. `[=]` - 외부의 모든 변수나 상수들(x, y, z)을 값으로 가져온다.
2. `[&]` - 외부의 모든 변수나 상수들(x, y, z)을 참조로 가져온다.
3. `[=, &x]` - 외부의 모든 변수를나 상수들(y, z)을 값으로 가져오지만, x는 참조로 가져온다.
4. `[&, y]` - 외부의 모든 변수나 상수들(x, z)을 참조로 가져오지만, y는 값으로 가져온다.
5. `[&y, z]` - 선택한 변수나 상수(y z)를 지정한 방식(y: 참조, z: 값)에 따라 가져온다.

> 참조로 복사한 외부 변수는 대입 연산자를 통해 값을 변경할 수 있지만, 값으로 복사한 외부 변수는 대입 연산자를 통해 값을 변경할 수 없습니다.

## params
람다는 결국 함수이기 때문에 매개 변수가 존재할 수 있습니다. 일반적인 함수들 처럼 매개 변수의 존재 여부에 상관 없이 소괄호(`()`)를 사용해야 하지만 람다에서 매개 변수가 없을 경우, 소괄호를 생략할 수 있습니다.

```cpp
// 매개 변수 X
[]() { cout << "매개 변수 X" << endl; }();
[] { cout << "매개 변수 X" << endl; }();

// 매개 변수 O
[](int a, int b) { cout << a + b << endl; }(1, 3);
```
> 람다 식을은 인수로 전달하거나 `auto` 키워드로 함수 포인터를 지정해 사용하지 않아도 본문을 정의한 중괄호(`{}`) 뒤에 소괄호(`()`)에 매개 변수를 전달하면 람다 식이 실행됩니다. 

## Specifiers
일반적으로 람다 식의 호출 연산자는 const-by-value이지만 `mutable` 키워드를 사용하면 이를 취소합니다. 

## exception
`noxcept` 키워드를 사용하여 람다 식이 예외를 throw하지 않음을 나타낼 수 있습니다. 일반 함수와 마찬가지로 람다 식이 `noexcept` 예외 사양을 선언하고 람다 본문이 예외를 throw하는 경우 에러가 발생합니다.

```cpp
int main()
{
	[] () noexcept { throw 5; } (); // error!
}
```

## trailing return type
람다 식의 반환 형식은 자동으로 추론되기 때문에 후행 반환 형식(trailing return type)을 지정하지 않아도 됩니다. 후행 반환 형식을 지정하고 싶은 경우 중괄호(`{}`) 앞에 `->`(후행 반환 형식) 키워드를 작성해 지정합니다.

```cpp
[] { return 100; }();
[] () -> int { return 200; }();
```

## body
람다 식의 본문은 일반 함수 또는 멤버 함수의 본문에 허용되는 모든 항목을 포함할 수 있습니다. 일반 함수와 람다 식 모두의 본문은 다음고 같은 종류의 변수에 접근할 수있습니다.
* 캡처 절에 의해 캡처된 변수
* 매개 변수
* 로컬 변수
* 클래스 내에서 선언되고 캡처되는 경우 클래스 `this` 데이터 멤버
* 정적 스토리지 기간에 있는 모든 변수(예: 전역 변수)

# 참고 자료
[cppreference](https://en.cppreference.com/w/cpp/language/lambda)  
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170)  
[BlockDMask](https://blockdmask.tistory.com/491)  
[모두의 C++](https://modoocode.com/196)