---
layout: post
title: Move semantics and r-value reference
categories: C_CPP
comments: true
---

C++에서 도입된 새로운 특성 중 가장 중요한 것 중 하나로 Move Semantic을 손꼽을 수 있다.
 
이 특성은 C++의 큰 설계 목적인 '불필요한 복사와 임시 변수를 없애기' 를 더욱 멀리 밀고 나간 것이다.
 
나는 개인적으로 오늘 공부하면서 Call by Value, Call by Reference, Call by 이동생성자 라고 부르고 싶을 정도로
깊은 감명을 받았다.
 
그럼 지금부터 이동생성자(Move semantic)기법은 왜 필요하고 어떻게 사용되는지 알아보자.
 
우선 이동생성자를 설명하기에 앞서 이전 C++내용을 복습해 보겠다.


```cpp
#include <iostream>
#include <vector>
 
using namespace std;
 
class Test{
public:
    char *data;    
    char name;    
    int id;            
    
public:
    
    //생성자
    Test(char _name) 
    {
        //데이터 생성
        data = new char[1000000];
        id = cnt++;
        name = _name;
 
        cout << "id : " << id << " 생성자 호출 - 메모리의 주소 : " << (void*)data << endl;
        cout << "name : " << name << endl;
    }
    
    //복사생성자 (Deep Copy)
    Test(const Test &other) 
    {
        data = new char[1000000];
        memcpy(data, other.data, 1000000);
        id = cnt++;
        name = other.name;
 
        cout << "id : " << id << " 복사생성자 호출 - 메모리의 주소 : " << (void*)data << endl;
        cout << "name : " << name << endl;
    }
 
    //할당연산자
    Test& operator =(const Test &other)
    {
        if (this == &other)
            return *this;
        delete []date; // 기존 메모리 소멸
        data = new char[1000000];
        memcpy(data, other.data, 1000000);
        name = other.name;
 
        cout << "id : " << id << " 할당연산자 - 메모리의 주소 : " << (void*)data << endl;
        cout << "name : " << name << endl;
 
        return *this;
    }
 
    //소멸자
    ~Test()    
    { 
        cout << "id : " << id << " 소멸자 호출 - 메모리의 주소 : " << (void*)data << endl;
        cout << "name : " << name << endl;
        delete[]data;    
    }
 
    static int cnt;
};
 
int Test::cnt = 1;
 
int main()
{
    //Test 객체 a생성
    Test a{'a'};    
    cout << "------------" << endl;
 
    //Test 객체 b생성
    Test b{'b'};    
    cout << "-------------" << endl;
    
    //Test객체를 저장할 vector클래스 생성
    std::vector<Test> vec;
 
    //vec에 a 복사
    vec.push_back(a);
    cout << "-------------" << endl;
 
    //vec에 b 복사
    vec.push_back(b);
    cout << "-------------" << endl;
}

```

위 코드는 클래스 생성시 디폴트의 형태로 제공되는
생성자, 복사생성자, 소멸자, 할당연산자를 정의한 클래스를 나타낸 것이다.
 
객체는 생성시 data가 가르키는 공간에 1MB 만큼의 메모리를 할당하고,
객체a ,b 이렇게 두 개를 생성하였으며 각각을 vector클래스에 push_back하여 복사한 형태이다.
 
여기서 우리가 알고 싶은 것은 바로 메모리가 생성되고 삭제되는 과정이다.
 
먼저 출력결과를 보자!

![출력결과](/assets/img/MoveSemantics/res0.png)

결과값을 보면 알겠지만

먼저 a가 스텍영역에 생성된 후, data가 가르키는 공간(heap영역) 에 1MB만큼 메모리가 할당되었다.

b도 마찬가지로 스텍영역에 생성된 후 heap영역에 1MB만큼 공간을 할당하고, 그 후
 
```cpp
std::vector<Test> vec;
```

vec이 스택에 생성된 것을 알 수 있다.

![출력결과](/assets/img/MoveSemantics/note0.jpg)

```cpp
  vec.push_back(a);
```

이후 벡터클래스 vec에 a를 복사하는데
 
vec안 어딘가에 복사생성자가 호출되면서 a가 복사되고, 깊은복사(DeepCopy)가 일어나면서 

data가 가르키는 공간 또한 복사된 것을 볼 수 있다.

![출력결과](/assets/img/MoveSemantics/note1.jpg)

여기까지는 평범한 메모리의 생성, 복사 과정인데 이제부터 vec안에 b를 복사하려 할 때 문제가 발생한다.

![출력결과](/assets/img/MoveSemantics/res0.png)

출력 결과를 보면 a를 push\_back 하여 복사생성자가 호출된 이후에
b를 push_back 한 뒤에 보았더니

b가 복사되는것이 아니라 뜬금없이 a가 복사되고, 

다음에는 이전에 복사되었던 3번 a가  소멸된다.

그리고 난 후에야 b가 복사되는데 
이유는 바로 vector의 내부 구조의 비밀에 있다.
 
vector라는 컨테이너 클래스는 생성되면 사용자가 얼마만큼 자료를 넣을지 알 수 없으므로
처음에 외부로부터 데이터를 받을 공간을 0 만큼 생성한다.
 
그후 데이터가 하나 들어오면 공간이 1,

또 하나가 들어오면 2,

그다음에는 3

그다음에는 4

다음에는 6, 9, 13....

이런식으로 vector내부에서 사용자가 입력하는 양을 예상해서 주먹구구식으로 수용량을 늘려가기 때문이다.
 
그래서 b를 push\_back할 때 먼저 a, b두 개를 수용할 수 있는 공간을 할당하고,

a를 복사생성자를 이용해서 복사하고,

이전에 vec내부에 생성된 a를 소멸하고,

다시 b를 복사하는 것이다.

그림은 아래와 같다.

![출력결과](/assets/img/MoveSemantics/note2.jpg)
![출력결과](/assets/img/MoveSemantics/note3.jpg)

이제 지금까지의 메모리 생성 과정이 이해가 되었을 것이다.
 
위 과정을 보고 문제점이 무엇이라고 생각하는가?
 
프로그래머의 의도는 a,b두 개의 메모리를 생성한 뒤에
vector컨테이너에 담아서 관리하고 싶은데, 

그 과정에서 메모리가 한번, 두번,세번,네번, 다섯번이나 생성되어 불필요한 메모리 생성삭제 과정이 반복되었다는 것이다.
 
만일 vector에 집어넣을 데이터가 a,b두 개가 아니라 100개라고 생각해보자.
 
위에 과정대로라면 100개의 데이터를 저장하는데 일단 100번 복사를 할테고,

또 100번 복사하는 과정의 벡터 내부 메모리 할당 과정에서 복사 삭제를 반복하니까
처음에 미리 할당량을 정하지 않는 이상 생성/삭제를 300~400번이나 반복할 것이다.
 
아... 이럴수가 (  ~~미리 할당하면 상관없지만 그건 생각하지 말자..~~)

 
절망하지 마라 그대여
 
친절한 C++11에서는 
 
rvalue참조자라고 불리는 문법을 이용하여
이동생성자 ( Move Semantic)이라는 기법을 사용할 수 있다.
 
방법은 간단하다.
클래스내부에 이동생성자를 정의하면 된다.

```cpp
//이동생성자
    Test(Test &&other)
    {
        id = other.id;
        data = other.data;
        other.data = nullptr;
        cout << id << " 이동생성자 - 메모리의주소 : " << (void *)data << endl;
    }
```

Test클래스의 이동생성자는 이런 형태로 정의될 수 있겠는데 우선 이 코드를 살펴봄에 앞서 rvalue참조자와 임시객체에 대해 알아보자.

```cpp

	// 어떤 컨테이너
    container con;
    // X라는 객체
    X x;    
    
	 //x가 복사됨
    con.insert(x);

	// x+x라는 임시 rvalue값이 복사됨 
    con.insert(x + x); 
 
    //
    // 이제 더이상 이 지역에서 x를 쓸 일이 없다고 가정하자.
    //
 
	//X x는 더 이상 사용하지 않고 , con내부에서만 사용하고싶지만 복사됨
    con.insert(x);  

```

번째 줄과, 13번째 줄에서의 x삽입을 보면

호출자가 전달된 값 (x+x의 결과와 x)을 더이상 사용하지 않기 때문에

con이 내부적으로 복사본을 만들지 않고 원래 내용을 새로운 요소로 이동시킬 수 있다면 멋질 것이다.

특히 x의 복사 비용이 비쌀경우 ( Test의 예를 보아도 한번에 1MB절약) 성능이 어마어마하게 향상될 것이다.
 
C++11 은 이런 마법같은 기능을 지원하는데 그 때 호출되는 것이 바로 이동생성자이다!!!

```cpp
class X{
public:
    X(const X& lValue);    //복사 생성자
    X(X&& rValue);        //이동 생성자
};
```

이동 생성자는 다음과 같이 (타입&& rValue) 의 형태로 선언한다.

물론 이동 생성자의 내부는 Test내부에 정의한 것과 같이 

깊은복사(Deep Copy) 가 아닌 얕은복사 (Shallow Copy)를 하도록 하고,

원래 가르키던 포인터는 nullptr로 초기화하여 다시는 가르키지 못하도록 하여야 한다.

```cpp
//이동생성자
    Test(Test &&other)
    {
        id = other.id;
		// 얕은복사
  		data = other.data; 
 		// 이전의 포인터를 nullptr로 초기화         
        other.data = nullptr;       
        cout << id << " 이동생성자 - 메모리의주소 : " << (void *)data << endl;
    }
```

자 그럼 새롭게 정의한 이동생성자를 추가하여 이전의 예제를 실행해 보자.

![출력결과](/assets/img/MoveSemantics/res1.png)

아까 전과의 출력 결과와는 조금 다르지만 우리가 기대했던 것과는 조금 다른 결과가 나왔다.

살펴보자면
 
push_back(a)를 하는 과정에서 이동생성자가 호출되기를 바랬지만

복사생성자가 호출되었는데, 이는 사용자가 이동을 원하는지, 복사를 원하는지 모르기 때문에 당연한 결과이다.
 
그다음을 push_back(b)를 할때를 보면 아까와 조금 다르다.

이동생성자가 호출되고, vec안에있는 a가 소멸되었다.
 
지금 이 과정을 그림으로 설명해 보겠다.

![출력결과](/assets/img/MoveSemantics/note4.jpg)

우선 a를 push_back하는 과정까지는 이전과 같다.
 
이제 b를 push_back하기 위해 벡터 내부 수용량을 늘리는 과정에서 벡터 내부를 재구성할때
 
이전과는 다르게 이동생성자가 호출되어 벡터내부에 생성된 a의 메모리를 새로만든 a가 주소만 가르키게 된다.

![출력결과](/assets/img/MoveSemantics/note5.jpg)

벡터내부를 재 구성하는 과정에서는 벡터 스스로가 복사가 필요없고, 이동생성을 하면 더 효율적이라는 것을 알기 때문에
이동생성자가 정의된 경우 이동생성자를 호출하는 것이다!!
 
자 그럼 이제 다시 vector 안에 데이터를 100개를 집어넣는다고 가정해보자.
 
벡터 내부구조를 재구성할때는 이동생성자가 호출 되므로, 맨 처음 벡터에 push_back할때만 복사생성자가 호출되어
처음 100개 +  집어넣을떄 100개 하여 200개가 생성될 것이다.
 
아까와 비교하면 거의 두 배정도 향상된 성능을 보인다.
엄청난 성능향상이다!
하지만 아직 우린 배가 고프다.
 
vector에 처음 데이터를 넣을 때에도 이미 생성된 메모리를 복사하는것이니 
이동생성자가 호출되어 vector에서만 데이터를 관리하고 싶을수 있지 않은가?
 
이를 위해서는 우린 이 데이터를 더 이상 여기서 사용하지 않을꺼야 라는 뜻을 전해주어야 하는데

\<utility\>에 선언된 std::move()를 사용하면 그것이 가능하다.

std::move()는 전달된 인자를 rValue참조자라고 불리는 것으로 변경하여 이동생성자가 호출되게하는 일을한다.

rValue 참조자란 아까 이동생성자를 정의할 때 사용했던 Test &&와 같이 두 개의 &&로 선언된 데이터형을 말한다.
 
그럼이제 push_back을 할떄 move()를 사용하고 출력값을 확인해 보자.

```cpp
//vec에 a 복사
    vec.push_back(std::move(a));
    cout << "-------------------------------" << endl;
 
    //vec에 b 복사
    vec.push_back(std::move(b));
    cout << "-------------------------------" << endl;
```

![출력결과](/assets/img/MoveSemantics/res2.png)

와우! 브라보!!!

드디어 긴 여정의 끝이다.
 
a를 push_back할 때를 보면 바로 이동 생성자가 호출되어 처음 생성한 메모리 00280040을 그대로 사용하는 것이 보인다.
 
이후 벡터 내부를 재구성할 때에도 당연히 같은 메모리를 사용하고, b를 복사할떄에도 마찬가지이다.
 
이제 vector안에 데이터를 100개를 넣는다고 가정하면
처음 100개를 생성할 때 메모리를 생성하고, 그 이후로는 아무런 메모리 생성과정이 없는 것이다!!
 
와우.. 정말 놀라운 결과다.

이렇게 C++11의 새로운 문법인 rValue참조자를 이용한 이동 생성자만 호출하더라도, 이전의 C++11에서 짠 코드가
상당한 성능 향상이 가능하다.
 
이번 포스트를 통해 이동 생성자 문법의 파워를 실감하였고, 공부할 마음이 생겼다면, 여러분은 분명 훌륭한 독자이다.
