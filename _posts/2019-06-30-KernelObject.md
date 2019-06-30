---
layout: post
title: 커널 오브젝트
categories: OS
comments: true
language: ko
---



**커널 오브젝트란 커널에 의해 관리되는 데이터 블록을 말한다.**
이건 약간 추상적인 개념이라 우선 커널 오브젝트들의 공통적인 특성에 대해 살펴보자.

운영체제는 Access token object, Event object, File object, Mutex object, Thread object 등 다양한 형태의 커널 오브젝트를 생성 및 조작하는데...

### 커널 오브젝트의 특징

**특징 1: 오직 커널만..**

- 커널 오브젝트는 커널에 의해서만 접근이 가능(어플리케이션에서 메모리 접근하여 수정 X)
- 하지만 MS는 커널 구조체 내용에 접근하기 위한 정제된 함수들을 제공하고 이것을 사용하면 접근 가능

**특징 2: 핸들**
- 커널 오브젝트 생성 함수를 호출하면 함수는 커널의 **핸들** 값을 반환.
- 이는 32비트 윈도우에서 32비트, 64비트 윈도우에서 64비트
- 다양한 함수의 매개변수로 전달되며 OS는 어떤 커널 오브젝트를 조작할 것인지 구분 가능

**특징 3: 사용 카운트(Usage count)를 갖는다.**
- 커널 오브젝트는 커널에 의해서 소유되므로 어떤 프로세스가 특정 함수로 커널 오브젝트를 생성 후 종료되더라도 커널 오브젝트가 프로세스와 함께 소멸되는 것은 아님.
- 사실 대부분의 경우는 커널 오브젝트도 같이 소멸되는 것이 맞는데 일종의 프로세스간의 글로벌 래퍼런스 카운팅을 하고 있어 다른 프로세스에서도 사용 중이면 생성한 프로세스가 소멸되도 커널 오브젝트는 삭제되지 않음.
- 이를 **Usage conut**라고 부르며 모든 커널오브젝트는 이 값을 가지고 있음. 0이 되면 소멸

**특징 4: 보안 디스크립터(Security descriptor)를 갖는다.**
- 누가 커널 오브젝트를 소유하고, 어떻게 공유되고, 각 사용자의 접근 권한은 어떻게 되는지 서술된 구조체.
- 커널 오브젝트 생성 함수의 인자로 전달하면 디테일 설정 가능.
- 파일 맵핑 함수를 예로들어 보겠음
```
    // 커널 오브젝트 생성 함수
    HANDLE CreateFileMappingA(
    HANDLE                hFile,
    LPSECURITY_ATTRIBUTES lpFileMappingAttributes, // 이 부분으로 보안 정보를 받는다.
    DWORD                 flProtect,
    DWORD                 dwMaximumSizeHigh,
    DWORD                 dwMaximumSizeLow,
    LPCSTR                lpName
    );

    // 커널 오브젝트 사용 함수. 사용자가 권한이 없으면 NULL Handle 반환
    // ERROR_ACCESS_DENIED 에러 발생
    HANDLE OpenFileMappingA(
    DWORD  dwDesiredAccess,
    BOOL   bInheritHandle,
    LPCSTR lpName
    );
```
- 윈도우에서 LPSECURITY_ATTRIBUTES를 인자로 받는 함수가 있다면 커널 오브젝트를 생성한다고 봐도 무방함

### 프로세스의 커널 오브젝트 핸들 테이블

프로세스가 초기화되면 운영체제는 프로세스를 위해 커널 오브젝트 핸들 테이블을 생성한다.
이는 커널 오브젝트의 메모리 블록을 가리키는 포인터를 갖고 있으며 프로세스에서 커널 오브젝트를 생성하면 테이블이 채워진다.
만약 CloseHandle() 등의 함수를 호출하며 명시적으로 커널 오브젝트를 더 이상 사용 안하겠고 알려주면 Usage count를 감소시키고 커널 테이블의 값도 다시 비워준다.

### 프로세스간 커널 오브젝트의 경우

프로세스간에 커널 오브젝트를 공유하는 경우는 빈번하게 발생할 수 있다. 예를들면
- file mapping object는 두 프로세스 사이에서 데이터의 블록을 공유하게 해준다.
- mailslot과 named pipe를 사용하면 네트워크로 연결된 서로 다른 머신에서 데이터 주고 받을 수 있음
- mutex, semaphore, event는 서로 다른 프로세스에서 수행되는 스레드간 동기화를 가능하게 해줌. 이를 이용하면 한 어플리케이션이 특정 작업을 완료했을때 다른 어플리케이션 그 사실을 알게 할 수 있다.

하지만 프로세스간에 커널 오브젝트를 공유하는 것은 쉽지 않은데, 커널 오브젝트의 핸들은 프로세스별로 고유한 값을 가지기 때문이다 -0-;; 보안과 안정성을 얻기 위해서 이렇게 설계되었는데 만약 커널 오브젝트 핸들 값이 전역적인 값이라면 어떤 프로세스든 다른 프로세스에서 사용하는 커널 오브젝트를 쉽게 얻어올 수 있고 이를 통해 다른 프로세스가 오동작하게 만드는 것도 쉽게 가능할 것이다.

그럼 어떻게 커널 오브젝트를 공유할 수 있는지 알아보자.

#### 1. 오브젝트 핸들의 상속을 이용하는 방법

우선 이 방법은 오브젝트를 공유하고자 하는 프로세스들이 parent-child 관계를 가질 때에만 사용될 수 있다.

즉 커널 오브젝트가 부모 프로세스에 의해 사용되는 중에 부모 프로세스가 새로운 자식 프로세스를 생성하면 자식 프로세스에서 부모 프로세스가 사용하는 커널 오브젝트 핸들에 접근할 수 있도록 하는 방법이다. 이를 오브젝트 핸들의 상속이라고 표현한다. 오브젝트 핸들의 상속이 정상 작동하려면 다음과 같은 작업들의 수행이 필요하다.

**1) 부모 프로세스는 커널 오브젝트를 생성할 때 이를 가리키는 핸들이 상속될 수 있음을 시스템에게 알려줘야 한다.(오브젝트 상속이 아닌 '오브젝트 핸들'의 상속임을 유념하자.)**

상속 가능한 핸들을 만들기 위해서는 커널 오브젝트를 생성할 때 SECURITY_ATTRIBUTES 구조체를 다음과 같이 초기화 한다.

```
SECRUITY_ATTRIBUTES sa;
sa.nLength = sizeof(sa);
sa.lpSecurityDescriptor = NULL;
sa.bInheritHandle = TURE; // 상속 가능한 핸들을 만든다.

HANDLE hMutex = CreateMutex(&sa, FALSE, NULL);
```

위에서 언급했던 커널 오브젝트 핸들 테이블에는 커널 오브젝트에 대한 메모리 블록 포인터 이외에도 각 커널 오브젝트 핸들이 상속 가능한지 여부를 가리키는 flag bit를 갖고 있다. 커널 핸들 객체를 만들때 위와 같은 설정을 사용하면 해당 플래그의 비트는 1이 된다.

**2) 부모 프로세스에서 자식 프로세스를 생성한다.**
이러한 작업은 CreateProcess 함수를 사용하면 된다.

```
BOOL CreateProcessA(
  LPCSTR                lpApplicationName,
  LPSTR                 lpCommandLine,
  LPSECURITY_ATTRIBUTES lpProcessAttributes,
  LPSECURITY_ATTRIBUTES lpThreadAttributes,
  BOOL                  bInheritHandles, // 보통은 FALSE, 
  DWORD                 dwCreationFlags,
  LPVOID                lpEnvironment,
  LPCSTR                lpCurrentDirectory,
  LPSTARTUPINFOA        lpStartupInfo,
  LPPROCESS_INFORMATION lpProcessInformation
);
```
bInheritHandles를 TRUE로 설정하면 자식은 부모 프로세스에서 상속 가능한 핸들 값을 상속한다. 부모로부터 상속 가능한 핸들을 찾아 자식 프로세스 핸들 테이블에 같은 값을 복사한다.
결국 부모-자식 프로세스에서 동일한 핸들값으로 커널 오브젝트를 이용할 수 있게 된다.
자식에서 사용 가능하므로 핸들 오브젝트의 Usage counter는 자연스래 1 증가한다.

커널 오브젝트를 파괴할 떄 C++에서의 클래스 상속관계처럼 반드시 자식의 커널 오브젝트에서 먼저 CloseHandle() 함수를 호출해야 한다는 규칙 같은 것은 없다.

다만 <u>오브젝트 핸들 상속은 자식 프로세스를 생성할 때만</u> 적용이 가능한 것을 기억해야 한다. 아무리 부모가 상속 가능하도록 커널 오브젝트를 생성하더라도 이미 생성된 자식 프로세스가 갑자기 이 핸들을 상속 받는 것은 불가능하다.  

그리고 또한 이 핸들 상속에서 재밌는 점이 있는데 바로 자식 프로세스에서 자신이 어떤 커널 오브젝트 핸들을 상속받았는지 알지 못한다는 점이다! 물론 핸들 테이블은 알고 있지만 어플리케이션 레벨에서 프로그래머는 알지 못한다. 따라서 부모 프로세스로부터 어떤 핸들을 상속 받을 예정인지 전달 받아야 하는데 보통은 다음과 같은 방법을 사용한다.

1. 자식 프로세스의 명령행 인자를 통해 핸들 값을 받는다
2. 프로세스간 통신 방법을 이용하여 부모가 자식 프로세스에게 핸들을 전달한다.



#### 2. 명명된 오브젝트를 사용하는 방법.

프로세스간에 커널 오브젝트를 공유하는 두 번째 방법이다. 모두는 아니지만 대부분의 커널 오브젝트는 이름을 가질 수 있는데 예를 들면 다음과 같은 함수들이다.

```

HANDLE CreateMutexA(
  LPSECURITY_ATTRIBUTES lpMutexAttributes,
  BOOL                  bInitialOwner,
  LPCSTR                lpName // 이름 지정 가능
);

HANDLE CreateEventA(
  LPSECURITY_ATTRIBUTES lpEventAttributes,
  BOOL                  bManualReset,
  BOOL                  bInitialState,
  LPCSTR                lpName
);

HANDLE CreateSemaphoreA(
  LPSECURITY_ATTRIBUTES lpSemaphoreAttributes,
  LONG                  lInitialCount,
  LONG                  lMaximumCount,
  LPCSTR                lpName
);

...

```

위 함수들은 공통적으로 pszName을 갖는다. 이 매개변수로 NULL을 전달하면 익명의 커널 오브젝트를 생성하지만 문자열을 전달하는 것으로 커널 오브젝트에 이름을 붙일 수 있다. 특정한 규칙은 없으며 중복된 이름은 사용 불가능하다. 예를 들어 다음과 같은 코드는 에러를 발생시킨다.

```
HANDLE hMutex = CreateMutex(NULL, false, TEXT("jidonObj"));
HANDLE hSem = CreateSemaphore(NULL, 1, 1, TEXT("jidonObj"));
DWORD dwErrorCode = GetLastError(); // ERROR_INVALID_HANDLE 에러 발생.
```

굉장히 불분명한 에러 코드를 발생시키므로 이름을 지을 때 주의할 것.

자 그럼 이름을 이용하여 어떻게 커널 오브젝트를 공유하는지 알아보자.

먼저 프로세스 A에서 다음과 같은 코드로 커널 오브젝트를 생성한다.
```
HANDLE hMutextProcessA = CreateMutex(NULL, FALSE, TEXT("jidonMutex"));
```

이제 프로세스 B에서 다음과 같은 코드를 실행한다. 이 프로세스는 A와 부모-자식 관계일 필요가 없다.

```
HANDLE hMutextProcessB = CreateMutex(NULL, FALSE, TEXT("jidonMutex"));
```

1. B에서 CreateMutex 함수를 호출하면 os는 우선 jidonMutex라는 이름의 커널 오브젝트가 존재하는지 체크한다.
2. 만약 동일 오브젝트가 존재하면 이제 타입을 체크한다. 당연히 Mutex를 생성하려는 것이기 때문에 이미 생성된 오브젝트도 mutex 이어야 하기 때문이다.
3. 프로세스 B가 해당 오브젝트에 대해 접근 권한이 있는지 체크한다.

위 조건을 모두 만족하면 B 프로세스의 핸들 테이블에서 A에서 만들었던 jidonMutex 오브젝트 핸들을 생성하여 설정한다. 만약 위 조건을 만족하지 않는 경우 NULL을 반환할 것이다.
여기서도 커널 오브젝트에 핸들은 프로세스별로 고유의 값을 사용함을 잊지 말자(핸들 상속과는 다르다). Usage count 역시 증가할 것이다.

명시적으로 이미 생성된 커널 오브젝트에 대한 접근을 시도하는 함수도 있다.

```

HANDLE WINAPI OpenMutex(
  
  DWORD  dwDesiredAccess,
  DWORD   dwDesiredAccess,
  BOOL    bInheritHandle,
  LPCTSTR lpName
);

HANDLE OpenEventA(
  DWORD  dwDesiredAccess,
  BOOL   bInheritHandle,
  LPCSTR lpName
);

HANDLE OpenSemaphoreW(
  DWORD   dwDesiredAccess,
  BOOL    bInheritHandle,
  LPCWSTR lpName
);


```

예상대로 이들은 CreateXXX 함수와는 다르게 이미 생성되어 있지 않은 경우 생성을 시도하지는 않는다.

TIP: **Private namespace**

- 비스타 이전에는 다른 프로세스가 공유 오브젝트의 이름을 훔치는 것(선점)에 대한 방어책이 없었다. 이는 Dos(Denial of service) 공격에 기본이 되는 원리.
따라서 프로세스간에 공유할 필요가 없다면 이름을 가지지 않는 것이 바람직하며 공유할 일이 있더라도 private namespace를 만들어 공유하는 것을 권장한다.

#### 3. 오브젝트 핸들의 복사를 이용하는 방법.

프로세스간 커널 오브젝트를 공유하는 마지막 방법은 DuplicateHandle 함수를 사용하는 것이다.

```
BOOL DuplicateHandle(
  HANDLE   hSourceProcessHandle,
  HANDLE   hSourceHandle,
  HANDLE   hTargetProcessHandle,
  LPHANDLE lpTargetHandle,
  DWORD    dwDesiredAccess,
  BOOL     bInheritHandle,
  DWORD    dwOptions
);
```

이 함수는 한마디로 말하면 특정 프로세스 핸들 테이블 내 항목을 다른 프로세스 핸들 테이블로 복사하는 일을 한다.
첫번째 세번째 인자로 '프로세스 커널 오브젝트'에 대한 핸들을 필요로 하는데 이는 다음 포스팅(책 4장)에서 설명될 예정이다. 우선은 프로세스가 생성되면 프로세스 커널 오브젝트가 하나 생긴다고 생각하면 된다.

두번째 인자는 복사할 커널 오브젝트 핸들이며 네번째 인자는 함수 호출 이후에 전달받게 될 TargetProcess에서 사용될 핸들이다.(프로세스간의 핸들은 서로 고유한 값을 갖는다.)

나머지 세 인자는 복사될 TargetProcess의 핸들이 갖게될 핸들상속여부, 권한을 지정한다. 마지막 dwOption으로는 Source handle의 속성 정보를 그대로 가져와서 사용할지, 아니면 기존의 핸들을 소멸시킬지 등의 옵션을 추가할 수 있다.

이 방법 또한 커널 오브젝트 핸들을 상속하는 방법과 마찬가지로 target process가 새로운 커널 오브젝트에 접근 가능하게 되었다는 사실을 통보받지 못한다.
따라서 source process로부터 그 사실(handle)을 전달받아야 하는데 이때 윈도우 메시지나 프로세스간 통신 매커니즘(IPC)를 사용한다.

