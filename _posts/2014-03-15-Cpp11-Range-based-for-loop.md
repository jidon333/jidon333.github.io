---
layout: post
title: 범위 기반 for loop
categories: C_CPP
comments: true
language: ko
---

C++11 에서는 다른 프로그래밍 언어에서는 foreach 루프라고 하는 것과 동일한 새로운 형태의 for루프를 도입하여 주어진 범위의 배열이나 모음의 각 요소를 반복하게 했다.
 
일반적인 문법은 다음과 같다
 
```cpp
for( decl : coll )
{
     // statement...
}
```
 
여기서 decl은 전달된 모음 coll의 각 요소에 대한 선언으로, 각 decl에 대해 명시된 명령문이 호출된다.
예를들어 아래 예는 전달된 초기화자 목록의 각 값에 대해서 명시한 명령문을 실행해 표준 출력 cout 으로 값을 출력한다.
 
```cpp
for (int i : {2, 5, 8, 99, 40, 21})
  cout << i << " ";
 
```
출력값 :  2 5 8 99 40 21

물론 일반적인 배열에도 가능하다

``` cpp
int num[5]{1, 2, 3, 4, 5};
 
//num이라는 배열을 돌면서 int형 데이터를 꺼내줘!
    for (int i : num)              
        cout << i << " ";
```

벡터 vec의 요소인 elem 각각에 1을 더하고 싶다면 다음과 같이 프로그램을 작성할 수 있다.

```cpp
std::vector<double> vec{ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
 
// for문의 내부에서 vec요소의 복사본을 가지고 동작하지 않도록 레퍼런스 사용
    for (double& elem : vec)   
        elem += 1; 
```

위 코드를 다시 해석해보자면 vec에 있는 double데이터를 하나씩 가져와라! 라고 해석할 수 있다.
 
그렇다면 벡터와같은 STL컨테이너 이외에 사용자가 정의한 데이터타입에도 사용할 수 있을까??

다음 예를 보자

```cpp
struct Test{   
    int a;
    char b;
    double c;
 
    static int cnt;
 
    Test()
    {
        a = cnt;
        b = (char)cnt;
        c = (double)cnt;
  cnt++;
    }
    
};
 
int Test::cnt = 1;
 
int main()
{
    Test test1[10];
 
  	// test1 배열을 돌면서 int형데이터(Test::a)를 꺼내고 싶어!!
    for (int i : test1) 
    {
        cout<<i<<endl;
    }
 
    
}
```

main ()문을 보면
test1이라는 이름의 10개짜리 배열을 생성후
각 배열의 int자료형 데이터에 접근하려고 한다.
 
하지만 컴파일 해보면 

> error C2440: '초기화 중' : 'Test'에서 'int'(으)로 변환할 수 없습니다.

이런 컴파일 에러가 발생한다.

​이유는 사실 for문 내부에서 Test타입을 int 타입으로 암시적 변환이 되어야하는데 vector과 같은 STL컨테이너들은 내부적으로 이러한 변환을 구현해 놓았지만 우리가 생성한 데이터 타입은 아무런 정의도 하지 않았으니 컴파일러가 Test를 int로 변환할때 어떤식으로 처리해야하는지 알 수 없어 생기는 문제이다.
 
이를 해결하기 위해서는 int형으로 암시적 변환이 일어날 때에 대해서 정의해주면 된다.
 
방법은 아래와 같다.

```cpp
struct Test{
    int a;
    char b;
    double c;
 
    static int cnt;
 
// ...
 
    operator int()
    {
        return a;
    }
    
}
```

10번째 줄의 operator int()를 살펴보자.

지금까지 알고 있던 연산자 오버로딩의 형태와는 모습이 조금 다르다.

operator는 연산자 오버로딩에 사용되는 키워드인데 리턴형태도 명시하지 않았는데, 당당하게 a를 리턴까지 하고 있다!

도대체 이게 뭘까?

MSDN을 뒤져보니

<http://msdn.microsoft.com/ko-kr/library/5s5sk305.aspx#fbid=daOU-EhT8Y8>

# 변환 함수(conversion operator)
라는 문법이 나온다.

간단하게 설명하자면
클래스 형식이 암묵적으로 다른 형태로 변환될때, 그 변환 형태를 정의하는 문법이다.

다음 예제에서는 Money 형식을 double 형식으로 변환하는 변환 함수를 지정한다.

```cpp
// spec1_conversion_functions1.cpp
    struct Money {
        operator double() { return _amount; }
 
    private:
        double _amount;
    };
 
    int main() {
        Money Account;
	// 객체 Account를 double형태로 변환하려고한다 operator double() 이 이때 호출됨
        double CashOnHand = Account;  
    }
```

```cpp
cout << (double)Account << endl;
```

변환함수가 무엇인지 알았으니 이제 Test객체에 범위 기반 for루프를 적용하는 예제를 실행해보자.

```cpp
struct Test{
    int a;
    char b;
    double c;
 
    static int cnt;
 
    Test()
    {
        a = cnt;
        b = (char)cnt;
        c = (double)cnt;
        cnt++;
    }
 
    operator int()
    {
        return a;
    }
    
};
 
int Test::cnt = 1;
 
int main()
{
    Test test1[10];
 
    for (int i : test1)
    {
        cout << i << endl;
    }
 
    
}
```

![출력결과](/assets/img/foreach_res.png)

 **\[ 출력 결과 ]**


