---
layout: post
title: C++11 Initializer list(ko)
categories: C_CPP
comments: true
---


​C++ 에서는 지금까지 초기화를 위해서 (),=,{}등 여러 가지 방법이 사용되었다. 

이는 초보자에게 변수나 객체를 어떻게 초기화해야 하는지 혼동을 줄 수 있었는데 C++11부터는 초기화 방식을 통합했다.

이제 초기화를 할때에는 공통된 문법인 {}를 사용하면 된다.
 
```cpp
int values[] {1, 2, 3};
double db{ 1.34 };
std::vector<std::string> fruit{ "Banana", "Apple", "Orange" };
```
 
{}초기화자 목록은 초기화(Value Initialization)라고 불리는 작업을 시행한다.

지역변수라 하더라도 0으로, 포인터일 경우 nullptr로 초기화한다. (예전에는 정의되지 않은 초기화값을 가졌다)
 
```cpp
- int i;  //쓰레기값
- int j{}; //j는 0으로 초기화
- int *p; //쓰레기 주소값
- int *q{}; //q는 nullptr로 초기화
```
 
하지만 {} 안에서는 하향 초기화(정밀도를 줄이거나 명시한 값을 수정하는 것)을 할 수 없다.
 
```cpp
- int x1(1.2);   //문법상 문제 없지만 1.2 가 1이됨
- int x2 = 3.2;  //문법상 문제 없지만 3.2 가 2가 됨
- int x3{ 7.0 };  // 오류! 하향초기화
- int x4{ 3.4 };  // 오류! 하향초기화
- char c1{ 7 };  // ok 문제없음
- char c2{ 256 };  // 오류! 하향초기화
``` 

```cpp
 	std::vector<int> v1{ 1, 2, 3, 4 };
	int a = 10;
```

 
이처럼 C++11의 {}초기화자를 이용해 초기화를 하게 되면 , 실수로 하향초기화를 하는 일을 막을 수 있다.

또한 C++11에서는 사용자 정의 데이터형에 대한 초기화자 목록을 지원하기위해 std::initializer_list<> 라는 클래스 템플릿을 제공한다.

이 템플릿은 값들의 목록으로 초기화를 하려하거나, 목록만으로 어떠한 일을 처리하고 싶을때 언제든지 사용할 수 있다.

그리고 목록의 개수의 제한이 없기때문에 이전의 가변인자(...) 대신 사용해도 좋을 것 같다.


```cpp

void showData(std::initializer_list<char> _nums)
{
    //std::initializer_list<int>::iterator i 와 auto i는 같다.
    for (auto i = _nums.begin(); i < _nums.end(); i++)
    {
        std::cout << *i << endl;
    }
}

int main()
{
    showData( { 'A', 'B', 'C', 'D' } );
    
}

```

인자와 초기화자 목록 모두를 위한 생성자가 있다면, 초기화자 목록을 위한 생성자가 우선순위를 갖는다.

```cpp
 
class Test{
public:
    Test(int, int);
    Test(std::initializer_list<int>);
};


int main()
{
    Test a(1, 2);              //Test::Test(int,int) 호출
    Test b{ 1, 2 };            //Test::Test(initializer_list)호출
    Test c{ 1, 2, 3, 4, 5 };   //Test::Test(initializer_list)호출
    Test d = { 1, 2 };         //Test::Test(initializer_list)호출
    
}
 
```

```cpp
#include <iostream>
using namespace std;
 
class Test{
 
public:
    Test(int a, int b)
    {
        cout << "Test(int a, int b)" << endl;
    }
    Test(std::initializer_list<int> vals)
    {
        cout << "Test(std::initializer_list<int> vals)" << endl;
    }
    void showData(std::initializer_list<int> vals)
    {
        cout << "showData() " << endl;
        for (auto i = vals.begin(); i < vals.end(); ++i)
            cout << *i << " "<<endl;
    }
};
int main()
{
    Test test1(1, 2);            // Test(int a, int b) 호출됨
    Test test2{ 3, 4 };            // Test(std::initializer_list<int> vals) 호출
    Test show{1,2,3,4,5};        // Test(std::initializer_list<int> vals) 호출
 
    // showData()호출, 멤버이니셜라이저를 인자로 사용하라면
    // ( {} ) 괄호 안에 중괄호를 넣어주어야 한다.
    show.showData({ 1, 2, 3, 4, 5, }); 

```

![initializer_res](/assets/img/initializer_res.png)
![Thinking Cat](/assets/img/ThinkingCat.jpg)

