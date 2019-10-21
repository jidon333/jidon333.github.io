---
layout: post
title: 유저 모드에서의 스레드 동기화
categories: OS
comments: true
language: ko
---


### 목표
- 원자적 접근: interlocked 함수들
- 캐시 라인
- 고급 스레드 동기화 기법
- 크리티컬 섹션
- 슬림 리더-라이터 락
- 조건변수 


시스템에서 수행되는 모든 스레드는 수많은 리소스에 접근하게 된다. 다수의 스레드가 동일한 리소스를 사용해야 하는 경우 반드시 동기화가 필요하다.

다음 두 가지의 기본적인 상황에서 스레드는 상호 통신을 수행해야 한다.
- 다수의 스레드가 공유 리소스에 접근해야 하며, 리소스가 손상되지 않도록 해야 하는 경우
- 어떤 스레드가 하나 혹은 다수의 다른 스레드에게 작업이 완료되었음을 알려야 하는 경우.

#### 1. 원자적 접근: interlocked 함수들

스레드 동기화를 위해서는 리소스에 원자적으로 접근해야 한다.

```
(공유 자원에 대해 두 개의 스레드가 접근해서 읽고/쓰기를 하는 코드)
(멀티 프로세서에서 원하는 결과가 나오지 않는다!)
```

이런 경우에 윈도우에서 제공하는 interlocked 함수들을 사용할 수 있다.

```
LONG InterlockedExchangeAdd(
  LONG volatile *Addend,
  LONG          Value
);

LONG64 InterlockedExchangeAdd64(
  LONG64 volatile *Addend,
  LONG64          Value
);

```

함수를 사용할 때 LONG 값을 저장하고 있는 변수의 주소와 얼마만큼 증가시킬지 인자로 전달하기만 하면 된다!!
- 인터락 함수의 동작 방식은 CPU 플렛폼마다 다르다
- 하지만 이런 함수들을 사용하면 그런 것들을 고려하지 않고도 변수의 값을 Atomic 하게 변경할 수 있다.
- 함수에 전달하는 주소 값은 align되어 있어야 한다.
- 매우 빠르다. (보통 50 CPU 사이클 미만이며 유저모드와 커널 모드의 전환도 발생시키지 않음)
- 값을 감소시키기 위해서는 매개변수로 음수값을 전달하면 된다.
- 스핀락(spinlock)을 구현해야 하는 경우 매우 유용하다.

스핀락(spinlock) 예시

```
// 공유 리소스의 사용 여부를 나타내는 전역변수
BOOL g_fResourceInUse = FALSE;

void Func1()
{
  // 리소스의 접근을 기다림. 접근 가능하면 TRUE
  while(InterlockedExchange(&g_fResourceInUse, TRUE) == TRUE)
    Sleep(0);

  // 리소스에 접근함
  ...

  // 리소스에 더 이상 접근할 필요가 없음.
  InterlockedExchange(&g_fResourceInUse, FALSE)

}
```

- 스핀락과 같은 기법은 CPU 시간을 많이 낭비시킬 수 있으므로 주의가 필요
- 하지만 크리티컬 섹션(뒤에서 설명할)과 같이 스레드가 대기 모드로 진입할 필요가 없으므로 빠르다.
- 불필요한 CPU 자원 낭비를 막기 위해 일정 횟수(대략 4000회) 정도만 스핀락을 시도하고 그동안 자원을 점유하는데 실패하면 그냥 스레드를 대기시키자.(크리티컬 섹션)
- 락 변수와 락으로 보호하고자 하는 데이터는 서로 다른 캐시라인에 있도록 하는 것이 좋다.(안그러면 락 변수 접근하느라 CPU끼리 경쟁해버리니까...)
- 리소스가 매우 짧은 시간 동안 사용될 것이라고 가정되는 경우에만 스핀락을 사용하도록 하자.


#### 2. 캐시 라인

CPU가 메모리로부터 값을 가져올 때는 바이트 단위로 가져오는 것이 아니라 캐시 라인을 가득 채울 만큼 충분한 양을 가져온다. 캐시 라인은 예전에는 32bit, 최근에는 64bit, 128bit 크기로 구성된다(CPU 마다 다름). 보통의 경우 어플리케이션은 인접한 바이트들을 주로 사용하는 경향이 있으므로 같은 캐시 라인에 있는 데이터를 사용할때 비교적 많은 시간을 소모하는 메모리 버스에 대해 CPU가 추가적으로 접근할 필요가 없다.

하지만 멀티프로세서 환경에서 다른 CPU가 이미 캐시 라인으로 가져왔던 데이터를 변경하면 CPU는 이미 가져왔던 캐시 라인 정보를 무효화 시켜야 한다.(안그러면 혼돈의 카오스)
이러한 특성 때문에 
- 어플리케이션이 사용하는 데이터는 캐시 라인의 크기와 그 경계 단위로 묶어서 다루는 것이 좋다.
- 읽기 전용과 데이터와 읽고 쓰는 데이터를 분리해서 관리하는 것이 좋다.
- 동일한 시간에 접근하는 데이터를 묶어서 구성하는 것이 좋다.

```
나쁜 예

struct CUSTINFO {
    DWORD       dwCustomerID        // 거의 읽기 전용으로 사용
    int         nBalanceDue;        // 읽고 쓰기용으로 사용
    wchar_t     szName[100];        // 거의 읽기 전용으로 사용
    FILETIME    ftLastOrderDate;    // 읽고 쓰기용으로 사용
};
```

```
좋은 예

#define CACHE_ALIGN 64
// 구조체의 인스턴스가 각기 다른 캐시 라인에 들어갈 수 있도록 한다.
struct_ __declspec(align(CACHE_ALIGN)) CUSTINFO {
    DWORD dwCustomerId;         // 거의 읽기 전용으로 사용
    wchar_t szName[100];        // 거의 읽기 전용으로 사용
 
    // 아래 필드는 다른 캐시 라인에 들어갈 수 있도록 한다.
    __declspec(align(CACHE_ALIGN))
    int nBalanceDue;            // 읽고 쓰기용으로 사용
    FILETIME ftLastOrderDate;   // 읽고 쓰기용으로 사용
};

```



CPU 의 캐시 라인 크기가 알고 싶으면 다음의 함수를 사용할 수 있음.
```
BOOL GetLogicalProcessorInformation(
  PSYSTEM_LOGICAL_PROCESSOR_INFORMATION Buffer,
  PDWORD                                ReturnedLength
);
```

#### 3. 고급 스레드 동기화 비법

- 인터락 계열 함수들은 하나의 값에 원자적 접근이 필요한 경우 유용하다. 따라서 동기화가 필요하다면 위 계열 함수로 우선 문제를 해결할 수 있는지 검토하는 것이 좋다. 
- 단일 프로세서 머신에서는 스핀락을 사용하지 말자.(CPU 시간이 너무 낭비됨)
- volatile 과 polling 을 통해서 스레드 동기화 기법을 직접 구현할 수 있지만 그러지 말자.(운영체제가 지원하는거 씁시다)

**Volatile: 타입 한정자로 변수가 운영체제 또는 동시에 수행중인 다른 스레드에 의해서 변경될 수 있음을 컴파일러에게 알려주는 키워드. 해당 변수에 대해 최적화가 진행되지 않는다.** 

#### 4. 크리티컬 섹션

- 공유 리소스에 대해 배타적으로 접근해야 하는 작은 코드의 집합을 의미
- 공유 리소스를 다루는 여러 줄의 코드를 Atomic하게 수행하기 위한 방법
- 인터락 함수로 동기화 문제를 해결할 수 없다면 크리티컬 섹션을 사용하자.
- 크리티컬 섹션은 내부적으로 인터락을 이용하기 때문에 빠른 편에 속한다.

```
VOID InitializeCriticalSection(PCRITICAL_SECTION pcs);
BOOL InitializeCriticalSectionAndSpinCount(PCRITICAL_SECTION pcs, DWORD dwSpinCount);
 
VOID DeleteCriticalSection(PCRITICAL_SECTION pcs);
 
VOID EnterCriticalSection(PCRITICAL_SECTION pcs);
BOOL TryEnterCriticalSection(PCRITICAL_SECTION pcs);
 
VOID LeaveCriticalSection(PCRITICAL_SECTION pcs);
```

**크리티컬 섹션의 세부사항**
- 크리티컬 섹션 구조체는 일반적으로 모든 스레드에서 접근 가능하게 전역변수로, 클래스 멤버로 생성시 private 멤버로 선언한다.
- 공유 리소스에 접근하고자 하는 모든 스레드는 크리티컬 섹션의 주소를 알고 있어야 한다.
- 이 구조체를 사용하기 위해 반드시 InitializeCriticalSection 함수를 통해 구조체를 초기화 해야 한다.
- 스레드가 더 이상 공유 자원을 사용하지 않는다면 리소스 반환을 위해 DeleteCriticalSection를 호출하여 크리티컬 섹션을 삭제해야 한다.
- 공유 리소스에 접근하는 코드를 작성하는 경우 코드 앞에 EnterCriticalSection 함수를 호출해야 한다.
- 공유 리소스 사용을 마치면 LeaveCriticalSection을 호출해 자원 점유를 마쳐야 한다.
- 다른 스레드가 진입한 크리티컬 섹션에 특정 스레드가 진입을 시도하면, 스레드는 바로 대기 상태로 변경된다. 이 때 스레드는 유저 모드에서 커널 모드로 변경되는데 이것은 값비싼 동작에 해당한다.
- 위 성능 이슈로 인해 윈도우는 InitializeCriticalSectionAndSpinCount 함수를 제공한다. 이 함수는 일정 횟수 동안 스핀락을 시도하고 그 이후에 크리티컬 섹션으로 전환한다.
- InitializeCriticalSection 함수는 드물게 실패할 수 있다. (메모리 부족) 이런 경우 예외 핸들링이 필요하다.

#### 5. 슬림 리더-라이터 락

SRWLock(Slim Reader-Writer Lock)은 크리티컬 섹션과 유사하게 다수의 스레드로부터 단일의 리소스를 보호하는 목적으로 사용된다. 하지만 자원을 읽기만 하는 Reader와 읽고 쓰는 Writer가 완전히 구분되어 있는 경우에 사용한다. 읽기만 하는 동작에는 리소스에 대한 변경을 하지 않으므로 동시에 수행되어도 무방하기 때문이다. 공유 자원에 대한 읽기 동작이 쓰기 동작보다 훨씬 많이 일어난다면  크리티컬 섹션보다 성능이 더 뛰어날 것이다.

```
VOID InitializeSRWLock(PSRWLOCK SRWLock);
 
VOID AcquireSRWLockExclusive(PSRWLOCK SRWLock);     // For Writer
VOID ReleaseSRWLockExclusive(PSRWLOCK SRWLock);     // For Writer
 
VOID AcquireSRWLockShared(PSRWLOCK SRWLock);        // For Reader
VOID ReleaseSRWLockShared(PSRWLOCK SRWLock);        // For Reader
 
typedef struct _RTL_SRWLOCK {
    PVOID Ptr;
} RTL_SRWLOCK, *PRTL_SRWLOCK;
```

#### 6. 조건변수

조건변수(Condition variable)을 사용하면 스레드가 리소스에 대한 락을 해제하고 SleepCondotionVariableCS나 SleepConditionVariableSRW 함수에서 지정한 상태가 될 때까지 스레드를 블로킹해 준다. FALSE를 반환받는 경우 락을 수행하지 않음에 주의해야 한다.

동일한 조건변수를 인자로 SleepConditionVariable 함수를 호출한 스레드가 여러 개 있는 경우, WakeConditionVariable이 호출되면 이 중 하나의 스레드만이 락을 설정한 상태로 수행을 재개하게 된다.



```
BOOL SleepConditionVariableCS(PCONDITION_VARIABLE pConditionVariable, PCRITICAL_SECTION pCriticalSection, DWORD dwMilliseconds);
BOOL SleepConditionVariableSRW(PCONDITION__VARIABLE pConditionVariable, PSRWLOCK pSRWLock, DWORD dwMilliseconds, ULONG Flags);
 
VOID WakeConditionVariable(PCONDITION_VARIABLE ConditionVariable);
VOID WakeAllConditionVariable(PCONDITION_VARIABLE ConditionVariable);
```