---
layout: post
title: <분석방법론> 코호트 분석 파헤치기
categories: 데이터분석
tags: [데이터분석, 코호트분석]
---


<div class="tip">
이번 시리즈에서는 실전 데이터 분석을 다룬다. 주로 SaaS 기업에서 행해지는 널리 알려진 여러 기법들을 소개하고, 실제 구현해보는 시간을 갖는다.

코호트 분석(Cohort Analysis)이 그 시작이다.
</div>

## 1. 코호트 분석이란? Cohort Analysis
코호트(동질 집단)란 무엇일까? **특정 기간동안 공통된 특성이나 경험을 공유하는** 사용자 집단을 의미한다. 코로나 발생 초기에, 병원에서 코호트 격리를 한다는 기사가 많이 나왔는데 거기서 사용되는 의미와 동일하다.

코호트 분석(동질 집단 분석)은 이러한 유사 성질을 갖는 코호트를 만들어, **시간에 따른** 각 코호트의 행동과 여러 지표들을 분석할 때 사용하는 기법이다.

여러 출처로부터 다양한 정의를 내리자면,
- [코호트 분석](https://ko.wikipedia.org/wiki/%EC%BD%94%ED%98%B8%ED%8A%B8_%EC%97%B0%EA%B5%AC)은 동종 집단이 나타내는 시간적 변모 양태를 분석하여 예측하고자 하는 연구이다.
- [코호트 분석](https://www.beusable.net/blog/?p=4355#:~:text=%EC%BD%94%ED%98%B8%ED%8A%B8%20%EB%B6%84%EC%84%9D%EC%9D%80%20%EC%82%AC%EC%9A%A9%EC%9E%90%EB%A5%BC,%EC%A0%81%EA%B7%B9%EC%A0%81%EC%9C%BC%EB%A1%9C%20%ED%99%9C%EC%9A%A9%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.)은 사용자를 기간에 따라 그룹으로 분류하여 그룹의 행동과 유지율을 분석할 때 활용하는 기법이다.
- [코호트 분석](https://mixpanel.com/ko/blog/cohort-analysis/)은 시간 경과에 따라 유저 그룹을 추적하는 방법이다.

여기서 중요한 점은, **특정 기간 경험을 공유한다**는 점과, **유사 속성을 갖는다**라는 점이다.

### 코호트 테이블
이제 실제 코호트 분석에 쓰이는 테이블을 살펴보자. [출처](https://analytics.google.com/analytics/web/#/report/visitors-cohort/a54516992w87479473p92320289/)는 GA에 있는 샘플 차트이고, 조건은 아래와 같다.

- 코호트 유형은 '획득 날짜'이다. 즉, 최초로 사이트에 접속한 날짜를 기준으로 코호트를 만들겠다는 의미이다.
- 코호트의 크기는 '일별'이다. 일 단위로 코호트를 분리하겠다는 의미이다.
- 측정 항목은 사용자 유지율이고, 7일간의 데이터를 살펴본다.

![Cohort Table](/public/img/da_01_01.png){: width="100%" height="100%"}

위 이미지에서 행(날짜 데이터)이 코호트이다. 간단히 해석해보자.

- 4월 10일, 1714명의 사용자가 최초로 접속했다. (사실 정말로 최초는 아닐테지만, 코호트의 시작점이라 그렇다).
- 4월 10일에 방문한 고객 중, 다음 날(+1일 열) 3.68%의 고객만이 재방문했다(유지되었다).
- 4월 10일에 방문한 고객 중, 6일 뒤(+6일 열), 0.88%의 고객만이 재방문했다.
- 4월 14일, 2149명의 사용자가 최초로 접속했다. (신규 유저)
- 4월 14일에 방문한 고객 중, 다음 날(+1일 열) 3.82%의 고객만이 재방문했다.

그럼 4월 14일에 방문한 총 고객은 몇 명일까? 2149명일까? 정답은 '그렇지 않다'이다. 코호트에 표기된 숫자는 새롭게 획득된 사용자 수를 의미한다. 즉, 4월 14일에 새롭게 유입된 고객이 2149명이라는 의미이다. 코호트 테이블만을 보고 4월 14일의 전체 방문자는 알 수 없다. 다만, 코호트 내의 값으로 어느정도 계산은 할 수 있다. 4월 14일의 방문자는 코호트 내의 아래 방문자들로 구성된다.

- 4월 14일의 신규 유입
- 4월 13일의 +1일 유저들 (재방문)
- 4월 12일의 +2일 유저들 (재방문)
- 4월 11일의 +3일 유저들 (재방문)
- 4월 10일의 +4일 유저들 (재방문)
- 그리고 코호트 기간을 벗어나는, 이전 방문자들 중 재방문자들

### 코호트 테이블의 구현
여기서 짚고 넘어가야 할 부분이 있다. 코호트 분석에서 집계 방식은 크게 두 가지가 있다.

1. 코호트의 유입 날짜를 기준으로 하는 것
2. 코호트의 유입 날짜와 직전 기간을 기준으로 하는 것 (Rolling retention)

예를 들어, 사용자 A가 4월 10일의 코호트에 속하는데, 4월 11일과 4월 13일에 방문했다고 해보자. 1번 방식으로는 4월 11일, 4월 13일이 모두 집계되지만 2번 방식에서는 4월 11일이 마지막 집계가 된다. 왜냐하면 4월 12일에는 방문을 하지 않았기 때문에, 직전 기간이 12일이 되는 13일도 집계가 되지 않는 것임.

이에 대한 자세한 내용을 찾아보려 했는데, 구체적으로 다룬 내용을 찾지 못했다. 나중에 찾게 되면 추가하겠음. GA는 1번 방식을 택하고 있다.

## 2. 코호트 분석의 의미
코호트 분석이 왜 중요한걸까? 개인적으로는 행동 분석(Behavioral Analytics)을 기반으로 하는 분석 기법들이 대부분 동일한 목적을 갖는다고 생각하는데, 데이터를 전체로 보는 것이 아닌 부분으로 나누어 보아야 행동 분석의 의미가 있기 때문이다.

- 이벤트를 기반으로 데이터를 나누게 되면, **_퍼널 분석_**이나 **_AARRR_**과 같이 고객의 각 행동(이벤트)을 단계별로 나누어 각 단계를 면밀히 관찰할 수 있다.
- 사용자를 기반으로 데이터를 나누게 되면, **_일반적인 세그먼트_**나 세그먼트의 일종인 **_코호트 분석_**, **_RFM 분석_**처럼 유사한 고객별로 면밀히 관찰할 수 있다.

이벤트를 중심으로 살펴보든, 사용자를 중심으로 살펴보든 유사한 특성을 지닌 단계(그룹)별로 나누어 보는 게 데이터 분석이 훨씬 용이하기 때문이다. 더 나아가, 사용자를 기반으로 타겟을 쪼개고 그 타겟별로 이벤트 기반 분석을 할 수 있다. 즉, 세그먼트별로 퍼널이 어떻게 다른지도 비교해 볼 수 있고 더 세분화된 분석이 가능하다. 이러한 세분화된 분석은 맞춤 마케팅을 가능하게 해준다.

좀 더 직접적인 의미를 찾자면, 코호트 분석은 고객의 유지율(그리고 이탈률)을 분석하는 데 탁월한 지표이다. 사용자가 감소하는 시기를 포착하여 개선방안을 제시할 수 있기 때문이다. 예를 들어, 사용자가 급격히 이탈하는 지점이 발생하면 그 시기에 대한 조사를 통해 문제가 있는 부분을 개선할 수 있고, 푸시 알림 등을 통해 이탈한 고객을 다시 유입시킬 수도 있다.

## 3. 코호트 분석 해보기

이제 실제 데이터셋을 통해 코호트 테이블을 그려보고, 간단한 분석을 진행해보자. 

### 1) 데이터 소개
캐글에서 데이터를 가져왔고, [여기](https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop)서 다운로드 받을 수 있다. '2019-Oct' 부터 '2020-Feb' 까지 5개 파일 모두 다운받으면 된다.

이 과정이 귀찮은 사람을 위해 이미 정제한 데이터도 올려두겠다. [여기를 눌러서 다운로드하기](https://raw.githubusercontent.com/ud803/ud803.github.io/main/public/data/ecommerce_cosmetics_sampled.csv)

데이터는 화장품을 다루는 이커머스가 출처이며, '상품 클릭 view', '장바구니 담기 cart', '장바구니 제외 remove_from_cart', '구매 purchase' 네 가지 이벤트에 대한 정보를 담고 있다. 아쉬운 점은 단순히 상품 클릭이 아닌, 사이트에 최초 접속한 레코드가 없다는 점인데 그냥 감안하고 진행하도록 하겠다.

### 2) 데이터 정제

여기에는 2019년 10월부터 2020년 2월까지 총 5개월의 데이터가 담겨있다. 하지만 개별 데이터가 너무 크기 때문에, 각 월별로 고유한 1000명의 데이터만 뽑아서 샘플링한 데이터를 쓴다.

더 구체적으로, 고유한 1000명에 더해 이전 기간의 코호트를 유지하기 위해 앞선 기간의 유저는 그대로 유지하는 방식을 택한다. 즉,

- 파일별 1000명을 샘플링하여, 해당 1000명의 데이터를 모두 가져온다.
- 이전 기간의 사용자를 기록해서, 이후 파일에서도 동일한 사용자가 나오면 포함한다.


```python
import pandas as pd
import random

whole_target_pool = set()

for file in ['2019-Dec', '2019-Nov', '2019-Oct', '2020-Feb', '2020-Jan']:
    target = pd.read_csv('{}.csv'.format(file))
    sample_targets = random.sample(list(target.user_id.unique()), 1000)

    whole_target_pool.update(sample_targets)

    # 1000명의 샘플에 있거나, 이미 뽑은 앞선 목록에 있으면 타겟으로 넣는다.
    new_target = target[target.user_id.isin(sample_targets) | target.user_id.isin(whole_target_pool)]

    print(len(new_target.user_id.unique()), new_target.shape)
    new_target.to_csv('{}-1000.csv'.format(file), index=False)
```

    1000 (9418, 9)
    1141 (16677, 9)
    1241 (17031, 9)
    1270 (21948, 9)
    1482 (27129, 9)


이제 5개의 파일을 하나로 합쳐서, 최종 파일을 만든다. 앞서 말한대로 이미 정제된 파일은 위에 링크를 달아두었다.


```python
whole_df = pd.DataFrame()

for file in ['2019-Dec', '2019-Nov', '2019-Oct', '2020-Feb', '2020-Jan']:
    target = pd.read_csv('{}-1000.csv'.format(file))
    whole_df = pd.concat([whole_df, target])

print(whole_df.shape)
whole_df.head(3)
```

    (92203, 9)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>event_time</th>
      <th>event_type</th>
      <th>product_id</th>
      <th>category_id</th>
      <th>category_code</th>
      <th>brand</th>
      <th>price</th>
      <th>user_id</th>
      <th>user_session</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-12-01 04:09:27 UTC</td>
      <td>view</td>
      <td>5724233</td>
      <td>1487580005092295511</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>14.60</td>
      <td>504201396</td>
      <td>10c40773-05d3-4fd8-af4c-16e8c9999bf4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-12-01 05:18:49 UTC</td>
      <td>view</td>
      <td>5900639</td>
      <td>1487580005713052531</td>
      <td>NaN</td>
      <td>ingarden</td>
      <td>4.44</td>
      <td>502359395</td>
      <td>8dad7a10-5dc3-4c94-9880-bedbf53eeb1a</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-12-01 05:46:02 UTC</td>
      <td>view</td>
      <td>5651938</td>
      <td>1487580012902088873</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.33</td>
      <td>574133915</td>
      <td>4c7bf0a0-5f68-414c-b126-93b372b26a65</td>
    </tr>
  </tbody>
</table>
</div>




```python
whole_df.to_csv("ecommerce_cosmetics_sampled.csv", index=False)
```

### 3) SQLite에 데이터 쌓기

다른 방식으로 진행해도 되지만, 일반적으로 데이터가 DB에 들어있다고 가정하고 SQL을 통해 코호트 테이블의 기초 데이터를 만들고자 한다.

여기서는 데이터베이스로 SQLite을 사용한다. SQLite은 이름 그대로 정말 라이트하게 사용할 수 있는 SQL DB 엔진이다. 별도의 설정도 필요 없고, 그냥 설치하고 곧바로 파이썬 소스로 데이터를 넣어주면 된다. 나는 맥OS를 쓰는데, 이미 설치가 되어 있어 생략했다. [여기](https://www.sqlite.org/index.html)서 최신 버전을 다운받자.

관계형 DB에서 가장 먼저 할 일은, DB 서버에 연결하고, 개별 DB에 접속하는 일이다. SQLite은 별도로 서버 개념이 없기 때문에, 실제로 코드를 돌리는 곳에 데이터를 저장한다.

아래서 쓰인 SQLite 관련 도큐멘테이션은 [여기](https://docs.python.org/3/library/sqlite3.html)를 참고하자. 별도로 다루지는 않는다.


```python
import sqlite3

# 일반적으로 connect는 host에 연결하는 행위이지만, SQLite은 곧바로 DB에 연결한다.
con = sqlite3.connect('ecommerce.db')

# 명령어를 실행해주기 위한 커서 선언
cur = con.cursor()

# event 테이블을 만들어준다. 일단 모두 text 타입으로 선언한다.
cur.execute("""
    CREATE TABLE events
        (event_time text, event_type text, product_id text, category_id text, category_code text, brand text, price text, user_id text, user_session text)
""")

import pandas as pd

# 아까 만든 데이터를 불러온다.
data = pd.read_csv('ecommerce_cosmetics_sampled_10000.csv')

lists = []
# 튜플 형태로 리스트에 넣어준다
for idx, row in data.iterrows():
    lists.append(tuple([*row.values]))

# executemany를 통해 한 번에 데이터를 넣는다
cur.executemany("""INSERT INTO events VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)""",lists)

# 트랜잭션을 수행하는 모든 함수는 반드시 커밋을 해줘야 실제 트랜잭션이 수행된다
con.commit()

# 연결을 닫는다
con.close()
```


이제 데이터가 생성되었는지 확인해 볼 차례다. 위에서 SQLite을 설치했다면, 터미널을 열고 해당 소스를 실행시킨 위치로 이동하자. 즉, `ecommerce.db` 파일이 생성된 위치로 가면 된다.

해당 위치에서 아래 명령어를 실행한다.

```shell
sqlite3 ecommerce.db #이제 sqlite3 DB에 접속했다

# sqlite에서는 시스템 명령어를 쓸 때 '.'을 앞에 붙인다
sqlite> .tables #전체 테이블 목록을 읽어온다
events #아까 생성한 events 테이블이 있어야 정상이다

sqlite> .headers on #출력 시 헤더를 보여주도록 설정을 바꾼다

sqlite> SELECT COUNT(*) FROM events; #테이블의 전체 행 개수를 불러온다
95806 #랜덤 샘플링이기 때문에 값은 약간씩 다르다

sqlite> SELECT * FROM events LIMIT 1;
# 결과가 출력된다

sqlite> .exit #혹은 .quit으로 쉘을 종료한다
```

데이터가 쌓인 것을 확인했다. 이번 글의 중점은 코호트 분석이므로 데이터를 탐색하고, 들여다보고, 빈 값을 채우는 등의 프로세스는 생략한다.

### 4) SQL로 데이터 기반 만들기

이제 실제 데이터를 만들어보자. 그 전에, 어떤 코호트 분석을 할 것인지 정하고 가자.

- 코호트의 유형은? 최초 상품 클릭(view) 기준 코호트
- 코호트의 크기는? 1주 단위
- 코호트의 측정 항목은? 유지율
- 코호트의 전체 기간은? 9주
- 코호트 집계 방식은? 일반적인 방식. 기준 롤링을 하지 않는다. 즉, 코호트 기준일에만 부합하면 집계한다.

데이터는 어떻게 만들까? 코호트 테이블을 행렬로 볼 때, 각 원소는 (기준 코호트, 기준 코호트로부터의 경과일)의 쌍이라고 할 수 있다. 

즉, (코호트1, 코호트1 +1주), (코호트1, 코호트1 +2주)... 를 기준으로 그룹을 만든 것이 코호트 테이블이 된다.


```python
con = sqlite3.connect('ecommerce.db')
cur = con.cursor()

result = con.execute("""
WITH 
base AS (
-- 문자열을 날짜로 바꿔주기 위한 용도
SELECT
    user_id,
    -- '주간'만 빼내는데, 연도가 바뀌면 계산이 틀어지기 때문에 현재 연도에서 가장 낮은 연도인 2019년을 뺀 만큼에 52를 곱한 값을 더해준다
    -- 즉, 2019년 마지막 주는 52가 되고, 2020년의 첫 주는 1 + (2020-2019)*52 = 53이 된다
    STRFTIME('%W', DATE(SUBSTR(event_time, 1, 10))) + (STRFTIME('%Y', DATE(SUBSTR(event_time, 1, 10))) - 2019) * 52 AS event_week
FROM events
WHERE event_type = 'view'
-- 9개의 주간으로 나누기 위해 기간을 제한해준다
AND STRFTIME('%W', DATE(SUBSTR(event_time, 1, 10))) + (STRFTIME('%Y', DATE(SUBSTR(event_time, 1, 10))) - 2019) * 52 <= 47
)
,first_view AS (
-- 우선 사용자별로 최초 유입 월을 찾는다. 이게 코호트가 된다.
SELECT
    user_id,
    MIN(event_week) AS cohort
FROM base
GROUP BY user_id
)
,joinned AS (
-- 기존 데이터에 위에서 찾은 코호트를 조인해준다. 그리고 기존 이벤트 월과 코호트 월의 차이를 빼준다
SELECT
    t1.user_id,
    t2.cohort,
    t1.event_week,
    t1.event_week - t2.cohort AS week_diff
FROM base t1
LEFT JOIN first_view t2
ON t1.user_id = t2.user_id
)

-- (기준 코호트, 기준 코호트로부터의 경과주) 쌍을 만들어 고유한 사용자 수를 센다
SELECT
    cohort,
    week_diff,
    COUNT(DISTINCT user_id)
FROM joinned
GROUP BY cohort, week_diff
ORDER BY cohort ASC, week_diff ASC
""").fetchall()
```

이제 데이터 추출이 끝났다. 판다스로 가져와서 행렬로 만들고, 추가적인 시각화까지 진행해보자.

### 5) 코호트 테이블 만들기


```python
# 데이터프레임으로 만들고
# 컬럼의 이름을 바꿔주고
# 피벗 기능을 이용해 코호트 테이블 형태로 만들어준다
# 빈 값은 0으로 채운다
pivot_table = pd.DataFrame(result)\
    .rename(columns={0: 'cohort', 1: 'duration', 2: 'value'})\
    .pivot(index='cohort', columns='duration', values='value')\
    .fillna(0)

pivot_table
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>duration</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
    </tr>
    <tr>
      <th>cohort</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>39</th>
      <td>3402.0</td>
      <td>576.0</td>
      <td>456.0</td>
      <td>398.0</td>
      <td>355.0</td>
      <td>248.0</td>
      <td>226.0</td>
      <td>261.0</td>
      <td>253.0</td>
    </tr>
    <tr>
      <th>40</th>
      <td>2790.0</td>
      <td>405.0</td>
      <td>296.0</td>
      <td>230.0</td>
      <td>204.0</td>
      <td>185.0</td>
      <td>189.0</td>
      <td>186.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>41</th>
      <td>2324.0</td>
      <td>315.0</td>
      <td>198.0</td>
      <td>141.0</td>
      <td>116.0</td>
      <td>140.0</td>
      <td>143.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>42</th>
      <td>2185.0</td>
      <td>244.0</td>
      <td>147.0</td>
      <td>137.0</td>
      <td>137.0</td>
      <td>136.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>43</th>
      <td>2113.0</td>
      <td>196.0</td>
      <td>114.0</td>
      <td>132.0</td>
      <td>107.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>44</th>
      <td>2101.0</td>
      <td>180.0</td>
      <td>193.0</td>
      <td>145.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>45</th>
      <td>1988.0</td>
      <td>233.0</td>
      <td>148.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>46</th>
      <td>2229.0</td>
      <td>277.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2323.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



위 상태는 유지되는 고유 유저수를 보여주고 있기 때문에 비율로 바꾸어 주겠다. 또한 판다스에서 제공하는 `background_gradient`를 이용해 히트맵을 그려준다.


```python
# 첫 번째 기간으로 나누어 비율로 만들어주고
# %가 나오도록 포맷팅을 해주고
# 색을 입혀준다

round(pivot_table.div(pivot_table[0], axis='index'), 2)\
    .style.format({k: '{:,.0%}'.format for k in pivot_table})\
    .background_gradient(cmap ='Blues', axis=None, vmax=0.2) 
```




<style  type="text/css" >
#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col0,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col0{
            background-color:  #08306b;
            color:  #f1f1f1;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col1{
            background-color:  #0d57a1;
            color:  #f1f1f1;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col2{
            background-color:  #3b8bc2;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col1,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col1{
            background-color:  #4a98c9;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col4{
            background-color:  #6aaed6;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col2,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col2{
            background-color:  #a6cee4;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col3{
            background-color:  #94c4df;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col1{
            background-color:  #2171b5;
            color:  #f1f1f1;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col2,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col1{
            background-color:  #5ba3d0;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col2,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col8,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col1,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col2,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col7,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col8{
            background-color:  #f7fbff;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col1{
            background-color:  #2e7ebc;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col2,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col1,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col1,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col2{
            background-color:  #7fb9da;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col6,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col3,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col5,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col3{
            background-color:  #b7d4ea;
            color:  #000000;
        }#T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col4,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col2,#T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col4{
            background-color:  #c6dbef;
            color:  #000000;
        }</style><table id="T_3037e81c_c47c_11ec_b8ea_7210a9758097" ><thead>    <tr>        <th class="index_name level0" >duration</th>        <th class="col_heading level0 col0" >0</th>        <th class="col_heading level0 col1" >1</th>        <th class="col_heading level0 col2" >2</th>        <th class="col_heading level0 col3" >3</th>        <th class="col_heading level0 col4" >4</th>        <th class="col_heading level0 col5" >5</th>        <th class="col_heading level0 col6" >6</th>        <th class="col_heading level0 col7" >7</th>        <th class="col_heading level0 col8" >8</th>    </tr>    <tr>        <th class="index_name level0" >cohort</th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>    </tr></thead><tbody>
                <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row0" class="row_heading level0 row0" >39</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col0" class="data row0 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col1" class="data row0 col1" >17%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col2" class="data row0 col2" >13%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col3" class="data row0 col3" >12%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col4" class="data row0 col4" >10%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col5" class="data row0 col5" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col6" class="data row0 col6" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col7" class="data row0 col7" >8%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row0_col8" class="data row0 col8" >7%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row1" class="row_heading level0 row1" >40</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col0" class="data row1 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col1" class="data row1 col1" >15%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col2" class="data row1 col2" >11%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col3" class="data row1 col3" >8%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col4" class="data row1 col4" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col5" class="data row1 col5" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col6" class="data row1 col6" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col7" class="data row1 col7" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row1_col8" class="data row1 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row2" class="row_heading level0 row2" >41</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col0" class="data row2 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col1" class="data row2 col1" >14%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col2" class="data row2 col2" >9%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col3" class="data row2 col3" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col4" class="data row2 col4" >5%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col5" class="data row2 col5" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col6" class="data row2 col6" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col7" class="data row2 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row2_col8" class="data row2 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row3" class="row_heading level0 row3" >42</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col0" class="data row3 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col1" class="data row3 col1" >11%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col2" class="data row3 col2" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col3" class="data row3 col3" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col4" class="data row3 col4" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col5" class="data row3 col5" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col6" class="data row3 col6" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col7" class="data row3 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row3_col8" class="data row3 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row4" class="row_heading level0 row4" >43</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col0" class="data row4 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col1" class="data row4 col1" >9%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col2" class="data row4 col2" >5%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col3" class="data row4 col3" >6%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col4" class="data row4 col4" >5%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col5" class="data row4 col5" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col6" class="data row4 col6" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col7" class="data row4 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row4_col8" class="data row4 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row5" class="row_heading level0 row5" >44</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col0" class="data row5 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col1" class="data row5 col1" >9%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col2" class="data row5 col2" >9%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col3" class="data row5 col3" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col4" class="data row5 col4" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col5" class="data row5 col5" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col6" class="data row5 col6" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col7" class="data row5 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row5_col8" class="data row5 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row6" class="row_heading level0 row6" >45</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col0" class="data row6 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col1" class="data row6 col1" >12%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col2" class="data row6 col2" >7%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col3" class="data row6 col3" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col4" class="data row6 col4" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col5" class="data row6 col5" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col6" class="data row6 col6" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col7" class="data row6 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row6_col8" class="data row6 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row7" class="row_heading level0 row7" >46</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col0" class="data row7 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col1" class="data row7 col1" >12%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col2" class="data row7 col2" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col3" class="data row7 col3" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col4" class="data row7 col4" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col5" class="data row7 col5" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col6" class="data row7 col6" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col7" class="data row7 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row7_col8" class="data row7 col8" >0%</td>
            </tr>
            <tr>
                        <th id="T_3037e81c_c47c_11ec_b8ea_7210a9758097level0_row8" class="row_heading level0 row8" >47</th>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col0" class="data row8 col0" >100%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col1" class="data row8 col1" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col2" class="data row8 col2" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col3" class="data row8 col3" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col4" class="data row8 col4" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col5" class="data row8 col5" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col6" class="data row8 col6" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col7" class="data row8 col7" >0%</td>
                        <td id="T_3037e81c_c47c_11ec_b8ea_7210a9758097row8_col8" class="data row8 col8" >0%</td>
            </tr>
    </tbody></table>



### 6) 해석하기. 가져갈 수 있는 의미

이제 코호트 테이블을 읽어보자. 왼쪽의 39는 2019년의 39번째 주를 의미한다. 그리고 오른쪽의 duration에서 1~8은 코호트 주로부터 그만큼 경과한 주를 의미한다.

- 모든 코호트에서 1기간이 경과했을 때의 유지율은 평균 10%가 조금 넘는다.
- 코호트39에서 8기간이 경과했을 때의 유지율은 9%이다. 즉 39번째 주에 방문하여 상품을 본 고객 중, 8주 이후에도 똑같이 행동한 고객은 그 중 9%라는 의미이다. (단, 이 고객이 1~7주차 사이에도 그랬는지는 알 수 없다.)
- 3기간이 경과했을 때의 유지율은 4~8% 정도로 매우 낮은 편이다.

전반적으로, 유지율이 매우 낮은 것을 알 수 있다. 물론 상품을 본(view) 이벤트를 기준으로 하기 때문에 사이트 방문이라고는 볼 수 없지만, 일반적으로 이커머스에서 유지율(재방문율)은 낮아도 20% 정도인 업체가 많다. 이런 경우 대부분 유입 트래픽 광고를 엄청나게 뿌려대지만 당장 그 때만 고객이 유입하고 이후에 리타겟팅이 안 되는 경우가 많다.

또 어떤 데이터를 읽을 수 있을까? 테이블만 봐서는 사실 잘 와닿지 않는다. 우리는 이 분석을 통해 얻어가고자 하는 바를 미리 정의하지 않았기 때문이다. 미리 정의하지 않았기 때문에 코호트 분석을 통해 찾아내고자 하는 내용이 없었고, 그에 맞는 평가 지표도 선정되지 않았다. 

### 7) 또다른 코호트 접근

이번에는 10월 한 달 동안 사용자가 본 브랜드별로 코호트를 만든다고 가정해보자. 마케터인 나는 브랜드별로 사용자의 성향이 다르고, 방문 성향도 다를 것이라고 생각한다. 따라서 브랜드에 따라 코호트를 나누었다.

<div class="exclamation">
일반적으로 코호트는 상호배타적(mutually exclusive)이다. 즉, 코호트끼리 겹치지 않는다. 아래 예시에서는 번거로움을 피하고자 그런 과정을 거치지 않았는데, 이 점은 주의하기 바람!
</div>


```python
result_2 = con.execute("""
WITH 
base AS (
-- 문자열을 날짜로 바꿔주기 위한 용도
SELECT
    user_id,
    STRFTIME('%W', DATE(SUBSTR(event_time, 1, 10))) + (STRFTIME('%Y', DATE(SUBSTR(event_time, 1, 10))) - 2019) * 52 AS event_week,
    event_type,
    brand
FROM events
-- 9개의 주간으로 나누기 위해 기간을 제한해준다
WHERE STRFTIME('%W', DATE(SUBSTR(event_time, 1, 10))) + (STRFTIME('%Y', DATE(SUBSTR(event_time, 1, 10))) - 2019) * 52 <= 47
AND brand IS NOT NULL
AND event_type = 'view'
AND DATE(SUBSTR(event_time, 1, 10)) >= '2019-10-01'
AND DATE(SUBSTR(event_time, 1, 10)) <= '2019-10-31'
)
,first_view AS (
SELECT
    user_id,
    brand AS cohort,
    MIN(event_week) AS cohort_time
FROM base
GROUP BY user_id, brand
)
,joinned AS (
SELECT
    t1.user_id,
    t2.cohort,
    t1.event_week,
    t1.event_week - t2.cohort_time AS week_diff
FROM base t1
LEFT JOIN first_view t2
ON t1.user_id = t2.user_id
AND t1.brand = t2.cohort
)

SELECT
    cohort,
    week_diff,
    COUNT(DISTINCT user_id)
FROM joinned
GROUP BY cohort, week_diff
ORDER BY cohort ASC, week_diff ASC
""").fetchall()
```


```python
# 데이터프레임으로 만들고
# 컬럼의 이름을 바꿔주고
# 피벗 기능을 이용해 코호트 테이블 형태로 만들어준다
# 빈 값은 0으로 채운다
pivot_table_2 = pd.DataFrame(result_2)\
    .rename(columns={0: 'cohort', 1: 'duration', 2: 'value'})\
    .pivot(index='cohort', columns='duration', values='value')\
    .fillna(0)\
    .sort_values(by=[0], ascending=False)\
    .iloc[:10, :]

# 상위 10개만 잘랐다
pivot_table_2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>duration</th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
    <tr>
      <th>cohort</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>runail</th>
      <td>1641.0</td>
      <td>182.0</td>
      <td>104.0</td>
      <td>67.0</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>irisk</th>
      <td>1354.0</td>
      <td>126.0</td>
      <td>77.0</td>
      <td>52.0</td>
      <td>25.0</td>
    </tr>
    <tr>
      <th>masura</th>
      <td>840.0</td>
      <td>99.0</td>
      <td>52.0</td>
      <td>26.0</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>grattol</th>
      <td>730.0</td>
      <td>106.0</td>
      <td>50.0</td>
      <td>29.0</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>estel</th>
      <td>649.0</td>
      <td>28.0</td>
      <td>13.0</td>
      <td>8.0</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>jessnail</th>
      <td>595.0</td>
      <td>38.0</td>
      <td>13.0</td>
      <td>6.0</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>ingarden</th>
      <td>560.0</td>
      <td>61.0</td>
      <td>28.0</td>
      <td>15.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>kapous</th>
      <td>512.0</td>
      <td>24.0</td>
      <td>14.0</td>
      <td>5.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>bpw.style</th>
      <td>467.0</td>
      <td>46.0</td>
      <td>30.0</td>
      <td>13.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>uno</th>
      <td>466.0</td>
      <td>34.0</td>
      <td>19.0</td>
      <td>9.0</td>
      <td>5.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 첫 번째 기간으로 나누어 비율로 만들어주고
# %가 나오도록 포맷팅을 해주고
# 색을 입혀준다

round(pivot_table_2.div(pivot_table_2[0], axis='index'), 2)\
    .style.format({k: '{:,.0%}'.format for k in pivot_table_2})\
    .background_gradient(cmap ='Blues', axis=None, vmax=0.2) 
```




<style  type="text/css" >
#T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col0,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col0{
            background-color:  #08306b;
            color:  #f1f1f1;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col1,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col1{
            background-color:  #5ba3d0;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col1,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col2{
            background-color:  #b7d4ea;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col1,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col2{
            background-color:  #d0e1f2;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col4{
            background-color:  #eef5fc;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col1{
            background-color:  #7fb9da;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col3{
            background-color:  #e3eef9;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col1{
            background-color:  #4a98c9;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col3,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col3{
            background-color:  #d9e8f5;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col1{
            background-color:  #2171b5;
            color:  #f1f1f1;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col1{
            background-color:  #a6cee4;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col4,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col4{
            background-color:  #f7fbff;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col2,#T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col1{
            background-color:  #c6dbef;
            color:  #000000;
        }#T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col1{
            background-color:  #6aaed6;
            color:  #000000;
        }</style><table id="T_32648b2c_c47c_11ec_b8ea_7210a9758097" ><thead>    <tr>        <th class="index_name level0" >duration</th>        <th class="col_heading level0 col0" >0</th>        <th class="col_heading level0 col1" >1</th>        <th class="col_heading level0 col2" >2</th>        <th class="col_heading level0 col3" >3</th>        <th class="col_heading level0 col4" >4</th>    </tr>    <tr>        <th class="index_name level0" >cohort</th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>        <th class="blank" ></th>    </tr></thead><tbody>
                <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row0" class="row_heading level0 row0" >runail</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col0" class="data row0 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col1" class="data row0 col1" >11%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col2" class="data row0 col2" >6%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col3" class="data row0 col3" >4%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row0_col4" class="data row0 col4" >1%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row1" class="row_heading level0 row1" >irisk</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col0" class="data row1 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col1" class="data row1 col1" >9%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col2" class="data row1 col2" >6%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col3" class="data row1 col3" >4%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row1_col4" class="data row1 col4" >2%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row2" class="row_heading level0 row2" >masura</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col0" class="data row2 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col1" class="data row2 col1" >12%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col2" class="data row2 col2" >6%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col3" class="data row2 col3" >3%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row2_col4" class="data row2 col4" >1%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row3" class="row_heading level0 row3" >grattol</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col0" class="data row3 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col1" class="data row3 col1" >15%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col2" class="data row3 col2" >7%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col3" class="data row3 col3" >4%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row3_col4" class="data row3 col4" >2%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row4" class="row_heading level0 row4" >estel</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col0" class="data row4 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col1" class="data row4 col1" >4%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col2" class="data row4 col2" >2%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col3" class="data row4 col3" >1%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row4_col4" class="data row4 col4" >0%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row5" class="row_heading level0 row5" >jessnail</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col0" class="data row5 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col1" class="data row5 col1" >6%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col2" class="data row5 col2" >2%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col3" class="data row5 col3" >1%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row5_col4" class="data row5 col4" >1%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row6" class="row_heading level0 row6" >ingarden</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col0" class="data row6 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col1" class="data row6 col1" >11%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col2" class="data row6 col2" >5%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col3" class="data row6 col3" >3%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row6_col4" class="data row6 col4" >1%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row7" class="row_heading level0 row7" >kapous</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col0" class="data row7 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col1" class="data row7 col1" >5%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col2" class="data row7 col2" >3%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col3" class="data row7 col3" >1%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row7_col4" class="data row7 col4" >0%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row8" class="row_heading level0 row8" >bpw.style</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col0" class="data row8 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col1" class="data row8 col1" >10%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col2" class="data row8 col2" >6%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col3" class="data row8 col3" >3%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row8_col4" class="data row8 col4" >1%</td>
            </tr>
            <tr>
                        <th id="T_32648b2c_c47c_11ec_b8ea_7210a9758097level0_row9" class="row_heading level0 row9" >uno</th>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col0" class="data row9 col0" >100%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col1" class="data row9 col1" >7%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col2" class="data row9 col2" >4%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col3" class="data row9 col3" >2%</td>
                        <td id="T_32648b2c_c47c_11ec_b8ea_7210a9758097row9_col4" class="data row9 col4" >1%</td>
            </tr>
    </tbody></table>



이번에는 어떨까? 브랜드 코호트마다 확실히 유지율이 다른 것을 알 수 있다. 특정 브랜드를 찾았던 고객은 계속해서 방문을 하고, 다른 브랜드들은 그렇지 않다. 여기에는 브랜드의 속성이 주는 재구매주기라든지 여러 요소들이 작용할 것이다. 브랜드를 떠나 화장품의 카테고리별로 보는 것도 좋은 분석이 될 수 있다.

## 4. 마치며

코호트 분석에 대해 정리해두었더 내용들을 글로 옮길 수 있어 마음이 편안하다. 😃 아무쪼록 누구에게든 도움이 되었으면 좋겠고, 앞으로도 데이터 분석과 관련한 글들을 계속해서 작성할 예정이다!

생각중인 주제로는 퍼널 분석, RFM 세그멘테이션, LTV, 전환기여(FirstTouch, LastTouch) 등이 있는데, 다양하게 다루어보도록 하겠다 :)
