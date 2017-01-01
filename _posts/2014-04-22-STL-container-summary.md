---
layout: post
title: STL 컨테이너 정보 요약
categories: C_CPP
comments: true
language: ko
---

 
# Sequence Container

## vector 
 
삽입성능
- 끝 부분 :  O(1)
- 임의의 위치 : O(N)
 
삭제성능 
- 끝 부분: O(1)
- 임의의 위치 : O(N)
 
접근 성능
- O(1)
 
사용
- 삽입/삭제 성능보다 접근 성능이 더 중요한 경우. 
- 표준 C 스타일 배열을 사용하면서 크기가 동적으로 커지거나 줄어들 수 있는 경우.
 
 
## list
 
삽입성능
- O(1)
 
삭제성능 
- O(1)
 
접근 성능
- O(N)
 
사용
- 접근 성능은 별로 중요하지 않고 삽입/삭제 성능이 중요한 경우.
 
## deque
 
삽입성능
- 시작 또는 끝 : O(1)
- 임의의 위치 : O(N)
 
삭제성능 
- 시작 또는 끝 : O(1)
- 임의의 위치 : O(N)
 
접근 성능
- O(1)
 
사용
- 양쪽 끝에 요소를 삽입하거나 제거할 때(보통 vector나 list를 사용한다. 잘 사용 안함)


## array
 
삽입성능
- 지원하지 않음
 
삭제성능 
- 지원하지 않음
 
접근 성능
- O(1)
 
사용
- 고정 크기의 C 스타일 배열 대신 사용
 
# Associative Container

## set/multiset
 
삽입성능
O(log(N))
 
삭제성능 
O(log(N))

접근 성능
O(log(N))

### 사용
정렬된 저장 상태와 균형있는 삽입/삭제/룩업 성능이 필요할 때

## map/multimap
 
삽입성능
- O(log(N))
 
삭제성능 
- O(log(N))

접근 성능
- O(log(N))

사용
- 키와 값을 연관 지어 쌍으로 저장하면서 고른 삽입/삭제/룩업 성능이 필요할 때
 
# Unordered Associative Container

## unordered_set/unordered_multiset

삽입성능
- 평균 : O(1)
- 최악의 경우 O(N)
 
삭제성능 
- 평균 : O(1)
- 최악의 경우 O(N)

접근 성능
- 평균 : O(1)
- 최악의 경우 O(N)

### 사용
- 키와 값을 연관지어 쌍으로 저장하되 정렬될 필요가 없고 고른 삽입/삭제/룩업 성능이 필요할 때.
정렬된 Associative Container인 일반적인 map보다 좋은 성능을 보여주나 저장된 항목에 따라 다를 수 있음.
 
삽입성능
- 평균 : O(1)
- 최악의 경우 O(N)
 
삭제성능 
- 평균 : O(1)
- 최악의 경우 O(N)

접근 성능
- 평균 : O(1)
- 최악의 경우 O(N)

사용
- 항목을 저장하되 정렬될 필요가 없고 고른 삽입/삭제/룩업 성능이 필요할 때.
정렬된 Associative Container인 일반적인 set보다 좋은 성능을 보여주나 저장된 항목에 따라 다를 수 있음.
 
 
 
[ 참고 문헌 : 마크 그레고리, 니콜라스 솔터, 스캇 클레퍼 저,  Professional C++ (전문가를 위한 C++ ) ]
 
 