---
layout: post
title: Windows via c/c++ 시작
categories: OS
comments: true
language: ko
---

오늘부터 제프리리처 Windows via c/c++ 책을 공부하면서 간략하게 포스팅하려고 한다. 학창 시절에 읽고 싶었던 책인데 절판 되었기도 하고 나랑은 별 상관 없을 거 같아서 신경 안쓰고 있었는데 이번에 복간판으로 새로 나온데다가 게임도 슬슬 질리고 공부나 할까 해서 시작하려는 중. 이번 포스팅을 통해 블로그 사용법도 다시 익히고, 공부 감도 되찾고, 이래저래 나에게 도움이 될 것 같다.
최종 목표는 22장의 DLL Injection, API 후킹 따라해보고 아무 프로그램이나 한번 까보고 더러운 ~~해커~~Cheater 놈들이 어떻게 핵 만드는지 감 잡아보자.


우선 목차는 아래와 같다.

**1부. 준비하기**
- 1장. 에러 핸들링
- 2장 문자와 문자열로 작업하기
- 3장 커널 오브젝트

**2부. 목표 달성**
- 4장 프로세스
- 5장 잡
- 6장 스레드의 기본
- 7장 스레드 스케줄링, 우선순위, 그리고 선호도
- 8장 유저 모드에서의 스레드 동기화
- 9장 커널 오브젝트를 이용한 스레드 동기화
- 10장 동기 및 비동기 장치 I/O
- 11장 윈도우 스레드 풀
- 12장 파이버

**3부. 메모리 관리**
- 13장 윈도우 메모리의 구조
- 14장 가상 메모리 살펴보기
- 15장 애플리케이션에서 가상 메모리 사용 방법
- 16장 스레드 스택
- 17장 메모리 맵 파일
- 18장 힙

**4부. 다이내믹 링크 라이브러리(DLL)**
- 19장 DLL의 기본
- 20장 DLL의 고급 기법
- 21장 스레드 지역 저장소(TLS)
- 22장 DLL 인젝션과 API 후킹

**5부. 구조적 예외 처리**
- 23장 종료 처리기
- 24장 예외 처리기와 소프트웨어 예외
- 25장 처리되지 않은 예외, 벡터화된 예외 처리, 그리고 C++ 예외
- 26장 에러 보고와 애플리케이션 복구


원래는 "DLL Injection 나도 한 번 해보자" 가 목표였으니까 대충 13~22장까지 보면 될 것 같았으나 기왕 공부하는거 꼼꼼하게 봐서 나쁠것도 없고, 시간이 없는 것도 아니니까 3장부터 포스팅할 예정이다.

그럼 다음 페이지에서 계속...

