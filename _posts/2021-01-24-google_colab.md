---
# layout: post
title: "[Programming] Google Colaboratory"
categories:
 - programming
tags: 
 - programming
 - colab
# share: true 
comments: true 
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
---

## Google Colaboratory

> [Google Colab](https://colab.research.google.com/) = Google Drive + Jupyter Notebook

구글 드라이브 상의 파일을 사용할 수 있고, 웹 상에서 협업이 가능하다는 장점이 있다.

특히, 구글 드라이브 용량이 무제한인 학생의 경우 굉장히 유용하게 사용할 수 있다.

구글 코랩은 기본 버전과 유료로 사용하는 Pro 버전이 있는데, 한 달에 9.99 달러로 사용할 수 있다.

아래 표(2021.01.04 기준)를 통해 비교를 하면, Pro 버전의 가장 큰 장점은 유지시간과 RAM이라는 생각이 든다. 


|구분|무료|유료|비고|  
|:---:|:---:|:---:|:---:|  
|CPU|Intel(R) Xeon(R) CPU @ 2.30GHz|Intel(R) Xeon(R) CPU @ 2.30GHz||
|GPU|Nvidia Tesla K80 또는 T4|T4, P100 등 무료보다 좋은 사양||
|RAM|12.72 GB|25.51 GB|고용량 RAM을 설정했을 때만 증가하니 필요시에는 런타임 유형 설정에서 변경해주어야 한다.|
|유지시간|12시간|24시간|정확히 24시간이 아니라, 끊기거나 줄어들 수 있다고 하니 유의해야 한다.|  

## 구글 코랩 사용하기

구글 드라이브에 접속한 후 오른쪽 버튼을 누르면 다음과 같은 화면이 뜨고, '더보기'에서 구글 코랩을 선택하면 새로운 파일을 생성할 수 있다.

### 런타임 유형 변경

![colab](/img/colab_0.jpg)

상단의 탭 중 '런타임'을 선택하면 런타임 유형을 변경할 수 있다.

![colab](/img/colab_1.jpg)

여기서 GPU를 사용하고자 하면 하드웨어 가속기를 변경하고, 이후 다시 output을 보고 싶다면 '셀 출력 생략'의 체크를 취소하면 된다.

![colab](/img/colab_2.JPG)

Pro 버전의 경우, 고용량 RAM을 사용하고자 하면 '런타임 구성'에서 '고용량 RAM'을 선택할 수 있다.

![colab](/img/colab_3.JPG)

사용하고 있는 코랩의 사양을 확인하고 싶으면, 아래 코드를 입력하고 실행하면 된다.

```python
!cat /etc/issue.net # OS 확인

!head /proc/cpuinfo # CPU 정보

!head -n 3 /proc/meminfo # 메모리 정보

!nvidia-smi # GPU 정보
```

### 모듈 설치

대부분의 모듈은 코랩에 자체적으로 설치가 되어있지만, 그렇지 않는 모듈도 존재한다. 이 때는 로컬에서 설치했던 것과 동일하게 설치할 수 있다.

```python
!pip install 모듈명
```

### 구글 드라이브와 Colab 연동

드라이브에 위치한 파일을 불러오기 위해서는 드라이브와 코랩을 연동(Mount)해주어야 한다. 매번 접속할 때마다 연동을 해주어야 하고, 코랩을 작동하지 않은 상태로 2시간이 경과할 경우 다시 연결해주어야 한다. 이 때 연동할 수 있는 방법은 두 가지가 있다.

드라이브에 위치한 파일을 불러오기 위해서는 드라이브와 코랩을 연동(Mount)해주어야 한다. 매번 접속할 때마다 연동을 해주어야 하고, 코랩을 작동하지 않은 상태로 2시간이 경과할 경우 다시 연결해주어야 한다. 이 때 연동할 수 있는 방법은 두 가지가 있다.

1. 왼쪽 탭의 '폴더' 버튼 클릭

    협업하지 않고 개인이 혼자 사용할 때는 이 방법을 추천한다. 클릭 한 번으로 인증없이 바로 사용할 수 있고, 따로 연동하지 않아도 새로 실행할 때 연동되기도 하기 때문이다. 왼쪽 탭 폴더 모양을 클릭한 후 상단의 폴더 아이콘을 클릭하면 '노트북이 Google Drive 파일에 액세스하도록 허용하시겠습니까?'라는 문구가 뜬다.  

    ![colab](/img/colab_4.JPG)

    연결 버튼을 클릭하면 자동으로 마운팅이 된다.  

2. 코드를 실행하여 연결

    코드를 실행하는 방식으로 연결하려면, 다음과 같은 코드를 실행한다.

    ```python
    from google.colab import drive
    drive.mount('/content/gdrive')
    ```

    ![colab](/img/colab_5.JPG)  

    '/content/'뒤의 'gdrive' 자리에는 본인이 사용하려는 폴더명으로 지정해줄 수 있다.

    예를 들어, 구글 드라이브 'data.csv'라는 파일을 불러오고 싶을 때 구글드라이브 명을 'gdrive'로 설정할 경우 파일 경로는 '/content/gdrive/data.csv'가 되는 것이고 구글드라이브 명을 'MyDrive'로 설정할 경우 파일 경로는 'content/MyDrive/data.csv' 이런 식으로 되는 것이다. 따라서 본인이 사용하기 편한 폴더명을 지정하면 된다.

    이렇게 코드를 통해 불러올 경우 한 번의 인증과정을 거쳐야 하는데, URL로 접속한 후 액세스를 허용하고, 주어지는 인증 코드를 복사해서 칸에 붙여넣으면 'Mounted at /content/구글드라이브폴더명' 라는 결과가 출력된다.

## Matplotlib에서 한글 사용

Matplotlib을 사용하고자 할 때 미리 폰트를 설정해주지 않으면 한글이 깨지는 현상이 발생한다. 따라서 다음 코드를 실행한 후 런타임을 재시작하면 정상적으로 출력하는 것을 볼 수 있다.

```python
!sudo apt-get install -y fonts-nanum
!sudo fc-cache -fv
!rm ~/.cache/matplotlib -rf
```

```python
import matplotlib.pyplot as plt
plt.rc('font', family='NanumBarunGothic')
```


## 사용자 파일(.py) 불러오기

사용자가 제작한 py 파일을 코랩에서 사용해야할 때가 있다. 그럴 때는 두 가지 방법을 사용하여 사용자 파일을 불러올 수 있다.

1. py 파일을 로컬 환경에 저장한 후 불러온다.

    ```python
    from google.colab import files
    src = list(files.upload().values())[0]
    open('사용자_파일.py','wb').write(src)
    ```

    위 코드를 입력하면 다음과 같이 파일을 업로드할 수 있다.

    ![colab](/img/colab_8.png)

    업로드를 한 후 다음 코드를 실행하여 사용자 파일을 임포트한다.

    ```python
    import 사용자_파일
    ```

    (+ 왼쪽 판넬에서 업로드를 선택한 후 직접적으로 업로드하는 방식도 존재한다.

    ![colab](/img/colab_10.png)

    Files 밑의 첫번째 아이콘을 클릭한다.)

2. 드라이브 환경에 저장한 후 불러온다.

    이 방법은 드라이브 환경에 있는 파일을 코랩 상으로 복사해와서 사용하는 방식이다.

    코랩에서 마운팅을 진행한 후, `!pwd`를 입력하면 현재 위치를 확인할 수 있다. 

    ![colab](/img/colab_6.png)

    'content'가 뜬다면 다음 코드를 통해 해당 파일을 불러올 수 있다.

    ```python
    ##### !cp [복사할 파일 경로] [이동할 파일 경로]
    !cp drive/마운팅한 구글드라이브/사용자 파일이 위치한 경로/사용자_파일.py .
    ```

    이 때 파일 경로에 특수문자(띄어쓰기, 괄호 등)이 포함된다면 앞에 `\`를 입력해준다. (예를 들자면 '사용자 폴더/사용자_파일.py' 대신 '사용자\ 폴더/사용자_파일.py' 이런 식)

    코드를 실행한 후 바로 해당 파일을 임포트하면 좌측 판넬에 py 파일이 생기면서 사용할 수 있게 된다.

    ![colab](/img/colab_7.png)

- 참고자료:
    - [https://zzsza.github.io/data/2018/08/30/google-colab/](https://zzsza.github.io/data/2018/08/30/google-colab/)
    - [https://stackoverflow.com/questions/48905127/importing-py-files-in-google-colab](https://stackoverflow.com/questions/48905127/importing-py-files-in-google-colab)