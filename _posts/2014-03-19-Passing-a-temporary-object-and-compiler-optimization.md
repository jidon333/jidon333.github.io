---
layout: post
title: Passing a temporary object and compiler optimization
categories: C_CPP
comments: true
language: ko
---

임시객체를 매개변수로 전달하는 여러 가지 경우에 대해서 생각해보다가 신기한 사실을 알게 되었다.

우선 아래의 코드는 
임시객체를 전달 받을 때 생성자와, 소멸자가 어떻게 호출이 되는지
알아보기 위해서 만든 코드이다.


``` cpp
class Test{
 
private:
    char c;
public:
    Test(char _c)
    {
        c = _c;
        cout << c <<" 생성자호출"<< endl;
    }
 
    Test(const Test &p)
    {
        c = p.c;
 
        cout << c << "복사생성자 호출" << endl;
    }
 
    ~Test()
    {
        cout << c << "소멸자 호출" << endl;
    }
};
 
 
Test funcOfRef(const Test &t1,<wbr /> const Test &t2)
{
    Test temp{'t'};
    cout << "-------------<wbr />funcOfRef 호출됨-----------" << <wbr />endl;
 
    return temp;
}
 
Test funcOfVal(const Test t1, <wbr />const Test t2)
{
    Test temp{ 't' };
    cout << "-------------<wbr />funcOfVal 호출됨-----------" << <wbr />endl;
 
    return temp;
}
 
int main()
{
 
    cout << " < call by ref 로 <wbr />임시객체 전달 >" << endl;
    funcOfRef(Test{ 'a' }, Test{ 'b' });
    cout << "-------------<wbr />funcOfRef 종료-----------" << <wbr />endl;
 
    cout << endl << endl;
 
    cout << "<call by Val 로 임시객체 전달 >" << endl;
    funcOfVal(Test{ 'a' }, Test{ 'b' });
    cout << "-------------funcOfVal 종료-----------" << <wbr />endl;
 
 
}

```

위 코드에서 임시객체를 전달 받을때의 함수로
value를 복사하여 받는 방식과,
참조의 형태로 받아 복사가 일어나지 않도록 하는 방식으로
두 개의 함수를 만들었다.

```cpp
 //참조자의 형태로 전달
Test funcOfRef(const Test &t1, const Test &t2);
```
```cpp
// value 복사의 형태로 전달  
Test funcOfVal(const Test t1, const Test t2);
```

그리고 각각의 함수에 임시객체를 전달한 뒤 객체의 생성, 삭제 과정을 살펴보았는데
예상하지 못한 일이 일어났다.

![출력결과](/assets/img/PassingTempObject/res0.png)


내 예상은 다음과 같았다.

 **A: 먼저 첫 번째 참조자의 의해 전달받은 경우**

1. 우선 임시 객체가 생성된 후 (a,b객체 생성)
2. 함수 전달인자를 래퍼런스 형태로 전달받으니 복사는 일어나지 않는다.
3. 함수 내부 코드가 실행되고 함수가 종료될 때
4. 전달인자로 생성했던 임시 객체들이 소멸될 것이다.

A의 경우에는 나의 예상이 맞았다.

**B: value 복사본에 의해 전달받은 경우**

1. 우선 임시 객체가 생성된 후 (a,b객체 생성)
2. call by value의 형태로 인자를 전달 받으니 생성된 임시 객체가 또다시 복사 되어 함수 내부로 들어온다.
3. 함수 내부 코드가 실행되고 함수가 종료 될 때
4. 복사되어 들어왔던 복사본a,b객체가 소멸되고
5. 생성했던 임시객체 a,b 가 소멸될 것이다.

B의 경우에는 위와 같은 예상이 빗나가고 A의 흐름과 비슷해 보였다.

그래서 아? '임시객체' 로 전달할 때에는

```cpp
//참조자의 형태로 전달
- Test funcOfRef(const Test &t1, const Test &t2);
```
```cpp
// value복사의 형태로 전달
Test funcOfVal(const Test t1, const Test t2);     
```  

두 개의 형태가 똑같이 생각되는구나? 라고 결정을 내릴 뻔 하였는데 출력창을 자세히 살펴보았더니 객체 소멸 과정이 약간 달라진 것을 볼 수 있었다.

funcOfVal 함수가 소멸될 때에는

함수 내부에서 할당된 t가 소멸되는 시간은 매개변수로 넘긴 a, b의 소멸 이후였던 것이다!

분명히 출력창을 확인 했을때에는 a,b의 복사생성자는 호출되지 않았다.

그렇다면 복사가 된 것 같기는 한데 도대체 무슨 일이 있어난 것일까????

여기 저기 찾다가 이유를 알아냈다. 

<u>바로 컴파일러 최적화 때문이었다</u>


**B: value 복사본에 의해 전달받은 경우**

1. 우선 임시 객체가 생성된 후 (a,b객체 생성)
2. call by value의 형태로 인자를 전달 받으니 생성된 임시 객체가 또다시 복사 되어 함수 내부로 들어온다.

   
 ---> 라고 생각했지만 사실 컴파일러는 이렇게까지 멍청하지 않았다.
```cpp

 funcOfVal(Test{ 'a' }, Test{ 'b' }); 
```
  
임시 객체를 전달 하는 순간에 컴파일러는
어차피 다시 사용되지 않을 임시 객체를 만들고 그것을 또 복사 하는것이 아니라,
애초에 함수 내부에 임시객체를 생성하여 내부 스텍공간에 저장되도록 최적화했던 것이었다.
 
당연히 임시객체의 생존 기간 또한 함수호출~함수리턴까지이다.

