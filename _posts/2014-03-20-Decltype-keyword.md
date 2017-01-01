---
layout: post
title: Decltype 키워드
categories: C_CPP
comments: true
language: ko
---


decltype 키워드
새로 도입된 decltype 키워드를 사용하면 컴파일러가 표현식의 데이터형을 찾아낼 수 있다.



>> 문제: 서로 자료형이 다른 두 개의 인자를 받아 덧셈을 하여 그 결과를 되돌려 주는 함수를 작성하라

```cpp
template < class U, class V >
auto add(U u, V v) ->decltype(u + v)
{
    return u + v;    /// 서로 다른 자료형 두 개를 더했을때 어떤 자료형을 리턴하지?
                    ///decltype 키워드가 자료형을 판단해준다.
}
 
//ex)
decltype(13123); // type : int
decltype (123 + 1.2)A; //type :  double
decltype (string("안녕?")); // type :  string
 
int main()
{
 
    cout<<add('1', 3.14)<<endl; 
    cout << add(string("안녕"), string("하세요")) << endl;
    cout<<add(100, 0.1234)<<endl;
    
}
```

![res](/assets/img/decltype/res0.png)

함수의 리턴 타입을 함수의 선언 시에 결정할 수 없는 경우에 사용하면 좋다.


---
# 추가 내용

C++11에서는 새로운 함수 정의 문법 (alternative function syntax)를 도입하였다.
함수 정의 앞의 auto키워드는 함수의 정의가 새로운 문법을 따른다는 것을 의미한다.

```cpp
auto func(int i) -> int
{
    return i + 10;
}
```

리턴 타입은 화살표 -> 뒤에 지정하고, auto 키워드는 그냥 새로운 문법임을 알리는 키워드라고 생각하면 된다.
 
main() 함수 또한 새로운 문법 스타일로 정의할 수 있다.


```cpp
// 기존의 main문
int main()
{
    
}
 
// 새로운 문법의 함수정의로 만든 main문
auto main() -> int
{
    
}
```