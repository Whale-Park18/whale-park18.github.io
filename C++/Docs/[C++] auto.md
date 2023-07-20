# 구문
> `auto` `variable-name` = `initializer` `;`
> `auto` `variable-name` = `lambda expression|Function Pointer` `;`

* `[variable-name]` - 식별자
* `initializer` - 초기화 할 값
* `expression` - 람다식

# 설명
`auto` 키워드는 C++11 이전에는 다른 의미로 사용됐지만, 이후에는 컴파일러가 선언된 변수 또는 람다 식 매개 변수의 초기화 식을 사용하여 형식을 추록하도록 지시하는 키워드입니다.

즉, Java, C#의 'var' 키워드 처럼 초기화 값에 따라 타입을 정해주는 <u>**타입 추론**</u> 키워드입니다. `char`, `int` 등의 기본 타입형과 `struct`, `class` 등의 타입까지 추론 가능합니다. 또한 함수 포인터나 <u>함수 자체(람다 함수)</u>도 될 수 있습니다.

다음과 같은 이점을 제공하기 때문에 Visual C++에서는 대부분의 상황에서 `auto` 키워드를 사용하는 것을 권장하고 있습니다.
* 경고성 - 함수의 반환 형식이 변경되는 경우를 포함하여 식의 형식이 변경되어도 작동한다.
* 성능 - 변환이 없음을 보장한다.
* 유용성 - 형식 이름 맞춤법 오류 및 오타에 대한 걱정이 없다.
* 효율성 - 코딩이 더 효율적이다.


하지만 `auto` 키워드가 어느 곳에서나 사용되는 만능인 것은 아닙니다. 다음과 같은 제약사항이 존재합니다.
* `auto` 키워드는 다른 형식 지정자(`int` 등)와 결합할 수 없다.
* `auto` 키워드가 선언된 곳에는 이니셜라이저가 있어야 한다.
* 함수의 매개변수로 사용될 수 없다.
* 초기화 하기 전에 사용할 수 없다.
* `auto` 키워드 선언된 형식으로 캐스팅할 수 없다.
* `auto` 키워드에 `sizeof` 및 `typeid` 연산자를 사용할 수 없다.
* 구조체나 클래스 등의 멤버 변수로 사용할 수 없다.

> Visual Studio 기준으로 `auto`로 선언한 변수명 위에 마우스를 가져다 대면 추론된 타입을 확인할 수 있습니다.

**예) 값**
```cpp
auto type1 = 10;			// int
auto type2 = 10.f;			// float
auto type3 = 'c';			// char
auto type4 = "c";			// char const *
auto type5 = "String";		// char const *
auto type6 = { 1, 2, 3 };	// std::initializer_list<int>
```

`auto` 키워드는 C++11에서는 함수의 반환형으로 사용할 수 있지만 후행 반환 형식을 작성했을 경우에만 사용 가능합니다. 하지만 C++14 이상 부터는 제약없이 사용 가능합니다.

**예) 함수**
```cpp
#include <iostream>
#include <vector>
#include <numeric>

using namespace std;

void increase(int value)
{
    cout << value + 1 << endl;
}

// `-> int`: 후행 반환 형식
// C++14 버전 이상이라면 작성하지 않아도 됨.
auto sum(vector<int> origin) -> int
{
    return accumulate(origin.begin(), origin.end(), 0);
}

int main()
{
    // 함수 포인터
    auto Increase = increase;

    // 람다 함수
    vector<int> indexList = { 1, 2, 3, 4, 5 };
    auto printIndexList = [&] {
        for (const auto& index : indexList)
            cout << index << endl;
    };

    Increase(5);
    printIndexList();
    cout << sum(indexList) << endl;
}
```

## *, &, const
기본적으로 `auto` 키워드는 일반 타입 키워들과 동일한 문법을 공유하기 때문에 `auto` 키워드 앞과 뒤에 붙여서 사용할 수 있습니다.

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
	const auto target = 3;
	vector<int> indexList = { 1, 2, 3, 4, 5 };
    
    // 범위 기반 for문
    for(auto& index : indexList)
    {
    	if(index >= target)
        {
        	cout << index << endl;
        }
    }
}
```

# 참고 자료
[cppreference](https://en.cppreference.com/w/cpp/keyword/auto)
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/auto-cpp?view=msvc-170)
[BlockDMask](https://blockdmask.tistory.com/384)
[불로그](https://m.blog.naver.com/kyed203/220068115571)