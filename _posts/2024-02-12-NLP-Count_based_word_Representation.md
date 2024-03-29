---
title: "[자연어 처리] 카운트 기반의 단어 표현"
author: <author_id>
date: 2024-02-12 18:00:00 +09:00
categories: [NLP, Count based word representation]
math: true
tag: [자연어처리, 카운트 기반 단어 표현, NLP, Count based word representation, BoW, DTM, TF-IDF]
toc: true
---

> 이 포스트는 [**딥 러닝을 이용한 자연어 처리 입문**](https://wikidocs.net/book/2155)을 읽고 공부 겸 정리한 글입니다.

## 단어 표현 방법

자연어 처리에서 텍스트를 표현하는 방법은 다양하다. 여러 방법들은 크게 두 표현으로 나뉘는데 **국소 표현**과 **분산 표현**이다. 국소 표현은 단어 그 자체를 특정한 값에 매핑하고, 분산 표현은 주변을 참고하여 그 단어를 표현하는 방법이다. 비슷한 의미로 국소 표현을 이산 표현, 분산 표현을 연속 표현이라 칭하기도 한다. 둘의 큰 차이점은 분산 표현은 주변을 참고하여 단어를 표현하므로 뉘앙스를 표현할 수 있지만 국소 표현을 할 수 없다. 

![Desktop View](/assets/img/wordrepresentation.png)
_단어 표현의 카테고리화_

이 포스트에서는 정보 검색과 텍스트 마이닝 분야에서 주로 사용되는 카운트 기반의 텍스트 표현 방법인 DTM(Document Term Matrix)과 TF-IDF(Term Frequency-Inverse Document Frequency)를 다룬다. 위 방법을 이용하면 통계적인 접근 방법을 통해 문서 내에서 단어의 중요도, 문서의 핵심어 추출, 검색엔진에서 검색 순위 결정, 문서들 간의 유사도를 알 수 있다.

## Bag of Words(BoW)

**Bag of Words**란 말 그대로 단어들의 가방이란 뜻으로, 단어의 등장 순서를 고려하지 않는 빈도수 기반의 단어 표현 방법이다. 구현하는 방법은 간단하다. 

1. 각 단어에 고유한 정수 인덱스를 부여 # 단어 집합 생성
2. 각 인덱스에 토큰의 등장 횟수를 기록하는 벡터(리스트) 생성

BoW를 사용한다는 것은 문서에서 단어가 얼마나 등장했는 지를 보고 이를 수치화하여 단어의 중요도를 보기 위함이다. 따라서 불용어처리는 BoW에서 정확도를 높일 수 있는 전처리 방법이다. 또한 어떻게 토큰화를 하냐에 따라 같은 단어도 다르게 인식하여 카운트가 될 수도 있기 때문에 적절한 토큰화도 성능을 좌우할 수 있다.

> 예를 들어, 띄어쓰기를 기준으로 낮은 수준의 토큰화를 한다면 조사에 따라 "그녀는"와 "그녀가"를 다른 단어로 인식할 수 있다.

이를 지원하는 라이브러리로는 사이킷 런의 CountVectorizer가 있다.

## 문서 단어 행렬(Document-Term Matrix, DTM)

**문서 단어 행렬(Document-Term Matrix, DTM)**이란 다수의 문서의 등장하는 모든 단어들의 빈도를 각 문서마다 카운트하여 행렬로 표현한 것으로, BoW표현을 다수의 문서에 대해서 행렬로 표현한 것이다. 예를 들어, 3개의 문서가 있다고 하자.

1. 갖고 싶은 옷
2. 갖고 싶은 물건
3. 싸고 좋은 옷

띄어쓰기 단위의 토큰화를 진행하고 DTM으로 표현하면 아래와 같다.

|   | 갖고 | 물건 | 싶은 | 싸고 | 옷 | 좋은 |
|:--|:-----|:----|:-----|:-----|:--|:-----|
| 1 | 1    | 0   | 1    | 0    | 1 | 0    |
| 2 | 1    | 1   | 1    | 0    | 0 | 0    |
| 3 | 0    | 0   | 0    | 1    | 1 | 1    |

DTM은 간단하고 구현하기에도 쉽지만 몇 가지 한계들도 존재한다.

다수의 문서에 대해 DTM으로 표현할 때, 원-핫 벡터와 같이, 코퍼스가 방대하다면 행렬에서 각 벡터의 크기가 증가하며 대부분의 값이 0을 가질 수도 있다. 원-핫 벡터와 DTM같이 대부분의 값이 0인 표현을 희소 벡터 또는 희소 행렬이라고 하는데, 이는 많은 저장 공간과 높은 계산 복잡도를 요구하여 비효율적일 수 있다. 앞서 배운 여러 전처리 기법을 이용하여 단어 집합의 크기를 줄이는 것이 최선이다.

또한, 단순 빈도 수 기반 접근이기에 문서에서 적절한 전처리를 하지 않았다면, 중요한 단어와 불필요한 단어를 혼동하여 잘못된 결과를 내놓게 될 수도 있다.

>예를 들어, 영어 문서들에 대해 DTM을 만들었다고 하자. 어떤 문서에서도 자주 등장할 수 밖에 없는 the의 빈도수가 여러 문서에서 높다고 해서 그 문서들을 유사하다고 판단해선 안된다.

## TF-IDF(Term Frequency-Inverse Document Frequency)

TF-IDF는 위의 문제점들을 보완하기 위한 개념으로 DTM 내에 각 단어의 중요도를 알 수 있는 가중치이다. TF-IDF를 이용하면 많은 경우에서 DTM보다 좋은 성능을 얻을 수 있고, 주로 문서의 유사도, 문서에서 특정 단어의 중요도를 구하는 작업 등에 쓰일 수 있다.

TF-IDF는 TF와 IDF를 곱한 값을 의미하는데 문서를 d, 단어를 t, 문서의 총 개수를 n이라고 하면 TF-IDF는 아래와 같이 구한다.

1. tf(d,t) : 문서 d에서 단어 t의 등장 횟수. DTM에서 각 단어가 가진 값과 같다.
2. df(t) : 단어 t가 등장한 문서의 수
3. idf(t) : log(n/df(t)+1)
4. TF-IDF(d,t) = tf(d,t) * idf(t) 

> 위의 문서 2에서 물건에 대해 계산해보겠다. tf는 문서 2에서 물건이란 단어가 한번 등장했기에 1이고, df는 물건이란 단어가 문서 2에서만 등장했기에 1을 가진다. 그리고 총 문서가 3개이므로 idf는 log(3/1+1)이다. 따라서 TF-IDF값은 1*log(3/2) = 0.176 이다.

idf에서 log를 취하는 이유는 n이 커질수록 빈도가 적은 희귀 단어들의 idf값이 기하급수적으로 커질 수 있기 때문이다. 또한 분모에 1을 더하는 이유는 df값이 0이 되는 상황을 방지하기 위함이다. 

tf는 말 그대로 하나의 문서에서 특정 단어가 등장하는 빈도이고, idf는 총 문서의 개수에서 특정 단어가 등장하는 문서의 수를 나눈 값이다. 따라서 TF-IDF는 모든 문서에 자주 등장하는 단어의 중요도는 낮고, 특정 문서에서 자주 등장하는 단어가 중요도가 높게 판단한다. TF-IDF 값이 크면 중요도가 높은 것이다. 이를 적용하면 위의 예시에서 the와 같은 불용어는 모든 문서에서 자주 등장하기에 TF-IDF값이 낮게 계산될 것이다.

사이킷런에서는 idf에서 log안의 분자와 분모의 비가 1이 되어 값이 0으로 계산되는 상황을 방지하기 위해 위의 보편적인 식에서 보정한 식을 사용한다. 