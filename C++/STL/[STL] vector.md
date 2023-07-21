# 구문
```cpp
template <class Type, class Allocator = allocator<Type>>
class vector
```

# 설명
`std::vector` 컨테이너는 원소의 개수에 따라 자동으로 길이를 변경하는 **동적 배열**인 순차(시퀀스) 컨테이너입니다. 벡터(`std::vector`)를 사용하려면 `vector` 헤더를 포함해야 합니다.

## 멤버 함수
> STL에서 제공하는 컨테이너의 멤버 함수는 거의 비슷하기 때문에 일부 멤버 함수들은 생략했습니다. 생략된 멤버 함수의 설명을 필요한 경우 [array]([STL]%20array.md)을 보시면 됩니다.

### 생성자
* `vector<T> v()`  
    → 비어있는 벡터를 생성한다.
* `vector<T> v(n)`  
    → 크키가 `n`인 벡터를 생성한다.  
    → 단, 원소는 `T` 타입의 기본값으로 초기화된다.
* `vector<T> v(n, value)`  
    → 크기가 `n`이고 `value`로 초기화된 벡터를 생성한다.
* `vector<T> v2(v1)`  
    →  `v1` 벡터를 복사한 `v2` 벡터를 생성한다.

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
	vector<int> v1;
	vector<int> v2(3);
	vector<int> v3(2, 2);
	vector<int> v4(v2);
	vector<int> v5 = { 1, 2, 3, 4, 5 };	// C++11에서 추가된 초기화 방법

	cout << "=====v1 elements=====" << endl;
	for (auto item : v1)
		cout << item << ' ';
	cout << endl;

	cout << "=====v2 elements=====" << endl;
	for (auto item : v2)
		cout << item << ' ';
	cout << endl;
	
	cout << "=====v3 elements=====" << endl;
	for (auto item : v3)
		cout << item << ' ';
	cout << endl;
	
	cout << "=====v4 elements=====" << endl;
	for (auto item : v4)
		cout << item << ' ';
	cout << endl;

	cout << "=====v5 elements=====" << endl;
	for (auto item : v5)
		cout << item << ' ';
	cout << endl;
}
```
```
[출력 결과]
=====v1 elements=====

=====v2 elements=====
0 0 0
=====v3 elements=====
2 2 
=====v4 elements=====
0 0 0
=====v5 elements=====
1 2 3 4 5
```

### 반복자
* `v.begin()`
* `v.end()`
* `v.rbegin()`
* `v.rend()`
* `v.cbegin()`
* `v.cend()`
* `v.crbegin()`
* `v.crend()`

### 원소 접근
* `v.at(pos)`
* `v[pos]`
* `v.front()`
* `v.back()`
* `v.data()`

### 크기
* `empty()`
* `size()`
    → 벡터의 길이(원소의 개수)를 반환한다.
* `max_size()`  
    → 벡터의 최대 길이를 반환한다.
* `capacity()`  
    → 벡터에게 할당된 메모리 크기를 반환한다.
* `reserve(count)`  
    → 벡터에게 할당된 메모리 크기를 `count`로 변경한다.  
    → `count`가 현재 할당된 메모리 크기보다 작거나 같다면 작동하지 않는다.
* `shrink_to_fit()`  
    → 벡터에게 할당된 메모리 중, 사용하지 않는 메모리를 해제한다.
* `resize(count)`  
    → 벡터의 크기를 `count`로 변경한다.
    → 새롭게 추가된 원소는 기본값으로 초기화된다.
    → `count`가 현재 크기보다 작거나 같다면 작동하지 않는다.
* `resize(count, value)`  
    → 추가된 원소의 초기화 되는 값을 `value`로 설정한다.

```cpp
#include <iostream>
#include <array>
#include <vector>

using namespace std;

int main()
{
	vector<int> v1(1);
	vector<int> v2 = { 1, 2, 3, 4, 5 };

	cout << "max_size" << endl;
	cout << "v1: " << v1.max_size() << endl;
	cout << "v2: " << v2.max_size() << endl;
	cout << endl;
	
	cout << "capacity" << endl;
	cout << "v1: " << v1.capacity() << endl;
	cout << "v2: " << v2.capacity() << endl;
	cout << endl;
	
	cout << "reserve" << endl;
	cout << "v1's Size before reserve(size/capacity): " << v1.size() << '/' << v1.capacity() << endl;
	v1.reserve(5);
	cout << "v1's Size after reserve(size/capacity): " << v1.size() << '/' << v1.capacity() << endl;
	cout << endl;
	
	cout << "shrink_to_fit" << endl;
	cout << "v1's Size before shrink_to_fit(size/capacity): " << v1.size() << '/' << v1.capacity() << endl;
	v1.shrink_to_fit();
	cout << "v1's Size after shrink_to_fit(size/capacity): " << v1.size() << '/' << v1.capacity() << endl;
	cout << endl;
	
	cout << "resize" << endl;
	cout << "v1's Size before resize(size/capacity): " << v1.size() << '/' << v1.capacity() << endl;
	v1.resize(5);
	cout << "v1's Size after resize(size/capacity): " << v1.size() << '/' << v1.capacity() << endl;
	cout << endl;
}
```
```
[출력 결과]
max_size
v1: 1073741823
v2: 1073741823

capacity
v1: 1
v2: 5

reserve
v1's Size before reserve(size/capacity): 1/1
v1's Size after reserve(size/capacity): 1/5

shrink_to_fit
v1's Size before shrink_to_fit(size/capacity): 1/5
v1's Size after shrink_to_fit(size/capacity): 1/1

resize
v1's Size before resize(size/capacity): 1/1
v1's Size after resize(size/capacity): 5/5
```

### 제어
* `clear()`  
    → 벡터의 모든 원소를 제거한다.
* `insert(const_iterator pos, T&& value)`  
    → 벡터의 pos에 취
* `emplace(const_iterator pos, Args&&... args)`  
    → 
* `erase()`  
    → 
* `push_back()`  
    → 
* `emplace_back()`  
    → 
* `pop_back`  
    → 
* `swap(vector& other)`  

---
* [`operator=`]()


* [제어]()

# 참고 자료
[cpprefernce](https://ko.cppreference.com/w/cpp/container/vector)
[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/standard-library/vector-class?view=msvc-170)
[모두의 코드](https://modoocode.com/175)
[BlockDMask](https://blockdmask.tistory.com/70)