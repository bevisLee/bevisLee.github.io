---
layout: post
title:  "PR12-060: Deep Neural Networks for YouTube Recommendations - YouTube"
comments: true
categories : [Data Science/DL]
tags: [Paper, Recommendation]

---

## 들어가기

 딥러닝에 학습을 깊이있게 하고자, 작년부터 논문을 공부하는 것을 시도해봤다. 다만, 많은 경험이 없고 혼자서 하기엔 많은 어려움이 있어
 작년부터 시작된 PR12의 유튜브 동영상을 참고하여 내가 관심 갖는 주제에 대해서는 정리하여 포스팅을 하고자 하였다.
 이번에 선택한 논문은 YouTube에서 사용된 실제 논문으로 추천시스템 공부를 계속 진행해왔기 때문에 제목을 보자 마자 시청한 논문이다.

 * youtube - [youtube 링크](https://www.youtube.com/watch?v=V6zixdCIOqw&feature=youtu.be)
 * slide - [slideshare 링크](https://www.slideshare.net/keunbongkwak/deep-neural-networks-for-youtube-recommendations)
 * blog - [blog 링크](http://keunwoochoi.blogspot.kr/2016/09/deep-neural-networks-for-youtube.html)
 * papers - [papers 링크](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/45530.pdf)


## 개요

 * Candidate Generation Model
 * Ranking Model
 * A/B Test를 통한 실제 환경 개선

# 실제 상황에서 겪게 되는 이슈들

 * Scale : 엄청난 양의 데이터와 제한된 컴퓨팅 파워
 * Freshness : 새로운 컨텐츠의 빠른 적용
 * Noise : 낮은 meta data 퀄리티, Implicit Feedback 위주 데이터

# 전체 모델

<img src='https://www.dropbox.com/s/mi6v7st3y45s7qc/1.png?raw=1'>
 * 단계 1 (candidate generation) : 추천할 후보 비디오를 몇 백개 내외로 뽑아내고
 * 단계 2 (ranking) : 다시 그중에서 자세하게 순위를 매겨서 추천


# 단계 1 : Candidate Generation

 * 문제 정의
	* 추천 : 엄청나게 클래스가 많은 multiclass 분류 문제로 재정의 (Extreme multiclass classification)
	* user, context 가 주어지면 특정 시간에 이 비디오를 볼 확률을 구함 

	<img src='https://www.dropbox.com/s/z6k34p3exb7djvy/2.png?raw=1'>
             
   * u_j : context embedding
	* u : user embedding 
	* 이 학습 과정에서 사용자가 누른 '좋아요' 같은 정보는 사용하지 않고, 비디오를 끝까지 "봤는지/아닌지만 사용"
 * Embeddings
	* Video Embedding과 Search Token Embedding
	* Dense Vector : 256 (CBOW에서 영감을 받음 / CBOW - Continuous Bag-Of-Words : 소스 컨텍스트에서 타켓 단어를 예측) 
	
	<img src='https://www.dropbox.com/s/72r07rinbork4mw/3.png?raw=1'>
    [ CBOW 예시 - http://solarisailab.com/archives/374 ]
	* Backpropagation을 통해서 embedding도 함께 학습
 * Combiner
	* 고정된 사이즈의 Input으로 바꿈
	* 다양한 방법을 사용해봤는데 average가 성능이 제일 좋음 : sum, componete-wise max, average 등 사용해봄
 * Additional Features
	* 단순하게 옆에 전부 concatenate 해버림
 * ReLu Stack
	* Fully connected layer + ReLu 사용
	* Output으로 User Embedding이 나옴
 * Softmax Prediction : 미리 계산
	* 각 video별 가중치가 output으로 나옴
	* loss function 으로는 각 class (비디오)마다 (binary) cross-entropy를 사용
	* Negative Sampling : 엄청난 데이터와 제한된 컴퓨팅 환경으로 모든 환경은 성능 이슈가 있어, 수천개 샘플을 뽑아 학습 진행
 * Serving
	* 상위 N개의 비디오 출력
	* Previous systems at YouTube relied on hashing [24]와 같이 hashing 사용
	* Dot-Product Space에서 Nearest Neighbor 사용하여 가장 가까운 아이템을 추천
 * Heterogeneous signals
	* example age : Video age (업로드 날짜/시간 기준) -> 새로 나온 비디오가 성능이 좋게 나타남 / 사용자는 신규 비디오를 선호
	* 성별, 나이 : 0,1 사용    
		<img src='https://www.dropbox.com/s/y2sv33owlfypdhq/4.png?raw=1'>  
	* Label and context selection
	* 모든 비디오 시청 이력을 확인해야 bias가 없음 : youtube + 외부 사이트 / youtube는 추천 영상을 시청하기 때문에 bias 발생
	* 학습에 사용할 이용자별 영상 횟수를 fix 해야 heavy user에 치우치지 않음
	* 새로운 검색 쿼리에 즉시 추천엔진에 반영하지 않음 : 예 - deep learning만 즐겨보던 user가 갑자기 가수 psy를 검색했을 때, deep learning psy를 보여주면 적절하지 않은 결과
	* 비대칭적인 감상 패턴을 적용해서 학습 : 예 - user는 종류별로 검색하고 감상하는 것이 아니라, 스타크래프트 -> deep learning -> kpop 과 같이 비대칭적인 감상 패턴을 지님   
		<img src='https://www.dropbox.com/s/femplbs90klh86v/5.png?raw=1'>
         
# 단계 2 : Rank
 * 문제 정의
	* 여기에서 더 많은 feature를 이용해 영상과 이용자의 관계를 구함 / deep neural network 이용
	* A/B TEst를 통해 계속 업데이트 됨 / 평가 잣대는 추천된 횟수 (= 화면에 뜬 횟수) 대비 평균 감상 시간
 * Embeddings
	* 앞의 모델과 같은 ID space, 같은 Embedding 사용
	* 직전에 시청한 비디오는 직업 embedding / 이외는 average embedding
	* continuous feature 들은 normalize 해서 사용 : root , 그대로, 제곱 하였을 때 알아서 좋은 feature 선택
 * Modeling Expected Watch Time
	* 추천된 영상을 얼마나 오랫동안 볼지 예측하는 것을 목표로 함
	* 감상시간은 안 봤으면 0, 봣으면 본 시간을 값으로 넣음
	* 감상 시간으로 가중치를 줌 (weighted logistic regression : 새로 정의)   
		<img src='https://www.dropbox.com/s/56qzjoerhjoaspv/6.png?raw=1'>
   * Feature Engineering
	* 각 feature 들은 어느 정도 가공해줘야 함
	* 특히나 시간 연속성을 가진 데이터들은 summarizing 필요
	* 사용자 이용패턴, 추천했는데 보지 않았던 영상 등도 활용
	* 가장 좋은 feature 는 비슷한 비디오에 대한 유저의 반응   
		<img src='https://www.dropbox.com/s/x1on10uzoen8i7e/7.png?raw=1'>

# 결론
 * 이 모델로 기존의 방법(Matrix Factorisation) 보다 성능을 많이 향상
 * 모든 것을 딥러닝으로 하기는 쉽지 않음
 * "영상의 나이"가 성능을 크게 개선
 * 감상 시간별 가중치를 주는 것도 개선점이 큼 (weighted logistic regression > CTR)

