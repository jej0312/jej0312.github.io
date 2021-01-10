---
# layout: post
title: "lambda, map, filter, zip"
categories:
 - programming
tags: 
 - programming
 - functions
 - lambda
 - map
 - filter
 - zip
# share: true 
comments: true 
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
---


크기가 큰 데이터를 정제하면서 하나의 작업을 하는 데도 시간이 굉장히 많이 필요한다는 것을 느꼈다. 그래서 검색을 해 가장 효율적인 코드를 찾게 되었고, 그 중 가장 많이 사용하는 코드 네 가지에 대해 정리하였다.



## lambda argument: expression

- 함수를 한 줄로 만들어낼 수 있고, 함수를 따로 정의해 저장하지 않기 때문에 메모리를 많이 사용하지 않음

```python
(lambda x, y: x * y)(10, 20)
# 30
```



## map(function_object, iterable1, ...)

- list와 같은 iterable한 객체의 모든 element에 해당 함수를 적용
- for 문으로 함수를 돌리는 것보다 훨씬 빠름
- 함수의 매개변수가 두 개일 경우 두 집합을 입력
- map의 결과는 <map>이므로 print하거나 값을 사용하려면 list 형식으로 변경해야 함
- 매개변수가 한 개일 경우

    ```python
    # 데이터 타입 변경

    l = list(map(int, ['1', '2', '3]'))

    print(l) # 1, 2, 3
    ```

- 매개변수가 두 개일 경우

    ```python
    # 두 리스트를 사용하여 계산

    l = list(map(lambda x, y: x * y, [0, 1, 2, 3], [4, 5, 6, 7]))

    print(l) #[0, 5, 12, 21]
    ```

- 직접 만든 함수를 적용할 경우

    ```python
    # 두 매개변수를 사용하는 함수

    def multiply (x, y):
    	return x * y

    l = list(map(multiply, [0, 1, 2, 3], [4, 5, 6, 7])) 

    print(l) #[0, 5, 12, 21]
    ```



## filter(function_object, iterable)

- list와 같은 iterable한 객체의 모든 element에 해당 함수를 적용한 후 True에 해당하는 iterable을 리턴

```python
l = [0, 3, 1, 2]

list(filter(lambda x : x < 3, l))

# [0, 1, 2]
```



## zip(*iterables)

- 여러 개의 iterable한 element를 매핑하여 하나의 tuple로 만들어줌

```python
fruit = ['apple', 'orange', 'cherry']
price = [100, 200, 300]

list(zip(fruit, price))
# [('apple', 100), ('orange', 200), ('cherry', 300)]
```

- 두 리스트를 합칠 때도 사용할 수 있음

```python
a = [1, 2, 3, 4, 5]
b = [0, 1, 2, 3, 4]

list(x*y for x, y in zip(a, b))
# [0, 2, 6, 12, 20]
```

- 참고자료:
    [https://wikidocs.net/64](https://wikidocs.net/64)
    [[Python] Map, Filter, Zip](https://redcrow.tistory.com/399)