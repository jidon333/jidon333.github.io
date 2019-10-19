---
layout: post
title: 스레드 스케줄링, 우선순위, 선호도
categories: OS
comments: true
language: ko
---


### 목표
- 스레드의 정지와 계속 수행
- 프로세스의 정지와 계속 수행
- 슬리핑
- 다른 스레드로의 전환
- 하이퍼스레드 CPU 상에서 다른 스레드로 전환
- 스레드 수행 시간
- 컨텍스트 내의 CONTEXT 구조체
- 스레드 우선순위
- 우선순위의 추상적인 의미
- 우선순위 프로그래밍
- 선호도

**운영체제의 스레드 스케줄링**

운영체제는 아래 동작을 시스템이 부팅되고 종료될 때까지 반복한다.
1. 스레드 가능한 커널 오브젝트 선택
2. 스레드 커널 오브젝트에 저장된 레지스터 값을 CPU 레지스터로 로드
3. 컨텍스트 전환이 일어나면 프로세스 주소 공간 내 위치한 코드를 실행.
4. CPU 레지스터 정보를 스레드 컨텍스트에 저장하고 다시 1로 

각 스레드에 대한 정보는 MS의 Spy++를 이용하여 확인할 수 있음.

**윈도우에서 특정 시간 내에 스레드를 수행시키는 것이 가능할까?**
- 불가능하다.
- 윈도우 운영체제는 실시간 운영체제가 아님. 실시간 운영체제는 하드웨어 장치에 대한 세부 정보와 밀접하게 연관되어 있음.
- 윈도우 운영체제는 다양한 CPU, 드라이브, 네트워크 장비에서 수행되는 것을 목표로 설계됨.

#### 1. 스레드의 정지와 계속 수행

- 스레드 커널 오브젝트에는 정지 카운트(suspend count)라는 값이 저장되어 있음. 
    - CreateProcess 또는 CreateThread를 호출하면 커널 오브젝트를 생성하고 정지 카운트를 1로 초기화.
    - 이 상태에서 스레드는 스케줄 불가능 상태.
    - 스레드가 완전히 초기화되면 다시 정지 카운트가 0으로 변경되고 스케줄 가능한 상태가 된다.
    - 스레드 정지 상태에서는 스레드 환경 변수들을 설정할 수 있음.
- SuspendThread, ResumeThread 함수로 스레드를 Suspend, Resume 가능
    - 스레드가 자기 자신을 정지시키면 깨어날 수 없으니 조심
    - 스레드가 어떤 작업을 수행하다가 정지될지 알 수 없으므로 SuspendThread는 신중하게 사용
    - 어떤 작업에서 중지될지 모르므로 데드락 같은 문제를 조심해야 한다.

#### 2. 프로세스의 정지와 계속 수행

- 프로세스는 CPU 시간을 할당받는 대상이 아니기 떄문에 정지와 수행이라는 개념은 적용되지 않음.
- SuspendProcess는 프로세스 내 모든 스레드를 정지시키는 의미
- 디버깅 목적으로 WaitForDebugEvent 함수를 사용하면 ContinueDebugEvent가 호출될 때까지 프로세스 내 모든 스레드를 정지시킴

#### 3. Sleeping

- 스레드는 Sleep 함수를 호출하여 일정 시간 동안 자신을 스케줄하지 않도록 운영체제에 명령을 내릴 수 있음.
    - 이 함수를 사용하면 매개변수로 주어진 시간동안 스레드를 일시정지
    - 시스템은 주어진 시간 동안 스레드를 스케줄 불가능 상태로 변경.(전달받은 시간보다 좀 더 기다릴 수 있음)
    - Sleep 함수의 매개변수로 0 전달 가능. 이렇게하면 스레드가 자발적으로 남은 타임슬라이스를 포기하여 다른 스레드가 시간을 할당받음. 다른 사용 가능 스레드가 없는경우 Sleep을 호출한 스레드가 다시 스케줄 될 수 있다.


#### 4. 다른 스레드로의 전환

- BOOL SwitchToThread();
    - 이 함수를 호출하면 시스템은 일정 시간 동안 CPU 시간을 받지 못해 수행되지 못한 스레드를 찾는다.
    - 수행할 스레드를 찾지 못하면 바로 결과를 반환하지만 찾으면 해당 스레드를 스케줄한다.
    - 할당받은 스레드는 단일 퀀텀 시간 동안만 수행된다.
    - Sleep(0) VS SwitchToThread(): SwitchToThread는 호출하는 스레드보다 더 낮은 우선순위의 스레드도 호출 가능하다.


#### 5. 하이퍼스레딩(Hyper-threading)

- 다수의 "논리적" CPU를 사용하여 각기 다른 스레드를 수행할 수 있다.
- 위 기술로 수행되는 스레드들은 자신만의 구조적 상태(레지스터 정보)는 갖지만 CPU 캐시와 같은 주요 자원은 공유한다.

#### 6. 스레드 수행 시간

- 특정 작업을 위해 얼마만큼의 스레드 시간을 사용하였는지 알아야 할 필요가 있다.
- 많은 사람들이 다음과 같은 코드를 사용함.

```
ULONGLONG qwStartTime = GetTickCount64();

// 알고리즘을 수행한다.

ULONGLONG qwElapsedTime = GetTickCount64() - qwStartTime;

```

위 코드가 동작하려면 수행 중에 인터럽트가 수행되지 않는다는 가정이 필요하다. 선점형 운영체제에서는 스레드가 언제 CPU에 의해 수행될지를 알 수 없으며 현재 코드를 수행하는 도중에 언제든지 다른 작업을 수행할 수 있다. 따라서 실제로 필요한 함수는 스레드가 부여받은 CPU 시간이 얼마나 되는지 알아내는 함수이다.
- 윈도우는 스레드가 사용한 시간을 가져오는 GetThreadTimes 함수를 제공한다.

```
BOOL GetThreadTimes(
  HANDLE     hThread,
  LPFILETIME lpCreationTime,
  LPFILETIME lpExitTime,
  LPFILETIME lpKernelTime,
  LPFILETIME lpUserTime
);
```

- 프로세스에 대해서는 GetProcessTimes() 사용

```
BOOL GetProcessTimes(
  HANDLE     hProcess,
  LPFILETIME lpCreationTime,
  LPFILETIME lpExitTime,
  LPFILETIME lpKernelTime,
  LPFILETIME lpUserTime
);
```

#### 7. 컨텍스트 내의 CONTEXT 구조체

- CONTEXT 구조체는 시스템이 저장하는 스레드의 상태 정보로, 다음번 CPU가 스레드를 수행할 때 어디서부터 수행해야 할지를 알려주는 역할을 한다.
- CONTEXT 구조체는 여러개의 영역으로 나뉘어져 있다.
    - CONTEXT_CONTROL: Instruction pointer, stack pointer, flags, function reture address와 같은 CPU 제어 레지스터 값들.
    - CONTEXT_INTEGER: integer register
    - CONTEXT_FLOATING_POINT: floating-point register
    - CONTEXT_SEGMENTS: segment register
    - CONTEXT_DEBUG_REGISTERS: debug register
    - CONTEXT_EXTENDED_REGISTERS: extended register

- 윈도우는 스레드 커널 오브젝트 내부에 저장된 컨텍스트 정보를 확인하고, 값을 가져올 수 있도록 GetThreadContext 함수를 제공.
- 다만 GetThreadContext 함수를 사용하기 위해서는 반드시 먼저 Thread를 Suspend 시켜야 함.(원하는 결과가 나오지 않을 수 있음)
```
BOOL GetThreadContext(
  HANDLE    hThread,
  LPCONTEXT lpContext
);
```

- SetThreadContext 함수를 사용하면 반대로 스레드 커널 오브젝트의 레지스터 값을 지정된 값으로 변경할 수 있음.
- 역시 사용전 SuspendThread, 사용후 ResumeThread 호출하여 정지/재개가 필요


#### 8. 스레드의 우선순위

- 스레드들은 다양한 우선순위를 가질 수 있고 이것은 스케줄러가 수행 스레드를 결정하는데 영향을 미친다.
- 모든 스레드는 0(가장 낮은)~31(가장 높음) 범위 내의 우선순위 번호를 갖는다.
- 시스템은 다음에 수행할 때 가장 높은 번호 순으로 스케줄 가능한 스레드를 선택한다.
- 31번 우선순위를 가진 스레드가 스케줄중이라면 0~30번까지의 스레드는 절대로 CPU 시간을 할당받지 못한다.(Starvation)
- Starvation은 멀티 프로세서 머신에서는 비교적 적게 발생함.
- 대부분의 스레드는 스케줄 불가능 상태를 유지함
- 낮은 우선순위의 스레드가 수행중이더라도 높은 우선순위의 스레드가 스케줄 가능 상태가 되면 기존 스레드를 정지시키고 높은 우선순위 스레드에 CPU 시간을 할당 


#### 9. 우선순위의 추상적 의미

- MS는 세월이 흐름에 따라 컴퓨터의 사용 목적이 변경되고 스케줄링 알고리즘이 변경되더라도 소프트웨어가 정상 작동하도록 설계해야 했다.
    - MS는 스케줄러의 동작 방식을 완벽하게 문서화 하지 않음
    - 어플리케이션이 스케줄러의 기능상 장점을 완벽하게 이용하지 못하도록 함
    - 스케줄러의 알고리즘은 변경될 수 있으므로 코드를 방어적으로 작성할 것을 지속적으로 강조 함.

- MS API는 시스템과 스케줄러에 대해 매우 추상적인 모습만을 드러내고 있음.
- 어플리케이션을 설계할 때 스레드의 응답성과 프로세스의 우선순위를 고려해야 함.

**프로세스 우선순위 클래스**
```
- 실시간(Realtime)
- 높음(High)
- 보통 이상(Above normal)
- 보통(normal)
- 보통 이하(Below normal)
- 유휴 상태(Idle)
```
- 실시간 ~ 높음: 즉각적인 응답이 필요한 프로세스의 스레드에서 사용
- 유휴 상태: 시스템이 아무것도 하지 않는 상태에서 수행되는 어플리케이션에 적합
- 대부분의 프로세스는 보통에서 수행
- 어플리케이션 개발자는 우선순위 클래스를 이용할 수 있지만 위에서 설명한 0~31단계의 수행 단계를 직접 설정할 수는 없다.
- 높은 우선순위 레벨의 스레드는 가급적 최소한의 시간동안만 스케줄 가능 상태로 남아있게 하자.

#### 10. 우선순위 프로그래밍


**우선순위 설정하기**

1. CreateProcess를 호출할때 fdwCreate 매개변수로 우선순위 클래스를 전달하여 우선순위 설정[(링크)](https://docs.microsoft.com/en-us/windows/win32/procthread/scheduling-priorities)
```
IDLE_PRIORITY_CLASS
BELOW_NORMAL_PRIORITY_CLASS
NORMAL_PRIORITY_CLASS
ABOVE_NORMAL_PRIORITY_CLASS
HIGH_PRIORITY_CLASS
REALTIME_PRIORITY_CLASS
```
2. SetPriorityClass/GetPriorityClass 함수를 사용
```
BOOL SetPriorityClass(
  HANDLE hProcess,
  DWORD  dwPriorityClass
);
```

3. 명령 쉘에서 프로그램 수행 시 우선순위 클래스 지정
```
ex) C:\START /LOW CALC.EXE
LOW대신, BELOWNORMAL, NORMAL, ABOVENORMAL, HIGH, REALTIME 등의 스위치 지원
```
4. 윈도우 작업 관리자를 통해 프로세스의 우선순위 변경


**동적인 우선순위 레벨 상승**
- 시스템은 프로세스 우선순위와 속한 스레드의 우선순위를 종합하여 스레드의 우선순위 레벨을 산출한다.
- I/O 이벤트나 윈도우 메시지, 디스크 읽기 등의 이유로 시스템은 스레드의 우선순위 레벨을 상승시키기도 한다.
    - ex) 기본 우선순위 13 + KeyInput에 의한 우선순위 2 = 현재 우선순위 15
- 임시로 상승된 우선순위는 1 타임슬라이스 만큼만 유지. 이후 타임슬라이스마다 1씩 감소
    - ex) 13(기본) -> 15(상승) -> 14 -> 13 -> 13 -> 13
    - 기본 이하로는 떨어지지 않는다.

- 시스템은 기본 우선순위가 1~15 사이의 스레드에 대해서만 동적 우선순위 상승을 수행함(OS 동작에 방해되지 않도록)
- SetProcessPriorityBoost 함수를 사용하면 해당 프로세스 내의 모든 스레드에 대해 동적 우선순위 조작을 가능/불가능하게 제어 가능.
```
BOOL SetProcessPriorityBoost(
  HANDLE hProcess,
  BOOL   bDisablePriorityBoost
);
```
- 우선순위가 낮아 너무 오랫동안 CPU 시간을 할당받지 못한 스레드가 감지되면 우선순위를 15까지 증가시켜 두 번의 퀸텀동안 스레드가 수행되게 해준다.(이후 기본 레벨로 변경)

**포어그라운드 프로세스를 위한 스케줄러 변경**

- 윈도우를 가지고 있고 그 윈도우에서 수행되는 프로세스를 포어그라운드 프로세스라 한다(<-> 백그라운드 프로세스)
- 당연히 사용자는 현재 사용중인 프로세스가 빠르게 응답하기를 원할 것임.
- OS는 이를 위해 포어그라운드 프로세스에 다른 알고리즘을 적용하여 좀 더 많은 퀸텀 시간이 할당되게 함
- 포어그라운드 프로세스가 '보통 우선순위 클래스'를 갖는 경우에만 적용


**I/O 요청 우선순위 스케줄링**

- 낮은 우선순위의 스레드가 아주 짧게 CPU시간을 획득하였을 경우라도, 많은 건의 I/O 요청을 생성하게 되면 시스템 전반적으로 응답성이 나빠진다.
- SetThreadPriority를 호출할 때 THREAD_MODE_BACKGROUND_BEGIN을 인자로 전달하면 낮은 우선순위로 I/O 수행 가능
- 원래대로 돌려놓기 위해서는 SetThreadPriority에 THREAD_MODE_BACKGROUND_END를 넘겨주면 된다.
- 프로세스 버전: SetPriorityClass 함수 인자로 PROCESS_MODE_BACKGROUND_BEGIN/PROCESS_MODE_BACKGROUND_END 인자 전달.

```
ex)
FILE_IO_PRIORITY_HINT_INFO phi;
phi.PriorityHint = IoPriorityHintLow;
SetFileInformationByHandle(hFile, FileIoPriorityHintInfo, &phi, sizeof(PriorityHint));
```


#### 11. 선호도

**소프트 선호도**
- 윈도우 비스타가 스레드를 프로세스에 할당할때 사용
- 다른 조건이 모두 동일하다면 마지막으로 스레드를 수행한 프로세서가 동일 스레드를 다시 수행하도록 유도함
- 프로세스 메모리 캐시 데이터 재사용을 노림
- NUMA 시스템에서는 CPU가 자신과 동일 보드에 있는 메모리에 접근할 때 최적의 성능을 발휘
![](/assets/img/Scheduling/NUMA.png)  
**[그림: NUMA Machine]**

**하드 선호도**
- 윈도우 비스타는 시스템 구조를 고려하여 어떤 스레드를 어떤 CPU에서 수행할지 선호도(Affinity) 지정 가능
- GetSysteminfo 함수로 사용 가능한 CPU 개수 얻을 수 있음
- SetProcessAffinityMask 함수로 어떤 프로세스가 어떤 CPU에서 작동할지 지정
```
BOOL SetProcessAffinityMask(
  HANDLE    hProcess,
  DWORD_PTR dwProcessAffinityMask
);
```
- GetProcessAffinityMask로 프로세스의 선호 마스크 값을 획득하는 것도 가능
- SetThreadAffinityMask를 사용하면 스레드별로 선호도 마스크를 설정하는 것도 가능
```
DWORD_PTR SetThreadAffinityMask(
  HANDLE    hThread,
  DWORD_PTR dwThreadAffinityMask
);
```
- 실행 파일의 헤더 정보에 프로세서 선호도를 설정하는 것도 가능(ImageHlp.h)

```
// EXE 파일 읽어오기
PLOADED_IMAGE pLoadedImage = ImageLoad(szExeName, NULL);
 
// EXE의 로드 환경정보 획득
IMAGE_LOAD_CONFIG_DIRECTORY ilcd;
GetImageConfigInformation(pLoadedImage, &ilcd);
 
// 프로세서 선호도 마스크 변경
ilcd.ProcessAffinityMask = 0x00000003;  // CPU 0 & 1
 
// 환경정보 저장
SetImageConfigInformation(pLoadedImage, &ilcd);
 
// 메모리 해제
ImageUnload(pLoadedImage);
```

- 윈도우 작업 관리자를 통해 CPU 선호도 설정도 가능하다.