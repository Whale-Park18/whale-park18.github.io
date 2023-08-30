> 본 글은 `std::weak_ptr`의 소개와 간단한 사용법을 중심으로 설명합니다.
> 
> `std::weak_ptr` 클래스가 갖는 멤버 변수나 함수에 대한 자세한 내용은 시간이 되면 나중에 작성하겠습니다.

# 설명

## `std::weak_ptr`란?

![`std::weak_ptr 생성시, 제어 블록`](../../Resources/C++/Docs/weak_ptr/weak_ptr%20도식화.jpg)

`std::weak_ptr`는 `std::shared_ptr`에 의해 관리되는 자원 객체를 공유하지만 소유하지 않는 (`std::shared_ptr`의 약점(순환 참조)를 보강해주는)스마트 포인터입니다. 자원 객체를 소유하지 않기 때문에 `std::shared_ptr`로 관리되는 자원 객체를 가리켜도 참조 개수에 영향을 미치지 않습니다(대신 약한 개수(weak count)에 영향을 미칩니다).

`std::weak_ptr`는 이름처럼 **"약한"** 포인터(자원 객체와의 관계가)이기 때문에 `std::unique_ptr`나 `std::shared_ptr`처럼 자체적으로 포인터로서의 역할을 수행할 수 없습니다.

또한 `std::unique_ptr`나 `std::shared_ptr`는 자신이 자원 객체를 관리하기 때문에 자원 객체가 살아있는지, 소멸했는지 알 수 있지만 `std::weak_ptr`는 자원 객체를 가리킬 뿐, 관리하지 않기 때문에 자신이 가리킨 자원 객체가 살아있는지 소멸했는지 알 수 없습니다. 하지만 `std::weak_ptr`는 `std::shared_ptr`와 같은 구조(자원 객체 + 제어 블록)이기 때문에 제어 블록을 통해 자원 객체가 살아있는지 소멸했는지 판단할 수 있습니다.

## 사용법

> `std::weak_ptr`를 사용하기 위해선 `<memory>` 헤더를 포함해야 합니다.

### 생성자를 이용한 생성 및 초기화

`std::waek_ptr`는 위에서 언급했듯이 `std::shared_ptr`가 관리하는 자원 객체를 가리킵니다. 즉, 생성자, 복사 연산자, 복사 배정 연산자의 인자로 포인터 객체(`std::shared_ptr`, `std::weak_ptr`)를 넘겨주면 됩니다.

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string m_name;
    int m_age;

public:
    Person(std::string name, int age) : m_name(name), m_age(age)
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
    std::shared_ptr<Person> sPtr = std::make_shared<Person>("whale", 10);
    std::weak_ptr<Person> wPtr1(sPtr);
    std::weak_ptr<Person> wPtr2 = wPtr1;
    std::weak_ptr<Person> wPtr3;
    wPtr3 = wPtr2;

    std::cout << "sPtr 참조 개수: " << sPtr.use_count() << std::endl;
    std::cout << "wPtr1 참조 개수: " << wPtr1.use_count() << std::endl;
    std::cout << "wPtr2 참조 개수: " << wPtr2.use_count() << std::endl;
    std::cout << "wPtr3 참조 개수: " << wPtr3.use_count() << std::endl;
}
```

```
[출력 결과]

생성자 호출
sPtr 참조 개수: 1
wPtr1 참조 개수: 1
wPtr2 참조 개수: 1
wPtr3 참조 개수: 1
소멸자 호출
```

`std::weak_ptr`는 `std::shared_ptr`와 동일한 멤버 함수인 `use_count`를 갖고 있습니다. `std::weak_ptr`는 약한 개수에 관여하기 때문에 약한 개수의 수를 반환할 것 같지만 `std::shared_ptr`와 동일하게 참조 개수를 반환합니다.

### `std::weak_ptr`가 가리키는 자원 객체 사용하기

`std::weak_ptr`는 자원 객체를 가리킬 뿐, 역참조를 할 수 없습니다. 때문에 포인터 객체를 `std::shared_ptr`로 업그레이드해서 사용해야 합니다. 하지만 그 전에 반드시 현재 가리키고 있는 자원 객체가 소멸되었는지 확인해야 합니다.

이를 위한 멤버 함수 `expired`가 존재하지만 원자적으로 연산되지 않기 때문에 멀티 쓰레드 환경에서 잘못 사용할 경우 미정의 행동이 발생할 수 있습니다.

이런 문제를 해결할 수 있는 것이 `lock` 멤버 함수입니다 `lock` 멤버 함수는 원자적으로 연산되며 `std::weak_ptr`가 가리키는 자원 객체가 소멸했다면 아무 것도 가리키지 않는 `std::shared_ptr`를 반환하지만 존재한다면 `std::weak_ptr`와 동일한 자원 객체를 가리키는 `std::shared_ptr`를 반환합니다.

```cpp
#include <iostream>
#include <memory>

class Person
{
private:
    std::string m_name;
    int m_age;
    std::weak_ptr<Person> m_wPtrFirend;

public:
    Person(std::string name, int age) : m_name(name), m_age(age)
    {
        std::cout << "생성자 호출" << std::endl;
    }

    ~Person()
    {
        std::cout << "소멸자 호출" << std::endl;
    }

    void SetFirend(std::shared_ptr<Person>& sPtrFirend) { m_wPtrFirend = sPtrFirend; }

    const std::string GetName() { return m_name; }
    const int GetAge() { return m_age; }
    const std::weak_ptr<Person>& GetFirend() { return m_wPtrFirend; }

    void ResetFirend()
    {
        m_wPtrFirend.reset();
    }

    void AccessFirend()
    {
        auto sPtrFirend = m_wPtrFirend.lock();

        if (sPtrFirend)
        {
            std::cout << "접근 성공, firend 정보 { " << sPtrFirend->GetName() << ", " << sPtrFirend->GetAge() << " }";
        }
        else
        {
            std::cout << "접근 실패(참조한 객체가 소멸됨)" << '\n';
        }
    }
};

int main()
{
    std::shared_ptr<Person> sPtrWhale = std::make_shared<Person>("whale", 10);
    std::shared_ptr<Person> sPtrPark18 = std::make_shared<Person>("park18", 20);

    sPtrWhale->SetFirend(sPtrPark18);
    sPtrPark18->SetFirend(sPtrWhale);

    std::cout << "sPtrWhale 참조 개수: " << sPtrWhale.use_count() << '\n'
              << "sPtrPark18 참조 개수: " << sPtrPark18.use_count() << '\n';

    std::cout << std::boolalpha << "sPtrWhale의 firend 만료: " << sPtrWhale->GetFirend().expired() << '\n'
                                << "sPtrPark18의 firend 만료: " << sPtrPark18->GetFirend().expired() << '\n';

    sPtrWhale->AccessFirend();
    sPtrWhale.reset();

    sPtrPark18->AccessFirend();
}
```

```
[출력 결과]

생성자 호출
생성자 호출
sPtrWhale 참조 개수: 1
sPtrPark18 참조 개수: 1
sPtrWhale의 firend 만료: false
sPtrPark18의 firend 만료: false
접근 성공, firend 정보 { park18, 20 }소멸자 호출
접근 실패(참조한 객체가 소멸됨)
소멸자 호출
```

# 참고

[cppreference - en](https://en.cppreference.com/w/cpp/memory/weak_ptr)

[VisualC++ Docs](https://learn.microsoft.com/ko-kr/cpp/cpp/how-to-create-and-use-weak-ptr-instances?view=msvc-170)

[TCP School](https://www.tcpschool.com/cpp/cpp_template_smartPointer)

[모두의 코드](https://modoocode.com/252)

[식빵맘(ansohxxn)](https://ansohxxn.github.io/cpp/chapter15-7/)

[ozt88](https://ozt88.tistory.com/29?category=118910)

[Effective Modern C++](https://ebook.insightbook.co.kr/book/117)