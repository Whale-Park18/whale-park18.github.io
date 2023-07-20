# 설명
`nullptr` 키워드는 null 포인터를 나타내는 상수 리터럴로 `std::nullptr_t` 타입의 prvalue입니다. `nullptr` 키워드는 헤더를 포함하지 않고 사용할 수 있지만 `std::ptr_t` 타입을 사용하려면 `<cstddef>` 헤더를 포함해야 합니다.

> `std::nullptr_t` 타입은 암시적으로 모든 타입의 포인터로 형 변환이 가능합니다.

## NULL의 문제
C++에 객체지향 패러다임이 적용되면서 오버로딩을 지원하게 되면서 `NULL`을 사용할 때, 문제가 발생하게 됩니다.

> 오버로딩을 간단하게 설명하면 똑같은 이름을 사용하지만 다른 매개 변수를 갖는 함수를 뜻합니다.

```cpp
#include <iostream>

using namespace std;

void func(int n)
{
    cout << "call void func(int)" << endl;
}

void func(int* p)
{
    cout << "call void func(int*)" << endl;
}

int main()
{
    func(0);
    func(NULL);
    func(nullptr);
}
```

```
[출력 결과]
call void func(int)
call void func(int)
call void func(int*)
```

프로그래머는 `NULL`을 전달하며 `void func(int*)`가 호출되는 것을 의도했겠지만 `NULL`은 `0`으로 정의된 매크로이기 때문에 `void func(int)`가 호출됩니다. 하지만 `nullptr`을 전달할 경우 의도대로 `void func(int*)`가 호출되는 것을 볼 수 있습니다. 

포인터를 초기화할 때, `nullptr` 키워드와 `NULL`은 똑같이 `0`으로 초기호되지만, 매개 변수로 전달될 때, `NULL`은 `0`으로 정의된 상수이기 때문에 컴파일러가 `void func(int)`로 호출하지만 `nullptr`은 `std::nullptr_t` 타입의 상수 리터럴이기 때문에 `void func(int*)`가 호출되는 것입니다.

## nullptr을 사용해야 하는 이유
`nullptr` 키워드를 사용하면 위의 상황처럼 의도치 못한 오류의 가능성을 없애 안전성을 높일 뿐만 아니라 코드의 가독성까지 높이기 때문에 C++11 이후의 버전에서 `nullptr` 키워드 대신 `NULL`을 사용해야 할 이유가 없습니다.

> `NULL`을 비어있다라는 의미로 초기화 하는 경우도 있기 때문에 `NULL`을 보고 null 포인터라고 판단할 수 없지만, `nullptr` 키워드는 null 포인터임을 알 수 있어 코드의 가독성이 높아집니다.

# 참고 자료
[cppreference](https://en.cppreference.com/w/cpp/language/nullptr)
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/nullptr?view=msvc-170)
[BlockDMask](https://blockdmask.tistory.com/501)
[냉정과 열정 사이](https://psychoria.tistory.com/21)