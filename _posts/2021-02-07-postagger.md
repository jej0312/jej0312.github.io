---
# layout: post
title: "형태소 분석기 설치"
categories:
 - textmining
tags: 
 - textmining
 - POStagger
 - Mecab
 - NLTK
 - Soynlp
# share: true 
comments: true 
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
last_modified_at: 2021-02-07
---


- 한글 형태소 분석을 진행할 때는 konlpy, 영어를 진행할 때는 nltk를 사용한다.  
- 전체 과정은 Pycharm에서 진행하였다.  

## KoNLPy

### konlpy 설치

- 전반적인 설치 과정은 [이 사이트]([https://konlpy-ko.readthedocs.io/ko/v0.4.3/install/#id2](https://konlpy-ko.readthedocs.io/ko/v0.4.3/install/#id2))를 참고한다.
    - konlpy 설치 전 필요한 파일은 **Java, JPype**이다.
        - JAVA를 설치한 후 JAVA_HOME도 설정해주어야 한다. 관련 내용은 [이 사이트]([https://m.blog.naver.com/PostView.nhn?blogId=hs_715&logNo=221548450575&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=hs_715&logNo=221548450575&proxyReferer=https:%2F%2Fwww.google.com%2F))를 참고한다.
1. pip 업그레이드를 진행한다.

    ```python
    $ (venv) python -m pip install --upgrade pip
    ```

2. 본인의 파이썬 버전을 확인한다.

    ```python
    $ (venv) python --version

    # script에서 확인하고자 한다면 다음 코드 활용.
    import platform
    platform.architecture()
    ```

    그 후 파이썬 버전에 맞는 JPype 파일을 [다운로드]([https://www.lfd.uci.edu/~gohlke/pythonlibs/#jpype](https://www.lfd.uci.edu/~gohlke/pythonlibs/#jpype))한다. 다운로드 경로는 현재 터미널의 경로로 설정하면 된다. JPype 설치는 터미널에 다음과 같이 입력한다.

    ```python
    $ (venv) python -m pip install JPype1-1.2.0-cp37-cp37m-win32.whl
    ```

    버전이 맞지 않을 경우 플랫폼이 적합하지 않다는 오류가 발생한다. 파이썬 버전과 파이썬 bit를 확인하여 다시 설치한다.

3. 다음 코드를 입력해 Konlpy를 설치한다.

    ```python
    $ (venv) python -m pip install konlpy
    ```

    - 만약, 전 과정을 진행하기 전에 이미 설치 코드를 실행했다면 Requirement already satisfied 가 출력되며, import konlpy를 해도 실행이 안될 수 있다. 이 때는 konlpy를 제거한 후 다시 설치 코드를 실행한다.

        ```python
        $ (venv) pip uninstall konlpy
        ```

4. 콘솔에 `import konlpy`를 통해 제대로 설치되었는지 확인한다.

- 참고자료
    - [https://m.blog.naver.com/PostView.nhn?blogId=hs_715&logNo=221548450575&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=hs_715&logNo=221548450575&proxyReferer=https:%2F%2Fwww.google.com%2F)
    - [https://ellun.tistory.com/46](https://ellun.tistory.com/46)

### Mecab 설치

- mecab은 은전한닢이라고도 불리며, 비교적 많이 사용하는 형태소 분석기이다. 사전을 추가하여 사용할 수 있다.  
- 다음은 Mecab을 설치하는 방법이다.  

1. 설치에 필요한 파일을 다운로드한다.  

    - [mecab-ko-msvc]([https://github.com/Pusnow/mecab-ko-msvc/releases/tag/release-0.9.2-msvc-3](https://github.com/Pusnow/mecab-ko-msvc/releases/tag/release-0.9.2-msvc-3))
    - [mecab-ko-dic-msvc]([https://github.com/Pusnow/mecab-ko-dic-msvc/releases/tag/mecab-ko-dic-2.1.1-20180720-msvc-2](https://github.com/Pusnow/mecab-ko-dic-msvc/releases/tag/mecab-ko-dic-2.1.1-20180720-msvc-2))
    - [whl 파일]([https://github.com/Pusnow/mecab-python-msvc/releases/tag/mecab_python-0.996_ko_0.9.2_msvc-2](https://github.com/Pusnow/mecab-python-msvc/releases/tag/mecab_python-0.996_ko_0.9.2_msvc-2))

2. C 드라이브에 'mecab' 폴더를 생성하고, 다운로드한 파일의 알집을 풀어준다.  

    ![mecab](/img/mecab_1.JPG)

3. 파이썬 커맨드 창에서 메캅을 설치한다.  

    ```python
    $ pip install C:/mecab/mecab_python-0.996_ko_0.9.
    2_msvc-cp37-cp37m-win_amd64.whl
    ```

4. 가상환경을 사용한 경우 Pycharm 콘솔에서 설치를 진행한다.  

    ```python
    import pip
    from pip._internal import main as pipmain

    def install_whl(path):
      pipmain(['install', path])

    install_whl('C:/mecab/mecab_python-0.996_ko_0.9.2_msvc-cp37-cp37m-win_amd64.whl')
    ```

5. `from konlpy.tag import Mecab` 으로 설치가 제대로 되었는지 확인한다.  

  - 사용자 사전을 추가할 수도 있다. 사전 파일(.csv)의 형태는 다음과 같아야 한다.  

        > 단어,,,단어비용,품사태그,의미분류,종성유무,읽기,타입,첫품사,마지막품사,표현

      - 단어비용은 형태소 분석 시 해당 단어를 우선순위로 할 것인지를 결정한다. 예를 들어, '머신'과 '머신러닝'이 모두 사전에 존재한다면, '머신러닝을 활용한 텍스트 분류'라는 문장의 형태소를 분석할 경우 '머신'을 단어로 먼저 추출할지 '머신러닝'을 먼저 추출할지 결정한다. 단어비용이 높을 수록 우선 추출하게 된다. 단어비용은 생략할 수 있다.  
      - 품사태그의 종류는 [이 페이지](https://docs.google.com/spreadsheets/d/1OGAjUvalBuX-oZvZ_-9tEfYD2gQe7hTGsgUpiiBSXI8/edit?usp=sharing)에서 확인할 수 있다.  
      - 의미분류의 경우 인명, 지명 등을 표현하는 것으로, '*'로 대체하여 입력할 수 있다.  

        ```
        언택트,,,,NNG,*,F,언택트,*,*,*,*,*
        인공신경망,,,,NNG,*,T,인공신경망,*,*,*,*,*
        ```

    - 사용자 사전 설치 ([참고]([https://bitbucket.org/eunjeon/mecab-ko-dic/src/ce04f82ab0083fb24e4e542e69d9e88a672c3325/final/user-dic/?at=master](https://bitbucket.org/eunjeon/mecab-ko-dic/src/ce04f82ab0083fb24e4e542e69d9e88a672c3325/final/user-dic/?at=master)))

            ```
            # 사용자 사전 위치로 이동
            $ cd mecab-ko-dic/user-dic
            $ ll # 사전 리스트

            # 사용자 사전 추가
            $ ./tools/add-userdic.sh

            # 사용자 사전 확인
            $ cat user-사전명.csv

            # 사전 컴파일
            $ make install
            ```

- 윈도우에서는 Konlpy에서 Mecab을 불러오는 것을 지원하지 않는다는 이야기도 있어, `pip install eunjeon`을 사용하여 설치하는 방법도 있다.) 설치 후 다음 코드를 사용한다.

    ```python
    from eunjeon import Mecab
    mecab = Mecab()
    ```

    - Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools" 오류가 뜬다면, [이 페이지]([https://visualstudio.microsoft.com/ko/vs/older-downloads/](https://visualstudio.microsoft.com/ko/vs/older-downloads/))에서 "Microsoft Build Tools 2015 업데이트 3"를 설치한다.

- 참고자료
    - [https://joyhong.tistory.com/127](https://joyhong.tistory.com/127)
    - [https://somjang.tistory.com/entry/Python-pip-install-시-error-Microsoft-Visual-C-140-is-required-오류-해결-방법](https://somjang.tistory.com/entry/Python-pip-install-%EC%8B%9C-error-Microsoft-Visual-C-140-is-required-%EC%98%A4%EB%A5%98-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95)
    - [https://edkoon35.github.io/2017/08/03/elasticsearch-seunjeon-user-dic/](https://edkoon35.github.io/2017/08/03/elasticsearch-seunjeon-user-dic/)

### Colab에서 Mecab 사용하기

- Mecab 형태소 분석기를 설치한다. (한 줄씩 실행한다.)

    ```python
    ! git clone https://github.com/SOMJANG/Mecab-ko-for-Google-Colab.git

    cd Mecab-ko-for-Google-Colab

    ! bash install_mecab-ko_on_colab190912.sh
    ```

- 사전을 설치한다.

    ```python
    import os

    # mecab-ko-dic 설치
    os.chdir('/tmp')
    !curl -LO https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/mecab-ko-dic-2.1.1-20180720.tar.gz
    !tar -zxvf mecab-ko-dic-2.1.1-20180720.tar.gz
    os.chdir('/tmp/mecab-ko-dic-2.1.1-20180720')
    !./autogen.sh
    !./configure
    !make
    # !sh -c 'echo "dicdir=/usr/local/lib/mecab/dic/mecab-ko-dic" > /usr/local/etc/mecabrc'
    !make install
    ```

- 사용자 사전을 추가하고 싶다면, /tmp/mecab-ko-dic-2.1.1-20180720/user-dic 폴더에 파일을 저장하고 코드를 실행한다.

    ```python
    import os
    os.getcwd()
    # user_dict을 /tmp/mecab-ko-dic-2.1.1-20180720/user-dic 여기 폴더에 넣고 돌리자.

    cd /tmp/mecab-ko-dic-2.1.1-20180720

    ! ./tools/add-userdic.sh # 사전 컴파일

    !make install
    ```

- 오류가 뜬다면 [여기]([https://github.com/konlpy/konlpy/issues/144#issuecomment-372553210](https://github.com/konlpy/konlpy/issues/144#issuecomment-372553210))를 참고한다.

- 참고자료
    - [https://beausty23.tistory.com/61](https://beausty23.tistory.com/61)
    - [https://victorydntmd.tistory.com/264](https://victorydntmd.tistory.com/264)


## SoyNLP

- [Soynlp](https://github.com/lovit/soynlp)는 미등록 단어들을 인식할 수 있는 형태소 분석기이다. 비지도 방식을 사용해 사전을 추가하기 때문에 아주 세부적인 분야에 대해서만 분석을 진행하거나 새로운 용어들이 많이 출현하는 분야에 대해 분석을 할 때 사용하기 용이하다.
- 그러나 아직(2021/01) 형태소 분석기를 제공하고 있지 않고, 각종 '점수'를 사용하여 단어를 추출하는 방식으로 제공된다.
- 설치는 다음 코드를 실행한다. (Colab에서도 동일한 방식으로 진행된다.)

    ```python
    $ pip install soynlp
    ```

- Soynlp의 경우 어절 내 명사(L token)의 오른쪽에 등장하는 조사 혹은 준조사(R token)의 분포를 이용하여 명사를 추출한다.  
- 따라서 형태소 분석기를 학습할 때 한 명사가 여러 종류의 조사와 함께 등장할 수 있도록 문장 단위의 데이터를 사용하는 것을 권장한다.  
  - 예를 들어, '다파장 측광 및 분광자료를 이용한 조기형 은하의 형성에 관한 연구'와 같이, 키워드 중심으로 이루어진 제목과 같은 문장을 학습할 경우 명사 추출기의 성능이 저하될 수 있다.  


## NLTK

- 다음 코드를 실행하여 NLTK를 설치한다.

    ```python
    $ pip install nltk
    ```

- NLTK 데이터를 설치한다. 데이터를 설치하지 않을 경우 LookupError가 발생한다.

    ```python
    import nltk
    nltk.download() # 'popular'만 받아도 된다. 괄호 안에 넣는다.
    # nltk.download("treebank")
    # nltk.download("popular")
    ```

- 참고자료
    - [https://wikidocs.net/22488](https://wikidocs.net/22488)




## 형태소 분석

- 특허 분야의 데이터를 가지고 형태소 분석기를 비교한 후 순위를 메긴 결과는 다음과 같다. 다만, 개인적인 의견이 반영된 것이며 특정 분야 내에서 비교한 것이므로 직접 확인이 필요하다.  

| 형태소분석기 | 특징 | 추출시간(순위) | 정확도(순위) |
|--------|------------|---------------|
| Khaiii | 카카오에서 세종계획 성과물을 활용하여 CNN으로 학습 | 4 | 4 |
| Soynlp | 학습데이터를 이용하지 않고 비지도 학습 접근법 이용 | 1 | 2 |
| Komoran | 공백이 포함된 형태소 단위로 분석 가능 | 3 | 3 |
| Mecab-ko | 일본의 MeCab 분석기로 세종계획 성과물을 학습 | 2 | 1 |

- 공식적인 문서에서 속도와 성능을 분석한 결과는 [이 페이지](https://konlpy-ko.readthedocs.io/ko/v0.4.3/morph/)에서 확인가능하다.  

---

## 한자 변환
- 분석 시 한자를 한글로 변환해야할 경우, hanja 모듈을 사용한다. 모든 한자를 변환해주는 것은 아니니 주의해야 한다.

    ```python
    $ pip install hanja
    ```

    ```python
    import hanja

    data = hanja.translate('STRING', 'substitution')
    ```
