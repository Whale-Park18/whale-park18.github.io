# 구문
> `for` `(` `element-type` `element-name` `:` `data-list` `)`

* `element-type` - `data-list`의 요소의 데이터 타입
* `element-name` - `data-list`의 요소에 접근할 변수명
* `data-list` - 배열, `vector` 등의 순회가 가능한 데이터 리스트

# 설명
배열, 벡터 등의 데이터 리스트 자료형을 순회해야 할 때, 보통 인덱스와 데이터 리스트의 길이를 이용합니다. 정확한 인덱스를 설정했다면 문제없지만 프로그래머의 실수로 잘못된 인덱스를 설정한다면 에러가 발생하게 됩니다.

범위기반 for 문은 기존의 for 문과 달리 스스로 데이터 리스트의 처음부터 끝까지 순회하는 반복문입니다. 스스로 순회 범위를 판단하기 때문에 인덱스 실수로 인한 에러를 줄일 수 있습니다.

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
	// 1. 배열
	int arr[] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
    for(auto element : arr)
    {
    	cout << element << endl;
    }
    
    // 2. STL
    vector<int> v = { 9, 8, 7, 6, 5, 4, 3, 2, 1, 0 };
    for(auto element : v)
    {
    	cout << element << endl;
    }
}
```

범위기반 for 문에서 접근한 요소는 값을 복사한 것입니다. 때문에 범위기반 for 문의 본문에서 요소를 변경하더라도 데이터 리스트에는 적용되지 않습니다. 또한 요소의 크기가 크다면 복사로 인한 성능 하락이 발생할 수도 있습니다.

범위기반 for 문에서 접근한 요소를 변경하고 복사로 인한 성능 하락을 피하는 방법은 간단합니다. 참조를 이용하는 것입니다. 

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    vector<int> v = { 9, 8, 7, 6, 5, 4, 3, 2, 1, 0 };
    for(auto& element : v)
    {
    	element += 1;
    }
    
    for(auto& element : v)
    {
    	cout << element << endl;
    }
}
```

하지만 범위기반 for 문은 아이러니하게도 인덱스 때문에 기존의 for 문을 완전히 대체하지 못합니다. 범위기반 for문에는 인덱스를 나타내는 정보가 존재하지 않고 오직 요소의 정보만을 표시합니다. 때문에 인덱스를 분류나 연산을 할 수 없습니다.

굳이 하려면 할 수 있습니다. 하지만... '이럴거면 그냥 인덱스에 신경써서 기존의 for 문을 쓰는게 낮지 않나...?' 라는 생각이 들긴합니다.

```cpp
#include <iostream>

using namespace std;

int main()
{
	int arr[] = { 1, 2, 3, 4 };
    int index = 0;
    for(auto& element : arr)
    {
    	element += index++ * 2;
    	cout << element << endl;
    }
}
```

# 참고 자료
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/range-based-for-statement-cpp?view=msvc-170)
[BlockDMask](https://blockdmask.tistory.com/319)