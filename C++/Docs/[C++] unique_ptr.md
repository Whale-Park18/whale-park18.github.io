> 본 글은 `std::unique_ptr`의 소개와 간단한 사용법을 중심으로 설명합니다.
> 
> `std::unique_ptr` 클래스가 갖는 멤버 변수나 함수에 대한 자세한 내용은 시간이 되면 나중에 작성하겠습니다.

# 설명

## `std::unique_ptr`란?

`std::unique_ptr`는 <u>가리키는 자원 객체에 대한 유일한 소유권을 갖는 독점 소유권(exclusive ownership) 의미론을 체현한 스마트 포인터</u>입니다. `std::unique_ptr`은 생 포인터(raw pointer)와 같은 크기로 볼 수 있으며, 대부분의 연산(역참조를 비롯해서)에서 `std::unique_ptr`는 생 포인터와 동일한 명령을 실행할 수 있습니다. 즉, 메모리와 CPU 주기가 넉넉하지 않은 상황에서도 생 포인터가 충분히 작고 충분히 빠른 상황이라면 `std::unique_ptr`를 사용해도 비슷한 성능을 낼 수 있습니다.

`std::unique_ptr`는 <u>복사를 허용하지 않습니다(복사를 허용할 경우 같은 자원을 가리킬 수 있게 되면서 미정의 행동이 발생합니다). 하지만 소유권 이전은 허용합니다.</u> 즉, 자원 객체의 소유권을 집 명의처럼 복사할 수는 없지만 이전시킬 수 있으며 소유권을 이전 시키려면 `std::move` 함수를 사용해야 합니다. 소유권이 이전되면 원본 포인터 객체는 널을 가리키게되며 대상 포인터 객체가 자원 객체를 가리키게 됩니다.

`std::unique_ptr`는 자원 객체를 소유하고 있을 때만, 자원 객체를 해제할 수 있는 권한을 가집니다. 또한 포인터 객체는 생성할 때 커스텀 삭제자(custom deleter)를 사용하도록 지정할 수도 있습니다. 다만, 커스텀 삭제자를 지정하게 되면 커스텀 삭제자를 위한 함수 포인터가 필요하므로 `std::unique_ptr`의 크기가 1 워드에서 2 워드로 증가하게 됩니다.

`std::unique_ptr`는 개별 객체(`std::unique_ptr<T>`)와 배열(`std::unique_ptr<T[]>`) 형태 모두 지원합니다. `std::unique_ptr` API는 사용 대상에게 잘 맞는 형태로 설계되어 있어 개별 객체 형태는 색인 적용 연산(`operator[]`)를 제공하지 않으며, 배열 형태는 역 참조 연산자들(`operator*`와 `operator->`)제공하지 않습니다.

`std::unique_ptr`에서 배열 형태를 지원하지만 사용을 권장하지 않습니다. 배열 형태의 포인터 객체를 생성하기 보단 `std::array`나 `std::vector`, `std::string`을 사용하는 것이 거의 항상 더 나은 선택이기 때문입니다.

## 왜 엄격한 소유권을 지니는가?

`std::unique_ptr`가 이렇게 설계된 이유는 **double-free** 오류에 있습니다.

double-free 오류란 이미 해제한 자원 객체를 다시 해제하는 경우 발생하는 오류로 다음과 같은 상황에서 자주 발생합니다.

```cpp
int* ptr1 = new int();
int* ptr2 = ptr1;

// ...

// ptr1 사용 끝, 해제
delete ptr1;

// ..

// ptr2 사용 끝, 해제
delete ptr2;
```

동일한 자원 객체를 가리키는 포인터 `ptr1`, `ptr2`가 존재합니다. `ptr1`이 자원 객체를 다 사용해, 해제합니다. `ptr2`는 이 사실을 모르고 있다가 이미 소멸된 자원 객체를 다시 해제하는 불상하가 일어납니다. 이런 경우 메모리 오류가 발생해, 프로그램이 죽게됩니다.

위의 문제가 발생한 이유는 <u>동적 할당된 자원 객체의 소유권이 명확하지 않고 쉽게 접근할 수 있기 때문</u>입니다. 만약 처음 초기화된 포인터에게만 소유권을 갖고 있었다면 위와 같이 자원 객체를 두 번 해제하는 일은 발생하지 않았을 것입니다.

즉, `std::unique_ptr`는 소유권이 명확하지 않아 발생하는 오류에 의해 발생하는 미정의 행동에서 벗어나기 위해 독점 소유권 의미론을 체현한 스마트 포인터로 설계된 것입니다.

## 사용법

> `std::unique_ptr`를 사용하기 위해선 `<memory>` 헤더를 포함해야 합니다.

### 생성자를 이용한 생성 및 초기화

스마트 포인터는 모든 타입에 대한 자원 객체를 가리킬 수 있어야 하기 때문에 템플릿 클래스로 선언되어 있습니다. 즉, 스마트 포인터는 생성자를 통해 생성 및 초기화를 할 수 있습니다.

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string mName;
    int mAge;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    void doSomething()
    {
        // ...
    }
};

int main()
{
    std::unique_ptr<Person> uPtr(new Person("whale", 10));
    (*uPtr).doSomething();
    uPtr->doSomething();
}
```

위에서 언급했듯이 `std::unique_ptr`는 생 포인터와 동일하게 사용할 수 있게 설계되어 있어 생 포인터처럼 사용하면 됩니다.

또한 `std::unique_ptr`(와 `shared_ptr`)는 기본적으로 자원 객체의 해제는 `delete`를 통해서 일어나지만, 위에서 언급했듯이 커스텀 삭제자를 설정할 수 있습니다. 자원 객체의 해제 전, 로그를 기록하는 등의 작업이 필요할 때 사용합니다.

커스텀 삭제자를 설정하기 위해서는 포인터 객체를 선언할 때, `std::unique_ptr`의 두 번째 템플릿 형을 설정하고 생성자의 두 번째 인자로 함수 객체를 전달해야 합니다.

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string mName;
    int mAge;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    std::string GetName() { return mName; }
    int GetAge() { return mAge; }
};

void FuncDeleter(Person* pPerson)
{
    std::cout << "함수 커스텀 소멸자 호출" << std::endl;

    delete pPerson;
}

int main()
{
    auto lambdaDeleter = [](Person* pPerson) {
        std::cout << "람다 커스텀 삭제자 호출" << std::endl;

        delete pPerson;
    };

    std::unique_ptr<Person, decltype(lambdaDeleter)> uPtr1(new Person("whale", 10), lambdaDeleter);
    std::unique_ptr<Person, decltype(&FuncDeleter)> uPtr2(new Person("park18", 20), FuncDeleter);
}
```

```
[출력 결과]

생성자 호출
생성자 호출
함수 커스텀 소멸자 호출
소멸자 호출
람다 커스텀 삭제자 호출
소멸자 호출
```

`lambdaDeleter`와 `FuncDeleter`가 `Person`에 대한 커스텀 삭제자입니다. <u>모든 커스텀 삭제자 함수는 파괴할 자원 객체를 가리키는 포인터 하나를 받으며, 그 객체를 파괴하는데 필요한 일들을 수행</u>합니다. 지금 예제에서 삭제자는 별도의 일을 하고 있지는 않지만 로그를 남기거나 세이브 파일을 저장하는 등 삭제 전에 필요한 일을 수행하는 코드를 작성할 수 있습니다. 모든 일을 마친 후에는 인자로 넘겨받은 자원 객체를 반드시 해제해야 합니다.

삭제자 함수는 통상적인 함수를 작성해 전달하는 방법도 있지만 람다 표현식을 이용해 작성하는 것이 더 효율적입니다.

커스텀 삭제자를 사용하기 위해 `std::unique_ptr`의 두 번째 템플릿 형을 지정해야 하며 이를 위해 `decltype` 키워드를 사용합니다. `decltype` 키워드는 `auto` 키워드와 비슷한 개념으로 주어진 표현식의 타입을 컴파일러가 추론해 결정하도록 합니다.

최상위 기본 클래스로 하위 파생 클래스를 관리할 목적으로 포인터 객체를 생성하는 것이라면 삭제자가 인자로 전달받는 것이 기본 클래스 타입이기 때문에 기본 클래스의 소멸자를 가상화 시켜야합니다. 만약 기반 클래스의 소멸자를 가상화 시키지 않았다면 기본 클래스의 소멸자만 호출되어 미정의 행위가 발생하게됩니다.

```cpp
#include <iostream>
#include <memory>

class BaseClass
{
public:
    BaseClass()
    {
        std::cout << "(기본 클래스) 생성자 호출" << std::endl;
    }

    virtual ~BaseClass()
    {
        std::cout << "(기본 클래스) 소멸자 호출" << std::endl;
    }
};

class DerivedClass1 : public BaseClass
{
public:
    DerivedClass1()
    {
        std::cout << "(파생 클래스1) 생성자 호출" << std::endl;
    }

    ~DerivedClass1()
    {
        std::cout << "(파생 클래스1) 소멸자 호출" << std::endl;
    }
};

class DerivedClass2 : public BaseClass
{
public:
    DerivedClass2()
    {
        std::cout << "(파생 클래스2) 생성자 호출" << std::endl;
    }

    ~DerivedClass2()
    {
        std::cout << "(파생 클래스2) 소멸자 호출" << std::endl;
    }
};

int main()
{
    auto deleter = [](BaseClass* pBaseClass) {
        std::cout << "커스텀 삭제자 호출" << std::endl;

        delete pBaseClass;
    };

    std::unique_ptr<DerivedClass1, decltype(deleter)> uPtr1(new DerivedClass1, deleter);
    std::unique_ptr<DerivedClass2, decltype(deleter)> uPtr2(new DerivedClass2, deleter);
}
```

```
[출력 결과]

(기본 클래스) 생성자 호출
(파생 클래스1) 생성자 호출
(기본 클래스) 생성자 호출
(파생 클래스2) 생성자 호출
커스텀 삭제자 호출
(파생 클래스2) 소멸자 호출
(기본 클래스) 소멸자 호출
커스텀 삭제자 호출
(파생 클래스1) 소멸자 호출
(기본 클래스) 소멸자 호출
```

### `std::make_unique`를 사용한 생성 및 초기화

`std::unique_ptr`를 생성자로 초기화하기 위해선 반드시 주소값이 필요합니다. 만약 인자로 다른 포인터 객체가 가리키고 있는 자원 객체의 주소를 전달하여 초기화하는 것이 가능한지 의문이 생깁니다.

```cpp
#include <iostream>
#include <memory>

int main()
{
    int* pNumber = new int(5);
    std::unique_ptr<int> uPtr1(pNumber);
    std::unique_ptr<int> uPtr2(pNumber);
    std::unique_ptr<int> uPtr3;
    uPtr3.reset(pNumber);

    std::cout << "uPtr1: " << uPtr1.get() << std::endl;
    std::cout << "uPtr2: " << uPtr2.get() << std::endl;
    std::cout << "uPtr3: " << uPtr3.get() << std::endl;
}
```

```
[출력 결과]
uPtr1: 01450428
uPtr2: 01450428
uPtr3: 01450428

<런타임 오류>
```

놀랍게도 서로 다른 포인터 객체가 같은 자원 객체를 가리킬 수 있게 초기화하는 것이 가능합니다. 이렇게 되면 `std::unique_ptr`는 목적과 다르게 하나의 자원 객체를 여러 개의 포인터 객체가 소유하게 되며 double-free 문제까지 발생하게 됩니다.

> `std::unique_ptr`가 다른 자원 객체를 가리키고 싶게 하고 싶다면 `reset` 멤버 함수를 호출하면 됩니다.
>
> `std::unique_ptr`가 현재 가리키고 있는 자원 객체의 주소를 알고 싶으면 `get` 멤버 함수를 호출하면 됩니다.

이런 상황을 피하기 위해 C++14에서는 `std::make_unique` 함수를 제공합니다. `std::make_unique` 함수는 자원 객체의 생성자에게 전달할 인자들을 전달 받고 내부에서 `std::unique_ptr`를 생성해 반환하기 때문에 안전하게 포인터 객체를 생성할 수 있습니다.

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string mName;
    int mAge;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    void DoSomething()
    {
        // ...
    }
};

int main()
{
    std::unique_ptr<Person> uPtr = std::make_unique<Person>("whale", 10);
    uPtr->DoSomething();
}
```

이렇게 안전하게 포인터 객체를 생성해주는 `std::make_unique` 함수에도한 가지 단점이 존재합니다. 바로 커스텀 삭제자를 설정할 수 없다는 것입니다. 때문에 커스텀 삭제자를 지정해야 하는 경우에는 어쩔 수 없이 생성자를 통해 초기화해야 합니다.

> 같은 스마트 포인터인 `std::shared_ptr`의 경우에는 C++14가 아닌 C++11에 `std::make_shared` 함수가 존재합니다. 아마 C++11에는 이런 상황을 예상하지 못해 제외시킨 것 같습니다.

> C++11 환경에서 개발 중이라면 `std::make_unique`의 기본 버전을 직접 작성하는 것이 어렵지 않기 때문에 걱정하지 않아도 됩니다.
>
> ```cpp
> template<typename T, typename... Ts>
> std::unique_ptr<T> make_unique<Ts&... params>
> {
>     return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
> }
> ```
>
> 단, 직접 구현할 경우 `std::` 네임스페이스를 적성하지 않아야 합니다. 나중에 C++14로 업그레이드 했을 경우 표준 라이브러리와 이름 충돌이 발생하기 때문입니다.

### 복사

`std::unqie_ptr`은 복사할 수 없다고 설명했습니다. 그럼에도 불구하고 복사를 시도할 경우 어떻게 작동할까요?

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string mName;
    int mAge;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }
};

int main()
{
    std::unique_ptr<Person> uPtrOriginal = std::make_unique<Person>("whale", 10);
    std::unique_ptr<Person> uPtrCopy(uPtrOriginal);
}
```

```
<컴파일 오류>

심각도	코드	설명	프로젝트	파일	줄	비표시 오류(Suppression) 상태
오류	C2280	'std::unique_ptr<Person,std::default_delete<Person>>::unique_ptr(const std::unique_ptr<Person,std::default_delete<Person>> &)': 삭제된 함수를 참조하려고 합니다.
```

복사를 시도할 경우 위와 같은 오류가 발생하게 됩니다. 위 오류는 삭제된 함수를 사용하려 했을 때, 발생하는 것입니다.

삭제된 함수란 C++11에서 추가된 기능으로 프로그래머가 명시적으로 '이 함수는 사용하지 말 것!'을 표현한 것입니다. 혹시라도 삭제된 함수를 사용할 경우 위와 같이 컴파일 오류가 발생하게 됩니다.

`std::unique_ptr`의 경우 **유일하게 소유하는 엄격한 소유권** 때문에 복사 생성자와 대입 연산자(복사 기능 한정)를 *명시적으로 삭제*했습니다. 만약 복사 생성자나 대입 연산자를 사용할 수 있다면 유일한 소유권을 갖는 특성이 사라지게 되면서 `std::unique_ptr`는 존재 의의를 잃게 됩니다.

### `std::move` 함수를 사용한 소유권 이전

`std::unique_ptr`는 복사는 불가능 하지만 소유권을 이전할 수 있습니다. 다만, 소유권을 이전하기 위해선, `std::move` 함수를 사용해야 합니다.

```cpp
#include <iostream>
#include <memory>

int main()
{
    std::unique_ptr<int> uPtrOriginal = std::make_unique<int>(1);
    std::unique_ptr<int> uPtrTarget(nullptr);

    std::cout << "[Before] owner: uPtrOriginal" << std::endl;
    std::cout << "uPtrOriginal: " << uPtrOriginal.get() << ", uPtrTarget: " << uPtrTarget.get() << std::endl;
    std::cout << '\n';

    uPtrTarget = std::move(uPtrOriginal);

    std::cout << "[After] owner: uPtrTarget" << std::endl;
    std::cout << "uPtrOriginal: " << uPtrOriginal.get() << ", uPtrTarget: " << uPtrTarget.get() << std::endl;
}
```

```
[출력 결과]

[Before] owner: uPtrOriginal
uPtrOriginal: 00BF03F0, uPtrTarget: 00000000

[After] owner: uPtrTarget
uPtrOriginal: 00000000, uPtrTarget: 00BF03F0
```

`std::move` 함수를 사용해, `uPtrOriginal`의 소유권을 `uPtrTarget`에게 이전하였습니다. `uPtrOriginal`이 가리키던 주소는 널을 가리키게 되고 `uPtrTarget`은 `uPtrOriginal`이 가리키던 자원 객체를 가리키면서 소유권이 넘겨진 것을 확인할 수 있습니다.

> 소유권이 이전된 포인터 객체를 댕글리 포인터(dangling pointer)라고 하며 이를 재참조할 때, 런타임 오류가 발생합니다. 따라서 소유권 이전은 댕글링 포인터를 다시 참조하지 않겠다는 확신을 갖고 이동시켜야 합니다.

### 함수 인자로 전달하기

일반적으로 함수 인자를 전달하게 되면 복사가 발생하며 전달하게 됩니다. 하지만  `std::unique_ptr`은 위에서 설명했듯이 복사가 불가능합니다. 즉, `std::unique_ptr`은 함수 인자로 전달할 수 없다는 것입니다.

하지만 복사가 일어나지 않는 레퍼런스를 전달한다면 가능하지 않을까요?

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string mName;
    int mAge;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    std::string GetName() { return mName; }
    int GetAge() { return mAge; }
};

inline void PrintPersonalInfo(std::unique_ptr<Person>& uPtrReference)
{
    std::cout << "이름: " << uPtrReference->GetName() << std::endl;

    std::cout << "나이: " << uPtrReference->GetAge() << std::endl;
}

int main()
{
    std::unique_ptr<Person> uPtrOriginal = std::make_unique<Person>("whale", 10);
    PrintPersonalInfo(uPtrOriginal);
}
```

```
[출력 결과]

생성자 호출
이름: whale
나이: 10
소멸자 호출
```

포인터 객체를 레퍼런스로 전달할 경우 정상적으로 작동하는 것을 확인할 수 있습니다. `PrintPersonalInfo` 함수의 `uPtrPerson` 인자는 레퍼런스이기 때문에 함수가 종료되도 객체를 파괴되지 않고 `main` 함수가 종료되면서 파괴는 것을 확인할 수도 있습니다.

> 참고한 자료 중에서 레퍼런스이긴 하지만 유일하게 소유한다는 원칙을 벗어나기 때문에 문맥상 옳지 못하다고 의견을 내신 분도 있습니다.

> 위에선 값으로 전달하지 못한다고 했지만 정확하게 말하자면 불가능 한 것은 아닙니다. 소유권을 `get` 멤버 함수를 사용해 포인터 주소값을 전달하거나 `std::move` 함수를 사용해 소유권을 넘기는 것입니다. 하지만 소유권을 넘기게 된다면 함수가 끝날 때, 자원을 반환한다는 문제가 발생합니다.

### 컨테이너의 원소

C++는 `std::array`, `std::vector` 등의 다양한 STL을 지원하면서 C++로 프로그램을 개발하게 된다면 STL을 반드시 접하게 됩니다. 이 STL의 컨테이너의 원소로 `std::unique_ptr`를 저장할 수 있을까요?

기본적으로 `std::vector`의 `push_back` 멤버 함수 같이 컨테이너에 원소를 넣게 된다면 복사 과정이 발생하게 됩니다. 즉, `std::unique_ptr`는 컨테이너의 원소가 될 수 없다는 것을 의미합니다. 하지만 `std::move` 함수로 소유권을 이전 시킨다면 컨테이너의 원소로 넣을 수 있습니다.

다른 방법으로 `std::vecto`의 `emplace_back` 멤버 함수 같이 컨테이너 내부에서 생성하게 된다면 복사 과정이 발생하지 않기 때문에 컨테이너의 원소가 될 수 있습니다.

```cpp
#include <iostream>
#include <memory>
#include <vector>

class Person
{
private:
    std::string mName;
    int mAge;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    std::string GetName() { return mName; }
    int GetAge() { return mAge; }
};

void funcDeleter(Person* pPerson)
{
    std::cout << "함수 커스텀 소멸자 호출" << std::endl;

    delete pPerson;
}

int main()
{
    std::vector<std::unique_ptr<Person>> vec;

    std::unique_ptr<Person> uPtr = std::make_unique<Person>("whale", 10);

    vec.push_back(std::move(uPtr));
    vec.emplace_back(new Person("park18", 20));

    for (auto& item : vec)
    {
        std::cout << "name: " << item->GetName() << ", age: " << item->GetAge() << std::endl;
    }
}
```

범위 기반 for문은 기본적으로 복사된 원소에 접근하는 것이기 때문에 사용하려면 레퍼런스로 지정해야합니다. 일반 for문은 인덱스를 이용해 직접 접근하는 것이기 때문에 평소처럼 사용하면 됩니다.

# 참고

[cppreference - en](https://en.cppreference.com/w/cpp/memory/unique_ptr)

[cppreference - ko](https://ko.cppreference.com/w/cpp/memory/unique_ptr)

[cplusplus.com](https://cplusplus.com/reference/memory/unique_ptr/?kw=unique_ptr)

[Runebook.dev](https://runebook.dev/ko/docs/cpp/memory/unique_ptr)

[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/how-to-create-and-use-unique-ptr-instances?view=msvc-170)

[TCP School](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[모두의 코드](https://modoocode.com/229)

[식빵맘(ansohxxn)](https://ansohxxn.github.io/cpp/chapter15-5/#unique_ptr%EC%9D%98-%ED%95%A8%EC%88%98%EB%93%A4)

[duragon.gitbooks.io](https://duragon.gitbooks.io/c-11/content/chapter8.html)

[gamdekong](https://gamdekong.tistory.com/88)

[코드없는 프로그래밍](https://www.youtube.com/watch?v=oNqm04uL3v8)

[포프TV](https://www.youtube.com/watch?v=MGVSPZoOchE)

[Effective Modern C++](https://ebook.insightbook.co.kr/book/117)