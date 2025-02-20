---
title: "[Machine Learning Design Patterns] Chapter 1: 머신러닝 디자인 패턴의 필요성"
categories:
  - Machine Learning Design Patterns
tags:
  - Study
  - Machine Learning
  - MLOps
gallery:
gallery:
  - url: /assets/images/머신러닝디자인패턴.jpg
    image_path: /assets/images/머신러닝디자인패턴.jpg
    alt: "placeholder image 1"
    title: "머신러닝 디자인패턴"
---
![머신러닝디자인패턴](https://user-images.githubusercontent.com/104043279/164156828-b2e17094-7cdc-456e-84c2-321fada963de.jpg)     
             
## CHAPTER 1 머신러닝 디자인 패턴의 필요성

**디자인 패턴** 
- 각 패턴은 우리 환경에서 반복해서 발생하는 문제와 그 문제에 대한 솔루션의 핵심을 설명한다.
- 패턴을 활용하면 같은 방식을 두 번 고안하지 않고서도 패턴의 솔루션을 백만 번 사용할 수 있다.
  
  
**머신러닝**: 
- 데이터에서 학습하는 모델을 구축하는 프로세스
- *프로그램의 동작 방식을 알려주는 명시적 규칙을 작성하는 기존의 프로그래밍과는 대조적*
- 머신러닝 모델: 데이터에서 패턴을 학습하는 알고리즘
  - 순방향 신경망(feed-forward neural network) 모델, 선형 회귀(linear regression) 모델, 결정 트리(decision tree), 클러스터링(clustering) 모델 등
  
  
**순방향 신경망(feed-forward neural network)**
  - 일반적으로 '신경망'이라고 불림
  - 수많은 뉴런을 포함한 여러 계층이 각각 정보를 분석하고 처리한 다음, 처리한 정보를 다음 계층으로 보냄  
  -> 이러한 방식으로 최종 계층, 즉 출력층에 도달한 정보를 통해 예측 생성
  - 2개 이상의 은닉층(hidden layer. 입력층이나 출력층이 아닌 계층)이 있는 신경망은 딥러닝으로 분류됨
  - **선형 모델(linear model): 입력층과 출력층만 있는 신경망**
  
  
**결정 트리(decision tree)**
- 데이터를 사용하여 다양한 분기를 가진 경로의 집합을 만드는 머신러닝 모델
  
  
**클러스터링(clustering)**
- 데이터 내의 부분집합 간의 유사성을 찾고 식별된 패턴을 사용하여 클러스터로 데이터를 그룹화
  
  
**지도 학습(supervised learning)**
- 데이터에 대한 실측 검증 라벨(ground truth label)을 미리 알고 있는 문제에 적용
- 새로운 데이터가 들어왔을 때 자동으로 라벨을 달 수 있도록 충분히 학습시키는 것이 지도학습의 목적
- 분류 / 회귀
  
**비지도 학습(unsupervised learning)**
- 데이터에 라벨이 없을 때 적용
- 비지도 학습의 목적:
  - 데이터를 자연스럽게 그룹화(클러스터링) 
  - 정보를 압축(차원 축소) 
  - 연관 규칙을 찾을 수 있는 모델 구축
  
  
---
  
- 헷갈리는 용어 정리  
에폭(epoch): 학습 데이터셋에 대한 각 반복  
하이퍼파라미터(hyperparameter): 랜덤 포레스트 모델의 트리 수 등  
테이블 데이터(tabular): = 구조화된 데이터  
특징 가공(feature engineering): = 전처리  
인스턴스(instance): 예측을 위해 모델로 보내는 데이터 항목.   
추론(inference): 이미지 및 텍스트 분류 모델에 '예측'이라는 말을 사용하기는 조금 어색하기 때문에 이럴 경우 추론이라는 단어를 사용. 통계 용어인 추론과 정확하게 의미가 일치하지는 않지만 ML업계에서는 일반적으로 통용.
  

---
  

더 많은 데이터를 수집할 수 없을 때 데이터 균형을 이해하면 모델을 설계하는데 도움이 된다.
  

---
  

**예측(prediction. 모델에 새 데이터를 전송하고 출력을 사용하는 프로세스)**은 아직 배포되지 않은 로컬 모델에서 예측값을 생성하는 것은 물론, 배포된 모델에서 예측값을 가져오는 것 모두를 의미한다.
- 배포된 모델의 예측에는 온라인 예측과 배치 예측의 두 가지가 있다.
  - 온라인 예측(online prediction)은 거의 실시간으로 적은 수의 예측값을 얻고자 할 때 사용되며 지연 시간이 짧을수록 좋다.
  - 배치 예측(batch prediction)은 오프라인에서 대규모 데이터 집합에 대한 예측을 생성하는 것을 의미한다. (다만, 온라인이어도 업데이트 주기에 따라 배치 예측으로 보는 경우도 있다.) 온라인 예측보다 오래 걸리며, 예측을 사전에 계산*ex.추천 시스템*하고 대규모 새 데이터 샘플에서 모델의 예측을 분석할 때 유용하다.
  
학습 데이터 수집, 특징 가공, 학습, 모델 평가 프로세스는 프로덕션 파이프라인과 별도로 처리하는 경우가 많다. 이렇게 별도로 처리할 때에는 모델의 새 버전을 학습시키기에 충분한 추가 데이터가 있다고 결정할 때마다 모델을 다시 평가한다.  
반면, 이와 같은 과정을 프로덕션 파이프라인 내에서 소화하는 경우에는 새로운 데이터를 지속적으로 수집할 수 있어야 하며 학습 또는 예측을 위해 모델로 보내기 전에 이 데이터를 즉시 전처리해야 한다. 이를 **스트리밍(streaming)**이라고 한다.  
스트리밍 데이터를 처리하려면 특징 가공, 학습, 평가, 예측을 수행하기 위해 여러 단계에 걸친 시스템이 필요하다. 이러한 시스템을 **머신러닝 파이프라인(machine learning pipeline)**이라고 부른다.
  

---
  

**ML설계에 영향을 미치는 요인**
1. 데이터의 품질
  - 데이터 정확도(accuracy): 데이터 특징의 정확도 & 데이터 라벨의 정확도
  - 데이터 완전성(completeness): 학습 데이터에 각 라벨의 다양한 표현이 포함되도록 해야 함
  - 데이터 일관성(consistency)
  - 데이터 적시성(timeliness): 데이터의 사건이 발생한 시점과 데이터베이스에 추가된 시점 사이의 지연 시간
2. 재현성
  - 머신러닝 모델에는 근본적으로 무작위성이 내재되어 있다. ML모델을 처음 학습시킬 때 가중치는 임의의 값으로 초기화된다. 모델이 학습을 반복하다 보면 이러한 가중치는 특정한 값으로 수렴된다. -> 동일한 학습 데이터를 동일한 모델의 코드에 입력했다고 해도 두 모델은 학습이 끝나면 약간 다른 결과를 생성한다.  
  ==> 재현성의 문제 발생
  - 재현성 문제를 해결하는 일반적인 방법은 학습을 실행할 때마다 동일한 무작위성이 적용되도록 모델에서 사용하는 시드값을 설정하는 것이다.
  - ML모델 학습에는 재현성을 보장하기 위해 수정해야 하는 몇 가지 요소가 있다. 
    - *학습 데이터, 학습 및 검증을 위한 데이터셋 생성에 사용하는 분할 메커니즘, 데이터 준비와 모델 하이퍼파라미터, 배치 크기와 학습률 등*
  - 재현성은 머신러닝 프레임워크의 종속성, 모델 학습 환경의 영향도 받는다.
3. 데이터 드리프트
  - 머신러닝 모델이 입력과 출력 간의 관계를 유지할 수 있는지와 모델 예측이 사용 중인 환경을 정확하게 반영하는지에 대한 문제를 표현하는 용어
  - 데이터 드리프트 문제를 해결하려면 학습 데이터셋을 지속적으로 업데이트하고, 모델을 재학습하고, 모델이 특정 입력 데이터 그룹에 할당하는 가중치를 수정해야 한다.
4. 확장
5. 다양한 목표