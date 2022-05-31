---
title: "[논문 리뷰] The use of machine learning algorithms in recommender systems: A systematic review"
categories:
  - Review Paper
tags:
  - Study
  - Machine Learning
  - Review paper
  - Recommender Systems
---

# [The use of machine learning algorithms in recommender systems: A systematic review](https://arxiv.org/ftp/arxiv/papers/1511/1511.05263.pdf)     
     
## Abstract     
최근 추천 시스템은 인공지능 분야의 머신러닝 알고리즘을 사용하고 있다.    
→ 수없이 많은 접근법과 변주가 존재하기 때문에 해당 서비스의 추천 시스템에 적합한 알고리즘을 고르는 것은 쉽지 않다.   
본 논문은 추천 시스템에서 머신러닝 알고리즘의 사용을 분석하고 소프트웨어 엔지니어링 연구를 위한 연구 기회를 식별하는 문헌에 대한 체계적인 검토를 제시한다.   
**이 연구는 Bayesian과 Decision Tree 알고리즘이 상대적으로 단순(simplicity)하기 때문에 추천 시스템에 널리 사용되고 있다고 결론 짓는다.**     
     
## Theoretical Background            
### Recommender System    
추천 시스템(RS)은 사용자에게 아이템 추천을 할 때 AI 방법을 사용한다. 첫 번째 RS는 1992년 기업 Tapestry에서 나타났는데, __협업 필터링(*collaborative filtering*__ 이라는 용어를 사용했고, 이는 아직까지도 RS에서 사용되고 있다.   
추천 시스템은 3개의 메인 카테고리로 나뉜다.: **collaborative, content-based, hybrid filtering**    
협업(**collaborative**) 접근 방식을 사용하는 RS는 추천을 위한 정보를 처리할 때 **사용자 데이터**를 고려한다. 사용자 정보를 통해 시스템은 동일한 아이템 선호도를 공유하는 사용자를 식별한 다음, 유사한 사용자가 선택한 아이템을 제안한다.    
콘텐츠 기반(**content-based**) 필터링 접근 방식을 사용하는 RS는 접근할 수 있는 **아이템 데이터**에 따라 추천한다. 사용자가 특정 아이템을 선택하면 RS는 해당 아이템에 대한 정보를 수집하고, 유사한 속성을 가진 아이템에 대한 데이터베이스를 검색한다. 그 후 검색 결과가 권장 사항으로 사용자에게 반환된다.       
하이브리드 필터링(**hybrid filtering**) 접근법은 두 개의 이전 분류를 결합하여 추천하는 RS 방식이다.    
collaborative나 hybrid filtering 접근 방식을 사용할 때, RS는 추천을 발전시키기 위하여 반드시 사용자 정보를 수집해야 한다. 수집과정은 명시적일수도 암묵적일수도 있다.      
사용자에게 관심 있는 항목을 제공하는 일반적인 추천 프로세스 외에도 추천은 다른 방식들로도 제안될 수 있다.       
Trust-based recommendations은 사용자 간의 신뢰 관계를 고려한다. 신뢰 관계는 소셜 네트워크에서 친구 또는 팔로워로 연결해주는 링크로, 신뢰할 수 있는 친구를 기반으로 한 추천은 신뢰 링크가 없는 추천보다 더 가치가 있다.   
Context-aware recommendations는 사용자의 컨텍스트를 기반으로 한다. 컨텍스트는 사용자의 현재 상태에 대한 정보의 집합이다. 처리할 컨텍스트 정보의 양이 많기 때문에 context aware recommendations는 어려운 연구 분야로 꼽힌다.     
Risk-aware recommendations는 context-aware recommendations의 하위 집합으로 중요한 정보를 사용할 수 있는 컨텍스트를 고려한다. 잘못된 결정이 사용자에게 피해를 줄 수 있기 때문에 risk-aware이라고 불린다.      
          
### Machine Learning     
오늘날, 문헌에 제안된 수많은 ML 알고리즘이 있다. ML 알고리즘은 학습 과정에 사용되는 접근 방식에 따라 분류된다. 4가지의 메인 분류 방식이 있다.: **supervised, unsupervised, semi-supervised, reinforcement learning**    
지도 학습(**supervised learning**)은 알고리즘이 훈련 데이터와 정확한 답과 함께 제공될 때 발생한다.    
비지도 학습(**unsupervised learning**)에서 ML 알고리즘은 훈련 세트를 가지고 있지 않다. 실제 세계에서 약간의 데이터를 제공받고 알고리즘 스스로 그 데이터로부터 학습해야 한다. 비지도 학습 알고리즘은 대부분 데이터에서 숨겨진 패턴을 찾는데 중점을 둔다.    
준지도 학습(**semi-supervised learning**)은, 알고리즘이 누락된 정보가 있지만 여전히 그 정보로부터 학습할 필요가 있는 훈련 세트에서 작동해야 할 때, 발생한다. 예를 들어 ML 알고리즘이 영화 등급과 함께 제공되는 경우, 모든 사용자가 모든 영화를 평가한 것은 아니므로 일부 누락된 정보가 있다. 이런 상황에서 준지도 학습 알고리즘은 불완전 데이터로 학습하고 결론을 도출하게 된다.     
강화 학습(**reinforcement learning**)은 생각 주체 또는 환경에 의해 주어진 외부 피드백을 기반으로 학습할 때 발생한다. 긍정적 피드백으로 이어지는 동작은 학습/반복하고, 부정적 피드백으로 이어지는 동작은 회피하도록 하는 알고리즘이다.     
                   
## Systematic Review Results            
