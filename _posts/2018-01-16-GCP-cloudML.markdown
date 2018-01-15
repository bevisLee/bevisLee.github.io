---
layout: post
title:  "Qwiklabs - Cloud ML Engine"
subtitle:   "Cloud ML Engine - wide & Deep"
categories: gcp
tags: 
---

## 들어가기

 Google Cloud Platform을 잘 사용하기 위해, 여러 스터디에 참석하면서 [Qwiklabs](https://google.qwiklabs.com)에 대해 알게 되었다. 
 Qwiklabs에는 10개 카테고리로 구성되어 있다. 필자는 여러 카테고리 중 ML 관련된 사항에 대해 정리하고자 한다. 

 Cloud ML Engine 에서는 Adult Data Set을 이용하여, 연소득 5만달러 초과자를 예측하는 것이다. 

 * Data Set RAW
 [Data Set RAW](https://github.com/bevisLee/bevisLee.github.io/tree/master/assets/img/post/2018-01-16-GCP-cloudML/1.jpg)
 <img src="https://github.com/bevisLee/bevisLee.github.io/tree/master/assets/img/post/2018-01-16-GCP-cloudML/1.jpg?raw=true">

 * Data Set 설명
 [Data Set 설명](https://github.com/bevisLee/bevisLee.github.io/tree/master/assets/img/post/2018-01-16-GCP-cloudML/2.jpg)

## wide & deep

 이번 예측 분석에서는 Cloud에 많은 예시 중에서 wide & deep을 이용하였다. wide & deep에 대해서 알아보고 실습해보자.
 wide 선형 모델과 deep feed-forward 신경망으로 구성된 모델이다. 
 [![wide & deep](https://github.com/bevisLee/bevisLee.github.io/tree/master/assets/img/post/2018-01-16-GCP-cloudML/4.jpg)](#)

 쉬운 이해를 위해, 가상의 APP을 개발하고 Service 한다는 가정으로 설명하겠다. 
 FoodIO는 사용자가 자신이 원하는 음식을 말하고(쿼리), FoodIO는 음식을 추천해 준다.

 * FoodIO ver 1.0
 음식(쿼리 / 검색어)에 많이 일치하는 문자와 일치하는 항목을 사용자에게 추천 
 (사용자 : 프라이드 치킨 → FoodIO : 치킨 볶음밥)

 * FoodIO ver 2.0 : wide 모델 적용
 쿼리에 동시 발생 대상 레이블과 상관 관계가 있는지 파악 후 기억(추천)
 (사용자 : 프라이드 치킨 → FoodIO : 치킨 볶음밥, 치킨과 와플, ....)
 [![FoodIO wide](https://github.com/bevisLee/bevisLee.github.io/tree/master/assets/img/post/2018-01-16-GCP-cloudML/5.jpg)](#)

 * FoodIO ver 3.0 : Deep 모델 적용

 [![FoodIO deep](https://github.com/bevisLee/bevisLee.github.io/tree/master/assets/img/post/2018-01-16-GCP-cloudML/6.jpg)](#)

