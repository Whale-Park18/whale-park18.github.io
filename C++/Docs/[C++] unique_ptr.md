> 본 글은  `unique_ptr` 에 대한 설명입니다.
>
> `unique_ptr `클래스가 갖는 멤버 변수나 함수에 대한 자세한 설명은 다른 글에 작성하겠습니다.

# 설명

## `unique_ptr`이란?

`unique_ptr`은 *오직 하나의 객체의 유일한 소유권을 갖는 엄격한 소유권 개념을 도입한 스마트 포인터*입니다. `unique_ptr`을 사용하기 위해선 `memory` 헤더를 포함해야 합니다.

이 스마트 포인터는 해당 객체의 소유권을 가지고 있을 때만, 소멸자가 해당 객체를 삭제할 수 있습니다. `unique_ptr` 인스턴스는 `std::move()` 함수를 통해 소유권을 이동시킬 수 있지만, 복사할 수는 없습니다. 소유권이 이전되면, 이전 `unique_ptr` 인스턴스는 더 이상 해당 객체를 소유하지 않게 재설정됩니다.

>  `unique_ptr`은 이러한 특성 때문에 클래스나 구조체의 멤버 변수로 포인터가 필요할 경우에 자주 사용됩니다.

## 왜 엄격한 소유권을 갖는가?

`unique_ptr`이 이러한 특성을 갖게된 이유는 **double-free** 오류에 있습니다. double-free 오류란 이미 해제한 메모리를 다시 해제하는 경우 발생하는 오류로 다음과 같은 상황에서 자주 발생합니다.

```cpp
Data* p_data1 = new Data();
Data* p_data2 = p_data1;

// p_data1 사용 끝, 해제
delete P_data1;

// ..

// p_data2 사용 끝, 해제
delete p_data2;
```

`p_data1`과 `p_data2`가 같은 객체를 가리키고 있고, `delete p_data1`을 통해 그 객체를 소멸시켰습니다. 그런데, `p_data2`가 이미 소멸된 객체를 다시 소멸시킵니다. 이런 경우 메모리 오류가 발생하며 프로그램이 죽게됩니다.

위의 *문제가 발생한 이유는 생성된 객체의 소유권이 명확하지 않고 쉽게 접근할 수 있기 때문*입니다. 만약 처음 초기화된 포인터에게 소유권을 줬다면 위와 같이 같은 객체를 두 번 해제하는 일은 발생하지 않았을 것입니다.

즉, `unique_ptr`은 소유권이 명확하지 않아 발생하는 오류로부터 안전해지기 위해 엄격한 소유권을 지닌 스마트 포인터로 설계되었습니다.

## `unique_ptr` 사용하기

### 생성 및 멤버 접근

스마트 포인터는 모든 타입에 대한 포인터를 담은 객체여야 하기 때문에 템플릿 클래스로 작성되어 있습니다. 즉, 스마트 포인터는 생성자를 사용해 생성하며,  `operator*`와 `operator->`가 오버로딩되어 있기 때문에 원시 포인터처럼 사용할 수 있습니다.

```cpp
#include <iostream>
#include <memory>

using namespace std;

struct Data
{
	string name;
	unsigned int size;
};

int main()
{
	std::unique_ptr<Data> p_data(new Data({"Whale.Park18", 5}));
	cout << (*p_data).size << endl;
	cout << p_data->name << endl;
}
```

#### `make_unique` 생성하기

`unique_ptr`은 생성자로 초기화를 합니다. 그 중 원시 포인터를 인자로 받는 생성자가 있습니다. 만약 다른 `unique_ptr` 객체를 같은 포인터로 초기화하는 것이 가능할까요?

```cpp
#include <iostream>
#include <memory>

using namespace std;

int main()
{
	int* ptr = new int(5);
	unique_ptr<int> p_unique1(ptr);
	unique_ptr<int> p_unique2(ptr);

	*p_unique1 = 10;

	cout << "p_unique1: " << *p_unique1 << endl;
	cout << "p_unique2: " << *p_unique2 << endl;
}
```

```
[출력 결과]
10
10
<런타임 오류 발생>
```

놀랍게도 다른 `unique_ptr` 객체가 같은 포인터로 초기화하는 것이 가능합니다. 하지만 스코프를 벗어나면서 각각의 스마트 포인터가 같은 포인터를 해제하게 되면서 런타임 오류가 발생하게 됩니다.

C++14에서는 안전하게 `unique_ptr` 객체를 생성하기 위해 `make_unique` 함수를 제공합니다. `make_unique` 함수는 전달 받은 인자로 `unique_ptr` 객체를 만들어 반환합니다. 때문에 위의 예제처럼 같은 포인터를 갖는 `unique_ptr` 객체를 생성하는 실수에서 멀어질 수 있습니다.
```cpp
#include <iostream>
#include <memory>

using namespace std;

struct Data
{
	string name;
	unsigned int size;
};

int main()
{
	unique_ptr<Data> p_data = make_unique<Data> Data({ "Whale.Park18", 5 }));
}
```

### 복사하기

`unique_ptr`은 엄격한 소유권을 갖고 있다 설명했습니다. 그렇다면 복사를 시도할 경우 어떻게 작동할까요?

```cpp
#include <iostream>
#include <memory>

using namespace std;

struct Data
{
	string name;
	unsigned int size;
};

int main()
{
	unique_ptr<Data> p_data(new Data({"Whale.Park18", 5}));
	unique_ptr<Data> p_data_copy = p_data;
}
```

```
[오류 목록]
심각도	코드	설명	프로젝트	파일	줄	비표시 오류(Suppression) 상태
오류	C2280	'std::unique_ptr<Data,std::default_delete<Data>>::unique_ptr(const std::unique_ptr<Data,std::default_delete<Data>> &)': 삭제된 함수를 참조하려고 합니다.
```

복사를 시도할 경우 위와 같은 오류가 발생하게 됩니다. 위 오류는 삭제된 함수를 사용하려 했을 때, 발생하는 것입니다.

삭제된 함수란 C++11에서 추가된 기능으로 프로그래머가 명시적으로 '이 함수는 사용하지 말 것!'을 표현한 것입니다. 혹시라도 삭제된 함수를 사용할 경우 위와 같이 컴파일 오류가 발생하게 됩니다.

`unique_ptr`의 경우 **유일하게 소유하는 엄격한 소유권** 때문에 복사 생성자와 대입 연산자(복사 기능 한정)를 *명시적으로 삭제*했습니다. 만약 복사 생성자나 대입 연산자를 사용할 수 있다면 유일한 소유권을 갖는 특성이 사라지게 되면서 `unique_ptr`는 존재 의의를 잃게 됩니다.

### 소유권 이전하기

`unique_ptr`은 엄격한 소유권으로 복사는 불가능하지만 소유권을 이전할 수 있습니다.

```cpp
#include <iostream>
#include <memory>

using namespace std;

struct Data
{
	string name;
	unsigned int size;
};

int main()
{
	unique_ptr<Data> p_data1(new Data({"whale.park18", 3}));
	unique_ptr<Data> p_data2;

	cout << "Before move - owner: p_data1" << endl;
	cout << "p_data1: " << p_data1.get() << ", p_data2: " << p_data2.get() << endl;
	cout << endl;

	cout << "After move - owner: p_data2" << endl;
	p_data2 = move(p_data1);
	cout << "p_data1: " << p_data1.get() << ", p_data2: " << p_data2.get() << endl;
	cout << endl;
}


```

```plaintext
Before move - owner: p_data1
p_data1: 015F8BF0, p_data2: 00000000

After move - owner: p_data2
p_data1: 00000000, p_data2: 015F8BF0
```

`unique_ptr`은 복사 생성자는 삭제되어 있지만, 이동 생성자는 가능합니다. 집이나 자동차같이 소유권을 이동시키는 개념과 동일하다 생각하면 됩니다.

`std::move`를 사용하여 `p_data1`의 소유권을 `p_data2`에게 이전합니다. 그러면 `p_data1`이 가리키던 주소는 0(`nullptr`)을 가리키게 되고 `p_data2`는 `p_data1`이 가리키던 주소를 가리키면서 소유권이 이전 된 것을 확인할 수 있습니다.

> 소유권이 이전된 `unique_ptr`를 댕글리 포인터(dangling pointer)라고 하며 이를 재참조할 때, 런타임 오류가 발생합니다. 따라서 소유권 이전은 댕글링 포인터를 다시 참조하지 않겠다는 확신을 갖고 이동시켜야 합니다.

### 함수 인자로 전달하기

일반적으로 함수 인자를 전달하게 되면 복사가 발생하며 전달하게 됩니다. 하지만  `unique_ptr`은 위에서 설명했듯이 복사가 불가능합니다. 즉, `unique_ptr`은 함수 인자로 전달할 수 없다는 것입니다.

하지만 복사가 일어나지 않는 레퍼런스를 전달한다면 가능하지 않을까요?

```cpp
#include <iostream>
#include <memory>

using namespace std;

class Data
{
private:
	int* size;

public:
	Data()
	{
		size = new int(100);
		cout << "Get Resource" << endl;
	}

	~Data()
	{
		delete size;
		cout << "Free Resource" << endl;
	}

	int get_size() { return *size; }
};

inline void do_something(unique_ptr<Data>& p_data)
{
	cout << "[Start do_something]" << endl;

	cout << "p_data.size: " << p_data->get_size() << endl;

	cout << "[End do_something]" << endl;
}

int main()
{
	cout << "[Start Program]" << endl;

	unique_ptr<Data> p_data(new Data());
	do_something(p_data);

	cout << "[End Program]" << endl;
}
```

```
[출력 결과]
[Start Program]
Get Resource
[Start do_something]
p_data.size: 100
[End do_something]
[End Program]
Free Resource
```

`unique_ptr` 레퍼런스로 전달할 경우 정상적으로 작동하는 것을 확인할 수 있습니다. `do_somethint`의 `p_data`는 레퍼런스이기 때문에 함수가 종료되도 객체를 파괴되지 않고 `main` 함수가 종료되면서 파괴는 것을 확인할 수도 있습니다.

> 참고한 자료 중에서 레퍼런스이긴 하지만 유일하게 소유한다는 원칙을 벗어나기 때문에 문맥상 옳지 못하다고 의견을 내신 분도 있습니다.

위에선 값으로 전달하지 못한다고 했지만 정확하게 말하자면 불가능 한 것은 아닙니다. 소유권을 `get` 멤버 함수를 사용해 포인터 주소값을 전달하거나 `move`를 사용해 소유권을 넘기는 것입니다. 하지만 소유권을 넘기게 된다면 함수가 끝날 때, 자원을 반환한다는 문제가 발생합니다.

# 참고

[cppreference - en](https://en.cppreference.com/w/cpp/memory/unique_ptr)

[cppreference - ko](https://ko.cppreference.com/w/cpp/memory/unique_ptr)

[cplusplus.com](https://cplusplus.com/reference/memory/unique_ptr/?kw=unique_ptr)

[Runebook.dev](https://runebook.dev/ko/docs/cpp/memory/unique_ptr)

[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/how-to-create-and-use-unique-ptr-instances?view=msvc-170)

[TCP School](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[모두의 코드](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[식빵맘(ansohxxn)](https://ansohxxn.github.io/cpp/chapter15-5/#unique_ptr%EC%9D%98-%ED%95%A8%EC%88%98%EB%93%A4)

[코드없는 프로그래밍](https://www.youtube.com/watch?v=oNqm04uL3v8)

[포프TV](https://www.youtube.com/watch?v=MGVSPZoOchE)
