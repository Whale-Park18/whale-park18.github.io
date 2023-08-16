# 설명

C#이나 Java같은 프로그래밍 언어에는 가비지 컬렉터(**G**arbage **C**ollector:GC)라는 자원 청소기가 기본적으로 내장되어 있습니다. 이 가비지 컬렉터는 프로그램에서 더 이상 사용하지 않는 자원을 자동으로 해제하는 역할을 수행하기 때문에 프로그래머가 자원을 해제하는 것에 대한 신경을 쓰지 않아도 됩니다.

C++은 C언어를 계승했기 이러한 개념이 존재하지 않는 대신 프로그래머가 생 포인터(raw pointer)를 사용해, 수동으로 메모리를 할당받고 해제할 수 있기 때문에 보다 효율적인 프로그램을 만들 수 있습니다.

하지만 프로그래머가 메모리 할당과 해제 시기를 결정하기 때문에 만약 사용한 메모리를 실수로 해제하지 않고 메모리 주소를 다른 값으로 초기화 시킨다면 프로그램이 종료되기 전까지, 메모리에 계속 남아있는 **메모리 누수(Memory leak)**가 발생합니다.

> 메모리 누수 외에도 아래와 같은 생 포인터를 이용하기 어려운 이유가 존재합니다.
> 
> 1. 선언만 봐서는 하나의 객체를 가리키는지 배열을 가리키는지 구분할 수 없다.
> 
> 2. 선언만 봐서는 포인터를 다 사용한 후 포인터가 가리키는 객체(줄여서 피지칭 객체)를 독자가 직접 파괴해야 하는지 알 수 없다. 다른 말로 하면, 포인터가 피지칭 객체를 **소유**하고 있는지 여부를 알 수  없다.
> 
> 3. 피지칭 객체를 독자가 직접 파괴해야 한다는 점을 알게 된다고 해도, 구체적으로 어떻게 파괴해야 하는지에 대한 정보를 얻을 수 없다. `delete`를 사용해야 할 수도 있고, 다른 어떤 파괴(소멸) 매커니즘을 사용해야할 수도 있다(이를테면 그 포인터를 전용 파괴 함수에 넘겨주는 등).
> 
> 4. `delete`를 이용해서 파괴해야 함을 알게 된다고 해도, 1번 이유 때문에 단일 객체 삭제 버전(`delete`)을 사용해야 할지 아니면 배열 버전(`delete[]`)을 사용해야 할지 알 수 없는 경우가 있다. 만일 잘못된 버전을 사용하면 그 결과는 미정의 행동(undefined behavior)이다.
> 
> 5. 포인터가 피지칭 객체를 소유하고 있으며 그것을 파괴하는 구체적인 방법을 알아냈다고 해도, 코드의 모든 경로에서 파괴가 **정확히 한 번** 일어남을 보장하기가 어렵다(이를 테면 예외 때문에). 파괴가 일어나지 않는 경로가 하나라도 있으면 메모리 누수가 발생할 수 있으며, 파괴를 여러 번 수행하는 것은 미정의 행동으로 이어진다.
> 
> 6. 대체로, 포인터가 피지칭 객체를 잃었는지 알아내는 방법은 없다. 즉, 포인터가 가리키는 메모리 장소에 유효한 객체(포인터가 가리키게 되어 있는)가 더 이상 존재하지 않는 상황을 파악할 수 없다. 포인터가 객체를 여전히 가리키고 있는 상황에서 객체를 파괴하면 포인터는 대상을 잃게 된다.

'프로그래머가 신경 써서 메모리 해제만 했어도 일어나지 않았을 문제 아닌가?' 생각할 수 있습니다. 객관적으로 보자면 신경쓰지 못한 프로그래머의 잘못이 맞습니다. 하지만 우리는 사람이기 때문에 실수로 메모리 해제를 잊을 수도, 언어의 사양을 잘못 파악하거나 예상치 못한 곳에서 다른 흐름이 발생해 메모리가 해제되지 않는 실수를 할 수 밖에 없습니다.

예를 들어 소멸자를 이용해 메모리 해제하는 코드를 작성했지만 `Thrower()` 함수로 발생한 예외로 인해, 흐름이 바뀌게 되어 메모리를 해제하지 못하고 넘어가 소멸자가 실행되지 못하는 경우가 있습니다.

```cpp
#include <iostream>

class Socket
{
private:
	int* mPData;

public:
	Socket()
	{
		mPData = new int[100];
		std::cout << "생성자 호출" << std::endl;
	}

	~Socket()
	{
		delete[] mPData;
		std::cout << "소멸자 호출" << std::endl;
	}
};

inline void Thrower()
{
	// 예외 발생
	throw 1;
}

inline void DoSomething()
{
	Socket* pSocket = new Socket();

	// ...

	Thrower();

	// ...

	delete pSocket;
}

int main()
{
	try
	{
		DoSomething();
	}
	catch (int i)
	{
		std::cout << "예외 상황 발생" << std::endl;
	}
}
```

```
[출력 결과]
생성자 호출
예외 상황 발생
```

이러한 문제를 해결하기 위해 C++에서 자원을 관리하는 방법으로 **RAII(Resource Acquisition Is Initialization)**라는 디자인 패턴을 권장합니다. 이는 자원 관리를 스택에 할당한 객체를 통해 수행하는 것입니다.

스택을 이용하는 이유는 스택에 할당된 객체는 스코프를 벗어나게 되면 자동으로 스택에서 제거되면서 소멸자가 호출되기 때문입니다. 예를 들어 위 예제에서 `p_socket`은 객체가 아니기 때문에 소멸자가 호출되지 않은 것입니다. 즉, `p_socket`을 일반적인 포인터가 아닌, <u>포인터 객체로 만들어서 자신이 소멸될 때, 자신이 가리키고 있는 자원을 해제하면 되는 것</u>입니다.

이렇게 똑똑하게 작동하는 포인터 객체를 **스마트 포인터**라고 하며 C++11에서 부터 스마트 포인터를 지원하게 됐습니다. C++11 이전에도 `auto_ptr` 이라는 스마트 포인터가 존재했지만, [문제가 많아](https://stackoverflow.com/questions/3697686/why-is-auto-ptr-being-deprecated) C++11로 개정되면서 `memory` 헤더에 정의된 `unique_ptr`, `shared_ptr`, `weak_ptr`로 개편되어 이제 사용하지 않습니다.

> `auto_ptr` 키워드는 C++11 에서는 deprecated 되었고, C++17에서는 removed 되었습니다.

그렇다면 스마트 포인터를 사용하면 정말로 문제가 해결되는지 확인해보겠습니다.

```cpp
#include <iostream>
#include <memory>

class Socket
{
private:
	int* mPData;

public:
	Socket()
	{
		mPData = new int[100];
		std::cout << "생성자 호출" << std::endl;
	}

	~Socket()
	{
		delete[] mPData;
		std::cout << "소멸자 호출" << std::endl;
	}

	void Something()
	{
		std::cout << "소켓이 무언가를 하는 중..." << std::endl;
	}
};

inline void Thrower()
{
	// 예외 발생
	throw 1;
}

inline void DoSomething()
{
	std::unique_ptr<Socket> uPtrSocket(new Socket());
	uPtrSocket->Something();

	Thrower();

	// ...
}

int main()
{
	try
	{
		DoSomething();
	}
	catch (int i)
	{
		std::cout << "예외 상황 발생" << std::endl;
	}
}
```

```plaintext
[출력 결과]

생성자 호출
소켓이 무언가를 하는 중...
소멸자 호출
예외 상황 발생
```

`Thrower` 함수에 의해 스코프를 벗어나자 스마트 포인터가 스택에서 해제되면서 소멸자를 호출하는 것을 확인할 수 있습니다.

# 참고

[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170)

[TCP School](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[모두의 코드](https://modoocode.com/229)

[Effective Modern C++](https://ebook.insightbook.co.kr/book/117)