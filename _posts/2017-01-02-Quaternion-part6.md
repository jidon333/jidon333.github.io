---
layout: post
title: Quaternion 개념 정리-part6(구면선형보간)
categories: Math
comments: true
language: ko
---

spherical linear interpolation


![](/assets/img/QuatPart6/slerp.png)

Slerp라고 줄여서 주로 불리우는 구면선형보간은 다음과 같이 계산된다.

![](/assets/img/QuatPart6/rotation_with_quaternion_12.jpg)

[그림1]

이는 평행사변형 법칙을 따르기에 유도을 위해서는 r의 위치에따른 L1, L2를 구하면 된다.

![](/assets/img/QuatPart6/rotation_with_quaternion_13.jpg)

**끝.**