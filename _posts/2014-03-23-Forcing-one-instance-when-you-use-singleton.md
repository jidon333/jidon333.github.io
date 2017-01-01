---
layout: post
title: 싱글톤(singleton)으로 객체를 하나만 생성하도록 강요할 때 재밌는 방법 소개
categories: C_CPP
comments: true
language: ko
---

프로그래밍을 하다보면 오직 하나의 객체만 존재해야하는 경우가 생긴다. 
또는 하나의 객체만 생성하도록 강요해야하는 경우가 생기는데 
이 때 보통 싱글톤(singleton) 패턴을 사용한다.
 
싱글톤 패턴을 구현할 때 추가적인 생성을 막으려고 생성자를 private멤버로 구현하는데
여기에 재미있는 방법이 있다.

```cpp
const int DEFAULT_ACCOUNT_NUM = 1000;
 
class AccountManager{
 
private:
    // 계좌 관리 배열
    vector<Account> accArr;
    //개설된 계좌의 수
    int accountNumber;
 
private:
    
    // 디폴트 생성 방지 (일반적인 싱글톤구현에 사용하는 방법)
    AccountManager() : accountNumber(DEFAULT_ACCOUNT_NUM) {}
 
//생성자,할당자로 이 매니저 클래스의 복사 생성자를 호출하거나,
//할당 연산자를 호출하는 경우를 방지
// ( =연산자와, 복사생성자를 선언하는데 정의부분 대신 = delete를 넣어서 명시적표현을 함)
    AccountManager operator=(const AccountManager& other) = delete;
    AccountManager(const AccountManager& other) = delete;
public:
 
    // 싱글톤 GetInstance함수
    static AccountManager& GetInstance(){
        static AccountManager instance;
        return instance;
    }
```

19,20번 째 코드를 보면
함수 정의부분 대신 = delete;를 하고 있는데

이렇게 하면 일반적인 구현보다 더욱 명시적인 효과를 낼 수 있다.
 ```cpp
 AccountManager operator=(const AccountManager& other) = delete;
```
 
이런 표현은 AccountManager클래스의 = 연산자를 지워버리겠다는 뜻으로 해석된다.

그러므로 위의 코드처럼 private이 아닌 public에 저렇게 선언하여도

할당연산자를 호출하려 하면 컴파일 에러를 발생시킨다 ㅋ

