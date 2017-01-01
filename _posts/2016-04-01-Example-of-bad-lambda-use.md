---
layout: post
title: 람다(Lambda)를 잘못쓰기 쉬운 예
categories: C_CPP
comments: true
language: ko
---

```cpp
#include "stdafx.h"
#include <iostream>
#include <functional>
#include <vector>
using namespace std;
 
 
// int를 하나 받아 뭔가 조건을 검사하여 반환하는 필터링 함수가 있다고 가정하자.
// 필터링 함수들을 담을 컨테이너
vector<function<bool(int)>> Filters;
 
 
int main()
{
 
    // Do Something with Filter fucntions
    // ...
}
 
 
 
/**
* 예제 1
*/
void AddFilterA()
{
    int LocalCopiedValue = 1 + 2 + 3;
 
    Filters.emplace_back([=](int value)
    {
        return (value + LocalCopiedValue) > 0;
    });
}
 
/**
* 예제 2
* 다음 람다 식에서 만들어진 클로저를 호출하는 것은 어떤 문제가 있을까?
*/
void AddFilterDangerous()
{
    int LocalRefValue = 1 + 2 + 3;
 
    Filters.emplace_back([&](int value)
    {
        return (value + LocalRefValue) > 0;
    });
}
 
 
/**
* 예제 3
*/
 
class Widget
{
public:
    // ..
 
    void AddFilterMoreDangerous();
 
private:
    int MemberValue;
};
 
void Widget::AddFilterMoreDangerous()
{
    /**
    * 캡쳐모드가 Capture by Value[=]이므로 MemberValue가 람다가 생성하는 클로저 안으로 복사되어 
    * 매우 안전하고 닫힌 상태처럼 보인다.
    * 하지만 그것은 매우, 치명적이며, 위험한 생각이다!!!
    * 캡쳐블록은 오직 static이 아닌 지역 변수(매개변수 포함)에만 적용되기 때문!
    */
    Filters.emplace_back([=](int value)
    {
        return value + MemberValue > 0;
    });
 
    /// 그렇다면 어떻게 MemberValue를 람다 내에서 사용해도 컴파일 에러가 발생하지 않는 것일까??
    /// -> 사실은 다음과 같이 변환됨
 
    auto CurrentObjectPointer = this;
    Filters.emplace_back([CurrentObjectPointer](int value)
    {
        return (value + CurrentObjectPointer->MemberValue) > 0;
    });
 
    /// 만약 멤버변수를 캡쳐하고 싶다면 다음과 같이 하는 것이 안전하다.
 
    int CopyValue = MemberValue;
    Filters.emplace_back([CopyValue](int value)
    {
        return (value + CopyValue) > 0;
    });
}
 
 
void AddFilterAlsoDangerous()
{
    /// 정적 변수
    static int StaticValue = 10;
 
 
    /**
    * 무심코 [=]를 보고는 "아 이 람다는 자신이 사용하는 모든 객체의 복제본을 만들테니 완전히 닫혀있겠지?" 
    * 라고 오해하기 쉽지만 그렇지 않다. 
    * 람다는 static이 아닌 지역변수(매개변수 포함)에만 적용되기 때문!
    */
    
    Filters.emplace_back([=](int value)
    {
        return StaticValue + value > 0;
    });
 
    /// 정적 변수의 값을 수정한다!
    ++StaticValue; 
    
    /// 위에서 생성된 클로저를 실행할 때 StaticValue의 값은 무엇일까??
}
 
```