---
# layout: post
title: "[Programming] DataFrmae - Join"
categories:
 - programming
tags: 
 - programming
 - howto
# share: true 
comments: true 
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
---

<style>
  table, tr, td {
    /* border-collapse: collapse; */
    /* border: solid 1px black; */
    text-align: center;
    width: 35%;
    margin: 0 auto;
    padding: 5px;
  }
</style>


- 데이터를 병합하는 방법은 다양하나, 큰 데이터를 다룰 때에는 어떤 방식이 가장 효율적인지 비교할 필요가 있다.  
- 한 칼럼에 대해 병합할 경우 `pd.concat`과 `pd.merge` 함수 중 어떤 함수가 빠른 속도로 진행이 되는지 비교하였다.  

## pd.concat(objs)

- `pd.concat()`은 데이터프레임을 물리적으로 붙이는 함수이다.
- 데이터프레임과 데이터프레임, 데이터프레임과 시리즈, 시리즈와 시리즈를 병합할 수 있다.
- 옵션은 다음과 같다.
    - `pandas.concat(objs, axis=0, join='outer', ignore_index=False, keys=None, levels=None, names=None, verify_integrity=False, sort=False, copy=True)`
    - 자주 사용하는 옵션으로는 axis, join, ignore_index 등이 있다.
        - axis: 0의 경우 행방향으로(위 아래 병합), 1의 경우 열방향으로 결합이 된다.
        - join: {'inner', 'outer'} 중 선택이 가능하다. 디폴트 값은 outer이다.
        - ignore_index: 인덱스에 따라 이어붙일 것인가를 선택할 수 있다. 만약 인덱스 번호가 다를 경우 ignore_index = True를 줘서 재배열한다.
        - 디폴트 값은 다음과 같다: 행단위로 병합(axis=0)이 되며, 인덱스는 중복되어(ignore_index=False) 표시한다.


```python
import pandas as pd

df1 = pd.DataFrame({'a':['a0','a1','a2','a3'],
                   'b':['b0','b1','b2','b3'],
                   'c':['c0','c1','c2','c3']},
                  index = [0,1,2,3])

df2 = pd.DataFrame({'a':['a0','a1','a2','a3','a4'],
                    'd':['d10','d11','d12','d13', 'd14'],
                    'e':['e10','e11','e12','e13', 'e14'],
                    'f':['f10','f11','f12','f13', 'f14']},
                   index = [2,3,4,5,6])

pd.concat([df1,df2]) # 행단위로 병합되며, 인덱스가 중복되어 표시된다.
```


<table class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a0</td>
      <td>b0</td>
      <td>c0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a3</td>
      <td>b3</td>
      <td>c3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>d10</td>
      <td>e10</td>
      <td>f10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a1</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>d11</td>
      <td>e11</td>
      <td>f11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>a2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>d12</td>
      <td>e12</td>
      <td>f12</td>
    </tr>
    <tr>
      <th>5</th>
      <td>a3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>d13</td>
      <td>e13</td>
      <td>f13</td>
    </tr>
    <tr>
      <th>6</th>
      <td>a4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>d14</td>
      <td>e14</td>
      <td>f14</td>
    </tr>
  </tbody>
</table>


- axis=1을 설정함으로써 열 단위로 병합하게 되고, ignore_index=True로 설정하여 병합 후 인덱스가 재배열된다.


```python
pd.concat([df1,df2], axis=1, ignore_index=True) 
```

<table class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a0</td>
      <td>b0</td>
      <td>c0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a1</td>
      <td>b1</td>
      <td>c1</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
      <td>a0</td>
      <td>d10</td>
      <td>e10</td>
      <td>f10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a3</td>
      <td>b3</td>
      <td>c3</td>
      <td>a1</td>
      <td>d11</td>
      <td>e11</td>
      <td>f11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>a2</td>
      <td>d12</td>
      <td>e12</td>
      <td>f12</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>a3</td>
      <td>d13</td>
      <td>e13</td>
      <td>f13</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>a4</td>
      <td>d14</td>
      <td>e14</td>
      <td>f14</td>
    </tr>
  </tbody>
</table>

- join='inner'을 설정하면 두 데이터프레임의 공통된 부분만 출력된다. 현재는 index 기준으로 병합이 되었다.  

```python
pd.concat([df1,df2], axis=1, join='inner') # 인덱스가 동일한 행
```


<table class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>a</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>a2</td>
      <td>b2</td>
      <td>c2</td>
      <td>a0</td>
      <td>d10</td>
      <td>e10</td>
      <td>f10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a3</td>
      <td>b3</td>
      <td>c3</td>
      <td>a1</td>
      <td>d11</td>
      <td>e11</td>
      <td>f11</td>
    </tr>
  </tbody>
</table>

```python
pd.concat([df1,df2], axis=0, join='inner') # 칼럼명이 동일한 열
```

<table class="dataframe" style="width: 10%;">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>a2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>a3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>a4</td>
    </tr>
  </tbody>
</table>



- 시리즈를 병합할 수도 있다.
  - 시리즈를 생성할 때 name을 등록해주면 칼럼명으로 사용할 수 있다.

```python
series = pd.Series(['a00','a01','a02'], name = 'aa', index = [3,4,5])
pd.concat([df2, series], axis=1)
```

<table class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
      <th>aa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>a0</td>
      <td>d10</td>
      <td>e10</td>
      <td>f10</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a1</td>
      <td>d11</td>
      <td>e11</td>
      <td>f11</td>
      <td>a00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>a2</td>
      <td>d12</td>
      <td>e12</td>
      <td>f12</td>
      <td>a01</td>
    </tr>
    <tr>
      <th>5</th>
      <td>a3</td>
      <td>d13</td>
      <td>e13</td>
      <td>f13</td>
      <td>a02</td>
    </tr>
    <tr>
      <th>6</th>
      <td>a4</td>
      <td>d14</td>
      <td>e14</td>
      <td>f14</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>



## data.merge(right)

- `pd.merge()`는 두 데이터프레임을 고유값 기준으로 병합할 때 사용한다.
- 옵션은 다음과 같다.
    - `DataFrame.merge(right, how='inner', on=None, left_on=None, right_on=None, left_index=False, right_index=False, sort=False, suffixes=('_x', '_y'), copy=True, indicator=False, validate=None)`


```python
import pandas as pd

df1 = pd.DataFrame({'a':['a0','a1','a2'],
                   'b':['b0','b1','b2'],
                   'c':['a0','a1','a2']},
                  index = [0,1,2])

df2 = pd.DataFrame({'a':['a0','a2','a5','a0'],
                    'd':['d10','d11','d12','d13'],
                    'e':['e10','e11','e12','e13']},
                   index = [2,3,4,5])

pd.merge(df1, df2) # 공통된 key값에 대한 inner join이 수행된다.
```

<table class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a0</td>
      <td>b0</td>
      <td>a0</td>
      <td>d10</td>
      <td>e10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a0</td>
      <td>b0</td>
      <td>a0</td>
      <td>d13</td>
      <td>e13</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a2</td>
      <td>b2</td>
      <td>a2</td>
      <td>d11</td>
      <td>e11</td>
    </tr>
  </tbody>
</table>

- 칼럼명이 다를 경우 병합할 key 칼럼을 지정할 수 있다.

```python
pd.merge(df1, df2, left_on='c', right_on='a') 
```

<table class="dataframe">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a_x</th>
      <th>b</th>
      <th>c</th>
      <th>a_y</th>
      <th>d</th>
      <th>e</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a0</td>
      <td>b0</td>
      <td>a0</td>
      <td>a0</td>
      <td>d10</td>
      <td>e10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a0</td>
      <td>b0</td>
      <td>a0</td>
      <td>a0</td>
      <td>d13</td>
      <td>e13</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a2</td>
      <td>b2</td>
      <td>a2</td>
      <td>a2</td>
      <td>d11</td>
      <td>e11</td>
    </tr>
  </tbody>
</table>

- 기본적으로는 inner join이 수행되나 outer join으로 병합할 수도 있다.

```python
pd.merge(df1, df2, how='outer')
```

<table class="dataframe" style="margin-left:auto;margin-right:auto;">
  <thead>
    <tr style="text-align: center;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>a0</td>
      <td>b0</td>
      <td>a0</td>
      <td>d10</td>
      <td>e10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>a0</td>
      <td>b0</td>
      <td>a0</td>
      <td>d13</td>
      <td>e13</td>
    </tr>
    <tr>
      <th>2</th>
      <td>a1</td>
      <td>b1</td>
      <td>a1</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>a2</td>
      <td>b2</td>
      <td>a2</td>
      <td>d11</td>
      <td>e11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>a5</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>d12</td>
      <td>e12</td>
    </tr>
  </tbody>
</table>

## 속도 비교

- 다음과 같은 코드를 통해 수행 속도를 비교하였다.  

```python
import time
startTime = time.perf_counter()

# 실행 코드 #

print(time.perf_counter() - startTime)
```

| 데이터 사이즈 | 실행 코드 | 수행 속도 |
|:--:|:--:|:--:|
|(50, 2), (50, 2)|pd.concat|0.003|
|(50, 2), (50, 2)|pd.merge|0.006|
|(50000, 2), (50000, 2)|pd.concat|0.009|
|(50000, 2), (50000, 2)|pd.merge|0.137|

- 비교에 사용한 데이터는 `pd.DataFrame(np.random.randint(1000, size=(N,2)))` 이며 첫번째 칼럼에 대해 결합을 시도하였다.  
- 데이터의 사이즈(`N`)와 상관없이 `concat` 함수의 속도가 더 우세하다는 것을 확인하였다.  
- 다만 이는 데이터 특성에 따라, 데이터 크기에 따라 상이할 수 있으므로 일반화하기는 어려우나, 타 블로그의 글을 참고하더라도 `concat`을 권장하는 분들이 많다는 것을 알 수 있다.
- 참고로, 행만 추가할 경우, dict 타입 데이터를 생성 후 `append`나 list 타입 데이터 생성 후 `extend`하는 방식이 보다 빠르게 처리된다고 한다.  


- 참고문서
  - [https://yganalyst.github.io/data_handling/Pd_12/](https://yganalyst.github.io/data_handling/Pd_12/)
  - [https://dailyheumsi.tistory.com/84](https://dailyheumsi.tistory.com/84)
  - [https://emilkwak.github.io/pandas-dataframe-concat-efficiently](https://emilkwak.github.io/pandas-dataframe-concat-efficiently)
  - [https://dowtech.tistory.com/39](https://dowtech.tistory.com/39)