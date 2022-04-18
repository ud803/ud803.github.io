---
layout: post
title: <Test> Test 2
categories: Test2
tags: [Test2]
---
  
# 1. 데이터 정제

앞으로의 고객 데이터 분석에 사용할 토대를 만들기 위해, 캐글에서 가져온 데이터를 샘플링한다.

https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop?select=2020-Jan.csv

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
    new_target = target[target.user_id.isin(sample_targets) | target.user_id.isin(whole_target_pool)]

    print(len(new_target.user_id.unique()))
    print(new_target.shape)
    new_target.to_csv('{}-1000.csv'.format(file), index=False)
```

    1000
    (9814, 9)
    1145
    (24577, 9)
    1240
    (21055, 9)
    1256
    (21378, 9)
    1516
    (27234, 9)



```python
whole_df = pd.DataFrame()

for file in ['2019-Dec', '2019-Nov', '2019-Oct', '2020-Feb', '2020-Jan']:
    target = pd.read_csv('{}-1000.csv'.format(file))
    whole_df = pd.concat([whole_df, target])
```


```python
print(whole_df.shape)
whole_df.head()
whole_df.to_csv("ecommerce_cosmetics_sampled.csv", index=False)
```

    (104058, 9)





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
      <td>2019-12-01 00:38:08 UTC</td>
      <td>view</td>
      <td>5861591</td>
      <td>1487580009143992338</td>
      <td>NaN</td>
      <td>lador</td>
      <td>2.22</td>
      <td>555562373</td>
      <td>be19f337-027d-48a8-b394-bc3a76f9a560</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-12-01 01:08:47 UTC</td>
      <td>view</td>
      <td>5861591</td>
      <td>1487580009143992338</td>
      <td>NaN</td>
      <td>lador</td>
      <td>2.22</td>
      <td>555562373</td>
      <td>c5136529-aa38-4acc-abe6-e208bfa36ba5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-12-01 03:39:32 UTC</td>
      <td>view</td>
      <td>5816564</td>
      <td>1487580005553668971</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>7.46</td>
      <td>566664583</td>
      <td>3d2a5a00-a114-1a4e-455d-4170f08f715b</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-12-01 03:39:42 UTC</td>
      <td>view</td>
      <td>5800960</td>
      <td>1487580005553668971</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.21</td>
      <td>566664583</td>
      <td>3d2a5a00-a114-1a4e-455d-4170f08f715b</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-12-01 03:40:12 UTC</td>
      <td>view</td>
      <td>5800963</td>
      <td>1487580005553668971</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.21</td>
      <td>566664583</td>
      <td>3d2a5a00-a114-1a4e-455d-4170f08f715b</td>
    </tr>
  </tbody>
</table>
</div>



# 2. 데이터 적재하기


```python
import sqlite3

con = sqlite3.connect('ecommerce.db')

cur = con.cursor()

cur.execute("""
    CREATE TABLE events
        (event_time text, event_type text, product_id text, category_id text, category_code text, brand text, price text, user_id text, user_session text)
""")

import pandas as pd

data = pd.read_csv('ecommerce_cosmetics_sampled.csv')

lists = []
for idx, row in data.iterrows():
    # print(tuple([*row.values]))
    lists.append(tuple([*row.values]))

cur.executemany("""INSERT INTO events VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)""",lists)

con.commit()
con.close()
```

# 3. 데이터 가져와서 판다스 작업하기

## 1) 획득 기반 코호트 - 주차별로


```python

```
