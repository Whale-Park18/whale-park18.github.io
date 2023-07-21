# 구문
```cpp
template <class Ty, size_t N>
class array;
```

# 설명
`std::array` 컨테이너는 C 스타일 배열과 같은 형식을 유지하면서, STL에서 추가된 반복자, 알고리즘, 대입 연산자 등을 사용할 수 있게 있어 많은 편리함을 제공하는 순차(시퀀스) 컨테이너입니다. 배열을 사용하려면 `array` 헤더를 포함해야 합니다.

배열(`array`)은 STL의 컨테이너 중 하나이기 때문에 템플릿으로 정의되어 있습니다. 또한 고정된 길이의 선형 구조이기 때문에 길이에 대한 정보도 필요합니다. 즉, 배열을 선언하기 위해서는 저장할 타입과 배열의 길이의 정보가 필요합니다.

```cpp
#include <array>

int main()
{
    std::array<int, 5> arr;
}
```

## 멤버 함수
STL에서 제공하는 컨테이너가 제공하는 멤버 함수는 거의 비슷합니다. 때문에 첫 번째인 현재 글 이후에는 간단하게 생략하겠습니다.

* [생성자](#예제-생성자-및-초기화)

* [`operator=`](#예제-생성자-및-초기화)

* [반복자](#예제-반복자)
    * `begin()`
        → 배열(컨테이너)의 첫 번쨰 원소를 가리키는 반복자를 반환한다.
    * `end()`
        → 배열(컨테이너)의 마지막 원소의 <u>**다음 원소**</u>를 가리키는 반복자를 반환한다.
    * `rbegin()`
        → 배열(컨테이너)의 반대 방향의 첫 번째 원소를 가리키는 반복자를 반환한다.
    * `rend()`
        → 배열(컨테이너)의 반대 방향의 마지막 원소의 <u>**다음 원소**</u>를 가리키는 반복자를 반환한다.
    * `cbegin()`
        → 배열(컨테이너)의 첫 번째 원소를 가리키는 `const` 반복자를 반환한다.
    * `cend()`
        → 배열(컨테이너)의 마지막 원소의 <u>**다음 원소**</u>를 가리키는 `const` 반복자를 반환한다.
    * `crbegin()`
        → 배열(컨테이너)의 반대 방향의 첫 번째 원소를 가리키는 `const` 반복자를 반환한다.
    * `crend()`
        → 배열(컨테이너)의 반대 방향의 마지막 원소의 <u>**다음 원소**</u>를 가리키는 `const` 반복자를 반환한다.

* [원소 접근](#예제-원소-접근)
    * `at(size_type pos)`
        → 배열(컨테이너)의 `pos` 위치에 있는 원소를 반환한다.
    * `operator[](size_type pos)`
        → 배열(컨테이너)의 `pos` 위치에 있는 원소를 반환한다.
    * `front()`
        → 배열(컨테이너)의 첫 번째 원소를 반환한다.
    * `back()`
        → 배열(컨테이너)의 마지막 원소를 반환한다.
    * `data()`
        → 배열(컨테이너)의 첫 번째 원소의 주소를 반환합니다.

* [크기](#예제-크기)
    * `empty()`
        → 배열(컨테이너)이 비어있는지 확인한다.
    * `max_size()`
        → 배열(컨테이너)의 최대 크기를 반환한다.
        → `array`에서는 `size()`와 같다.
    * `size()`
        → 배열(컨테이너)의 크기를 반환한다.
        → `array`에서는 `max_size()`와 같다.
        
* [제어](#예제-그-외)
    * `fill(const Ty& value)`
        → 배열의 모든 원소를 `val`로 바꾼다.
    * `swap(array& other)`
        → 두 배열(컨테이너)의 내용을 바꾼다.

## 예제
### 예제: 생성자 및 초기화
```cpp
#include <iostream>
#include <array>

using namespace std;

int main()
{
    array<int, 3> arr1 = { 1, 2, 3 };   // 지정된 값으로 초기화
    array<int, 2> arr2;                 // 쓰레기 값으로 초기화
    array<int, 4> arr3 = { 0 };         // 모든 원소 0으로 초기화
    array<int, 5> arr4 = { 10 };        // 첫 번째 원소만 10으로 초기화, 나머지 0으로 초기화
    array<int, 3> arr5{4, 5, 6};        // C++11에서 추가된 초기화 방법
    array<int, 4> arr6 = arr3;          // 대입 연산자를 사용한 초기화

    cout << "=====arr1=====" << endl;
    for (auto item : arr1)
        cout << item << endl;

    cout << "=====arr2=====" << endl;
    for (auto item : arr2)
        cout << item << endl;

    cout << "=====arr3=====" << endl;
    for (auto item : arr3)
        cout << item << endl;

    cout << "=====arr4=====" << endl;
    for (auto item : arr4)
        cout << item << endl;

    cout << "=====arr5=====" << endl;
    for (auto item : arr5)
        cout << item << endl;

    cout << "=====arr6=====" << endl;
    for (auto item : arr6)
        cout << item << endl;
}
```
```
[출력 결과]
=====arr1=====
1
2
3
=====arr2=====
-858993460
-858993460
=====arr3=====
0
0
0
0
=====arr4=====
10
0
0
0
0
=====arr5=====
4
5
6
=====arr6=====
0
0
0
0
```
`array`를 초기화 하는 방법은 변수 선언과 동시에 초기화하거나 선언 후, 인덱스를 통해 접근해 초기화 하는 등 C 스타일 배열과 동일하게 초기화할 수 있습니다. 초기화를 하지 않았을 경우 쓰레기 값이 들어가는 것 또한 동일합니다.

다른 점이 있다면 `operator=` 연산자를 사용한 덮어쓰기가 가능하다는 것입니다. 하지만 주의할 점이 존재합니다. 다른 컨테이너의 정보를 덮어쓸 경우에는 오버헤드가 발생하지 않지만 초기화 리스트를 덮어쓸 때는 

다른 컨테이너의 정보를 `operator=` 연산자를 이용해 복사하는 것은 오버헤드가 발생하지 않지만 초기화 리스트를 복사할 경우, 초기화 리스트의 정보들을 복사한 후(새롭게 변수 선언 == 기본 생성자), 복사(복사 생성자)하는 오버헤드가 발생합니다.

```cpp
#include <iostream>
#include <array>

using namespace std;

class Type
{
public:
    Type()
    {
        cout << "기본 생성자" << endl;
    }

    Type(Type& other)
    {
        cout << "복사 생성자" << endl;
    }
};

int main()
{
    Type type;
    array<Type, 1> arr;
    arr = { type };
}
```
```
[출력 결과]
기본 생성자
기본 생성자
복사 생성자
```

### 예제: 반복자
```cpp
#include <iostream>
#include <array>

using namespace std;

int main()
{
    array<int, 5> arr = { 1, 2, 3, 4, 5 };

    cout << "=====begin - end=====" << endl;
    for (array<int, 5>::iterator iter = arr.begin(); iter != arr.end(); iter++)
        cout << *iter << endl;

    cout << "=====rbegin - rend=====" << endl;
    for (auto iter = arr.rbegin(); iter != arr.rend(); iter++)
        cout << *iter << endl;

    cout << "=====Range-for=====" << endl;
    for(auto item : arr)
        cout << item << endl;
}
```
```
[출력 결과]
=====begin - end=====
1
2
3
4
5
=====rbegin - rend=====
5
4
3
2
1
=====Range-for=====
1
2
3
4
5
```
반복자를 변수로 사용하기 위해서는 `컨테이너 템플릿 선언문` `::` `iterator` 구문으로 선언해야 합니다. `컨테이너 템플릿 선언문`은 위의 예제로 설명하자면 `arr`의 변수 타입인 `array<int, 5>`입니다. 
반복자를 사용할 때마다 이렇게 반복자를 선언하는 것음 참으로 귀찮고 오타로 인한 에러가 발생할 수도 있습니다. 때문에 C++11에서는 `auto`라는 강력한 추론 타입이 존재하기 때문에 `auto`를 사용하는 것을 권합니다.

[STL]([STL]%20Standard%20Template%20Library.md)에서 반복자는 포인터의 개념을 품은(또는 비슷한) 객체로 설명했습니다. 때문에 반복자로 데이터에 접근하기 위해서는 포인터 변수처럼 참조 연산자(`*`)를 사용해 접근해야 합니다.

또한 기본 시퀀스를 반환하는 반복자(`begin`, `end` 등)과 반전 시퀀스를 반환하는 반복자(`rbegin`, `rend` 등)는 서로 다른 타입이기 때문에 비교 연산자를 이용한 비교가 불가능합니다.

> `begin`, `end` 등의 기본 시퀀스 반복자 타입: `iterator`
> `rebgin`, `rend` 등의 반전 시퀀스 반복자 타입: `reverse_iterator`

> `cbegin`, `cend`, `crbegin`, `crend`는 `begin`, `end`, `rbegin`, `rend`에서 `const`만 적용한 것이기 때문에 생략했습니다.

### 예제: 원소 접근
```cpp
#include <iostream>
#include <array>

using namespace std;

int main()
{
    array<int, 5> arr = { 1, 2, 3, 4, 5 };

    cout << "at(2): " << arr.at(2) << endl;
    cout << "[3]: " << arr[3] << endl;
    cout << "front: " << arr.front() << endl;
    cout << "back: " << arr.back() << endl;
    cout << "data: " << arr.data() << endl;
}
```
```
[출력 결과]
at(2): 3
[3]: 4
front: 1
back: 5
data: 00EFFC1C
```

`data`는 배열의 첫 번째 원소의 주소를 반환합니다. 그렇다는 것은 C 스타일 배열을 포인터로 접근한 것처럼 접근이 가능한 것일까요?

```cpp
#include <iostream>
#include <array>

using namespace std;

int main()
{
    array<int, 5> arr = { 1, 2, 3, 4, 5 };

    cout << "*(data + 0): " << *(arr.data() + 0) << endl;
    cout << "*(data + 1): " << *(arr.data() + 1) << endl;
    cout << "*(data + 2): " << *(arr.data() + 2) << endl;
    cout << "*(data + 3): " << *(arr.data() + 3) << endl;
    cout << "*(data + 4): " << *(arr.data() + 4) << endl;
}
```
```
[출력 결과]
*(data + 0): 1
*(data + 1): 2
*(data + 2): 3
*(data + 3): 4
*(data + 4): 5
```

### 예제: 크기
```cpp
#include <iostream>
#include <array>

using namespace std;

int main()
{
    array<int, 5> arr = {1, 2, 3, 4, 5};

    cout << "empty: " << std::boolalpha << arr.empty() << endl;
    cout << "max_size: " << arr.max_size() << endl;
    cout << "size: " << arr.size() << endl;
}
```
```
[출력 결과]
empty: false
max_size: 5
size: 5
```

### 예제: 제어
```cpp
#include <iostream>
#include <array>

using namespace std;

int main()
{
    array<int, 5> arr1 = {1, 2, 3, 4, 5};

    cout << "=====fill before=====" << endl;
    for (auto item : arr1)
       cout << item << endl;

    cout << "=====fill after=====" << endl;
    arr1.fill(10);
    for (auto item : arr1)
        cout << item << endl;
    cout << endl;

    array<int, 5> arr2 = { 6,7,8,9,0 };
    cout << "=====swap before=====" << endl;
    cout << "arr1" << '\t' << "arr2" << endl;
    for (size_t i = 0; i < arr1.size(); i++)
       cout << arr1.at(i) << '\t' << arr2.at(i) << endl;

    cout << "=====swap after=====" << endl;
    arr1.swap(arr2);
    cout << "arr1" << '\t' << "arr2" << endl;
    for (size_t i = 0; i < arr1.size(); i++)
       cout << arr1.at(i) << '\t' << arr2.at(i) << endl;
}
```
```
[출력 결과]
=====fill before=====
1
2
3
4
5
=====fill after=====
10
10
10
10
10

=====swap before=====
arr1    arr2
10      6
10      7
10      8
10      9
10      0
=====swap after=====
arr1    arr2
6       10
7       10
8       10
9       10
0       10
```

# 참고 자료
[cpprefernce](https://ko.cppreference.com/w/cpp/container/array)
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/standard-library/array-class-stl?view=msvc-170#array)
[모두의 코드](https://modoocode.com/314)
[소년코딩](https://boycoding.tistory.com/213)
[BlockDMask](https://blockdmask.tistory.com/332)