# 설명

C#이나 Java같은 프로그래밍 언어에는 가비지 컬렉터(**G**arbage **C**ollector:GC)라는 자원 청소기가 기본적으로 내장되어 있습니다. 이 가비지 컬렉터는 프로그램에서 더 이상 사용하지 않는 자원을 자동으로 해제하는 역할을 수행하기 때문에 프로그래머가 자원을 해제하는 것에 대한 신경을 쓰지 않아도 됩니다.

하지만 C++에는 이러한 개념이 존재하지 않아 프로그래머가 수동으로 메모리를 해제해야 합니다. 만약 사용한 자원을 해제하지 않으면 프로그램이 종료되기 전까지, 메모리에 계속 남아있는 **메모리 누수(Memory leak)**가 발생합니다.

'프로그래머가 신경 써서 메모리 해제만 했어도 일어나지 않았을 문제 아닌가?' 생각할 수 있습니다. 객관적으로 보자면 신경쓰지 못한 프로그래머의 잘못이 맞습니다. 하지만 우리는 사람이기 때문에 실수로 메모리 해제를 잊을 수도, 언어의 사양을 잘못 파악하거나 예상치 못한 곳에서 다른 흐름이 발생해 메모리가 해제되지 않는 실수를 할 수 밖에 없습니다.

예를 들어 소멸자를 이용해 메모리 해제하는 코드를 작성했지만 `thrower()` 함수로 발생한 예외로 인해, 흐름이 바뀌게 되어 메모리를 해제하지 못하고 넘어가 소멸자가 실행되지 못하는 경우가 있습니다.

```cpp
#include <iostream>

class Socket
{
private:
	int* p_data;

public:
	Socket()
	{
		p_data = new int[100];
		std::cout << "Get Resource" << std::endl;
	}

	~Socket()
	{
		delete[] p_data;
		std::cout << "Free Resource" << std::endl;
	}
};

inline void thrower()
{
	// 예외 발생
	throw 1;
}

inline void do_something()
{
	Socket* p_socket = new Socket();

	// ...

	thrower();

	// ...

	delete p_socket;
}

int main()
{
	try
	{
		do_something();
	}
	catch (int i)
	{
		std::cout << "Exception" << std::endl;
	}
}
```

```
[출력 결과]
Get Resource
Exception
```

이러한 문제를 해결하기 위해 C++에서 자원을 관리하는 방법으로 RAII(Resource Acquisition Is Initialization)라는 디자인 패턴을 권장합니다. 이는 자원 관리를 스택에 할당한 객체를 통해 수행하는 것입니다.

스택을 이용하는 이유는 스택에 할당된 객체는 스코프를 벗어나게 되면 자동으로 스택에서 제거되면서 소멸자가 호출되기 때문입니다. 예를 들어 위 예제에서 `p_socket`은 객체가 아니기 때문에 소멸자가 호출되지 않은 것입니다. 즉, `p_socket`을 일반적인 포인터가 아닌, 포인터 **객체로 만들어서 자신이 소멸될 때, 자신이 가리키고 있는 자원을 해제하면 되는 것입니다.**

이렇게 똑똑하게 작동하는 포인터 객체를 스마트 포인터라고 하며 C++11에서 언어 자체에서 스마트 포인터를 지원하게 됐습니다. C++11 이전에도 `auto_ptr` 이라는 스마트 포인터가 존재했지만, [문제가 많아](https://stackoverflow.com/questions/3697686/why-is-auto-ptr-being-deprecated) C++11로 개정되면서 `memory` 헤더에 정의된 `unique_ptr`, `shared_ptr`, `weak_ptr`로 개편되어 이제 사용하지 않습니다.

> `auto_ptr` 키워드는 C++11 에서는 deprecated 되었고, C++17에서는 removed 되었습니다.

그렇다면 스마트 포인터를 사용하면 정말로 문제가 해결되는지 확인해보겠습니다.

```cpp
#include <iostream>
#include <memory>

using namespace std;

class Socket
{
private:
	int* p_data;

public:
	Socket()
	{
		p_data = new int[100];
		std::cout << "Get Resource" << std::endl;
	}

	~Socket()
	{
		delete[] p_data;
		std::cout << "Free Resource" << std::endl;
	}

	void something()
	{
		cout << "Socket's something..." << endl;
	}
};

inline void thrower()
{
	// 예외 발생
	throw 1;
}

inline void do_something()
{
	unique_ptr<Socket> p_socket(new Socket());
	p_socket->something();

	thrower();

	// ...
}

int main()
{
	try
	{
		do_something();
	}
	catch (int i)
	{
		std::cout << "Exception" << std::endl;
	}
}
```

```plaintext
[출력 결과]
Get Resource
Socket's something...
Free Resource
Exception
```

`thrower`에 의해 스코프를 벗어나자 스마트 포인터가 스택에서 해제되면서 소멸자를 호출하는 것을 확인할 수 있습니다.

# 참고

[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170)

[TCP School](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[모두의 코드](https://modoocode.com/229)
