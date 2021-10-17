---
# layout: post
title: "[Project] 연희-연남-신촌동 카페 추천기"
categories:
 - textmining
excerpt: "카카오플레이스 평점 및 리뷰 기준으로"
tags: 
 - textmining
 - crawling
 - visualization
 - project
# share: true 
comments: true 
# last_modified_at: 2020-09-05T20:55:00-09:00
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
---




# 1. 데이터 수집하기   

- 데이터는 카카오맵을 기준으로 수집


```python
import folium
import warnings
from selenium import webdriver
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
plt.style.use('seaborn-whitegrid')
warnings.filterwarnings('ignore')
from bs4 import BeautifulSoup
import time
import re
import googlemaps
import missingno as msno
from tqdm import tqdm_notebook
import math
```


```python
# 스타일 변경으로 인해 폰트 다시 설정
plt.rc('font', family='NanumGothic')
plt.rc('font', size=13)
```


```python
driver = webdriver.Chrome('C:/Users\jej_0312_@naver.com/chromedriver_win32/chromedriver.exe')
driver.get('https://map.kakao.com/')
```


```python
# driver.find_element_by_id('search.keyword.bounds').click()    # 현 지도 내 장소 검색
# driver.implicitly_wait(10)
driver.find_element_by_id('search.keyword.query').send_keys('카페')    # 카페 검색
driver.implicitly_wait(10)
time.sleep(1)
driver.find_element_by_id('search.keyword.submit').click()    # 검색 클릭
driver.implicitly_wait(10)
time.sleep(1)
driver.find_element_by_xpath('//*[@id="info.search.place.more"]').click()    # 더 보기 클릭
```


```python
dict_cafe = {'name': [], 'address': [], 'score': [], 'score_cnt': [], 'review_cnt': []}
for pagenum in tqdm_notebook(range(1, 36)):
    try:
        page = pagenum % 5    # 페이지 이동 버튼의 id넘버가 5의 나머지로 되어있다. (1페이지 = p1, 6페이지 = p1, 7페이지 = p2 ...)
        html = driver.page_source
        soup = BeautifulSoup(html, 'lxml')
        for cafenum in range(15): # 한 페이지에 총 15개
            dict_cafe['name'].append(soup.find_all('a', 'link_name')[cafenum].text)
            dict_cafe['address'].append(soup.find_all('p', 'lot_number')[cafenum].text)
            dict_cafe['score'].append(soup.find_all('em', 'num')[cafenum].text)
            dict_cafe['score_cnt'].append(soup.find_all('a', 'numberofscore')[cafenum].text)
            dict_cafe['review_cnt'].append(soup.find_all('a', 'review', 'em')[cafenum].text)
        
        # 페이지가 5페이지씩 나뉘어져있는데(1-5, 6-10..), 마지막 페이지(5의 배수 페이지)에 도착했을 경우, 다음 버튼 클릭
        if page == 0:
            driver.find_element_by_id('info.search.page.next').click()
        else:
            driver.find_element_by_id('info.search.page.no{}'.format(page+1)).click() # 현재 페이지 +1인 페이지 선택
        time.sleep(1)
    except:
        # print('페이지 초과: {}'.format(pagenum))
        continue
```


```python
df_raw = pd.DataFrame(dict_cafe)
```


```python
df_raw.to_csv('source/cafe_in_yeonhee.csv')
```

- 크롤링 과정을 다시 거치지 않기 위해 우선은 여기까지를 저장했다.


```python
df_raw = pd.read_csv('source/cafe_in_yeonhee.csv', index_col=0)
```


```python
df_raw.head()
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>앤트러사이트 연희점</td>
      <td>(지번) 연희동 89-19</td>
      <td>3.5</td>
      <td>40건</td>
      <td>리뷰 135</td>
    </tr>
    <tr>
      <th>1</th>
      <td>콘하스 연희점</td>
      <td>(지번) 연희동 90-1</td>
      <td>2.8</td>
      <td>67건</td>
      <td>리뷰 211</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>(지번) 연희동 90-8</td>
      <td>4.3</td>
      <td>17건</td>
      <td>리뷰 50</td>
    </tr>
    <tr>
      <th>3</th>
      <td>스타벅스 연희DT점</td>
      <td>(지번) 연희동 87-8</td>
      <td>3.7</td>
      <td>20건</td>
      <td>리뷰 22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>(지번) 연희동 130-2</td>
      <td>4.5</td>
      <td>61건</td>
      <td>리뷰 157</td>
    </tr>
  </tbody>
</table>
</div>



# 2. 데이터 전처리   

##  Null 값 삭제   


```python
df_raw.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 505 entries, 0 to 504
    Data columns (total 5 columns):
    name          505 non-null object
    address       501 non-null object
    score         505 non-null float64
    score_cnt     505 non-null object
    review_cnt    505 non-null object
    dtypes: float64(1), object(4)
    memory usage: 23.7+ KB
    

- address가 다른 column들에 비해 4개가 적은 것을 보면 address 열에서 4개의 결측치가 발견되었다.


```python
df_raw[df_raw.address.isnull()]
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>280</th>
      <td>스타벅스 가재울뉴타운점</td>
      <td>NaN</td>
      <td>4.8</td>
      <td>5건</td>
      <td>리뷰 12</td>
    </tr>
    <tr>
      <th>316</th>
      <td>카페마이바움</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0건</td>
      <td>리뷰 0</td>
    </tr>
    <tr>
      <th>407</th>
      <td>다르다</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1건</td>
      <td>리뷰 28</td>
    </tr>
    <tr>
      <th>485</th>
      <td>스타벅스 이대ECC점</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>8건</td>
      <td>리뷰 11</td>
    </tr>
  </tbody>
</table>
</div>



- '카페마이바움'과 '다르다'는 평점이 없으므로 삭제해도 무방할 듯 하다.
- 나머지는 검색하여 값을 넣어주었다.


```python
df_raw.drop(df_raw[(df_raw["name"] == '카페마이바움') | (df_raw["name"] == '다르다')].index, inplace = True)
df_raw[df_raw.address.isnull()]
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>280</th>
      <td>스타벅스 가재울뉴타운점</td>
      <td>NaN</td>
      <td>4.8</td>
      <td>5건</td>
      <td>리뷰 12</td>
    </tr>
    <tr>
      <th>485</th>
      <td>스타벅스 이대ECC점</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>8건</td>
      <td>리뷰 11</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_raw.reset_index(drop=True, inplace=True)
df_raw[df_raw.address.isnull()]
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>280</th>
      <td>스타벅스 가재울뉴타운점</td>
      <td>NaN</td>
      <td>4.8</td>
      <td>5건</td>
      <td>리뷰 12</td>
    </tr>
    <tr>
      <th>483</th>
      <td>스타벅스 이대ECC점</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>8건</td>
      <td>리뷰 11</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_raw.iloc[280,1] = "(지번) 남가좌동 165-1"
df_raw.iloc[483,1] = "(지번) 대현동 11-1"
df_raw[df_raw.address.isnull()]
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
df_raw.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 503 entries, 0 to 502
    Data columns (total 5 columns):
    name          503 non-null object
    address       503 non-null object
    score         503 non-null float64
    score_cnt     503 non-null object
    review_cnt    503 non-null object
    dtypes: float64(1), object(4)
    memory usage: 19.8+ KB
    


## 텍스트 처리   
- address는 '(지번)'을 삭제하고
- 리뷰 건수와 평점 건수는 숫자만 남기자.


```python
for row in df_raw.index:
    df_raw.address[row] = df_raw.address[row][5:]
df_raw.head()
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>앤트러사이트 연희점</td>
      <td>연희동 89-19</td>
      <td>3.5</td>
      <td>40건</td>
      <td>리뷰 135</td>
    </tr>
    <tr>
      <th>1</th>
      <td>콘하스 연희점</td>
      <td>연희동 90-1</td>
      <td>2.8</td>
      <td>67건</td>
      <td>리뷰 211</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>연희동 90-8</td>
      <td>4.3</td>
      <td>17건</td>
      <td>리뷰 50</td>
    </tr>
    <tr>
      <th>3</th>
      <td>스타벅스 연희DT점</td>
      <td>연희동 87-8</td>
      <td>3.7</td>
      <td>20건</td>
      <td>리뷰 22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61건</td>
      <td>리뷰 157</td>
    </tr>
  </tbody>
</table>
</div>




```python
p = re.compile('\D')
for row in df_raw.index:
    df_raw.loc[row, 'score_cnt'] = p.sub('', df_raw.loc[row, 'score_cnt'])
    df_raw.loc[row, 'review_cnt'] = p.sub('', df_raw.loc[row, 'review_cnt'])
df_raw.head()
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>앤트러사이트 연희점</td>
      <td>연희동 89-19</td>
      <td>3.5</td>
      <td>40</td>
      <td>135</td>
    </tr>
    <tr>
      <th>1</th>
      <td>콘하스 연희점</td>
      <td>연희동 90-1</td>
      <td>2.8</td>
      <td>67</td>
      <td>211</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>연희동 90-8</td>
      <td>4.3</td>
      <td>17</td>
      <td>50</td>
    </tr>
    <tr>
      <th>3</th>
      <td>스타벅스 연희DT점</td>
      <td>연희동 87-8</td>
      <td>3.7</td>
      <td>20</td>
      <td>22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61</td>
      <td>157</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_raw['score_cnt'] = df_raw['score_cnt'].astype(int)
df_raw['review_cnt'] = df_raw['review_cnt'].astype(int)
df_raw['score'] = df_raw['score'].astype(float)
df_raw.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 503 entries, 0 to 502
    Data columns (total 5 columns):
    name          503 non-null object
    address       503 non-null object
    score         503 non-null float64
    score_cnt     503 non-null int32
    review_cnt    503 non-null int32
    dtypes: float64(1), int32(2), object(2)
    memory usage: 15.8+ KB
    


```python
# 전처리가 끝났으니 다시 저장하자.
# df_raw = pd.read_csv('source/cafe_in_yeonhee.csv', index_col=0)
```

# 3. 데이터 분석   

- 전처리는 끝났다.


## 별점이 높은 카페 찾기   


```python
df_raw.sort_values('score', ascending=False)
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>409</th>
      <td>스완카페</td>
      <td>남가좌동 211-19</td>
      <td>5.0</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>95</th>
      <td>브이카페 연희점</td>
      <td>연희동 193-10</td>
      <td>5.0</td>
      <td>1</td>
      <td>9</td>
    </tr>
    <tr>
      <th>481</th>
      <td>커피스튜디오</td>
      <td>남가좌동 119-27</td>
      <td>5.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>116</th>
      <td>카페로이</td>
      <td>연희동 163-1</td>
      <td>5.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>435</th>
      <td>매드히피</td>
      <td>연남동 239-44</td>
      <td>5.0</td>
      <td>3</td>
      <td>17</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>302</th>
      <td>제이쇼콜라</td>
      <td>홍은동 410-36</td>
      <td>0.0</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>304</th>
      <td>다향</td>
      <td>연희동 137-13</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>305</th>
      <td>피안타</td>
      <td>연희동 190-21</td>
      <td>0.0</td>
      <td>0</td>
      <td>6</td>
    </tr>
    <tr>
      <th>310</th>
      <td>오빌</td>
      <td>연희동 92-18</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>502</th>
      <td>커피볶는집 명지대점</td>
      <td>남가좌동 342-10</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>503 rows × 5 columns</p>
</div>



- 평가 수가 적은 카페들이 상위에 랭크된다.  
- 따라서 평가 수가 일정 기준 이상인 데이터들을 위주로 분석하자.
- 우선, 평가를 받지 않은 카페들을 제외하고 보자.


```python
plt.hist(df_raw[df_raw.score_cnt != 0].score_cnt)
```




    (array([338.,  32.,   9.,   2.,   2.,   0.,   0.,   0.,   1.,   1.]),
     array([  1. ,  25.8,  50.6,  75.4, 100.2, 125. , 149.8, 174.6, 199.4,
            224.2, 249. ]),
     <a list of 10 Patch objects>)




![png](/img/output_29_1.png)



```python
df_raw[df_raw.score_cnt != 0].score_cnt.describe()
```




    count    385.000000
    mean      12.150649
    std       22.015339
    min        1.000000
    25%        2.000000
    50%        5.000000
    75%       13.000000
    max      249.000000
    Name: score_cnt, dtype: float64



- 평균은 12개, 75%는 13개이다.
- 12번 이상의 평가를 받은 카페들을 대상으로 다시 구해보자.


```python
df_filter1 = df_raw[df_raw.score_cnt >= np.mean(df_raw[df_raw.score_cnt != 0].score_cnt)]
print('평가 개수가 평균 이상인 카페: 총 {}개 중 {}개 ({:.2f}%)'.format(len(df_raw), len(df_filter1), (len(df_filter1) / len(df_raw)*100)))
```

    평가 개수가 평균 이상인 카페: 총 503개 중 99개 (19.68%)
    


```python
df_filter1.sort_values('score', ascending=False).head()
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>89</th>
      <td>땡스오트</td>
      <td>연남동 375-113</td>
      <td>4.8</td>
      <td>17</td>
      <td>233</td>
    </tr>
    <tr>
      <th>303</th>
      <td>하이드미플리즈</td>
      <td>홍제동 330-226</td>
      <td>4.8</td>
      <td>17</td>
      <td>81</td>
    </tr>
    <tr>
      <th>28</th>
      <td>쿳사</td>
      <td>연희동 100-2</td>
      <td>4.8</td>
      <td>13</td>
      <td>4</td>
    </tr>
    <tr>
      <th>189</th>
      <td>미드나잇플레저</td>
      <td>연남동 241-18</td>
      <td>4.7</td>
      <td>14</td>
      <td>35</td>
    </tr>
    <tr>
      <th>20</th>
      <td>르솔레이</td>
      <td>연희동 192-4</td>
      <td>4.7</td>
      <td>15</td>
      <td>36</td>
    </tr>
  </tbody>
</table>
</div>



- 2번째로 나타난 '쿳사'는 리뷰 수가 상당히 적다.
- 리뷰 카운트도 조건으로 설정해서 검색하는 것이 더 합당한 듯 하다.


```python
df_filter1.review_cnt.describe()
```




    count     99.000000
    mean     142.212121
    std      128.119545
    min        1.000000
    25%       53.500000
    50%      104.000000
    75%      200.500000
    max      673.000000
    Name: review_cnt, dtype: float64




```python
plt.hist(df_filter1.review_cnt)
```




    (array([31., 31., 12., 10.,  6.,  3.,  3.,  1.,  1.,  1.]),
     array([  1. ,  68.2, 135.4, 202.6, 269.8, 337. , 404.2, 471.4, 538.6,
            605.8, 673. ]),
     <a list of 10 Patch objects>)




![png](/img/output_36_1.png)


- 25%에 해당하는 값도 53개 정도이기 때문에 꽤나 신뢰할 수 있는 정도인 한 것 같다.
- 따라서 25%를 기준으로 잡고 보자.


```python
df_filter2 = df_filter1[df_filter1.review_cnt >= (df_filter1.review_cnt).quantile(.25)]
print('평점 개수가 평균 이상, 리뷰 개수가 전체의 25% 이상인 카페: 총 {}개 중 {}개 ({:.2f}%)'.format(len(df_filter1), len(df_filter2), (len(df_filter2) / len(df_filter1)*100)))
```

    평점 개수가 평균 이상, 리뷰 개수가 전체의 25% 이상인 카페: 총 99개 중 74개 (74.75%)
    


```python
df_filter2.sort_values('score', ascending=False).reset_index(drop=True).head(10)
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>하이드미플리즈</td>
      <td>홍제동 330-226</td>
      <td>4.8</td>
      <td>17</td>
      <td>81</td>
    </tr>
    <tr>
      <th>1</th>
      <td>땡스오트</td>
      <td>연남동 375-113</td>
      <td>4.8</td>
      <td>17</td>
      <td>233</td>
    </tr>
    <tr>
      <th>2</th>
      <td>루온루온</td>
      <td>연남동 390-77</td>
      <td>4.7</td>
      <td>15</td>
      <td>59</td>
    </tr>
    <tr>
      <th>3</th>
      <td>오랑지</td>
      <td>연남동 390-71</td>
      <td>4.6</td>
      <td>21</td>
      <td>184</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61</td>
      <td>157</td>
    </tr>
    <tr>
      <th>5</th>
      <td>연남온도</td>
      <td>연남동 241-26</td>
      <td>4.5</td>
      <td>15</td>
      <td>77</td>
    </tr>
    <tr>
      <th>6</th>
      <td>테일러커피 서교1호점</td>
      <td>서교동 329-15</td>
      <td>4.4</td>
      <td>39</td>
      <td>174</td>
    </tr>
    <tr>
      <th>7</th>
      <td>러빈허</td>
      <td>동교동 177-12</td>
      <td>4.3</td>
      <td>15</td>
      <td>112</td>
    </tr>
    <tr>
      <th>8</th>
      <td>테일러커피 연남2호점</td>
      <td>연남동 224-57</td>
      <td>4.3</td>
      <td>27</td>
      <td>215</td>
    </tr>
    <tr>
      <th>9</th>
      <td>이미</td>
      <td>동교동 201-10</td>
      <td>4.3</td>
      <td>69</td>
      <td>150</td>
    </tr>
  </tbody>
</table>
</div>



- 하이드미플리즈는 홍제동에 위치해있다.  
  - 근처에 유명한 카페의 수가 적어 비교적 높은 평점을 받은 것이라 예상해본다.  
  - 검색결과, 인스타감성을 저격한 카페 겸 맥주 바였다.

## 가장 유명한 카페 찾기   

- 리뷰 수가 많을수록 사람들이 많이 찾는 카페라 생각하고 찾아보자.


```python
df_raw.sort_values(['score_cnt', 'review_cnt'], ascending=False).reset_index(drop=True).head(10)
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>호밀밭</td>
      <td>창천동 4-77</td>
      <td>3.9</td>
      <td>249</td>
      <td>120</td>
    </tr>
    <tr>
      <th>1</th>
      <td>커피리브레 연남점</td>
      <td>연남동 227-15</td>
      <td>3.3</td>
      <td>209</td>
      <td>112</td>
    </tr>
    <tr>
      <th>2</th>
      <td>테일러커피 서교2호점</td>
      <td>서교동 338-1</td>
      <td>4.2</td>
      <td>108</td>
      <td>198</td>
    </tr>
    <tr>
      <th>3</th>
      <td>클로리스 신촌점</td>
      <td>창천동 13-35</td>
      <td>4.0</td>
      <td>103</td>
      <td>123</td>
    </tr>
    <tr>
      <th>4</th>
      <td>아오이토리</td>
      <td>서교동 327-17</td>
      <td>3.5</td>
      <td>94</td>
      <td>249</td>
    </tr>
    <tr>
      <th>5</th>
      <td>수카라</td>
      <td>서교동 327-9</td>
      <td>4.2</td>
      <td>88</td>
      <td>134</td>
    </tr>
    <tr>
      <th>6</th>
      <td>옥루몽 신촌본점</td>
      <td>대신동 50-3</td>
      <td>3.1</td>
      <td>75</td>
      <td>11</td>
    </tr>
    <tr>
      <th>7</th>
      <td>이미</td>
      <td>동교동 201-10</td>
      <td>4.3</td>
      <td>69</td>
      <td>150</td>
    </tr>
    <tr>
      <th>8</th>
      <td>콘하스 연희점</td>
      <td>연희동 90-1</td>
      <td>2.8</td>
      <td>67</td>
      <td>211</td>
    </tr>
    <tr>
      <th>9</th>
      <td>연남살롱</td>
      <td>연남동 504-33</td>
      <td>3.6</td>
      <td>62</td>
      <td>117</td>
    </tr>
  </tbody>
</table>
</div>



- 아까와는 달리, 신촌 부근의 카페들이 많이 등장했다.
  - 연남동이나 연희동에 비해 유동인구가 많아 더 잘 알려진 곳이라 판단했다.  

## 인지도 지표   
- 이번에는 리뷰 수와 평가 수가 모두 높은 카페들을 찾아보려고 한다.
  - 평가보다 리뷰를 남기는 것이 더 정성적인 방법의 평가이며, 리뷰를 통해 또 다른 소비자를 끌어낼 수 있다는 점에서 리뷰수에 더 큰 점수를 부여하자.
  - 평가 수를 10%의 가산점이라 생각하고 계산한다.  
 
**인지도 지수 = 평가 수(70%) + 리뷰 수(100%)**  
- 우선, 각 기준들에 똑같은 범위의 값을 주기 위해, 스케일링을 해준 후에 사용하자.


```python
plt.hist(df_raw.review_cnt)
```




    (array([395.,  61.,  19.,  12.,   6.,   4.,   3.,   1.,   1.,   1.]),
     array([  0. ,  67.3, 134.6, 201.9, 269.2, 336.5, 403.8, 471.1, 538.4,
            605.7, 673. ]),
     <a list of 10 Patch objects>)




![png](/img/output_45_1.png)



```python
df_raw.review_cnt.describe()
```




    count    503.000000
    mean      46.276342
    std       81.662221
    min        0.000000
    25%        1.000000
    50%       13.000000
    75%       54.000000
    max      673.000000
    Name: review_cnt, dtype: float64




```python
plt.hist(df_raw.score_cnt)
```




    (array([454.,  34.,   8.,   3.,   2.,   0.,   0.,   0.,   1.,   1.]),
     array([  0. ,  24.9,  49.8,  74.7,  99.6, 124.5, 149.4, 174.3, 199.2,
            224.1, 249. ]),
     <a list of 10 Patch objects>)




![png](/img/output_47_1.png)



```python
df_raw.score_cnt.describe()
```




    count    503.000000
    mean       9.300199
    std       19.932639
    min        0.000000
    25%        1.000000
    50%        3.000000
    75%       10.000000
    max      249.000000
    Name: score_cnt, dtype: float64



- 두 기준 모두 right-skewed 되어있고 0의 값이 존재한다.  
- 리뷰 수와 평가 건수의 범위가 다르기 때문에 각각 스케일링을 한 후 지표에 사용해야할 것 같다.  
  - 같은 범위로 만들기 위해 min max scaler를 사용하여 0과 1 사이의 값들로 만들어준다.  


```python
# 정규화를 할 때는 제곱근 변환보다는 로그 변환이 적합할 것 같다.  
#  - 0 값들은 0.1로 대체한 후 로그 변환을 해준다?
#   - 이 방법은 기존에 건수가 0이었던 카페들의 인지도 지수가 음수가 나올 수 있다.
#   - 리뷰 건수가 많이 있다고 하더라도 평가 건수의 음수값이 커서 평점과 리뷰 건수가 모두 조금씩만 있는 카페들보다 더 낮은 값들이 나올 수 있겠다.
#  - 따라서 0은 영향이 없다는 의미에서 0으로 그대로 두고 1은 로그함수를 취하면 0이 되기 때문에 미묘한 차이를 두어 1.1로 바꾸어서 하자.
# df_new = df_raw.copy()
# df_new["score_cnt"] = df_new["score_cnt"].apply(lambda x: 1.1 if x == 1 else x)
# df_new["review_cnt"] = df_new["review_cnt"].apply(lambda x: 1.1 if x == 1 else x)
# df_new["score_cnt"] = df_new["score_cnt"].apply(lambda x: np.log(x) if x != 0 else x)
# plt.hist(df_new.score_cnt)
```


```python
df_new = df_raw.copy()
df_new["score_cnt"] = (df_new["score_cnt"] - df_new["score_cnt"].min())/(df_new["score_cnt"].max() - df_new["score_cnt"].min())
df_new["review_cnt"] = (df_new["review_cnt"] - df_new["review_cnt"].min())/(df_new["review_cnt"].max() - df_new["review_cnt"].min())
plt.hist(df_new["score_cnt"])
```




    (array([454.,  34.,   8.,   3.,   2.,   0.,   0.,   0.,   1.,   1.]),
     array([0. , 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1. ]),
     <a list of 10 Patch objects>)




![png](/img/output_51_1.png)



```python
plt.hist(df_new["review_cnt"])
```




    (array([395.,  61.,  19.,  12.,   6.,   4.,   3.,   1.,   1.,   1.]),
     array([0. , 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1. ]),
     <a list of 10 Patch objects>)




![png](/img/output_52_1.png)


- 이를 지표에 사용할 수 있도록 70%의 가중치를 주자.


```python
df_new["score_cnt"] = 0.7 * df_new["score_cnt"]
df_new["score_cnt"]
```




    0      0.112450
    1      0.188353
    2      0.047791
    3      0.056225
    4      0.171486
             ...   
    498    0.042169
    499    0.000000
    500    0.008434
    501    0.014056
    502    0.000000
    Name: score_cnt, Length: 503, dtype: float64




```python
plt.hist(df_new["score_cnt"])
```




    (array([454.,  34.,   8.,   3.,   2.,   0.,   0.,   0.,   1.,   1.]),
     array([0.  , 0.07, 0.14, 0.21, 0.28, 0.35, 0.42, 0.49, 0.56, 0.63, 0.7 ]),
     <a list of 10 Patch objects>)




![png](/img/output_55_1.png)



```python
df_raw['popularity'] = df_new['score_cnt'] + df_new['review_cnt']
df_raw.sort_values('popularity', ascending=False).reset_index(drop=True).head(10)
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
      <th>popularity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>레이어드 연남점</td>
      <td>연남동 223-20</td>
      <td>2.5</td>
      <td>44</td>
      <td>673</td>
      <td>1.123695</td>
    </tr>
    <tr>
      <th>1</th>
      <td>하이웨이스트</td>
      <td>연남동 223-80</td>
      <td>2.8</td>
      <td>20</td>
      <td>566</td>
      <td>0.897235</td>
    </tr>
    <tr>
      <th>2</th>
      <td>호밀밭</td>
      <td>창천동 4-77</td>
      <td>3.9</td>
      <td>249</td>
      <td>120</td>
      <td>0.878306</td>
    </tr>
    <tr>
      <th>3</th>
      <td>딩가케이크</td>
      <td>연남동 252-18</td>
      <td>2.6</td>
      <td>45</td>
      <td>496</td>
      <td>0.863505</td>
    </tr>
    <tr>
      <th>4</th>
      <td>커피리브레 연남점</td>
      <td>연남동 227-15</td>
      <td>3.3</td>
      <td>209</td>
      <td>112</td>
      <td>0.753969</td>
    </tr>
    <tr>
      <th>5</th>
      <td>모파상</td>
      <td>동교동 153-5</td>
      <td>2.6</td>
      <td>49</td>
      <td>412</td>
      <td>0.749935</td>
    </tr>
    <tr>
      <th>6</th>
      <td>콩카페 연남점</td>
      <td>연남동 223-114</td>
      <td>3.3</td>
      <td>41</td>
      <td>407</td>
      <td>0.720016</td>
    </tr>
    <tr>
      <th>7</th>
      <td>카페스콘</td>
      <td>연남동 239-4</td>
      <td>3.6</td>
      <td>20</td>
      <td>413</td>
      <td>0.669895</td>
    </tr>
    <tr>
      <th>8</th>
      <td>아오이토리</td>
      <td>서교동 327-17</td>
      <td>3.5</td>
      <td>94</td>
      <td>249</td>
      <td>0.634242</td>
    </tr>
    <tr>
      <th>9</th>
      <td>얼스어스</td>
      <td>연남동 239-49</td>
      <td>4.2</td>
      <td>40</td>
      <td>350</td>
      <td>0.632509</td>
    </tr>
  </tbody>
</table>
</div>



- 아까는 보이지 않던 카페들이 대거 등장했다.  
  - 리뷰는 남기지 않고 평점만 기록한 카페인 경우로 생각할 수 있다.  
  - 적극적으로 평가를 남기는 소비자보다 평점만 기록한 소비자가 많다는 것은 입소문을 타고 온 소비자들이 많다는 의미일 수 있겠다.  

### 인지도와 평점을 한 눈에 보자.   


```python
plt.figure(figsize=(10, 10))
sns.scatterplot('score', 'popularity', s=30, data=df_raw)
plt.title('별점 - 인지도 산점도')
for row in df_raw.sort_values('popularity', ascending=False).head(10).index:
    plt.text(df_raw.loc[row, 'score'],
             df_raw.loc[row, 'popularity'],
             df_raw.loc[row, 'name'],
             rotation=20)
```


![png](/img/output_59_0.png)


- 레이어드 연남이나 딩가케이크, 하이웨스트, 모파상 등의 인지도는 높은 반면, 평점은 3~4점 사이를 기록하고 있다.  
  - 소문에 비해 맛이나 가격 등이 적당하지 않았던 듯하다.  
  - 이런 카페들은 다시 가고 싶은 카페라기 보다는 일회성 방문이 많을 것이라 예상해본다.  
- 호밀밭은 적당한 인지도에 적당한 평점을 갖고 있다.  
  - 이런 카페들일수록 재방문율이 높고, 주변 지인들에게 소개할 확률이 높아질 것이다.  
- 얼스어스의 경우는 상위 10개의 카페들 중 평점은 가장 높고 인지도는 가장 낮다.  
  - 더 유명해질 가능성이 있는 카페이며, 재방문율이 가장 높을 것이라 예상된다.  


## 지도 시각화   


```python
gmap_key = "#######################"
gmaps = googlemaps.Client(key=gmap_key)
```


```python
# 혹시 모를 사태를 대비하여 원본을 복사하여 사용
df = df_raw.copy()

# 구글맵API를 활용하여 각 카페의 주소에 해당하는 지리 정보를 얻어오기
for row in tqdm_notebook(df.index):
    try:
        geo = gmaps.geocode(str(df.loc[row, 'address']))
        df.loc[row, 'lat'] = geo[0].get('geometry')['location']['lat']
        df.loc[row, 'lng'] = geo[0].get('geometry')['location']['lng']
    except:
        df.loc[row, 'lat'] = np.nan
        df.loc[row, 'lng'] = np.nan
df.head()
```


    HBox(children=(IntProgress(value=0, max=503), HTML(value='')))


    
    




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
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
      <th>popularity</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>앤트러사이트 연희점</td>
      <td>연희동 89-19</td>
      <td>3.5</td>
      <td>40</td>
      <td>135</td>
      <td>0.313044</td>
      <td>37.569648</td>
      <td>126.932820</td>
    </tr>
    <tr>
      <th>1</th>
      <td>콘하스 연희점</td>
      <td>연희동 90-1</td>
      <td>2.8</td>
      <td>67</td>
      <td>211</td>
      <td>0.501875</td>
      <td>37.570284</td>
      <td>126.932176</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>연희동 90-8</td>
      <td>4.3</td>
      <td>17</td>
      <td>50</td>
      <td>0.122085</td>
      <td>37.570100</td>
      <td>126.932233</td>
    </tr>
    <tr>
      <th>3</th>
      <td>스타벅스 연희DT점</td>
      <td>연희동 87-8</td>
      <td>3.7</td>
      <td>20</td>
      <td>22</td>
      <td>0.088914</td>
      <td>37.570118</td>
      <td>126.933859</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61</td>
      <td>157</td>
      <td>0.404770</td>
      <td>37.567758</td>
      <td>126.929598</td>
    </tr>
  </tbody>
</table>
</div>




```python
msno.matrix(df, figsize=(10, 5))
plt.show()
```


![png](/img/output_64_0.png)



```python
# 위치까지 포함된 파일로 다시 저장하자.
df.to_csv('source/cafe_in_yeonhee_map.csv')
```


```python
df = pd.read_csv('source/cafe_in_yeonhee_map.csv', index_col=0)
```


```python
# map = folium.Map([df.lat.median(), df.lng.median()], zoom_start=15)
# for row in df.index:
#     lat = df.lat[row]
#     lng = df.lng[row]
#     folium.Marker([lat, lng]).add_to(map)
# map
```


```python
map = folium.Map([df.lat.median(), df.lng.median()], zoom_start=15)
#                 tiles='Stamen WaterColor')

# 지하철 표시
folium.Marker([37.557646, 126.9222673], popup='홍대입구역 2호선').add_to(map)
folium.Marker([37.558335, 126.9237427], popup='홍대입구역 공항철도').add_to(map)
folium.Marker([37.559778, 126.9401363], popup='신촌역 경의중앙선').add_to(map)
folium.Marker([37.555143, 126.9346963], popup='신촌역 2호선').add_to(map)

# 카페 표시
for row in df.index:
    lat = df.lat[row]
    lng = df.lng[row]
    folium.CircleMarker([lat, lng], color='', fill=True, fill_color='#044275', radius=15, popup=([lat,lng])).add_to(map)

# 카페 밀집 지역 표시
folium.CircleMarker([37.5681379, 126.9311559], color='#F247F5', radius=50).add_to(map)
folium.CircleMarker([37.5652905, 126.9235006], color='#F247F5', radius=50).add_to(map)
folium.CircleMarker([37.5621316, 126.9264844], color='#F247F5', radius=50).add_to(map)
folium.CircleMarker([37.56632949999999, 126.9279715], color='#F247F5', radius=50).add_to(map)


map
```


- (시각화 결과는 생략한다.)  
- 카페가 많이 위치한 지역은 크게 4군데로 나눌 수 있다.
- 대부분은 연남동이거나 연희동이며 그 중에서도 연남동에 훨씬 많은 카페들이 있다는 것을 알 수 있다.  
  - 그 외에도 신촌역이나 명지대학교 근처는 대학교 주변 상권이라 카페들이 일정 수준 존재한다는 것을 볼 수 있다.  


```python
map = folium.Map([df.lat.median(), df.lng.median()], zoom_start=16)
#                tiles='stamen Toner')


for row in df.index:
    lat = df.lat[row]
    lng = df.lng[row]
    folium.CircleMarker([lat, lng], radius=df.loc[row, 'popularity']*20, color='', fill=True, fill_opacity=.6,
                        popup=(df.loc[row, 'name']),
                        fill_color='#F8F8C7' if df.score[row] < 1 else
                        ('#DFF7BE' if df.score[row] < 2 else 
                        ('#00AD2E' if df.score[row] < 3 else 
                        ('#00611A' if df.score[row] < 4 else '#007508')))).add_to(map)

# 지하철 표시
folium.Marker([37.557646, 126.9222673], popup='홍대입구역 2호선').add_to(map)
folium.Marker([37.558335, 126.9237427], popup='홍대입구역 공항철도').add_to(map)
folium.Marker([37.559778, 126.9401363], popup='신촌역 경의중앙선').add_to(map)
folium.Marker([37.555143, 126.9346963], popup='신촌역 2호선').add_to(map)


map.save('cafe_in_yeonhee_map.html')
map

```


- 색이 진할수록 평점이 높고, 원이 클수록 인지도가 높다.  
  - 노란색으로 나타난 카페들은 리뷰는 존재하지만 평점이 없는 경우이다.  
- 연남동에는 인지도가 높은 카페들이 많은 반면, 연희동은 그렇지 않다.  
  - 아직 연희동은 연남동에 비해 잘 알려지지 않은 동네이지만 유명한 카페들이 일정 수준 존재하기 때문에 이 카페들을 중심으로 주변 상권을 살리기 좋을 것 같다.  
- 연남동은 평균적으로 카페들의 평점이 높은 편이며, 연희동은 평점이 존재하는 대부분의 카페들이 좋은 평가를 받고 있다.  
  - 그러나 여전히 잘 알려지지 않은 카페들이 많고, 이 때문에 평점이 메겨지지 않는다고 생각할 수 있다.  
  - 혹은 이제 막 생겨나기 시작한 카페들이 많을 것이다. 유명해질 가능성이 있는 카페들이다.  
  - 숨은 보석 찾기가 필요하다..라고 해두자.  


```python
df[df["score"] == 0]
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
      <th>popularity</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>7</th>
      <td>일룸 엄마의서재</td>
      <td>연희동 87-5</td>
      <td>0.0</td>
      <td>0</td>
      <td>26</td>
      <td>0.038633</td>
      <td>37.570210</td>
      <td>126.934134</td>
    </tr>
    <tr>
      <th>16</th>
      <td>연희입니다</td>
      <td>연희동 151-148</td>
      <td>0.0</td>
      <td>0</td>
      <td>23</td>
      <td>0.034175</td>
      <td>37.575933</td>
      <td>126.933732</td>
    </tr>
    <tr>
      <th>32</th>
      <td>연희라운지</td>
      <td>연희동 80-1</td>
      <td>0.0</td>
      <td>0</td>
      <td>2</td>
      <td>0.002972</td>
      <td>37.571397</td>
      <td>126.934932</td>
    </tr>
    <tr>
      <th>39</th>
      <td>늬에게</td>
      <td>연희동 195-12</td>
      <td>0.0</td>
      <td>0</td>
      <td>4</td>
      <td>0.005944</td>
      <td>37.573028</td>
      <td>126.928954</td>
    </tr>
    <tr>
      <th>45</th>
      <td>라파르벨라</td>
      <td>연희동 76-19</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
      <td>0.001486</td>
      <td>37.573656</td>
      <td>126.936133</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>492</th>
      <td>마이 블러드타입이즈커피</td>
      <td>연남동 224-51</td>
      <td>0.0</td>
      <td>0</td>
      <td>4</td>
      <td>0.005944</td>
      <td>37.563560</td>
      <td>126.927189</td>
    </tr>
    <tr>
      <th>494</th>
      <td>515티룸</td>
      <td>홍은동 401-1</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
      <td>0.001486</td>
      <td>37.581797</td>
      <td>126.925234</td>
    </tr>
    <tr>
      <th>497</th>
      <td>커피볶는집 명지대점</td>
      <td>남가좌동 342-10</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
      <td>37.578273</td>
      <td>126.923529</td>
    </tr>
    <tr>
      <th>499</th>
      <td>515티룸</td>
      <td>홍은동 401-1</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
      <td>0.001486</td>
      <td>37.581797</td>
      <td>126.925234</td>
    </tr>
    <tr>
      <th>502</th>
      <td>커피볶는집 명지대점</td>
      <td>남가좌동 342-10</td>
      <td>0.0</td>
      <td>0</td>
      <td>0</td>
      <td>0.000000</td>
      <td>37.578273</td>
      <td>126.923529</td>
    </tr>
  </tbody>
</table>
<p>121 rows × 8 columns</p>
</div>



### 현재 거주지인 연희동만 위주로 보자.   


```python
df = pd.read_csv('source/cafe_in_yeonhee_map.csv', index_col = 0)
df_yh = df.copy()

for row in df_yh.index:
    if (df_yh.address[row][0:3] != "연희동"):
        df_yh.drop(row, inplace=True)

df_yh.head()
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
      <th>popularity</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>앤트러사이트 연희점</td>
      <td>연희동 89-19</td>
      <td>3.5</td>
      <td>40</td>
      <td>135</td>
      <td>0.313044</td>
      <td>37.569648</td>
      <td>126.932820</td>
    </tr>
    <tr>
      <th>1</th>
      <td>콘하스 연희점</td>
      <td>연희동 90-1</td>
      <td>2.8</td>
      <td>67</td>
      <td>211</td>
      <td>0.501875</td>
      <td>37.570284</td>
      <td>126.932176</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>연희동 90-8</td>
      <td>4.3</td>
      <td>17</td>
      <td>50</td>
      <td>0.122085</td>
      <td>37.570100</td>
      <td>126.932233</td>
    </tr>
    <tr>
      <th>3</th>
      <td>스타벅스 연희DT점</td>
      <td>연희동 87-8</td>
      <td>3.7</td>
      <td>20</td>
      <td>22</td>
      <td>0.088914</td>
      <td>37.570118</td>
      <td>126.933859</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61</td>
      <td>157</td>
      <td>0.404770</td>
      <td>37.567758</td>
      <td>126.929598</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(10, 10))
sns.scatterplot('score', 'popularity', s=30, data=df_yh)
plt.title('연희동 별점 - 인지도 산점도')
for row in df_yh.sort_values('popularity', ascending=False).head(10).index:
    plt.text(df_yh.loc[row, 'score'],
             df_yh.loc[row, 'popularity'],
             df_yh.loc[row, 'name'],
             rotation=20)
```


![png](/img/output_75_0.png)



```python
df_yh[df_yh.score_cnt != 0].score_cnt.describe()
```




    count    99.000000
    mean      7.181818
    std      11.251634
    min       1.000000
    25%       1.000000
    50%       3.000000
    75%       8.000000
    max      67.000000
    Name: score_cnt, dtype: float64




```python
df_filter = df_yh[df_yh.score_cnt >= np.mean(df_yh[df_yh.score_cnt != 0].score_cnt)]
print('평가 개수가 평균 이상인 카페: 총 {}개 중 {}개 ({:.2f}%)'.format(len(df_yh), len(df_filter), (len(df_filter) / len(df_yh)*100)))
df_filter.sort_values(['score', 'popularity'], ascending=False).head(10)
```

    평가 개수가 평균 이상인 카페: 총 161개 중 26개 (16.15%)
    




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
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
      <th>popularity</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>28</th>
      <td>쿳사</td>
      <td>연희동 100-2</td>
      <td>4.8</td>
      <td>13</td>
      <td>4</td>
      <td>0.042490</td>
      <td>37.572141</td>
      <td>126.929376</td>
    </tr>
    <tr>
      <th>104</th>
      <td>카페공든</td>
      <td>연희동 310-5</td>
      <td>4.8</td>
      <td>9</td>
      <td>10</td>
      <td>0.040160</td>
      <td>37.564220</td>
      <td>126.932816</td>
    </tr>
    <tr>
      <th>13</th>
      <td>시간이머무는홍차가게</td>
      <td>연희동 69-3</td>
      <td>4.7</td>
      <td>20</td>
      <td>33</td>
      <td>0.105259</td>
      <td>37.568310</td>
      <td>126.932584</td>
    </tr>
    <tr>
      <th>20</th>
      <td>르솔레이</td>
      <td>연희동 192-4</td>
      <td>4.7</td>
      <td>15</td>
      <td>36</td>
      <td>0.095661</td>
      <td>37.566329</td>
      <td>126.927972</td>
    </tr>
    <tr>
      <th>36</th>
      <td>푸어링아웃</td>
      <td>연희동 128-27</td>
      <td>4.6</td>
      <td>11</td>
      <td>38</td>
      <td>0.087387</td>
      <td>37.567252</td>
      <td>126.928118</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61</td>
      <td>157</td>
      <td>0.404770</td>
      <td>37.567758</td>
      <td>126.929598</td>
    </tr>
    <tr>
      <th>320</th>
      <td>마리아칼라스</td>
      <td>연희동 340-16</td>
      <td>4.5</td>
      <td>27</td>
      <td>3</td>
      <td>0.080361</td>
      <td>37.563277</td>
      <td>126.932272</td>
    </tr>
    <tr>
      <th>34</th>
      <td>맨팅</td>
      <td>연희동 79-8</td>
      <td>4.5</td>
      <td>8</td>
      <td>3</td>
      <td>0.026948</td>
      <td>37.572796</td>
      <td>126.936156</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>연희동 90-8</td>
      <td>4.3</td>
      <td>17</td>
      <td>50</td>
      <td>0.122085</td>
      <td>37.570100</td>
      <td>126.932233</td>
    </tr>
    <tr>
      <th>41</th>
      <td>올레무스</td>
      <td>연희동 220-40</td>
      <td>4.1</td>
      <td>16</td>
      <td>93</td>
      <td>0.183167</td>
      <td>37.565373</td>
      <td>126.926028</td>
    </tr>
  </tbody>
</table>
</div>



## 왜 인기가 많은걸까.   
- 평점과 함께 적혀있는 리뷰들을 분석해보겠다.
  - 평점의 점수와 직접적으로 관련이 있을 것이다.  
- 상위 10개의 카페들만 분석해보겠다.  
  
- 검색속도 향상을 위해 각 카페의 고유 번호를 먼저 찾았다.  
  - 쿳사: 705202577
  - 카페공든: 1369321588
  - 시간이머무는홍차가게: 23446983
  - 르솔레이: 1161755224
  - 푸어링아웃: 1980694762
  - 매뉴팩트커피 연희본점: 21542432 
  - 마리아칼라스: 22482531
  - 맨팅: 98345425
  - 로도덴드론: 205823441
  - 올레무스: 961299046


```python
df_high = df_filter.sort_values(['score', 'popularity'], ascending=False).head(10)
df_high['number'] = [705202577, 1369321588, 23446983, 1161755224, 1980694762, 21542432, 22482531, 98345425, 205823441, 961299046]
df_high
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
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>score</th>
      <th>score_cnt</th>
      <th>review_cnt</th>
      <th>popularity</th>
      <th>lat</th>
      <th>lng</th>
      <th>number</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>28</th>
      <td>쿳사</td>
      <td>연희동 100-2</td>
      <td>4.8</td>
      <td>13</td>
      <td>4</td>
      <td>0.042490</td>
      <td>37.572141</td>
      <td>126.929376</td>
      <td>705202577</td>
    </tr>
    <tr>
      <th>104</th>
      <td>카페공든</td>
      <td>연희동 310-5</td>
      <td>4.8</td>
      <td>9</td>
      <td>10</td>
      <td>0.040160</td>
      <td>37.564220</td>
      <td>126.932816</td>
      <td>1369321588</td>
    </tr>
    <tr>
      <th>13</th>
      <td>시간이머무는홍차가게</td>
      <td>연희동 69-3</td>
      <td>4.7</td>
      <td>20</td>
      <td>33</td>
      <td>0.105259</td>
      <td>37.568310</td>
      <td>126.932584</td>
      <td>23446983</td>
    </tr>
    <tr>
      <th>20</th>
      <td>르솔레이</td>
      <td>연희동 192-4</td>
      <td>4.7</td>
      <td>15</td>
      <td>36</td>
      <td>0.095661</td>
      <td>37.566329</td>
      <td>126.927972</td>
      <td>1161755224</td>
    </tr>
    <tr>
      <th>36</th>
      <td>푸어링아웃</td>
      <td>연희동 128-27</td>
      <td>4.6</td>
      <td>11</td>
      <td>38</td>
      <td>0.087387</td>
      <td>37.567252</td>
      <td>126.928118</td>
      <td>1980694762</td>
    </tr>
    <tr>
      <th>4</th>
      <td>매뉴팩트커피 연희본점</td>
      <td>연희동 130-2</td>
      <td>4.5</td>
      <td>61</td>
      <td>157</td>
      <td>0.404770</td>
      <td>37.567758</td>
      <td>126.929598</td>
      <td>21542432</td>
    </tr>
    <tr>
      <th>320</th>
      <td>마리아칼라스</td>
      <td>연희동 340-16</td>
      <td>4.5</td>
      <td>27</td>
      <td>3</td>
      <td>0.080361</td>
      <td>37.563277</td>
      <td>126.932272</td>
      <td>22482531</td>
    </tr>
    <tr>
      <th>34</th>
      <td>맨팅</td>
      <td>연희동 79-8</td>
      <td>4.5</td>
      <td>8</td>
      <td>3</td>
      <td>0.026948</td>
      <td>37.572796</td>
      <td>126.936156</td>
      <td>98345425</td>
    </tr>
    <tr>
      <th>2</th>
      <td>로도덴드론</td>
      <td>연희동 90-8</td>
      <td>4.3</td>
      <td>17</td>
      <td>50</td>
      <td>0.122085</td>
      <td>37.570100</td>
      <td>126.932233</td>
      <td>205823441</td>
    </tr>
    <tr>
      <th>41</th>
      <td>올레무스</td>
      <td>연희동 220-40</td>
      <td>4.1</td>
      <td>16</td>
      <td>93</td>
      <td>0.183167</td>
      <td>37.565373</td>
      <td>126.926028</td>
      <td>961299046</td>
    </tr>
  </tbody>
</table>
</div>



### 크롤링


```python
number = list(df_high.number)
name = list(df_high.name)
urls = []

for row in range(len(number)):
    url = 'https://place.map.kakao.com/' + str(number[row])
    urls.append(url)
    print(url)
    
```

    https://place.map.kakao.com/705202577
    https://place.map.kakao.com/1369321588
    https://place.map.kakao.com/23446983
    https://place.map.kakao.com/1161755224
    https://place.map.kakao.com/1980694762
    https://place.map.kakao.com/21542432
    https://place.map.kakao.com/22482531
    https://place.map.kakao.com/98345425
    https://place.map.kakao.com/205823441
    https://place.map.kakao.com/961299046
    


```python
driver = webdriver.Chrome('C:/Users/jej_0312_@naver.com/chromedriver_win32/chromedriver.exe')

dt_review = {'name': [], 'rating': [], 'review': []}
```


```python
for url in tqdm_notebook(urls):
    driver.get(url)
    time.sleep(4)
    html = driver.page_source # url마다 parsing하고
    soup = BeautifulSoup(html, 'lxml')
    reviewcount = int(soup.find_all('span', class_='color_b')[2].text)
    pagenum = 1

    while pagenum <= (math.ceil(reviewcount / 5)):
        html = driver.page_source # page마다 parsing하자
        soup = BeautifulSoup(html, 'lxml')

        for cafenum in range(len(soup.find_all('p', class_='txt_comment'))):
            information = []
            try:
                information.append(soup.select('.tit_location')[0].text)
                information.append(soup.find_all('em', class_='num_rate')[cafenum + 2].text)
                information.append(soup.find_all('p', class_='txt_comment')[cafenum].text)
            except Exception as e:
                print(soup.select('.tit_location')[0].text, e)
                continue
            if len(information) == 3: # 카페 이름, 평점, 리뷰가 다 있을 경우에만 parsing하자
                information[2].replace("더보기","")
                dt_review['name'].append(information[0])
                dt_review['rating'].append(information[1])
                dt_review['review'].append(information[2])
                # print('{} 페이지 {}/{}번째 댓글 완료'.format(pagenum, cafenum + 1, len(soup.find_all('p', class_='txt_comment'))))

        if pagenum == 5:
            driver.find_element_by_css_selector('a.btn_next').click()
            pagenum += 1
        if pagenum == 10:
            driver.find_element_by_css_selector('a.btn_next').click()
        elif pagenum == math.ceil(reviewcount / 5):
            break
        else:
            driver.find_element_by_xpath('//a[@data-page="{}"]'.format(pagenum+1)).click()

        pagenum += 1
        driver.implicitly_wait(10)
        time.sleep(2)

#print(dt_review)

```


    HBox(children=(IntProgress(value=0, max=10), HTML(value='')))

      name rating                                             review
    0   쿳사     5점                       너무 좋아요! 재방문 몇 번이고 하고 싶은 곳더보기
    1   쿳사     5점  맛있어요, 가게는 작지만 식물이 가득하고 햇살도 잘 들어서 오랫동안 앉아있다 오고싶...
    2   쿳사     4점  사장님이 굉장히 착하시고 요즘엔 마스크도 다 쓰고 영업 하신다. 서비스는 대만족. ...
    3   쿳사     5점   후 나만 알고싶지만 이걸 읽고있다면 어차피 여길 오겠죠 ㅋㅋㅋ 사진을 보세요.. 더보기
    4   쿳사     4점  에그 베네딕트와 뇨끼, 라떼. 매우 만족스러운 브런치였다. 근데 가게 분들 왜 마스...
    5   쿳사     5점                                     맛있고 멋있는 곳 !더보기
    6   쿳사     5점                                   친절하고 맛있고 주차되고더보기
    7   쿳사     5점  콰트로치즈뇨끼와 하우스와인이 정말 맛있었고 티라미수, 커피도 다 맛있었다. 매장 예...
    8   쿳사     5점                  밑의 분이 추천해 주셔서 잘 먹었습니다 뇨끼 맛있네요!더보기
    9   쿳사     5점  뇨끼뇨끼!!! 여러분 뇨끼를 드십시오 1인 1뇨끼도 아쉽습니다 식전 메뉴와 파스타는...
    


```python
df_yh = pd.DataFrame(dt_review)
df_yh.head(10)
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
      <th></th>
      <th>name</th>
      <th>rating</th>
      <th>review</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>너무 좋아요! 재방문 몇 번이고 하고 싶은 곳더보기</td>
    </tr>
    <tr>
      <th>1</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>맛있어요, 가게는 작지만 식물이 가득하고 햇살도 잘 들어서 오랫동안 앉아있다 오고싶...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>쿳사</td>
      <td>4점</td>
      <td>사장님이 굉장히 착하시고 요즘엔 마스크도 다 쓰고 영업 하신다. 서비스는 대만족. ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>후 나만 알고싶지만 이걸 읽고있다면 어차피 여길 오겠죠 ㅋㅋㅋ 사진을 보세요.. 더보기</td>
    </tr>
    <tr>
      <th>4</th>
      <td>쿳사</td>
      <td>4점</td>
      <td>에그 베네딕트와 뇨끼, 라떼. 매우 만족스러운 브런치였다. 근데 가게 분들 왜 마스...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>맛있고 멋있는 곳 !더보기</td>
    </tr>
    <tr>
      <th>6</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>친절하고 맛있고 주차되고더보기</td>
    </tr>
    <tr>
      <th>7</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>콰트로치즈뇨끼와 하우스와인이 정말 맛있었고 티라미수, 커피도 다 맛있었다. 매장 예...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>밑의 분이 추천해 주셔서 잘 먹었습니다 뇨끼 맛있네요!더보기</td>
    </tr>
    <tr>
      <th>9</th>
      <td>쿳사</td>
      <td>5점</td>
      <td>뇨끼뇨끼!!! 여러분 뇨끼를 드십시오 1인 1뇨끼도 아쉽습니다 식전 메뉴와 파스타는...</td>
    </tr>
  </tbody>
</table>
</div>




```python
for i in range(len(df_yh)):
    df_yh.iloc[i,2] = df_yh.iloc[i,2].replace("더보기","")
```

- 저장해주자.


```python
df_yh.to_csv('source/cafe_in_yeonhee_review.csv', encoding = 'UTF-8')
```


```python

```
