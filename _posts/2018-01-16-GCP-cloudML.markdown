---
layout: post
title:  "Qwiklabs - Cloud ML Engine"
subtitle:   "Cloud ML Engine - wide & Deep"
categories: GCP
tags: cloudML
---

## 들어가기

 Google Cloud Platform을 활용한 데이터 분석을 목표로, 여러 스터디에 참석하면서 [Qwiklabs](https://google.qwiklabs.com)에 대해 알게 되었다. 
 Qwiklabs에는 10개 카테고리로 구성되어 있다. 필자는 여러 카테고리 중 ML 관련된 사항에 대해 정리하고자 한다. 

 Cloud ML Engine 에서는 Adult Data Set을 이용하여, 연소득 5만달러 초과자를 예측하는 것이다. 

 * Data Set RAW
 [![Data Set RAW](/assets/img/post/2018-01-16-GCP-cloudML/1.png)](#)

 * Data Set 설명
 [![Data Set 설명](/assets/img/post/2018-01-16-GCP-cloudML/2.png)](#)


## wide & deep

 이번 예측 분석에서는 Cloud에 많은 예시 중에서 wide & deep을 이용하였다. wide & deep에 대해서 알아보고 실습해보자.
 wide 선형 모델과 deep feed-forward 신경망으로 구성된 모델이다. 
 [![wide & deep](/assets/img/post/2018-01-16-GCP-cloudML/4.png)](#)

 쉬운 이해를 위해, 가상의 APP을 개발하고 Service 한다는 가정으로 설명하겠다. 
 FoodIO는 사용자가 자신이 원하는 음식을 말하고(쿼리), FoodIO는 음식을 추천해 준다.

 * FoodIO ver 1.0
 - 음식(쿼리 / 검색어)에 많이 일치하는 문자와 일치하는 항목을 사용자에게 추천 
 - (사용자 : 프라이드 치킨 → FoodIO : 치킨 볶음밥)

 * FoodIO ver 2.0 : wide 모델 적용
 - 쿼리에 동시 발생 대상 레이블과 상관 관계가 있는지 파악 후 기억(추천)
 - (사용자 : 프라이드 치킨 → FoodIO : 치킨 볶음밥, 치킨과 와플, ....)
 [![wide](/assets/img/post/2018-01-16-GCP-cloudML/5.png)](#)

 * FoodIO ver 3.0 : deep 모델 적용
 - TensorFlow Deep Feed-Forward Neural Network 교육하여, 저 차원 밀도 표현(벡터)가 임베팅 공간에서 서로 가깝게 있는 쿼리와 
 쿼리를 일치 시켜 일반화
 - (사용자 : 프라이드 치킨 → FoodIO : 치킨 볶음밥, 새우 볶음밥, ... / FoodIO : 치킨과 와플, 후라이드 치킨, 버거, ...)
 [![deep](/assets/img/post/2018-01-16-GCP-cloudML/6.png)](#)

 * FoodIO ver 4.0 : wide & deep 모델 적용
 - 이전 버전에서 때때로 너무 많이 일반화 되고, 무관한 요리를 추천하게 되는 현상이 발생
 - wide & deep 모델을 사용하여, wide 모델과 deep 모델이 유사한 항목(동일한 추천)을 일반화 하여 사용자가 원하는 음식을 추천 
 [![wide & deep2](/assets/img/post/2018-01-16-GCP-cloudML/7.png)](#)

## Clone the example repo
 
 Google Cloud Shell에서 repo를 복사하겠다.
 > cloudshell_open --repo_url "https://github.com/googlecloudplatform/cloudml-samples"; --page "editor" --open_in_editor "census/estimator"

 Shell을 실행한 위치에 cloudml-samples 폴더가 생성된 것을 확인 할 수 있다. 만약에 Shell에서 튕기더라도, 아래의 이미지와 같이 이동하여
 명령어를 실행하면 된다.
  [![cloud shell](/assets/img/post/2018-01-16-GCP-cloudML/8.png)](#)

## Run your training job in the cloud

 Project ID, Bucket, Region 환경 셋팅을 한다.
 > PROJECT_ID=$(gcloud config list project --format "value(core.project)") 
 > BUCKET_NAME=${PROJECT_ID}-mlengine 
 > REGION=us-central1

 위의 설정 내역을 확인하기 위해서는 아래와 같이 실행한다.
 > echo $PROJECT_ID
 > echo $BUCKET_NAME
 > echo $REGION

 버킷을 생성하여, Adult Data 와 ML이 학습한 결과를 저장할 예정이다.
 > gsutil mb -l $REGION gs://$BUCKET_NAME
 
 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/9.png)](#)

 생성된 버킷에 Adult Data (train / eval)을 저장겠다.
 > gsutil -m cp gs://cloudml-public/census/data/* gs://$BUCKET_NAME/data

 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/10.png)](#)

 train 과 eval data를 바라보도록 환경 설정을 한다.
 > TRAIN_DATA=gs://$BUCKET_NAME/data/adult.data.csv 
 > EVAL_DATA=gs://$BUCKET_NAME/data/adult.test.csv

 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/11.png)](#)

## 클라우드에서 단일 인스턴트 트레이너 실행

 Job Name, Output 경로 환경 설정 한다. 
 > JOB_NAME=census1 
 > OUTPUT_PATH=gs://$BUCKET_NAME/$JOB_NAME 
 
 예측 Job 실행
 > gcloud ml-engine jobs submit training $JOB_NAME --job-dir $OUTPUT_PATH --runtime-version 1.4 --module-name trainer.task --package-path trainer/ --region $REGION -- --train-files $TRAIN_DATA --eval-files $EVAL_DATA --train-steps 5000 --verbosity DEBUG

 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/13.png)](#)

 예측 Job log 확인
 > gcloud ml-engine jobs stream-logs $JOB_NAME

 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/14.png)](#)

 Google Cloud Shell이 아닌, Google Cloud Platform에서도 확인이 가능하다. Menu > ML 엔진 > 작업 > 로그 보기
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/15.png)](#)

 Output 저장
 > gsutil ls -r $OUTPUT_PATH

 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/16.png)](#)

 TensorBoard 실행
 > tensorboard --logdir=$OUTPUT_PATH --port=8080

 - 실행 결과
 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/17.png)](#)
 
 실행된 결과 페이지에서 http://cs-6000-devshell-vm-32b33e7a-e588-4dca-9dbc-38159f7b45e1:8080 을 클릭하면 새탭에서
 TensorBoard가 출력된다.

 [![버킷 생성](/assets/img/post/2018-01-16-GCP-cloudML/18.png)](#)

 


