---
# layout: post
title: "[Blog] Markdown 작성법"
categories:
 - blog
excerpt: "마크다운 작성법 및 헤더 설정방법"
tags: 
 - howto
 - markdown
 - header
# share: true 
comments: true 
last_modified_at: 2020-09-03T22:05:00-09:00
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
mathjax: true
---

# 마크다운(Markdown) 작성법


## Front Matter

*layout*: post # post, page 중 하나를 넣어 이것이 한 페이지에 속하는 블로그 포스트인지, 홈페이지를 구성하는 하나의 큰 페이지인지를 결정한다.   
  
*title*: "제목"   

*date*: 2015-02-08T20:39:51-09:00 # 생성일로, 양식은 년-월-일T시:분:초-GMT시간 으로 추측된다.   
  
*modified*: 2015-02-08T22:21:31-09:00   

*excerpt*: "요약문" # 제목 아래 달리는 요약문   

*tags*: 태그 사용   
```
tags:
    - tag1
    - tag2
```

*categories*: 카테고리   
```
categories:
  - test
```

*author_profile*: true / false # 작성자 프로필 출력여부   

*read_time*: false # read_time을 출력할지 여부 1min read 같은것!   

*comments*: true # disqus 댓글 버튼   

*toc*: true # Table Of Contents 목차 보여줌   
*toc_label*: "My Table of Contents" # toc 이름 정의   
*toc_icon*: "cog" # font Awesome아이콘으로 toc 아이콘 설정   
*toc_sticky*: true # 스크롤 내릴때 같이 내려가는 목차   

*gallery*: 이미지 갤러리   
```
gallery:
  - url: /assets/images/unsplash-gallery-image-1.jpg
    image_path: /assets/images/unsplash-gallery-image-1-th.jpg
    alt: "placeholder image 1"
    title: "Image 1 title caption"
  - url: /assets/images/unsplash-gallery-image-2.jpg
    image_path: /assets/images/unsplash-gallery-image-2-th.jpg
    alt: "placeholder image 2"
    title: "Image 2 title caption"
```

캡션은 다음과 같이 본문에서 사용한다.   

```
{% include gallery caption="This is a sample gallery with **Markdown support**." %}
```

혹은   

"{% include gallery caption="This is a sample gallery with **Markdown support**." %}"

{% include gallery caption="This is a sample gallery with **Markdown support**." %}

*header*:  # 헤더에 유튜브 비디오 삽입   
```
header:
  video:
    id: XsxDH4HcOWA
    provider: youtube
```

*link*
```
참고링크: [참고1]   

[참고1]: https://gist.github.com/ihoneymon/652be052a0727ad59601/
```

참고링크: [참고1]   


[참고1]: https://gist.github.com/ihoneymon/652be052a0727ad59601/

*feature*: 사진 파일 이름
```   
feature: test.jpg
  credit: Me # 사진 제작자 및 저작권자   
  creditlink: me.org # 저작권자의 홈페이지이거나 이 사진을 직접 구할 수 있는 링크
```
  

## 마크다운 문법
### Header
- 글머리의 사이즈를 조절하기 위해서는 다음과 같이 '#'을 사용하면 된다.   
```
  # H1
  ## H2
  ### H3
```
이런 식으로 6개까지 지원함.   

### Block Quote
- 블럭인용문자는 '>'로 나타낸다.   
```
  > first quote
  > > second qoute
  > > > third one
```
> first quote
> > second qoute
> > > third one

### 목록
- 순서가 있다면 '1.'으로 작성하면 자동으로 번호가 입력되며,   
```
1. 가
1. 가
1. 가
```

1. 가
1. 가
1. 가

- 순서가 없다면 '*', '+', '-' 중에 선택하여 사용하면 된다.   

```
* 가
  + 가
    - 가
```

* 가
  + 가
    - 가

### 코드
- '```'로 시작하고 끝을 맺으면 된다.   


      ```
      이 안에 코드 작성
      a = 1
      ```


```
이 안에 코드 작성
a = 1
```

### 구분선
- 구분선을 작성하는 방법은 아래와 같다.   

```
  * * *
  ***
  - - -
  ---------------
```

* * *
***
- - -
---------------

### 강조
```
  *이탈릭체*
  **볼드체**
  ~~취소선~~
```

*이탈릭체*   
**볼드체**   
~~취소선~~   


- - -
- - -
- 줄바꿈을 하기 위해서는 문장 마지막에 2칸 이상을 띄워준다.  
- 참고로 Notion에서 작성한 후에 VSCode에 붙여넣으면 자동으로 마크다운 언어로 변경된다!    

### 이미지
```
![이미지](https://cdn.pixabay.com/photo/2019/04/10/11/56/watercolour-4116932_960_720.png)

![이미지](/img/profile.png)
```

- url 링크나 파일 경로를 입력해주면 이미지를 보여줄 수 있다.  

![이미지](https://cdn.pixabay.com/photo/2019/04/10/11/56/watercolour-4116932_960_720.png)


### 수식
```
$$ a_1 \times b_1 = a_1 b_1 $$
```
$$ a_1 \times b_1 = a_1 b_1 $$

- 수식을 사용하기 위한 설정은 [이 글](http://sgeos.github.io/github/jekyll/2016/08/21/adding_mathjax_to_a_jekyll_github_pages_blog.html)을 참고한다.  
- 더 자세한 수식 관련 Latex 구문은 [위키피디아](https://en.wikipedia.org/wiki/Help:Displaying_a_formula#Formatting_using_TeX)을 참고한다.  


- - -
[블로그 레이아웃 설정 참고링크](https://danggai.github.io/github.io/Archive%EC%97%90%EC%84%9C-%EC%B9%B4%ED%85%8C%EA%B3%A0%EB%A6%AC-%EC%9D%B4%EB%A6%84,-%EB%82%A0%EC%A7%9C-%EB%B3%B4%EC%9D%B4%EA%B2%8C%ED%95%98%EA%B8%B0!/)


