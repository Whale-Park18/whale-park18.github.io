> 본 글은 `std::shared_ptr`의 소개와 간단한 사용법을 중심으로 설명합니다.
> 
> `std::shared_ptr` 클래스가 갖는 멤버 변수나 함수에 대한 자세한 내용은 시간이 되면 나중에 작성하겠습니다.

# 설명

## `std::shared_ptr`란?

`std::shared_ptr`는 `std::unique_ptr`와 달리 <u>자원 객체에 대한 공유 소유권(shared ownership) 의미론을 체현한 스마트 포인터</u>입니다. `shared_ptr`는 하나의 자원 객체를 여러 포인터 객체가 가리킬 수 있으며 모든 포인터 객체가 자원 객체를 필요하지 않을 때 자원 객체를 해제하도록 설계되어 있습니다.

하지만 어느 시점에 모든 포인터 객체가 자원 객체를 필요하지 않은지 알 수 없습니다. `std::shared_ptr`는 자원 객체를 가리키는 포인터 객체의 수를 **참조 개수(reference count)**라 명명하고 관리합니다.  참조 개수는 생성자가 호출되면 증가, 소멸자가 호출되면 감소 그리고 복사 배정 연산자가 호출되면 증가와 감소 모두를 수행합니다.

이러한 참조 개수 관리는 성능에 많은 영향을 미치게 됩니다.

* **`std::shared_ptr`의 크기는 생 포인터의 두 배이다.**  
  자원 객체를 가리키는 생 포인터 뿐만 아니라 참조 개수를 가리키는 생 포인터도 저장해야 하기 때문에 <u>생 포인터의 두 배의 크기</u>를 가지게 됩니다.

* **참조 개수를 담는 메모리는 반드시 동적으로 할당해야 한다.**  
  개념적으로 참조 개수는 포인터 객체가 현재 자원 객체를 가리키는 포인터 객체의 수를 알기 위한 것이기 때문에 자원 객체가 현재 참조 개수를 담을 공간을 따로 마련하지 않습니다. 즉, 참조 개수는 `std::shared_ptr`가 관리해야 하며 정적 메모리로는 불가능하기 때문에 결국 동적 메모리를 이용해야 합니다. 만약 정적 메모리를 사용하면 참조 개수를 갱신하기 위해 같은 자원 객체를 가리키고 있는 모든 포인터 객체를 알아야합니다 즉, 참조 개수와 같은 자원 객체를 가리키는 모든 포인터 객체의 정보를 담은 공간이 필요하게 되므로 차라리 동적 메모리에 참조 개수를 저장해 같은 자원 객체를 가리키는 포인터 객체들이 접근하는 것이 이상적입니다.

* **참조 횟수의 증가와 감소가 반드시 원자적 연산이어야 한다.**  
  여러 스레드가 참조 개수에 동시 읽고 쓰려고 한다면 **데이터 레이스**가 발생할 수 있습니다. 이를 방지하기 위해 참조 개수는 원자적으로 연산됩니다. 그러다보니 비원자적 연산보다 느릴 수 밖에 없습니다.

`std::shared_ptr`는 `std::unique_ptr`과 다르게 오직 개별 객체(`std::shared_ptr<T>`)만을 지원합니다. '`std::shared_ptr<T>`로 가리키되, 커스텀 삭제자로 배열 해제(`delete[]`)하면 되지 않나?' 생각할 수 있지만 `std::shared_ptr`에서 `operator[]`를 제공하지 않기 때문만 아니라 설계 밖의 일이기 때문에 시스템에 구멍이 생기게 됩니다.

### Control Block

`std::shared_ptr`는 생 포인터의 두 배의 크기이고 그 중 하나는 참조 개수를 저장하기 위한 것이라 설명했습니다. 정확히 설명하자면 참조 개수는 제어 블록(Control Block)이라 불리는 더 큰 자료구조에 저장되는 하나의 목록이고 `std::shared_ptr`는 이 제어 블록을 가리킵니다. 이 제어 목록에는 참조 개수뿐만 아니라 커스텀 삭제자, 약한 개수(weak count) 등이 저장됩니다. 이러한 구현 방법이 제일 이상적이라 생각됩니다. 만약 각 항목마다 별도로 관리하게 된다면 `std::shared_ptr`의 크기는 계속 커지기만 할 것입니다. 하지만 제어 블록에서 모두 관리하기 때문에 `std::unique_ptr`처럼 커스텀 삭제자를 지정해도 `std::shared_ptr`의 크기가 변하지 않습니다.

![std::shared_ptr 도식화](https://learn.microsoft.com/ko-kr/cpp/cpp/media/shared_ptr.png?view=msvc-170)

## 사용법

> `std::shared_ptr`를 사용하기 위해선 `<memory>` 헤더를 포함해야 합니다.

## 생성자를 이용한 생성 및 초기화

* 자원 객체 초기화
* 소멸자 초기화

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
    std::shared_ptr<Person> sPtr1(new Person("whale", 10));
    std::shared_ptr<Person> sPtr2(sPtr1);
    std::shared_ptr<Person> sPtr3 = sPtr2;

    (*sPtr1).doSomething();
    sPtr2->doSomething();

    std::cout << "sPtr1 참조 개수: " << sPtr1.use_count() << std::endl;
    std::cout << "sPtr2 참조 개수: " << sPtr2.use_count() << std::endl;
    std::cout << "sPtr3 참조 개수: " << sPtr3.use_count() << std::endl;
}
```

```
[출력 결과]

생성자 호출
sPtr1 참조 개수: 3
sPtr2 참조 개수: 3
sPtr3 참조 개수: 3
소멸자 호출
```

`std::shared_ptr`도 `std::unique_ptr`처럼 생 포인터와 동일하게 사용할 수 있게 설계되어 있어 생 포인터처럼 사용하면 됩니다.

`std::shared_ptr`도 기본적으로 `delete`를 통해서 자원 객체를 해제하지만, 커스텀 삭제자를 설정할 수 있습니다. 하지만 커스텀 삭제자를 지원하는 구체적인 방식은 `std::unique_ptr`과 다릅니다. `std::unique_ptr`는 커스텀 삭제자를 템플릿의 두 번째 형으로 설정하고 생성자의 두 번째 인자로 전달해야 했지만 `std::shared_ptr`에서는 생성자의 두 번째 인자로 전달하기만 하면 됩니다.

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

    std::shared_ptr<Person> sPtr1(new Person("whale", 10), lambdaDeleter);
    std::shared_ptr<Person> sPtr2(new Person("park18", 10), FuncDeleter);
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

## `std::make_shared`를 사용한 생성 및 초기화

`std::shared_ptr`도 `std::unique_ptr`과 동일한 이유로 생성자(정확히는 생 포인터를 넘겨 받는 생성자)를 이용한 초기화 방법을 권장하지 않습니다. `std::shared_ptr`는 생 포인터를 인자로 넘겨 받는다면 첫 번째 생성으로 판단하고 제어 블록을 생성합니다. 즉, 서로 다른 포인터 객체의 생성자가 같은 자원 객체를 가리키는 생 포인터를 넘겨받는다면 서로 다른 제어 블록을 갖게 되는 문제가 발생하게 됩니다.

두 포인터 객체가 같은 자원 객체를 가리키지만 서로 다른 제어 블록을 갖게 된다면 두 포인터 객체 중 한 포인터 객체의 참조 개수가 0이 되어 남은 포인터 객체가 아직 자원 객체를 가리키고 있음에도 불구하고 자원 객체를 해제시켜 버립니다.

만약 운 좋게 남은 포인터 객체에서 자원 객체를 참조하지 않았어도 결국 포인터 객체의 참조 개수가 0이 되어 이미 해제된 자원 객체를 다시 해제시켜 오류를 발생시키게 됩니다.

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
    auto ptrPerson = new Person("whale", 10);

    std::shared_ptr<Person> sPtr1(ptrPerson);
    std::shared_ptr<Person> sPtr2(ptrPerson);

    std::cout << "sPtr1 참조 개수: " << sPtr1.use_count() << std::endl; 
    std::cout << "sPtr2 참조 개수: " << sPtr2.use_count() << std::endl; 
}
```

```
[출력 결과]

생성자 호출
sPtr1 참조 개수: 1
sPtr2 참조 개수: 1
소멸자 호출
소멸자 호출

<런타임 오류 발생>
```

이 같은 상황을 피하기 위해 포인터를 인자로 넘겨받는 생성자 초기화 방식을 지양하고 `std::make_shared` 함수의 사용을 지향해 안전하게 포인터 객체를 생성해야 합니다. 한 가지 주의점이 있는데 `std::make_shared` 함수는 항상 제어 블록을 생성하기 때문에 처음 포인터 객체를 생성하는 경우에만 사용해야 합니다.

> `std::make_shared` 함수는 이 같은 상황을 예견?했던 것인지 모르겠지만 C++11부터 지원했습니다.

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
    std::shared_ptr<Person> sPtr = std::make_shared<Person>("whale", 10);
    sPtr->DoSomething();
}
```

하지만 `std::make_shared` 함수 또한 `std::make_unique` 함수처럼 커스텀 삭제자를 지정할 수 없다는 단점이 존재합니다. 

## 주의점

### 자기 자신을 가리키는 경우

프로그램을 개발하다보면 객체 내부에서 자기 자신을 가리키는 포인터 객체를 만들어야 하는 상황이 발생하기도 앖니다. 자기 자신을 가리키기 위해 `std::shared_ptr`의 생성자에 `this`를 전달하게 된다면 다음과 같은 상황이 발생합니다.

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

    std::shared_ptr<Person> GetSelf() 
    {
        return std::shared_ptr<Person>(this);
    }
};

int main()
{
    std::shared_ptr<Person> sPtr1 = std::make_shared<Person>("whale", 10);
    std::shared_ptr<Person> sPtr2 = sPtr1->GetSelf();

    std::cout << "sPtr1 참조 개수: " << sPtr1.use_count() << std::endl;
    std::cout << "sPtr2 참조 개수: " << sPtr2.use_count() << std::endl;
}
```

```
[출력 결과]

생성자 호출
sPtr1 참조 개수: 1
sPtr2 참조 개수: 1
소멸자 호출

<런타임 오류 발생>
```

`this` 키워드 역시 생 포인터를 전달하는 것이기 때문에 두 포인터 객체는 서로 다른 제어 블록을 가지게 되면서 미정의 행위를 발생시키게 됩니다(예제에서는 간단하게 설명하기 위해 자신을 가리키는 포인터 객체를 생성해 반환했지만 벡터에 자신을 가리키는 포인터를 넣는 등의 상황도 있습니다).

![같은 자원 객체를 가리키지만 다른 제어 블록을 갖는 모습](../../Resources/C++/Docs/shared_ptr/같은%20자원%20객체를%20가리키지만%20다른%20제어%20블록을%20갖는%20모습.jpg)

이 문제는 `std::enable_shared_from_this`라는 기반 클래스 템플릿을 상속 받고 자신을 가리키는 포인터 객체를 생성해 반환하는 대신 `shared_from_this` 멤버 함수를 반환하는 것으로 간단하게 해결할 수있습니다.

한 가지 중요한 점은 `shared_from_this` 멤버 함수가 제대로 작동하기 위해선 해당 객체를 가리키고 있는 포인터 객체를 반드시 먼저 정의해야 한다는 것입니다. 왜냐하면 `shared_from_this` 멤버 함수는 해당 객체를 가리키고 있는 제어 블록을 확인할 뿐, 제어 블록을 생성하지 않기 때문입니다.

```cpp
#include <iostream>
#include <memory>

class Person : public std::enable_shared_from_this<Person>
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

    std::shared_ptr<Person> GetSelf() 
    {
        return shared_from_this();
    }
};

int main()
{
    std::shared_ptr<Person> sPtr1 = std::make_shared<Person>("whale", 10);
    std::shared_ptr<Person> sPtr2 = sPtr1->GetSelf();

    std::cout << "sPtr1 참조 개수: " << sPtr1.use_count() << std::endl;
    std::cout << "sPtr2 참조 개수: " << sPtr2.use_count() << std::endl;
}
```

```
[출력 결과]

생성자 호출
sPtr1 참조 개수: 2
sPtr2 참조 개수: 2
소멸자 호출
```

### 순환 참조

`std::shared_ptr`은 참조 개수가 0이 되어야 자원을 해제할 수 있습니다. 하지만 객체를 더 이상 사용하지 않음에도 불구하고 참조 개수가 0으로 떨어지지 않는 상황이 존재합니다.

포인터 객체 `sPtr1`과 `sPtr2`가 있을 때, 자원 객체의 내부에서 서로를 가리키는 포인터 객체가 있다면 서로를 가리키는 포인터 객체 때문에 참조 개수가 0으로 떨어지지않아 포인터 객체가 소멸되지 않는 상황으로 이런 구조를 **순환 구조**라고 합니다.

![순환 참조](../../Resources/C++/Docs/shared_ptr/순환%20참조.jpg)

```cpp
#include <iostream>
#include <memory>

class Person : public std::enable_shared_from_this<Person>
{
private:
    std::string mName;
    int mAge;

    std::shared_ptr<Person> mSPtrFriend;

public:
    Person(std::string name, int age) : mName(name), mAge(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    std::shared_ptr<Person> GetSelf() 
    {
        return shared_from_this();
    }

    void SetFriend(std::shared_ptr<Person>& sPtrFriend)
    {
        mSPtrFriend = sPtrFriend;
    }
};

int main()
{
    std::shared_ptr<Person> sPtrPerson1 = std::make_shared<Person>("whale", 10);
    std::shared_ptr<Person> sPtrPerson2 = std::make_shared<Person>("park18", 20);

    sPtrPerson1->SetFriend(sPtrPerson2);
    sPtrPerson2->SetFriend(sPtrPerson1);
}
```

```
[출력 결과]

생성자 호출
생성자 호출
```

출력 결과를 보면 서로 파괴할 수 없는 상태가 되어 소멸자가 호출되지 않는 것을 확인할 수 있습니다.

이 문제는 `std::shared_ptr`의 설계 한계에 의한 문제이기 때문에 `std::shared_ptr`을 통해서 이 문제를 해결할 수는 없습니다. 대신 이 문제를 해결하기 위해 설계된 것이 `std::weak_ptr`입니다.

다음에는 `std::weak_ptr`에 대해 알아보겠습니다.

# 참조

[cppreference - en](https://en.cppreference.com/w/cpp/memory/shared_ptr)

[cppreference - ko](https://ko.cppreference.com/w/cpp/memory/shared_ptr)

[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/how-to-create-and-use-shared-ptr-instances?view=msvc-170)

[TCP School](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[모두의 코드](https://modoocode.com/252)

[식빵맘(ansohxxn)](https://ansohxxn.github.io/cpp/chapter15-6/)

[ozt88](https://ozt88.tistory.com/28)

[Effective Modern C++](https://ebook.insightbook.co.kr/book/117)