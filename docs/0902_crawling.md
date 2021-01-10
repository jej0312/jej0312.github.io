

```python
from IPython.display import Image
```

# 스크래핑과 크롤링

- **크롤링(crawling)**: 무수히 많은 인터넷 상의 페이지(문서, html 등)를 수집해서 분류하고 저장한 후에 나중에 쉽게 찾아볼 수 있도록 하는 역할을 하는 일종의 로봇이다. 즉, 웹 크롤러(자동화 봇)가 일정 규칙으로 웹페이지를 브라우징 하는 것을 말한다.  
- **스크래핑(scarping)**: 웹 사이트 상에서 원하는 정보를 추출하는 기술을 말한다.  
- **파싱(parsing)**: 언어학적 측면에서 문장해석을 하여 문장 구조를 결정하는 것을 말한다. 즉, 분석의 측면에서 우리가 원하는 의미를 찾는 과정이라고 할 수 있다.  



- 크롤링은 돌아 다니는 것, 스크래핑은 긁어오는 것으로, 둘은 다른 의미를 갖고 있으나 일반적으로 '크롤링을 한다'고 하면 두 가지를 모두 하는 것이기에 **이 둘을 구분하는 것은 크게 의미가 없다.**  


## 스크래핑
- 정적인 페이지 스크래핑을 하기 위해서는 `requests`와 `BeautifulSoup`라는 모듈을 사용한다.  
- 동적인 페이지 스크래핑은 `Selenium`이라는 모듈을 사용하여 클릭, 스크롤 등의 액션을 수행하면서 페이지의 정보들을 받아온다.  



```python
from bs4 import BeautifulSoup
import requests
import pandas as pd
import numpy as np
```

- 페이지 정보는 아래와 같이 얻을 수 있다.  
- 구글 크롬을 이용하여 스크래핑을 진행하는 것을 추천한다.  


```python
## HTTP GET Request
req = requests.get('https://news.naver.com/')

## HTML 소스 가져오기
html = req.text
## HTTP Header 가져오기
header = req.headers
## HTTP Status 가져오기 (200: 정상)
status = req.status_code
## HTTP가 정상적으로 되었는지 (True/False)
is_ok = req.ok
```


```python
soup = BeautifulSoup(html, 'html.parser')
```


```python
"today's date: %s" % (', '.join(str(x) for x in ' '.join(header['date'].split(',')).split()[0:4]))
```




    "today's date: Tue, 01, Sep, 2020"



### 최근 기사 정보 크롤링
- 우선 네이버 최근 기사 정보를 받아오는 방법을 배워보자.  
- 스크래핑 하고자 하는 페이지에서 빈 부분에 오른쪽 마우스를 클릭하면 '검사'라는 옵션이 나타난다. 이를 클릭하면 html 구조를 확인할 수 있다.  
- 원하는 태그는 화살표 모양을 클린한 후 마우스를 가져다대면 그 부분이 회색으로 변한다. 이 때 클릭하면 오른쪽 검사창에서도 그 부분의 태그가 설정된다.  
- 정보를 가져오고 싶으면, 왼쪽의 '...' 버튼을 클릭한 후 'copy selector'를 선택하여 사용한다.  

![html 구조](./img/html구조.png)

- 이 외에도 정보를 불러오는 방법은 다양하다.  
  - 태그에 대해 이해할 때 도움이 되었던 [글](https://m.blog.naver.com/kiddwannabe/221177292446) 하나를 소개한다.  
  

- 일단 정보를 얻기 위한 코드를 짜보자.  
  - 개별 정보를 수집할 때 오류가 나는지 확인한 후에 for 문으로 한 번에 수집하고자 한다.  


```python
len(soup.select('div > h4.tit_sec')) # 카테고리 수
```




    6




```python
soup.select('div > h4.tit_sec')[0].text # 첫 번째 카테고리
```




    '정치'




```python
len(soup.select('div.mtype_list_wide > ul > li > a')) # 기사 수
```




    30




```python
soup.select('div.mtype_list_wide > ul > li > a')[0] # 첫 번째 기사 정보
```




    <a class="nclicks('hom.airscont','880000C2_000000000000000004464551', 'airsGParam', '0', 'news_sec_v2.0', 'oQoSbcD2uIQSPqgM')" href="https://news.naver.com/main/read.nhn?mode=LSD&amp;mid=shm&amp;sid1=100&amp;oid=008&amp;aid=0004464548">
    <strong>국방예산 증가폭 7%→5%, 여전히 MB-朴 시절보다 높다</strong>
    </a>




```python
soup.select('div.mtype_list_wide > ul > li > a')[0]['href'] # 첫 번째 기사 링크
```




    'https://news.naver.com/main/read.nhn?mode=LSD&mid=shm&sid1=100&oid=008&aid=0004464548'



- 출판사, 작성일자 등 필요한 정보가 빠져있기 때문에 더 자세한 정보를 크롤링하려 한다.  
- 앞서 추출한 기사 링크로 접속하여 웹페이지 정보를 재확인해보자.  


```python
req = requests.get('https://news.naver.com/main/read.nhn?mode=LSD&mid=shm&sid1=100&oid=008&aid=0004464548')
html = req.text
soup = BeautifulSoup(html, 'html.parser')

# 출판사
if len(soup.select('#main_content > div.article_header > div.press_logo > a > img')) > 0: 
    pb = soup.select('#main_content > div.article_header > div.press_logo > a > img')[0]['title']
    pb = pb.replace('기사목록','').replace('신문\r\n\t\t\t\t\t\t','').replace('신문게재기사만','')
    pb = pb.rstrip()
    pb = pb.lstrip()
else:
    pb = np.nan

# 기사 내용
content = soup.select('#articleBodyContents')[0].text
content = content.replace('\n\n\n\n\n// flash 오류를 우회하기 위한 함수 추가\nfunction _flash_removeCallback() {}\n\n','')
content = content.replace('\n','')
content = content.replace('\t','')


info = {
    'date': soup.select('span.t11')[0].text,
    'publisher': pb,
    'keyword': content 
};
info
```




    {'date': '2020.09.01. 오후 3:51',
     'publisher': '머니투데이',
     'keyword': '[머니투데이 최경민  기자] [[the300]전투기, 차세대 잠수함 개발예산 모두포함…병장 월급 60만원]2021년도 국방예산은 문재인 정부들어 가장 낮은 5%대 상승폭을 보였다. 그럼에도 문재인 정부의 자주국방 기조는 여전하다는 평가다. 국방예산 증가율도 앞선 정부들에 비해서는 여전히 높은 수준이다.국방부는 1일 내년도 국방예산을  52조9174억원으로 편성했다고 밝혔다. 올해 본예산(50조2000억원) 대비 5.5% 증가한 수치다.━여전히 이명박·박근혜 정부보다 큰 상승폭━증가 추세는 유지했지만 증가폭은 줄었다. 문재인 정부 출범 이후 국방비 증가율이 7%를 넘었던 것(2018년 7.0%, 2019년 8.2%, 2020년 7.4%)에 비해 낮아졌다. 지난달 발표한 \'21~25 국방중기계획\' 목표치(53조2000억원) 보다는 약 3000억원 적다.올해 국방예산 증가율 감소는 예견된 측면이 있다. 코로나19(COVID-19)의 영향으로 여타 경제 분야에 예산이 쓰일 곳이 늘어났기 때문이다. 군 관계자는 "국방예산 상승폭이 예년만은 못할 것이라고 지켜봐왔다"며 "코로나19를 생각하면 어쩔 수 없는 측면"이라고 설명했다.[양양=뉴시스] 김경목 기자 = 훈련 중인 국군. 2020.01.02.   photo31＠newsis.com그럼에도 \'5.5%\'라는 수치에는 자주국방 정책을 꾸준히 추진하겠다는 의지가 담겨있다는 평가다. 문재인 정부에서는 낮은 축이지만, 여타 정부의 평균 국방예산 증가율을 상회하기 때문이다. 박근혜 정부는 4.2%, 이명박 정부는 5.2%의 국방예산 상승폭을 보였다. 국방부 관계자는 "목표한 전력 증강과 군사력 운영을 차질 없이 달성할 수 있도록 내실있게 예산을 편성했다"고 설명했다.━방위력개선비 상승폭 줄어든 이유━내년도 국방예산 상승폭의 감소는 방위력개선비의 비중 축소 때문이다. 군사력 증강을 위한 방위력개선비는 전년 대비 2.4% 증가한 17조738억원 규모로 편성됐다. 문재인 정부들어 지난해까지 평균 11.0% 증가했던 분야다.국방부는 "내년에 기존 대형사업들이 최종 전력화 시기에 근접하면서 지불액이 줄어들었다"고 설명했다. 지난 2~3년 동안 투자해온 사업들의 결실을 맺는 해가 2021년이기 때문에 불가피하게 예산 증가율이 줄었다는 의미다.군 전력화에는 문제가 없다는 입장이다. 실제 한국형 전투기(KF-X)인 보라매(9069억원), 차세대 잠수함(5259억원), K-2전차(3094억원) 등 첨단무기체계 개발예산들이 모두 포함됐다. 국방개혁 2.0 추진의 핵심인 핵·WMD(대량살상무기) 대응체계 구축 및 전작권 전환, 군구조개편 추진에 필요한 재원 역시 반영됐다.【거제=뉴시스】박진희 기자 = 문재인 대통령과 부인 김정숙 여사가 14일 경남 거제시 대우조선해양 옥포조선소에서 열린 한국 최초 3000톤급 잠수함인 \'도산안창호함\' 진수 및 안전항해 기원의식을 하고 있다. 2018.09.14.   pak7130@newsis.com국방부 관계자는 "무기 획득 예산이 일시적으로 줄어드는 대신에 국방 R&D;(연구개발) 예산을 전년비 3333억원(8.5%) 증가한 4조2524억원으로 편성했다"며 "자주국방 역량 강화 기반 구축에 중점을 뒀다"고 강조했다.━전력운영비는 10년 간 최고 증가율━군사력 운영에 소요되는 전력운영비는 전년 대비 7.1% 증가한 35조8436억원 규모다. 최근 10년 간 가장 높은 수준의 증가율이다. 국방운영 첨단화·효율화 등에 중점을 뒀다.방위력개선비와 달리 전력운영비의 비중이 높아진 것에 대해 국방부는 "그동안 확보된 군사력을 효과적으로 운용하기 위한 취지"라고 설명했다. 최근 확보한 첨단전력을 위한 후속 군수지원 등이 포함됐다는 의미다. F-35A, 고고도무인정찰기(HUAV) 등의 장비유지비를 7.7% 증액한 이유다.강화도 \'배수로 월북\'을 통해 드러난 병력 중심 경계의 한계를 보완하는 예산도 대거 포함됐다. 노후 경계시설 대폭 보강(1389억원), AI(인공지능) 기반 고성능 감시장비 도입을 통한 해안경계력 강화(1968억원) 등의 사업이 추진된다.[인천=뉴시스]김병문 기자 = 합동참모본부는 지난 27일 정례브리핑에서 최근 월북한 것으로 추정되는 탈북민 김모 씨를 특정할 수 있는 유기된 가방을 발견, 확인하고 현재 정밀조사 중이라고 밝혔다. 사진은 28일 오전 김씨의 가방이 발견된 것으로 추정되는 인천 강화군 강화읍 월곳리의 한 배수로 모습. 2020.07.28.   dadazon@newsis.com장병복지 개선에도 중점을 뒀다. 일단 병 봉급을 12.5% 인상하기로 했다. 병장의 월급은 올해 54만900원에서 내년 60만8500원으로 오른다. 급식단가를 인상하고, 민간조리원을 확대하는 조치도 이뤄진다. 군 단체보험 제도의 도입을 추진하고, 현역 및 상근예비역 전원에게 1인당 월 1만원의 이발비를 지급할 계획이다.국방부 관계자는 "장병복무여건을 획기적 으로 개선하여 사기충천한 선진병영문화를 정착시켜 나갈 것"이라고 설명했다.최경민  기자 brown@mt.co.kr▶줄리아 투자노트▶조 변호사의 가정상담소 ▶머니투데이 구독하기 <저작권자 ⓒ \'돈이 보이는 리얼타임 뉴스\' 머니투데이, 무단전재 및 재배포 금지>'}



- 이제 진짜로 스크래핑을 시작해보자.  
- 참고로, copy selector에서 나오는 child 선택자 'nth-child'는 지원하지 않기 때문에 **'nth-of-type'**로 바꿔주어야 한다.  


```python
def scrape_detail_page(response):
    r = requests.get(response)
    h = r.text
    s = BeautifulSoup(h, 'html.parser')
    
    if len(s.select('#main_content > div.article_header > div.press_logo > a > img')) > 0:
        pb = s.select('#main_content > div.article_header > div.press_logo > a > img')[0]['title']
        pb = pb.replace('기사제공','').replace('신문\r\n\t\t\t\t\t\t','').replace('신문게재기사만','')
        pb = pb.rstrip().lstrip()
    else:
        pb = np.nan
        
    content = s.select('#articleBodyContents')[0].text
    content = content.replace('\n\n\n\n\n// flash 오류를 우회하기 위한 함수 추가\nfunction _flash_removeCallback() {}\n\n','')
    content = content.replace('\n','')
    content = content.replace('\t','')
    info = {
        'title': s.select('#articleTitle')[0].text,
        'date': s.select('span.t11')[0].text,
        'publisher': pb,
        'keyword': content 
    };
    return info
```


```python
req = requests.get('https://news.naver.com/')
html = req.text
soup = BeautifulSoup(html, 'html.parser')
news = {'section': [], 'title': [], 'date': [], 'publisher': [], 'keyword': [], 'url': []}
for i in range(len(soup.select('div > h4.tit_sec'))):
    for n in range(5):
        news['section'].append(soup.select('div > h4.tit_sec')[i].text)
        tt= (soup.select('div.mtype_list_wide > ul > li > a')[(i)*5+n].text)
        news['title'].append(tt.replace('\n',''))
        url = soup.select('div.mtype_list_wide > ul > li > a')[(i)*5+n]['href']
        news['url'].append(url)
        news['date'].append(scrape_detail_page(url)['date'])
        news['publisher'].append(scrape_detail_page(url)['publisher'])
        news['keyword'].append(scrape_detail_page(url)['keyword'])
```


```python
dt = pd.DataFrame({'section':news['section'],
                   'title':news['title'],
                   'date':news['date'],
                   'publisher':news['publisher'],
                   'keyword':news['keyword'],
                   'url':news['url']});
dt
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
      <th>section</th>
      <th>title</th>
      <th>date</th>
      <th>publisher</th>
      <th>keyword</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>정치</td>
      <td>박수 받던 김종인, 정강정책 제동에도 새 당명은 지켰다</td>
      <td>2020.09.01. 오후 4:46</td>
      <td>파이낸셜뉴스</td>
      <td>1일 온라인 의총서 의견수렴..‘반대 多’정치개혁특위 구성해 검토하기로김종인 "마음...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>정치</td>
      <td>'포스트 아베' 출마 日기시다 "한일대화 환경조성 중요"(종합)</td>
      <td>2020.09.01. 오후 4:43</td>
      <td>연합뉴스</td>
      <td>고립무원의 상황서 출사표…'스가 대세론' 막지는 못할 듯'포스트 아베' 출마선언 후...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>정치</td>
      <td>'성평등 어린이책 왜 회수했나'에 여가부 "코로나 때문에"</td>
      <td>2020.09.01. 오후 4:41</td>
      <td>뉴시스</td>
      <td>與 여가위원들, '나다움 어린이책' 회수 융단폭격유정주 "철회하라"…이정옥 "사회적...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>정치</td>
      <td>"그런 취지 전혀 아니다"…홍남기, '이재명 철 없다' 발언 해명</td>
      <td>2020.09.01. 오후 4:41</td>
      <td>아이뉴스24</td>
      <td>[아이뉴스24 권준영 기자] 홍남기 부총리 겸 기획재정부 장관이 1일 이재명 경기도...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>정치</td>
      <td>‘박근혜 사람’ 소리 들은 홍남기 “제가 어떻게 도지사한테…”</td>
      <td>2020.09.01. 오후 4:41</td>
      <td>세계일보</td>
      <td>이재명에 “철없다”는 野 의원에 동조 ‘논란’    홍남기 부총리 겸 기획재정부 장...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>경제</td>
      <td>임대사업자 등기 표기, 12월10일부터 의무화</td>
      <td>2020.09.01. 오후 4:54</td>
      <td>서울경제</td>
      <td>[서울경제] 오는 12월10일부터 임대사업자는 해당 주택이 공적 의무가 부여된 주택...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>경제</td>
      <td>‘비대면 식사’ 선호에 도시락 매출 ‘쑥쑥’</td>
      <td>2020.09.01. 오후 4:50</td>
      <td>한겨레</td>
      <td>편의점 3사, 8월 중순 이후 도시락 매출 일제히 증가사무실·가정 상권 불문하고 증...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>경제</td>
      <td>규제지역 묶인 '고양·양주·인천' 미분양 늘었다</td>
      <td>2020.09.01. 오후 4:49</td>
      <td>서울경제</td>
      <td>서울과 지방 줄면서전국은 감소세 지속수도권은 13.5% 늘어인천의 한 아파트 공사현...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>경제</td>
      <td>거래절벽에도...'상계주공16' 매매 3건 모두 신고가</td>
      <td>2020.09.01. 오후 4:48</td>
      <td>서울경제</td>
      <td>7·10대책후 갭투자 문의 줄었지만'노도강' 실거주 목적 수요는 꾸준'삼익세라믹' ...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>경제</td>
      <td>국내 최대 퍼블릭 골프장 스카이72 새 주인 찾는다</td>
      <td>2020.09.01. 오후 4:47</td>
      <td>중앙일보</td>
      <td>국내 최대 퍼블릭 골프 단지인 인천국제공항 부지 내 스카이72 골프장 전경. 사...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>사회</td>
      <td>3일 새벽 부산 상륙 ‘마이삭’, ‘돌 날아갈’ 강풍에 400㎜ 폭우</td>
      <td>2020.09.01. 오후 4:54</td>
      <td>한겨레</td>
      <td>제8호 태풍 바비보다 강도·강풍·강수 강해 중심 최대풍속 시속 150㎞ 강풍반경 3...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>사회</td>
      <td>제주 30대 여성 살해 피의자 “생활고에 탑차 끌고 대상 물색”</td>
      <td>2020.09.01. 오후 4:52</td>
      <td>국민일보</td>
      <td>제주공항 인근에서 30대 여성을 살해한 혐의를 받고 있는 20대 피의자는 제주시 민...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>사회</td>
      <td>태풍 '마이삭' 북상에 고층빌딩 많은 부산 바짝 긴장</td>
      <td>2020.09.01. 오후 4:49</td>
      <td>연합뉴스</td>
      <td>입주 후 첫 태풍 맞는 101층 엘시티·마린시티 강풍·월파 대비2003년 '매미' ...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>사회</td>
      <td>시민단체, ‘미필적 고의 살인’ 혐의 최대집 의협회장 고발</td>
      <td>2020.09.01. 오후 4:49</td>
      <td>이데일리</td>
      <td>애국국민운동대연합, 1일 최대집 의협 회장 등 고발단체 “파업으로 생명 잃은 환자 ...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>사회</td>
      <td>의대생 내부고발 플랫폼 등장... "동기끼리 사상 검증, 전체주의 분위기"</td>
      <td>2020.09.01. 오후 4:48</td>
      <td>오마이뉴스</td>
      <td>[스팟인터뷰] 운영자 "의대생 시험 거부 및 동맹휴학의 이면을 고발합니다"  [강연...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>생활/문화</td>
      <td>"사찰 법회 등 대면행사 전면 중단", 조계종 3일부터 코로나 추가 대응</td>
      <td>2020.09.01. 오후 4:34</td>
      <td>경향신문</td>
      <td>[경향신문] 서울 조계사 대웅전에서 일부 신도들이 지난 4월 개별적으로 기도를 드리...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>생활/문화</td>
      <td>문재인 대통령 "BTS 케이팝의 새 역사 썼다"</td>
      <td>2020.09.01. 오후 4:31</td>
      <td>파이낸셜뉴스</td>
      <td>사회관계망서비스 통해 축하 메시지 전해 문재인 대통령의 페이스북 캡쳐 화면  [파이...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>생활/문화</td>
      <td>"국민혈세로 치료해선 안돼"…시민단체, 전광훈 경찰 고발</td>
      <td>2020.09.01. 오후 4:25</td>
      <td>뉴스1</td>
      <td>"전광훈 목사·사랑제일교회·한기총에 구상권 청구"1일 경찰 고발에 나선 오천도 애국...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>생활/문화</td>
      <td>"교회 불신 떠나는 사람 늘어..."전광훈 사태, 반성해야"</td>
      <td>2020.09.01. 오후 4:20</td>
      <td>뉴시스</td>
      <td>교회개혁실천연대, '교단총회에 드리는 우리의 제안'[서울=뉴시스]교회개혁실천연대, ...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>생활/문화</td>
      <td>방대본 "코로나 위·중증 환자 104명 급증...사망자도 늘 듯"</td>
      <td>2020.09.01. 오후 4:16</td>
      <td>중앙일보</td>
      <td>지난 7월 20일 오후 광주 동구 산수동 문화마당에서 마스크를 착용한 어르신이 ...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>세계</td>
      <td>낮잠 잤는데 배가 ‘꿈틀꿈틀’…러 여성 입에서 꺼낸 1.2m 뱀</td>
      <td>2020.09.01. 오후 4:45</td>
      <td>국민일보</td>
      <td>인스타그램러시아 여성의 몸에서 뱀이 나왔다. 야외에서 자는 도중에 뱀이 몸에 들어간...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>세계</td>
      <td>대만 방문한 체코 상원의장 연설서 “나는 대만인이다”</td>
      <td>2020.09.01. 오후 4:44</td>
      <td>조선일보</td>
      <td>1960년대 케네디의 反共 연설 차용하며 중국어로 “나는 대만인”이라고 해 ” ’하...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>세계</td>
      <td>신문 보고 3년만에 알게 된 전 남편의 '바람'</td>
      <td>2020.09.01. 오후 4:44</td>
      <td>연합뉴스</td>
      <td>NYT 결혼란에 총각 행세한 전 남편 사연 실려"그가 현재 신부를 만날 때 나의 남...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>세계</td>
      <td>중국 또 실탄훈련 예고…2∼4일 서해남부서 훈련</td>
      <td>2020.09.01. 오후 4:42</td>
      <td>연합뉴스</td>
      <td>미중, 남·동중국해·대만해협·서해서 군사 긴장 고조[그래픽] 남·동중국해 미중 갈등...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>세계</td>
      <td>틱톡 美사업 새주인 찾나? '공'은 美中정부로</td>
      <td>2020.09.01. 오후 4:37</td>
      <td>파이낸셜뉴스</td>
      <td>- 中 수출 허가 새 걸림돌- 허가 지연하면 결정권은 美 [알링턴=신화/뉴시스] 중...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>IT/과학</td>
      <td>블라인드, 조직 내 숨은 문제 터지기 전 알려준다</td>
      <td>2020.09.01. 오후 4:37</td>
      <td>ZDNet Korea</td>
      <td>기업문화 컨설팅 플랫폼 &amp;apos;블라인드 허브&amp;apos; 출시(지디넷코리아=백봉삼...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>IT/과학</td>
      <td>이동통신 가입자 7천만 육박…5G 가입자 증가세 유지</td>
      <td>2020.09.01. 오후 4:33</td>
      <td>연합뉴스</td>
      <td>7월 5G가입자 48만7천여명↑ 총785만7천여명…8월 800만 돌파 예상5G[연합...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>IT/과학</td>
      <td>전공의 "추진 정책 원점 재논의 문서화" vs 정부 "의대 정원 정책 이미 중단"</td>
      <td>2020.09.01. 오후 4:33</td>
      <td>한국경제</td>
      <td>평행선 달리는 의·정 갈등정부의 의료 정책에 반대하는 전공의 파업이 12일째를 맞았...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>IT/과학</td>
      <td>이재용 결국 기소... 檢 "조직적 범죄" vs 삼성 "일방적 주장"</td>
      <td>2020.09.01. 오후 4:20</td>
      <td>블로터</td>
      <td>경영권 불법승계 의혹을 받고 있는 이재용 삼성전자 부회장이 결국 불구속 기소됐다. ...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>IT/과학</td>
      <td>"안드로메다 은하 둘러싼 가스 우리 은하와 이미 충돌 중"</td>
      <td>2020.09.01. 오후 4:18</td>
      <td>연합뉴스</td>
      <td>허블 망원경으로 은하 헤일로 분석…"예상보다 훨씬 크고 복잡"안드로메다 은하와 이를...</td>
      <td>https://news.naver.com/main/read.nhn?mode=LSD&amp;...</td>
    </tr>
  </tbody>
</table>
</div>



### 많이 본 기사 정보 크롤링
- 이번에는 많이 본 기사 정보를 크롤링하자.  


```python
soup.select('#ranking_100 h5')[0].text
```




    '정치'




```python
for i in range(10):
    print(soup.select('#ranking_100 ul li')[i].text[2:]) # 기사 제목
    print('https://news.naver.com'+soup.select('#ranking_100 > ul > li > a')[i]['href']) # 기사 링크
```

    정경두 “추미애 아들 휴가 행정처리 정확히 안돼”…秋 “사실 아냐”(종합) 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=081&aid=0003120399&date=20200901&type=1&rankingSeq=1&rankingSectionId=100
    [속보] 최재성 청와대 정무수석, 코로나19 ‘음성’ 판정 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=022&aid=0003499105&date=20200901&type=1&rankingSeq=2&rankingSectionId=100
    민경욱, 자가격리 중 무단이탈로 고발되자 "세번이나 음성이라더니" 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=018&aid=0004727177&date=20200901&type=1&rankingSeq=3&rankingSectionId=100
    [속보] '미열 증상' 최재성 청와대 정무수석, 코로나19 '음성' 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=015&aid=0004408273&date=20200901&type=1&rankingSeq=4&rankingSectionId=100
    文 "빌보드 1위 축하"…알고보니 BTS 정국과 노래한 인연 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=025&aid=0003031187&date=20200901&type=1&rankingSeq=5&rankingSectionId=100
    김종민 “흑서 100권 내도 40% 조국 편”…진중권 “60%는?” 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=020&aid=0003306683&date=20200901&type=1&rankingSeq=6&rankingSectionId=100
    민경욱 전 의원, 자가격리 중 무단이탈…지자체 고발 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=437&aid=0000246508&date=20200901&type=1&rankingSeq=7&rankingSectionId=100
    정경두 “추미애 아들 휴가 처리, 정확히 못 했다” 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=005&aid=0001357649&date=20200901&type=1&rankingSeq=8&rankingSectionId=100
    민경욱 전 의원, 자가격리 중 무단이탈… 경찰 고발 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=005&aid=0001357642&date=20200901&type=1&rankingSeq=9&rankingSectionId=100
     "추미애 아들 병가 기이하다" 육군 중장 출신 신원식의 지적 
    https://news.naver.com/main/ranking/read.nhn?mid=etc&sid1=111&rankingType=popular_day&oid=025&aid=0003031193&date=20200901&type=1&rankingSeq=10&rankingSectionId=100
    


```python
req = requests.get('https://news.naver.com/')
html = req.text
soup = BeautifulSoup(html, 'html.parser')

news = {'section': [], 'title': [], 'date': [], 'publisher': [], 'keyword': [], 'url': []}
for i in range(6):
    for n in range(10):
        sec = '#ranking_10{}'.format(i)
        news['section'].append(soup.select(sec+' h5')[0].text)
#        tt= soup.select(soup.select(sec+' ul li')[i].text)[2:]
#        news['title'].append(tt.rstrip())
        url = 'https://news.naver.com'+soup.select(sec+' > ul > li > a')[n]['href']
        news['url'].append(url)
        news['date'].append(scrape_detail_page(url)['date'])
        news['publisher'].append(scrape_detail_page(url)['publisher'])
        news['keyword'].append(scrape_detail_page(url)['keyword'])
        news['title'].append(scrape_detail_page(url)['title'])
```


```python
dt = pd.DataFrame({'section':news['section'],
                   'title':news['title'],
                   'date':news['date'],
                   'publisher':news['publisher'],
                   'keyword':news['keyword'],
                   'url':news['url']});
dt
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
      <th>section</th>
      <th>title</th>
      <th>date</th>
      <th>publisher</th>
      <th>keyword</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>정치</td>
      <td>정경두 “추미애 아들 휴가 행정처리 정확히 안돼”…秋 “사실 아냐”(종합)</td>
      <td>2020.09.01. 오후 2:26</td>
      <td>서울신문</td>
      <td>“지휘관 구두 승인 했더라도 휴가 명령 서류상에 안 남겨져 절차상 오류”신원식 “서...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>정치</td>
      <td>[속보] 최재성 청와대 정무수석, 코로나19 ‘음성’ 판정</td>
      <td>2020.09.01. 오후 3:06</td>
      <td>세계일보</td>
      <td>최재성 청와대 정무수석이 1일 오후 신종 코로나바이러스 감염증(코로나19) 검...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>정치</td>
      <td>민경욱, 자가격리 중 무단이탈로 고발되자 "세번이나 음성이라더니"</td>
      <td>2020.09.01. 오후 1:53</td>
      <td>이데일리</td>
      <td>[이데일리 박지혜 기자] 민경욱 미래통합당 전 의원이 자가격리 중 무단으로 이탈했다...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>정치</td>
      <td>[속보] '미열 증상' 최재성 청와대 정무수석, 코로나19 '음성'</td>
      <td>2020.09.01. 오후 3:11</td>
      <td>한국경제</td>
      <td>사진=연합뉴스'미열 증상' 최재성 청와대 정무수석, 코로나19 '음성'한경닷컴 뉴스...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>정치</td>
      <td>文 "빌보드 1위 축하"…알고보니 BTS 정국과 노래한 인연</td>
      <td>2020.09.01. 오후 2:31</td>
      <td>중앙일보</td>
      <td>문재인 대통령이 1일 한국인 최초로 빌보드 ‘핫 100’ 1위를 차지한 방탄소년단...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>정치</td>
      <td>김종민 “흑서 100권 내도 40% 조국 편”…진중권 “60%는?”</td>
      <td>2020.09.01. 오후 2:43</td>
      <td>동아일보</td>
      <td>더불어민주당 김종민 의원이 최근 베스트셀러 1위를 달리고 있는 책 ‘조국 흑서(한번...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>정치</td>
      <td>민경욱 전 의원, 자가격리 중 무단이탈…지자체 고발</td>
      <td>2020.09.01. 오후 1:58</td>
      <td>JTBC</td>
      <td>민경욱 전 미래통합당 의원이 자가격리 중 무단으로 이탈했다가 고발됐습니다....</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>정치</td>
      <td>정경두 “추미애 아들 휴가 처리, 정확히 못 했다”</td>
      <td>2020.09.01. 오후 1:51</td>
      <td>국민일보</td>
      <td>추미애 “그런 사실 없다” 부인정경두 국방부 장관이 1일 오전 서울 여의도 국회에서...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>정치</td>
      <td>민경욱 전 의원, 자가격리 중 무단이탈… 경찰 고발</td>
      <td>2020.09.01. 오후 1:18</td>
      <td>국민일보</td>
      <td>민경욱, 당국의 고발에 “법적 근거 대봐”민경욱 전 미래통합당 의원이 자가격리 중 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>정치</td>
      <td>"추미애 아들 병가 기이하다" 육군 중장 출신 신원식의 지적</td>
      <td>2020.09.01. 오후 2:52</td>
      <td>중앙일보</td>
      <td>정경두 "행정조치 완벽하지 못했다"  정경두 국방부 장관(왼쪽)과 추미애 법무부 장...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>경제</td>
      <td>[단독]기업은행 직원, 76억 '셀프 대출'로 부동산 29채 쇼핑</td>
      <td>2020.09.01. 오전 10:23</td>
      <td>중앙일보</td>
      <td>국책은행인 기업은행의 한 직원이 최근까지 자신의 가족 앞으로 76억원어치 부동산 담...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>경제</td>
      <td>빚 660조 물려받아 1000조 물려주는 文정부…이런 빚폭주 없었다</td>
      <td>2020.09.01. 오전 10:31</td>
      <td>중앙일보</td>
      <td>2년 후 국가채무 1000조원 넘어GDP대비 채무 비율도 60% 육박1인당 2000...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>경제</td>
      <td>1억 넣고 2주 현실로?…카카오게임즈 청약 첫날 15조 몰려(종합2보)</td>
      <td>2020.09.01. 오후 3:26</td>
      <td>연합뉴스</td>
      <td>3시 현재 삼성 429대1, 한투 319대1, KB 528대1…SK바이오팜 최종 경...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>경제</td>
      <td>"직접 살테니 나가라면서요"… 세입자가 집주인 실거주 직접 들여다본다</td>
      <td>2020.09.01. 오전 10:50</td>
      <td>아시아경제</td>
      <td>[아시아경제 이춘희 기자] 앞으로 집주인의 실거주를 이유로 계약을 갱신당한 임차인이...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>경제</td>
      <td>[속보] 이재용측 "처음부터 목표 정한 수사…무리한 기소"</td>
      <td>2020.09.01. 오후 3:35</td>
      <td>매일경제</td>
      <td>[디지털뉴스국 news@mkinternet.com]▶ 팬데믹위기 해법 찾는다! - ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>경제</td>
      <td>결국 이재용 기소…서초동에 발묶인 삼성</td>
      <td>2020.09.01. 오후 2:01</td>
      <td>한국경제TV</td>
      <td>檢, 이재용 불구속 기소…"전면 재검토 결과"'심의위' 스스로 무력화…'정치적 포석...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>경제</td>
      <td>[시그널] 카카오게임즈, 첫날 SK바이오팜 제껴···청약증거금 13조 돌파</td>
      <td>2020.09.01. 오후 2:42</td>
      <td>서울경제</td>
      <td>삼성증권 400대1, KB증권 500대1SK바이오팜 경쟁률(323대1) 훌쩍카카오게...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>경제</td>
      <td>[속보]이재용측 "기소 부당, 삼성물산 합병은 합법적 경영활동"</td>
      <td>2020.09.01. 오후 3:42</td>
      <td>아시아경제</td>
      <td>[아시아경제 이창환 기자] 이창환 기자 goldfish@asiae.co.kr▶ 20...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>경제</td>
      <td>“1억원 넣어봐야 고작 30만원?”…12조 몰린 카카오게임즈</td>
      <td>2020.09.01. 오후 3:09</td>
      <td>파이낸셜뉴스</td>
      <td>삼성증권 마포지점. 카카오게임즈 공모주 청약 첫날, 청약을 위해 고객들이 대기하고 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>경제</td>
      <td>카카오게임즈 청약 마비사태…개미 신청폭주에 경쟁률 역대최대 `예감`</td>
      <td>2020.09.01. 오후 1:49</td>
      <td>매일경제</td>
      <td>공모청약 첫날부터 열풍…KB증권 오후 1시 기준 경쟁률 400대1SK바이오팜 첫날 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>사회</td>
      <td>태풍 '마이삭' 진로 '나리', '매미'와 닮은꼴…위력은 역대급</td>
      <td>2020.09.01. 오후 2:05</td>
      <td>연합뉴스</td>
      <td>중간 부근 최대 풍속 47ｍ 강도 '매우 강'내일 밤 제주 최근접…막대한 피해 우려...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>사회</td>
      <td>[속보] 당국 "코로나19 백신, 접종 우선순위 등 논의 착수"</td>
      <td>2020.09.01. 오후 2:45</td>
      <td>아시아경제</td>
      <td>[아시아경제 김흥순 기자] 김흥순 기자 sport@asiae.co.kr▶ 2020년...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>사회</td>
      <td>[속보]코로나19로 전국 8052개교 등교 중단…최다 기록</td>
      <td>2020.09.01. 오후 3:14</td>
      <td>채널A</td>
      <td>[속보]코로나19로 전국 8052개교 등교 중단…최다 기록백승우 기자strip@d...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>사회</td>
      <td>[속보] 코로나19로 전국 8052개교 등교 못 해…또 최다 기록</td>
      <td>2020.09.01. 오후 3:07</td>
      <td>매일경제</td>
      <td>[디지털뉴스국 news@mkinternet.com]▶ 팬데믹위기 해법 찾는다! - ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>사회</td>
      <td>"나를 고발하겠다고?"…민경욱 자가격리 중 무단 이탈(종합)</td>
      <td>2020.09.01. 오후 2:34</td>
      <td>연합뉴스</td>
      <td>인천시 연수구, 감염병예방법 위반 혐의로 민 전 의원 고발민경욱 전 의원[연합뉴스 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>사회</td>
      <td>"모두 미안" 극단선택 암시글···'여행에 미치다' 조준기 의식불명</td>
      <td>2020.09.01. 오전 11:44</td>
      <td>중앙일보</td>
      <td>'여행에미치다' 조준기 대표. [사진 조준기 인스타그램]           소셜...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>사회</td>
      <td>'여행에 미치다' 대표, 극단적 선택 암시 뒤 의식불명</td>
      <td>2020.09.01. 오후 1:35</td>
      <td>SBS</td>
      <td>공식 SNS 계정에 불법 성적 촬영물을 올렸다는 논란이 불거진 유명 여행정보 소개 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>사회</td>
      <td>[속보] 전국 8052개교, 등교 중단…또 최다 기록</td>
      <td>2020.09.01. 오후 3:10</td>
      <td>헤럴드경제</td>
      <td>[속보] 코로나19로 전국 8052개교, 등교 중단…또 최다 기록지난 1일 오전 광...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>사회</td>
      <td>경실련 “강경화는 3채 27억 , 추미애는 2채 14억”</td>
      <td>2020.09.01. 오전 11:33</td>
      <td>조선일보</td>
      <td>“최기영은 50억 공장, 진영은 16억 상가 3채”올해 3월 기준 현직 장관 18명...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>사회</td>
      <td>[속보] 이재용측 "처음부터 목표 정한 수사…무리한 기소"</td>
      <td>2020.09.01. 오후 3:38</td>
      <td>헤럴드경제</td>
      <td>[헤럴드경제DB][속보] 이재용측 "처음부터 목표 정한 수사…무리한 기소"onlin...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>30</th>
      <td>생활/문화</td>
      <td>'마이삭'도 오는데…10호 태풍 '하이선' 주말쯤 한반도 덮친다</td>
      <td>2020.09.01. 오전 9:17</td>
      <td>중앙일보</td>
      <td>제9호 태풍 ‘마이삭’이 3일 우리나라 남해안에 상륙할 것으로 예상되는 가운데 곧바...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>31</th>
      <td>생활/문화</td>
      <td>'여미'엔 무슨 일이…음란물 논란→비판 쇄도→조준기 대표 극단적 시도→회복</td>
      <td>2020.09.01. 오후 2:00</td>
      <td>뉴스1</td>
      <td>'여행에 미치다' 조준기 대표, 구조대 출동해 병원 이송공식 SNS에 음란물 게재로...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>32</th>
      <td>생활/문화</td>
      <td>[더오래]잠자기 최적 온도는? 에어컨 이 온도에 맞추세요</td>
      <td>2020.09.01. 오후 12:01</td>
      <td>중앙일보</td>
      <td>━   [더,오래] 박세인의 밀레니얼 웰니스(6)        오랜만에 한의원에 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>33</th>
      <td>생활/문화</td>
      <td>'여행에 미치다' 조준기, 극단적 선택 암시→병원 이송 [전문]</td>
      <td>2020.09.01. 오후 1:27</td>
      <td>한국경제</td>
      <td>'여행에 미치다' 조준기, 유서 암시글 작성'여행에 미치다' "부조는 가족들에게" ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>34</th>
      <td>생활/문화</td>
      <td>[속보] 방역당국 "거리두기 성과 일부 보여…3차 고비도 극복할수 있어"</td>
      <td>2020.09.01. 오후 2:25</td>
      <td>한국경제</td>
      <td>[속보] 방역당국 "거리두기 성과 일부 보여…3차 고비도 극복할수 있어"한경닷컴 뉴...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>35</th>
      <td>생활/문화</td>
      <td>"교회, 사회에 씻을 수 없는 죄"…개신교 단체 사죄성명</td>
      <td>2020.08.31. 오후 8:34</td>
      <td>JTBC</td>
      <td>동영상 뉴스"극우 개신교 방조한 교회 책임 인정해야"[앵커]교회발 감염이 확산되고 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>36</th>
      <td>생활/문화</td>
      <td>태풍 `마이삭` 전남 상륙해 한반도 관통하나…美中日 예측</td>
      <td>2020.09.01. 오후 1:47</td>
      <td>매일경제</td>
      <td>韓 기상청 경로 서쪽으로 이동했지만 경남 상륙 예측`매미`보다 `루사`와 경로 유사...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>37</th>
      <td>생활/문화</td>
      <td>한국 대중음악 역사 새로 쓴 BTS…빌보드 ‘핫100’ 1위</td>
      <td>2020.09.01. 오전 8:16</td>
      <td>한겨레</td>
      <td>‘다이너마이트’로 빌보드 1위에 오른 방탄소년단(BTS). 빅히트엔터테인먼트 제공 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>38</th>
      <td>생활/문화</td>
      <td>`BTS` 정국 얼굴로 `KTX` 388m 덮었다…한국철도, KTX 20량 전체 생...</td>
      <td>2020.09.01. 오후 2:57</td>
      <td>매일경제</td>
      <td>중국 팬클럽서 코레일에 광고 제안KTX 랩핑 광고는 2004년 이후 처음코레일, 격...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>39</th>
      <td>생활/문화</td>
      <td>[이슈 컷] 이거 진짜 초등생용 맞아?…파격적 성교육 교재 논란</td>
      <td>2020.09.01. 오전 7:00</td>
      <td>연합뉴스</td>
      <td>(서울=연합뉴스) 한 그림책에 벌거벗은 남녀의 모습이 등장합니다.    '옷을 벗은...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>40</th>
      <td>세계</td>
      <td>코로나 확진자 600만명 넘은 미국, 집단면역 추진…WP "200만명 숨질 것"</td>
      <td>2020.09.01. 오후 2:36</td>
      <td>JTBC</td>
      <td>미국 트럼프 행정부가 코로나 19 대응책으로 '집단면역'을 추진하고 있다는 보도가 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>41</th>
      <td>세계</td>
      <td>러시아 여성 몸에서 길이 1.2m 뱀 꺼내…수면 중 입으로 들어간 듯</td>
      <td>2020.09.01. 오후 1:59</td>
      <td>경향신문</td>
      <td>[경향신문] 인스타그램 캡처러시아 여성 몸에서 길이 1.2m 뱀이 나왔다.  수술을...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>42</th>
      <td>세계</td>
      <td>자고 일어났더니 몸 속에 1.2m길이 뱀이?</td>
      <td>2020.08.31. 오후 9:24</td>
      <td>조선일보</td>
      <td>/인스타그램러시아의 한 여성의 몸에서 4피트(약 1.2m)가 넘는 뱀이 나왔다.31...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>43</th>
      <td>세계</td>
      <td>중국계 호주인 CCTV 유명 女앵커, 중국에 보름간 구금돼 발칵</td>
      <td>2020.09.01. 오전 1:55</td>
      <td>중앙일보</td>
      <td>중국 관영 CCTV 영어방송 채널 CGTN 앵커 청 레이(49). 사진 트위터 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>44</th>
      <td>세계</td>
      <td>찬물에 담그니 망사속옷이…바비 뛰어넘은 대두인형 성상품화 논란</td>
      <td>2020.09.01. 오후 2:36</td>
      <td>서울신문</td>
      <td>[서울신문 나우뉴스]기형적으로 큰 머리와 짧고 몽땅한 몸으로 늘씬한 바비인형에 도전...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>45</th>
      <td>세계</td>
      <td>어차피 대세는 스가?...일사불란하게 움직이는 日 자민당, ’담합총리’ 논란</td>
      <td>2020.09.01. 오후 1:48</td>
      <td>중앙일보</td>
      <td>당내 유력 파벌 지지 선언..의원표 60% 확보 '스가 독주'에 기시다, 이시바는 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>46</th>
      <td>세계</td>
      <td>日야쿠자도 고령화…곤궁한 생활 입에 풀칠도 힘들어</td>
      <td>2020.09.01. 오후 3:11</td>
      <td>서울신문</td>
      <td>10명 중 1명은 70대…조직 나와도 취업 ‘바늘구멍’일본 야쿠자들의 세계를 그린 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>47</th>
      <td>세계</td>
      <td>러시아 여성 몸서 나온 1.2m 뱀…의사도 놀라 뒷걸음질쳤다</td>
      <td>2020.09.01. 오전 6:29</td>
      <td>중앙일보</td>
      <td>러시아에서 한 여성의 입에서 1.2미터 길이의 뱀이 나왔다. 의사가 내시경을 통...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>48</th>
      <td>세계</td>
      <td>18명 성폭행 혐의에…포르노 스타 "여자들이 내게 몸을 던진 것"</td>
      <td>2020.09.01. 오후 1:47</td>
      <td>머니투데이</td>
      <td>[머니투데이 박수현 기자]  [로스앤젤레스=AP/뉴시스]미국의 포르노 스타 론 제러...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>49</th>
      <td>세계</td>
      <td>[World Now] 일본 차기 총리는? 팬케이크 좋아하는 '흙수저' 스가 급부상</td>
      <td>2020.09.01. 오후 12:08</td>
      <td>MBC</td>
      <td>최근 사의를 표명한 아베 신조 일본 총리의 후임으로 스가 요시히데 관방장관(71)이...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>50</th>
      <td>IT/과학</td>
      <td>“미안하다, 내 갈길 떠난다” ‘여행에 미치다’ 조준기, 응급처치 후 병원 이송[전문]</td>
      <td>2020.09.01. 오전 11:49</td>
      <td>조선일보</td>
      <td>1일 오전 서울 용산서 위중 상태로 발견 인스타에 극단 선택 암시글 올려…“불필요한...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>51</th>
      <td>IT/과학</td>
      <td>“하루 급여 47만원 드려요!”…그래도 품귀 ‘배달 라이더의 실상’ [IT선빵!]</td>
      <td>2020.09.01. 오전 9:49</td>
      <td>헤럴드경제</td>
      <td>라이더 연 수익 1억시대 본격화 고액수익에도 라이더 품귀현상사회적 인식, 교통사고 ...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>52</th>
      <td>IT/과학</td>
      <td>야생서 멸종한 줄 알았는데…뉴기니 ‘노래하는 개’ 살아있었다</td>
      <td>2020.09.01. 오후 3:11</td>
      <td>서울신문</td>
      <td>[서울신문 나우뉴스]야생서 멸종한 줄 알았는데…뉴기니 ‘노래하는 개’ 살아있었다독특...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>53</th>
      <td>IT/과학</td>
      <td>[핵잼 사이언스] “드래곤 닮아” 아프리카서 신종 독사 발견…학명에 메탈리카 이름</td>
      <td>2020.08.31. 오후 4:21</td>
      <td>서울신문</td>
      <td>[서울신문 나우뉴스]“드래곤 닮아” 아프리카서 신종 독사 발견…학명에 메탈리카 이름...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>54</th>
      <td>IT/과학</td>
      <td>100GB 데이터 월 2만원…알뜰폰 활성화 팔 걷어붙인 정부</td>
      <td>2020.09.01. 오후 3:17</td>
      <td>한국경제</td>
      <td>月 6GB 요금제, 최고 5만원 이상 저렴100GB의 경우 월 2만원대 VS 6만원...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>55</th>
      <td>IT/과학</td>
      <td>카카오게임즈, 청약 열기도 '후끈'…주주 '잭팟' 예고</td>
      <td>2020.09.01. 오후 1:46</td>
      <td>아이뉴스24</td>
      <td>온라인 청약 일시 중단도…남궁훈 대표 평가액 580억원대[아이뉴스24 문영수 기자]...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>56</th>
      <td>IT/과학</td>
      <td>이경규·이효리 나오는 카카오 예능, 오늘부터 카톡에서 본다</td>
      <td>2020.09.01. 오후 3:00</td>
      <td>한겨레</td>
      <td>카카오엠(M) 제공이경규, 이효리 등 유명 연예인이 출연하는 카카오의 오리지널 콘텐...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>57</th>
      <td>IT/과학</td>
      <td>'가로본능폰' LG윙, 美 가격 1000달러?…외신서 영상 또 유출</td>
      <td>2020.09.01. 오전 6:15</td>
      <td>뉴스1</td>
      <td>美 1000달러면 국내 출고가도 120만~150만원 수준 예상LG전자 "새로운 폼팩...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>58</th>
      <td>IT/과학</td>
      <td>'갤럭시Z폴드2' 오늘밤 펼친다…미리보는 스펙은?</td>
      <td>2020.09.01. 오후 12:56</td>
      <td>한국경제TV</td>
      <td>[한국경제TV 이지효 기자]'갤럭시 Z 폴드2' 미스틱 블랙.삼성전자의 세 번째 폴...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
    <tr>
      <th>59</th>
      <td>IT/과학</td>
      <td>[아하! 우주] 우리은하와 안드로메다는 이미 충돌 중?…거대 헤일로 확인</td>
      <td>2020.09.01. 오후 2:46</td>
      <td>서울신문</td>
      <td>[서울신문 나우뉴스]안드로메다 은하 헤일로가 눈에 보인다고 가정할 경우 모습. 사진...</td>
      <td>https://news.naver.com/main/ranking/read.nhn?m...</td>
    </tr>
  </tbody>
</table>
</div>



### 키워드로 뉴스 검색
- 이번에는 키워드로 뉴스를 검색해보자.  

## 셀레니움을 사용한 크롤링

- 인턴십 교육 과정 중 프로젝트를 진행하면서 작성한 코드이다.  
- 채용공고 사이트(원티드)를 통해 관심직무의 트렌드를 파악하는 프로젝트를 진행하였다.  

#### 1. 주제명
- 채용공고 사이트를 통해 관심직무의 트렌드를 한 눈에 파악해보자.  
- 대상 웹사이트: 원티드  
- 선정 이유   
  - 평소 자주 이용하는 사이트이고 직무나  카테고리가 상세히 나눠져있어 하나하나 살펴보기 번거로운 점을 해결해보고 싶었다.  
  - 원티드는 IT계열 공고가 많고, 대기업뿐만 아니라 타사이트 대비 소규모나 스타트업 기업들의 공고가 다수 올라온다.  
- 목표  
  - 직무에 따라 채용 중인 회사의 자격요건/우대사항/혜택 및 복지를 한 눈에 볼 수 있는 자료를 얻고 싶다!  
  - 기업 채용 공고를 더 자세히 보고 싶으면 크롤링 결과의 url을 클릭하면 된다!  

#### 2. 개요
- 직무별 채용 트렌드 분석: 자격요건/우대사항/혜택 및 복지  나눠서 트렌드를 분석해 볼 수 있다. 


#### 3. 기대효과
- 취준생의 검색 시간을 줄일 수 있다!  
- 회사 간 비교가 쉽고 전반적인 채용 트렌드를 볼 수 있다!  


```python
import warnings
from selenium import webdriver
import pandas as pd
import numpy as np
warnings.filterwarnings('ignore')
from bs4 import BeautifulSoup
import re
import requests
from tqdm import tqdm_notebook

import sys
import os
from glob import glob
from collections import Counter
from konlpy.tag import Kkma
from wordcloud import WordCloud
import time
```

- Selenium과 BeautifulSoup을 같이 쓸 수 있다면 좋겠지만, 원티드의 경우 페이지가 로딩이 된 후에 정보를 불러오는 방식이기 때문에 `soup.select`로 불러올 수 없었다.  
- 따라서 해당 페이지의 정보 역시 Selenium으로 불러와야 했고, 이 때문에 시간이 오래 걸릴 수 있다.  


```python
def scrape_detail_page(url):
    if url != 'https://www.wanted.co.kr/wd/undefined':
        driver = webdriver.Chrome(r'../chromedriver.exe')
        driver.get(url)
        h = driver.page_source
        s = BeautifulSoup(h, 'lxml')
    
        # 우대사항
        if len(s.select('section._1LnfhLPc7hiSZaaXxRv11H > p:nth-of-type(4) > span')[0].text) > 0:
            wd = s.select('section._1LnfhLPc7hiSZaaXxRv11H > p:nth-of-type(4) > span')[0].text
        else:
            wd = np.nan
        
        # 자격요건
        if len(s.select('section._1LnfhLPc7hiSZaaXxRv11H > p:nth-of-type(3) > span')[0].text) > 0:
            jk = s.select('section._1LnfhLPc7hiSZaaXxRv11H > p:nth-of-type(3) > span')[0].text
        else:
            jk = np.nan

        # 혜택 및 복지
        if len(s.select('section._1LnfhLPc7hiSZaaXxRv11H > p:nth-of-type(5) > span')[0].text) > 0:
            ht = s.select('section._1LnfhLPc7hiSZaaXxRv11H > p:nth-of-type(5) > span')[0].text
        else:
            ht = np.nan
        
        # 주소
        if len(s.select('section._3XP3DBqOgzsz7P6KrVpbGO > div:nth-of-type(2) > span.body')[0].text) > 0:
            address = s.select('section._3XP3DBqOgzsz7P6KrVpbGO > div:nth-of-type(2) > span.body')[0].text
        else:
            address = np.nan
        
        info = {
            '우대사항': wd.replace('•','\n'),
            '자격요건': jk.replace('•','\n'),
            '혜택 및 복지': ht.replace('•','\n'),
            '주소': address 
        };
        driver.close()
        return info
    else:
        driver.close()
```


```python
len(scrape_detail_page('https://www.wanted.co.kr/wd/41414')) # 잘 돌아가는지 확인
```




    4




```python
job = input('직무: (서버개발자, 웹개발자, 데이터엔지니어, 머신러닝엔지니어, 데이터사이언티스트, 데이터분석가)')

if job == '서버개발자':
    job = '518/872'
elif job == '웹개발자':
    job = '518/873'
elif job == '데이터엔지니어':
    job = '518/655'
elif job == '머신러닝엔지니어':
    job = '518/1634'
elif job == '데이터사이언티스트':
    job = '518/102'
elif job == '데이터분석가':
    job = '507/656'
else:
    print('잘못된 입력값입니다.')
    job = input('번호로 입력해주세요: (서버개발자: 518/872, 웹개발자: 518/873, 데이터엔지니어: 518/655, 머신러닝엔지니어: 518/1634, 데이터사이언티스트: 518/1024, 데이터분석가: 507/656)')
```

    직무: (서버개발자, 웹개발자, 데이터엔지니어, 머신러닝엔지니어, 데이터사이언티스트, 데이터분석가)머신러닝엔지니어
    


```python
driver = webdriver.Chrome(r'../chromedriver.exe')
html = 'https://www.wanted.co.kr/wdlist/'+job+'?country=kr&job_sort=job.latest_order&years=-1&locations=all'
driver.get(html)
```


```python
html = driver.page_source
soup = BeautifulSoup(html, 'lxml')
dict = {'회사명': [], '자격요건': [], '우대사항': [], '혜택 및 복지': [], '주소': [] ,'url': []}
for i in tqdm_notebook(range(len(soup.select('div.job-card-company-name')))):
    try:
        html = driver.page_source
        soup = BeautifulSoup(html, 'lxml')
        
        SCROLL_PAUSE_TIME = 2
        # Get scroll height
        last_height = driver.execute_script("return document.body.scrollHeight")
        
        while True:
            driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

            # Wait to load page
            time.sleep(SCROLL_PAUSE_TIME)
            driver.execute_script("window.scrollTo(0, document.body.scrollHeight-50);")
            time.sleep(SCROLL_PAUSE_TIME)

            new_height = driver.execute_script("return document.body.scrollHeight")

            if new_height == last_height:
                break

            last_height = new_height
            

        dict['회사명'].append(soup.select('div.job-card-company-name')[i].text)
        url = 'https://www.wanted.co.kr'+soup.select('div._3D4OeuZHyGXN7wwibRM5BJ > a')[i]['href']
        dict['url'].append(url)
        info = scrape_detail_page(url)
        dict['우대사항'].append(info['우대사항'])
        dict['자격요건'].append(info['자격요건'])
        dict['주소'].append(info['주소'])
        dict['혜택 및 복지'].append(info['혜택 및 복지'])


    except Exception as e:
        print('{}번째 오류: {}'.format(i, e))

driver.close()
```


    HBox(children=(FloatProgress(value=0.0, max=20.0), HTML(value='')))


    
    


```python
dt = pd.DataFrame({'회사명': dict['회사명'],
              '자격요건': dict['자격요건'],
              '우대사항': dict['우대사항'],
              '혜택 및 복지': dict['혜택 및 복지'],
              '주소': dict['주소'],
              'url': dict['url']})
dt
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
      <th>회사명</th>
      <th>자격요건</th>
      <th>우대사항</th>
      <th>혜택 및 복지</th>
      <th>주소</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>로아컨설팅</td>
      <td> 문제 해결에 필요한 데이터를 정의하고, 데이터를 분석할 능력을 갖추신 분 주어...</td>
      <td> 기본 수준 이상의 확률, 통계에 대한 지식 및 머신러닝에 대한 지식을 보유하고 ...</td>
      <td> 하루 7시간, 야근없이 집중하는 문화 정착(8시 ~ 13시 사이 협의하에 출근)...</td>
      <td>서울시 강남구 테헤란로 142(역삼동, 아크플레이스)</td>
      <td>https://www.wanted.co.kr/wd/36041</td>
    </tr>
    <tr>
      <th>1</th>
      <td>플랜아이</td>
      <td>\n 머신러닝/딥러닝 기반 프로젝트 경험 보유자\n Python을 활용한 연구 및 ...</td>
      <td>\n TensorFlow, Pytorch 프레임워크를 활용해 인공 신경망 설계/커스...</td>
      <td>&lt;너드팩토리는 이런 조직 문화가 있어요&gt;  \n 스타트업보다 더 스타트업스럽게 일하...</td>
      <td>대전광역시 유성구 문지로 282-10</td>
      <td>https://www.wanted.co.kr/wd/43507</td>
    </tr>
    <tr>
      <th>2</th>
      <td>셀렉트스타</td>
      <td>1. 머신러닝, 데이터과학, 크라우드소싱 관련 연구 논문 및 기술 등의 기본적 이해...</td>
      <td>1. 데이터 과학, 통계적 추론, 확률, 머신러닝 등 관련 업무 경험 및 관련 학과...</td>
      <td>## 선택적 근로제각각의 멤버가 자율과 책임 속에서 일하고 있는 우리 회사는, 개개...</td>
      <td>서울특별시 강남구 테헤란로 20길 20 삼정빌딩 11층</td>
      <td>https://www.wanted.co.kr/wd/33037</td>
    </tr>
    <tr>
      <th>3</th>
      <td>큐픽스</td>
      <td>\n C/C++, Python사용에 능숙 하신 분\n Boost, OpenCV, E...</td>
      <td>\n SLAM, Structure from Motion, Visual Odometr...</td>
      <td># 개발문화- 개발자로서 성장가능한 환경 (코칭 및 리뷰)- Angular, AWS...</td>
      <td>판교테크노밸리</td>
      <td>https://www.wanted.co.kr/wd/29701</td>
    </tr>
    <tr>
      <th>4</th>
      <td>플래티어</td>
      <td>\n 데이터 모델링 및 분석 경력 5년 이상\n 통계 모델링 또는 ML/DL 경험\...</td>
      <td>\n 데이터 관련 팀 리딩 경험\n 이커머스 관련 분야 업무 경험이 있으신 분\n ...</td>
      <td>\n 4대 보험\n 내일 채움 공제\n 장비 지원금 (노트북 / 디지털 장비)\n ...</td>
      <td>서울시 송파구 법원로 9길</td>
      <td>https://www.wanted.co.kr/wd/23802</td>
    </tr>
    <tr>
      <th>5</th>
      <td>화해(버드뷰)</td>
      <td>\n  데이터 웨어하우스 및 ETL 관련 경험 3년 이상\n  데이터베이스에 대한 ...</td>
      <td>\n AWS, GCP 등 클라우드 기반 데이터 플랫폼 구축 및 운영 경험\n Spa...</td>
      <td>\n 삼시세끼 지원(삼시세끼는 물론, 간식과 외부 음료 모두 지원!)\n 화장품 복...</td>
      <td>서울 서초구 서초대로 396 강남빌딩 19층</td>
      <td>https://www.wanted.co.kr/wd/35029</td>
    </tr>
    <tr>
      <th>6</th>
      <td>매드업</td>
      <td>\n Python을 통한 데이터 분석 능력을 갖추신 분\n 석박사 경력 포함 해당 ...</td>
      <td>\n Hadoop MapReduce, Hive, Spark 등 빅데이터 분석 플랫폼...</td>
      <td>&lt;복지제도&gt;\n 함께하는 성장을 위하여- 직무별 실무교육 및 스터디 지원- 외부교육...</td>
      <td>서울특별시 서초구 서초대로 74길 4 삼성생명 서초타워 20층</td>
      <td>https://www.wanted.co.kr/wd/26139</td>
    </tr>
    <tr>
      <th>7</th>
      <td>네오사피엔스(Neosapience)</td>
      <td>아래의 요건중 하나에 해당하면 됩니다.- 최신 딥러닝 논문을 코드로 구현하고 실험,...</td>
      <td>- 오픈소스 활동- 관련 분야 논문 개제 경험- 챗봇등 언어처리관련 개발 경험</td>
      <td>\n 국내 최고 음성/언어 인공지능 전문가들이 매일 매일 의기투합!\n 점심/저녁 ...</td>
      <td>서울시 서초구 매헌로 16(하이브랜드 빌딩), 12층 양재R&amp;D혁신허브 1208호</td>
      <td>https://www.wanted.co.kr/wd/20063</td>
    </tr>
    <tr>
      <th>8</th>
      <td>네오사피엔스(Neosapience)</td>
      <td>아래의 요건중 하나에 해당하면 됩니다.- 최신 딥러닝 논문을 코드로 구현하고 실험,...</td>
      <td>- 오픈소스 활동- 관련 분야 논문 개제 경험- 음성 신호처리관련 실무개발 경험</td>
      <td>\n 국내 최고 음성/언어 인공지능 전문가들이 매일 매일 의기투합!\n 점심/저녁 ...</td>
      <td>서울시 서초구 매헌로 16(하이브랜드 빌딩), 12층 양재R&amp;D혁신허브 1208호</td>
      <td>https://www.wanted.co.kr/wd/20066</td>
    </tr>
    <tr>
      <th>9</th>
      <td>코그넥스코리아</td>
      <td>\n 3년 이상의 C++ 개발 경력을 가진 사람\n Windows Applicati...</td>
      <td>\n Agile 방법론과 Scrum에 대한 경험\n Unit Test, TDD, 빌...</td>
      <td>▣ 우리 팀의 메리트\n 최신 딥러닝 기술을 사용하는 소프트웨어를 직접 개발\n...</td>
      <td>서울특별시 서초구 서초대로38길 12 마제스타시티 타워2 6층 (주)코그넥스코리아</td>
      <td>https://www.wanted.co.kr/wd/43411</td>
    </tr>
    <tr>
      <th>10</th>
      <td>이마고웍스</td>
      <td>\n 최신 기술에 대한 연구에 그치지 않고, 실제 사용 가능한 소프트웨어를 개발하고...</td>
      <td>\n 관련 분야에 대한 경력 2년(석사급)\n Git 을 활용한 코드 정리 및 협업...</td>
      <td>\n 자율 출퇴근제\n 점심 식사 제공\n 교육 지원 (도서, 온라인 강의, 학회 ...</td>
      <td>서울시 동대문구 회기로 117-3, 서울바이오허브 산업지원동 403호</td>
      <td>https://www.wanted.co.kr/wd/38503</td>
    </tr>
    <tr>
      <th>11</th>
      <td>이마고웍스</td>
      <td>[자격요건]\n 최신 기술에 대한 연구에 그치지 않고, 실제 사용 가능한 소프트웨어...</td>
      <td>[우대사항]\n 관련 분야에 대한 경력 2년(석사급)\n C++, Javascrip...</td>
      <td>\n 자율 출퇴근제\n 점심 식사 제공\n 자기 계발 지원 (도서, 온라인 강의, ...</td>
      <td>서울시 동대문구 회기로 117-3, 서울바이오허브 산업지원동 403호</td>
      <td>https://www.wanted.co.kr/wd/41986</td>
    </tr>
    <tr>
      <th>12</th>
      <td>뤼이드(Riiid)</td>
      <td>\n 관련 전공 석사 학위 (CS / EE) 혹은 이에 준하는 관련 업계 경력 또는...</td>
      <td>\n 관련 전공 박사 학위 (CS / EE)\n Top-tier ML confere...</td>
      <td>\n 최고복지인 '좋은 동료들'과 함께 일합니다.\n 식비 걱정 없는 점심, 저녁 ...</td>
      <td>서울특별시 강남구 테헤란로87길29</td>
      <td>https://www.wanted.co.kr/wd/3936</td>
    </tr>
    <tr>
      <th>13</th>
      <td>블루닷</td>
      <td>\n 컴퓨터비전/머신러닝/딥러닝 지식 보유 및 관련 프로젝트 수행 경험자\n C/C...</td>
      <td>\n 딥러닝 프레임워크 경량화를 위한 최적화 경험 및 기술 보유</td>
      <td>\n 개인 장비 - 고성능 컴퓨터 또는 맥 지원\n 식사, 간식 - 고급 커피 제공...</td>
      <td>서울시 강남구 역삼로 168 회성빌딩 5층</td>
      <td>https://www.wanted.co.kr/wd/40971</td>
    </tr>
    <tr>
      <th>14</th>
      <td>더기프팅컴퍼니</td>
      <td>#이런 분과 일하고 싶어요\n 데이터 분석 관련 직무 유 경험자 (관련 업무 3년 ...</td>
      <td>#이런 분은 저희가 더욱 주목합니다\n E-Commerce 서비스 경험이 있으신 분</td>
      <td>#기프팅 크루 베네핏 (Gifting Crew Benefits)\n 출퇴근 시간 및...</td>
      <td>서울특별시 성동구 상원12길 1, 유앤아이빌딩 3층</td>
      <td>https://www.wanted.co.kr/wd/43358</td>
    </tr>
    <tr>
      <th>15</th>
      <td>더기프팅컴퍼니</td>
      <td>#이런 분과 일하고 싶어요\n 백엔드 개발 경험이 있는 분\n RDBMS, NoSQ...</td>
      <td>#이런 분은 저희가 더욱 주목합니다\n E-Commerce 기반의 시스템 개발/운영...</td>
      <td>#기프팅 크루 베네핏 (Gifting Crew Benefits)\n 출퇴근 시간 및...</td>
      <td>서울특별시 성동구 상원12길 1, 유앤아이빌딩 3층</td>
      <td>https://www.wanted.co.kr/wd/43345</td>
    </tr>
    <tr>
      <th>16</th>
      <td>아이픽셀(iPIXEL)</td>
      <td>\n Primary languages/tools: Keras, python, ten...</td>
      <td>\n AI Computer Vision 경량화 및 On-Device AI 등 관련 ...</td>
      <td>\n iPIXEL은 능력과 책임을 중심으로 철저한 성과기반 정책을 지향하고 있습니다...</td>
      <td>강남구 테헤란로 151, 4층</td>
      <td>https://www.wanted.co.kr/wd/36715</td>
    </tr>
    <tr>
      <th>17</th>
      <td>플라스크</td>
      <td>\n Machine Learning / Computer Vision 연구∙개발 경력...</td>
      <td>\n Pose Estimation 관련 프로젝트 또는 연구를 수행한 경험이 있는 분...</td>
      <td>\n 주도적인 업무환경 및 성장 기회\n 주 5일 자율출퇴근 (주 40시간 근무)\...</td>
      <td>서울특별시 강남구 강남대로 382, 메리츠타워 16F</td>
      <td>https://www.wanted.co.kr/wd/42992</td>
    </tr>
    <tr>
      <th>18</th>
      <td>미디움</td>
      <td>\n 데이터 웨어하우스 및 ETL 관련 경험 3년 이상\n 데이터베이스에 대한 이해...</td>
      <td>\n 머신러닝 데이터/서비스 파이프라인 구축 경험이 있으신 분\n 머신러닝 모델을 ...</td>
      <td>◆ 미디움 회사소개미디움은 독자적 하드웨어 블록체인 기술로 상용 엔터프라이즈 시장에...</td>
      <td>서울시 강남구 학동로 211 2층, 4층</td>
      <td>https://www.wanted.co.kr/wd/43323</td>
    </tr>
    <tr>
      <th>19</th>
      <td>미디움</td>
      <td>\n 전산 관련학과 학사 이상 또는 동일한 자격이 있으신 분\n Computer v...</td>
      <td>\n Image processing, computer vision혹은 유관 분야에서...</td>
      <td>◆ 미디움 회사소개미디움은 독자적 하드웨어 블록체인 기술로 상용 엔터프라이즈 시장에...</td>
      <td>서울시 강남구 학동로 211 2층, 4층</td>
      <td>https://www.wanted.co.kr/wd/43322</td>
    </tr>
  </tbody>
</table>
</div>



### 키워드 추출
- 키워드를 추출하여 동향을 파악하고, 이를 통해 워드클라우드로 한 눈에 나타내자.  


```python
punc = '[!"#$%&\'()*+,-./:;<=>?[\]^_`{|}~“”·•◆▣]'
for i in dt.index:
    dt['자격요건'][i] = re.sub(punc, '', str(dt['자격요건'][i]))
    dt['자격요건'][i] = re.sub('\n', '', str(dt['자격요건'][i]))
    dt['우대사항'][i] = re.sub(punc, '', str(dt['우대사항'][i]))
    dt['우대사항'][i] = re.sub('\n+', '', str(dt['우대사항'][i]))
    dt['우대사항'][i] = re.sub('\n', '', str(dt['우대사항'][i]))
    dt['우대사항'][i] = str(dt['우대사항'][i]).lstrip()
    dt['혜택 및 복지'][i] = re.sub(punc, '', str(dt['혜택 및 복지'][i]))
    dt['혜택 및 복지'][i] = re.sub('\n', '', str(dt['혜택 및 복지'][i]))
    
dt
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
      <th>회사명</th>
      <th>자격요건</th>
      <th>우대사항</th>
      <th>혜택 및 복지</th>
      <th>주소</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>로아컨설팅</td>
      <td>문제 해결에 필요한 데이터를 정의하고 데이터를 분석할 능력을 갖추신 분 주어진 문...</td>
      <td>기본 수준 이상의 확률 통계에 대한 지식 및 머신러닝에 대한 지식을 보유하고 있으며...</td>
      <td>하루 7시간 야근없이 집중하는 문화 정착8시  13시 사이 협의하에 출근 편리한 ...</td>
      <td>서울시 강남구 테헤란로 142(역삼동, 아크플레이스)</td>
      <td>https://www.wanted.co.kr/wd/36041</td>
    </tr>
    <tr>
      <th>1</th>
      <td>플랜아이</td>
      <td>머신러닝딥러닝 기반 프로젝트 경험 보유자 Python을 활용한 연구 및 개발이 가...</td>
      <td>TensorFlow Pytorch 프레임워크를 활용해 인공 신경망 설계커스터마이징 ...</td>
      <td>너드팩토리는 이런 조직 문화가 있어요   스타트업보다 더 스타트업스럽게 일하고 싶어...</td>
      <td>대전광역시 유성구 문지로 282-10</td>
      <td>https://www.wanted.co.kr/wd/43507</td>
    </tr>
    <tr>
      <th>2</th>
      <td>셀렉트스타</td>
      <td>1 머신러닝 데이터과학 크라우드소싱 관련 연구 논문 및 기술 등의 기본적 이해 능력...</td>
      <td>1 데이터 과학 통계적 추론 확률 머신러닝 등 관련 업무 경험 및 관련 학과 석박사 우대</td>
      <td>선택적 근로제각각의 멤버가 자율과 책임 속에서 일하고 있는 우리 회사는 개개인이 ...</td>
      <td>서울특별시 강남구 테헤란로 20길 20 삼정빌딩 11층</td>
      <td>https://www.wanted.co.kr/wd/33037</td>
    </tr>
    <tr>
      <th>3</th>
      <td>큐픽스</td>
      <td>CC Python사용에 능숙 하신 분 Boost OpenCV Eigen Lapac...</td>
      <td>SLAM Structure from Motion Visual Odometry 개발 ...</td>
      <td>개발문화 개발자로서 성장가능한 환경 코칭 및 리뷰 Angular AWS 딥러닝 등...</td>
      <td>판교테크노밸리</td>
      <td>https://www.wanted.co.kr/wd/29701</td>
    </tr>
    <tr>
      <th>4</th>
      <td>플래티어</td>
      <td>데이터 모델링 및 분석 경력 5년 이상 통계 모델링 또는 MLDL 경험 고객 행동...</td>
      <td>데이터 관련 팀 리딩 경험 이커머스 관련 분야 업무 경험이 있으신 분 마케팅 솔루션...</td>
      <td>4대 보험 내일 채움 공제 장비 지원금 노트북  디지털 장비 동호회 활동 지원금 ...</td>
      <td>서울시 송파구 법원로 9길</td>
      <td>https://www.wanted.co.kr/wd/23802</td>
    </tr>
    <tr>
      <th>5</th>
      <td>화해(버드뷰)</td>
      <td>데이터 웨어하우스 및 ETL 관련 경험 3년 이상  데이터베이스에 대한 이해 및...</td>
      <td>AWS GCP 등 클라우드 기반 데이터 플랫폼 구축 및 운영 경험 Spark 기반 ...</td>
      <td>삼시세끼 지원삼시세끼는 물론 간식과 외부 음료 모두 지원 화장품 복지제도매달 화장...</td>
      <td>서울 서초구 서초대로 396 강남빌딩 19층</td>
      <td>https://www.wanted.co.kr/wd/35029</td>
    </tr>
    <tr>
      <th>6</th>
      <td>매드업</td>
      <td>Python을 통한 데이터 분석 능력을 갖추신 분 석박사 경력 포함 해당 업무 5...</td>
      <td>Hadoop MapReduce Hive Spark 등 빅데이터 분석 플랫폼 경험 대...</td>
      <td>복지제도 함께하는 성장을 위하여 직무별 실무교육 및 스터디 지원 외부교육 및 컨퍼런...</td>
      <td>서울특별시 서초구 서초대로 74길 4 삼성생명 서초타워 20층</td>
      <td>https://www.wanted.co.kr/wd/26139</td>
    </tr>
    <tr>
      <th>7</th>
      <td>네오사피엔스(Neosapience)</td>
      <td>아래의 요건중 하나에 해당하면 됩니다 최신 딥러닝 논문을 코드로 구현하고 실험 분석...</td>
      <td>오픈소스 활동 관련 분야 논문 개제 경험 챗봇등 언어처리관련 개발 경험</td>
      <td>국내 최고 음성언어 인공지능 전문가들이 매일 매일 의기투합 점심저녁 식사 간식 커...</td>
      <td>서울시 서초구 매헌로 16(하이브랜드 빌딩), 12층 양재R&amp;D혁신허브 1208호</td>
      <td>https://www.wanted.co.kr/wd/20063</td>
    </tr>
    <tr>
      <th>8</th>
      <td>네오사피엔스(Neosapience)</td>
      <td>아래의 요건중 하나에 해당하면 됩니다 최신 딥러닝 논문을 코드로 구현하고 실험 분석...</td>
      <td>오픈소스 활동 관련 분야 논문 개제 경험 음성 신호처리관련 실무개발 경험</td>
      <td>국내 최고 음성언어 인공지능 전문가들이 매일 매일 의기투합 점심저녁 식사 간식 커...</td>
      <td>서울시 서초구 매헌로 16(하이브랜드 빌딩), 12층 양재R&amp;D혁신허브 1208호</td>
      <td>https://www.wanted.co.kr/wd/20066</td>
    </tr>
    <tr>
      <th>9</th>
      <td>코그넥스코리아</td>
      <td>3년 이상의 C 개발 경력을 가진 사람 Windows Application 개발 ...</td>
      <td>Agile 방법론과 Scrum에 대한 경험 Unit Test TDD 빌드 자동화에 ...</td>
      <td>우리 팀의 메리트 최신 딥러닝 기술을 사용하는 소프트웨어를 직접 개발 Agile기...</td>
      <td>서울특별시 서초구 서초대로38길 12 마제스타시티 타워2 6층 (주)코그넥스코리아</td>
      <td>https://www.wanted.co.kr/wd/43411</td>
    </tr>
    <tr>
      <th>10</th>
      <td>이마고웍스</td>
      <td>최신 기술에 대한 연구에 그치지 않고 실제 사용 가능한 소프트웨어를 개발하고 싶으...</td>
      <td>관련 분야에 대한 경력 2년석사급 Git 을 활용한 코드 정리 및 협업에 익숙하신 ...</td>
      <td>자율 출퇴근제 점심 식사 제공 교육 지원 도서 온라인 강의 학회 등 경력에 따라 ...</td>
      <td>서울시 동대문구 회기로 117-3, 서울바이오허브 산업지원동 403호</td>
      <td>https://www.wanted.co.kr/wd/38503</td>
    </tr>
    <tr>
      <th>11</th>
      <td>이마고웍스</td>
      <td>자격요건 최신 기술에 대한 연구에 그치지 않고 실제 사용 가능한 소프트웨어를 개발하...</td>
      <td>우대사항 관련 분야에 대한 경력 2년석사급 C Javascript 등 다양한 언어 ...</td>
      <td>자율 출퇴근제 점심 식사 제공 자기 계발 지원 도서 온라인 강의 학회 등 경력에 ...</td>
      <td>서울시 동대문구 회기로 117-3, 서울바이오허브 산업지원동 403호</td>
      <td>https://www.wanted.co.kr/wd/41986</td>
    </tr>
    <tr>
      <th>12</th>
      <td>뤼이드(Riiid)</td>
      <td>관련 전공 석사 학위 CS  EE 혹은 이에 준하는 관련 업계 경력 또는 논문 실...</td>
      <td>관련 전공 박사 학위 CS  EE Toptier ML conference 논문 실적...</td>
      <td>최고복지인 좋은 동료들과 함께 일합니다 식비 걱정 없는 점심 저녁 식사 시간저녁 ...</td>
      <td>서울특별시 강남구 테헤란로87길29</td>
      <td>https://www.wanted.co.kr/wd/3936</td>
    </tr>
    <tr>
      <th>13</th>
      <td>블루닷</td>
      <td>컴퓨터비전머신러닝딥러닝 지식 보유 및 관련 프로젝트 수행 경험자 CC 능숙 딥러닝...</td>
      <td>딥러닝 프레임워크 경량화를 위한 최적화 경험 및 기술 보유</td>
      <td>개인 장비  고성능 컴퓨터 또는 맥 지원 식사 간식  고급 커피 제공 야근시 석식...</td>
      <td>서울시 강남구 역삼로 168 회성빌딩 5층</td>
      <td>https://www.wanted.co.kr/wd/40971</td>
    </tr>
    <tr>
      <th>14</th>
      <td>더기프팅컴퍼니</td>
      <td>이런 분과 일하고 싶어요 데이터 분석 관련 직무 유 경험자 관련 업무 3년 이상 통...</td>
      <td>이런 분은 저희가 더욱 주목합니다 ECommerce 서비스 경험이 있으신 분</td>
      <td>기프팅 크루 베네핏 Gifting Crew Benefits 출퇴근 시간 및 근무시간...</td>
      <td>서울특별시 성동구 상원12길 1, 유앤아이빌딩 3층</td>
      <td>https://www.wanted.co.kr/wd/43358</td>
    </tr>
    <tr>
      <th>15</th>
      <td>더기프팅컴퍼니</td>
      <td>이런 분과 일하고 싶어요 백엔드 개발 경험이 있는 분 RDBMS NoSQL 기반 데...</td>
      <td>이런 분은 저희가 더욱 주목합니다 ECommerce 기반의 시스템 개발운영 경험이 ...</td>
      <td>기프팅 크루 베네핏 Gifting Crew Benefits 출퇴근 시간 및 근무시간...</td>
      <td>서울특별시 성동구 상원12길 1, 유앤아이빌딩 3층</td>
      <td>https://www.wanted.co.kr/wd/43345</td>
    </tr>
    <tr>
      <th>16</th>
      <td>아이픽셀(iPIXEL)</td>
      <td>Primary languagestools Keras python tensorflo...</td>
      <td>AI Computer Vision 경량화 및 OnDevice AI 등 관련 경험 및...</td>
      <td>iPIXEL은 능력과 책임을 중심으로 철저한 성과기반 정책을 지향하고 있습니다  ...</td>
      <td>강남구 테헤란로 151, 4층</td>
      <td>https://www.wanted.co.kr/wd/36715</td>
    </tr>
    <tr>
      <th>17</th>
      <td>플라스크</td>
      <td>Machine Learning  Computer Vision 연구∙개발 경력 2년...</td>
      <td>Pose Estimation 관련 프로젝트 또는 연구를 수행한 경험이 있는 분 Te...</td>
      <td>주도적인 업무환경 및 성장 기회 주 5일 자율출퇴근 주 40시간 근무 월 36회 ...</td>
      <td>서울특별시 강남구 강남대로 382, 메리츠타워 16F</td>
      <td>https://www.wanted.co.kr/wd/42992</td>
    </tr>
    <tr>
      <th>18</th>
      <td>미디움</td>
      <td>데이터 웨어하우스 및 ETL 관련 경험 3년 이상 데이터베이스에 대한 이해 및 S...</td>
      <td>머신러닝 데이터서비스 파이프라인 구축 경험이 있으신 분 머신러닝 모델을 기반으로한 ...</td>
      <td>미디움 회사소개미디움은 독자적 하드웨어 블록체인 기술로 상용 엔터프라이즈 시장에 ...</td>
      <td>서울시 강남구 학동로 211 2층, 4층</td>
      <td>https://www.wanted.co.kr/wd/43323</td>
    </tr>
    <tr>
      <th>19</th>
      <td>미디움</td>
      <td>전산 관련학과 학사 이상 또는 동일한 자격이 있으신 분 Computer visio...</td>
      <td>Image processing computer vision혹은 유관 분야에서 2년 ...</td>
      <td>미디움 회사소개미디움은 독자적 하드웨어 블록체인 기술로 상용 엔터프라이즈 시장에 ...</td>
      <td>서울시 강남구 학동로 211 2층, 4층</td>
      <td>https://www.wanted.co.kr/wd/43322</td>
    </tr>
  </tbody>
</table>
</div>




```python
output_path = time.strftime('%y%m%d') +'_wanted.csv'
path = os.path.join(os.getcwd(),'output\\')
path+output_path
```




    'C:\\Users\\강병규PC\\internship_compa\\study\\output\\200902_wanted.csv'




```python
dt.to_csv(path+output_path, index = False)
```

- 크롤링과 전처리를 끝냈으므로 이제 키워드를 추출해보자.  


```python
# dt = pd.read_csv(r'G:\내 드라이브\과학기술일자리진흥원(인턴)\study\output\200902_wanted.csv')
```


```python
dt.isnull().sum()
```




    회사명        0
    자격요건       0
    우대사항       0
    혜택 및 복지    0
    주소         0
    url        0
    dtype: int64




```python
import matplotlib
import matplotlib.pyplot as plt

font_location = r"c:/Windows/fonts/malgun.ttf"
font_name = font_manager.FontProperties(fname=font_location).get_name()
matplotlib.rc('font', family=font_name)
matplotlib.rc('font', size=10)    
```


    <Figure size 1440x72 with 0 Axes>



```python
from konlpy.tag import Komoran
from collections import Counter
```


```python
def main(content, stopwords, title):
    plt.figure(figsize=(20, 4))
    
    komoran = Komoran()

    frequency = Counter()
    count_proccessed = 0

    for i in content.index:
        tokens = get_tokens(komoran, content[i], stopwords)
        frequency.update(tokens)
        count_proccessed += 1
    
    wordinfo = {}
    for token, count in frequency.most_common(30):
        print(token, count)
        wordinfo[token] = count
    
    plt.bar(range(len(wordinfo)), sorted(wordinfo.values(), reverse = True), align = 'center')
    plt.xticks(range(len(wordinfo)), list(sorted(wordinfo, key=wordinfo.get, reverse = True)), rotation = '70')
    plt.title(title)

def get_tokens(komoran, content, stopwords):
    tokens = []
    node = komoran.pos(content)
    for (taeso, pumsa) in node:
        # 고유 명사와 일반 명사만 추출
        if pumsa in ('NNG', 'NNP'):
            tokens.append(taeso)
    unique = set(tokens)
    for w in unique:
        if w in stopwords:
            while w in tokens: tokens.remove(w)
    return tokens
```


```python
stopwords = ['데이터', '분', '분석', '이상', '가능', '기반', '관련', '역량', '분야', '문제', '보유', '제공', '업무', '지급', '근무', '시', '회사', '복지', '우대', '지원']
```


```python
# main(dt['자격요건'], stopwords, '자격요건')
# main(dt['우대사항'], stopwords, '우대사항')
main(dt['혜택 및 복지'], stopwords, '혜택 및 복지')
```

    자율 20
    휴가 20
    블록체인 16
    출퇴근 14
    시간 13
    일 12
    간식 11
    도서 11
    기술 11
    장비 11
    자유 11
    사용 11
    운영 10
    개발 10
    커피 9
    교육 9
    팀 9
    제한 9
    환경 8
    제도 8
    시장 8
    세미나 7
    비용 7
    필요 7
    성장 7
    식사 7
    컨퍼런스 6
    재택 6
    신청 6
    엔터프라이즈 6
    


![png](output_45_1.png)



```python
from wordcloud import WordCloud
```


```python
def make_wordcloud(content):
    komoran = Komoran()
    tokens = []
    
    for i in dt.index:
        node = komoran.pos(content[i])
        for taeso, pumsa in node:
            if pumsa in ('NNG', 'NNP'):
                tokens.append(taeso)
    
    unique = set(tokens)
    for w in unique:
        if w in stopwords:
            while w in tokens: tokens.remove(w)
   # print(' '.join(tokens))
                
    wordcloud = WordCloud(font_path='c:/Windows/fonts/malgun.ttf',
                         background_color = 'white',
                         width=800, height=400).generate(' '.join(tokens))
    plt.figure(figsize = (40,20))
    plt.imshow(wordcloud)
    plt.axis('off')
#    plt.show()
```


```python
make_wordcloud(dt['자격요건'])
```


![png](output_48_0.png)



```python
make_wordcloud(dt['우대사항'])
```


![png](output_49_0.png)



```python
make_wordcloud(dt['혜택 및 복지'])
```


![png](output_50_0.png)


### (번외) 지도 상에 시각화


```python
import folium
import googlemaps
```


```python
gmap_key = "AIzaSyCiTzLA4PmX0hQ1cktTTNNWHvW_ryZeH7I"
gmaps = googlemaps.Client(key=gmap_key)
```


```python
for row in tqdm_notebook(dt.index):
    try:
        geo = gmaps.geocode(str(dt.loc[row, '주소']))
        dt.loc[row, 'lat'] = geo[0].get('geometry')['location']['lat']
        dt.loc[row, 'lng'] = geo[0].get('geometry')['location']['lng']
    except:
        dt.loc[row, 'lat'] = np.nan
        dt.loc[row, 'lng'] = np.nan
dt.head()
```


    HBox(children=(IntProgress(value=0, max=20), HTML(value='')))


    
    




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
      <th>회사명</th>
      <th>자격요건</th>
      <th>우대사항</th>
      <th>혜택 및 복지</th>
      <th>주소</th>
      <th>url</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>로아컨설팅</td>
      <td>문제 해결에 필요한 데이터를 정의하고 데이터를 분석할 능력을 갖추신 분 주어진 문...</td>
      <td>기본 수준 이상의 확률 통계에 대한 지식 및 머신러닝에 대한 지식을 보유하고 있으며...</td>
      <td>하루 7시간 야근없이 집중하는 문화 정착8시  13시 사이 협의하에 출근 편리한 ...</td>
      <td>서울시 강남구 테헤란로 142(역삼동, 아크플레이스)</td>
      <td>https://www.wanted.co.kr/wd/36041</td>
      <td>37.499794</td>
      <td>127.035021</td>
    </tr>
    <tr>
      <th>1</th>
      <td>플랜아이</td>
      <td>머신러닝딥러닝 기반 프로젝트 경험 보유자 Python을 활용한 연구 및 개발이 가...</td>
      <td>TensorFlow Pytorch 프레임워크를 활용해 인공 신경망 설계커스터마이징 ...</td>
      <td>너드팩토리는 이런 조직 문화가 있어요   스타트업보다 더 스타트업스럽게 일하고 싶어...</td>
      <td>대전광역시 유성구 문지로 282-10</td>
      <td>https://www.wanted.co.kr/wd/43507</td>
      <td>36.391740</td>
      <td>127.407118</td>
    </tr>
    <tr>
      <th>2</th>
      <td>셀렉트스타</td>
      <td>1 머신러닝 데이터과학 크라우드소싱 관련 연구 논문 및 기술 등의 기본적 이해 능력...</td>
      <td>1 데이터 과학 통계적 추론 확률 머신러닝 등 관련 업무 경험 및 관련 학과 석박사 우대</td>
      <td>선택적 근로제각각의 멤버가 자율과 책임 속에서 일하고 있는 우리 회사는 개개인이 ...</td>
      <td>서울특별시 강남구 테헤란로 20길 20 삼정빌딩 11층</td>
      <td>https://www.wanted.co.kr/wd/33037</td>
      <td>37.505377</td>
      <td>127.034355</td>
    </tr>
    <tr>
      <th>3</th>
      <td>큐픽스</td>
      <td>CC Python사용에 능숙 하신 분 Boost OpenCV Eigen Lapac...</td>
      <td>SLAM Structure from Motion Visual Odometry 개발 ...</td>
      <td>개발문화 개발자로서 성장가능한 환경 코칭 및 리뷰 Angular AWS 딥러닝 등...</td>
      <td>판교테크노밸리</td>
      <td>https://www.wanted.co.kr/wd/29701</td>
      <td>37.400246</td>
      <td>127.104784</td>
    </tr>
    <tr>
      <th>4</th>
      <td>플래티어</td>
      <td>데이터 모델링 및 분석 경력 5년 이상 통계 모델링 또는 MLDL 경험 고객 행동...</td>
      <td>데이터 관련 팀 리딩 경험 이커머스 관련 분야 업무 경험이 있으신 분 마케팅 솔루션...</td>
      <td>4대 보험 내일 채움 공제 장비 지원금 노트북  디지털 장비 동호회 활동 지원금 ...</td>
      <td>서울시 송파구 법원로 9길</td>
      <td>https://www.wanted.co.kr/wd/23802</td>
      <td>37.484287</td>
      <td>127.116988</td>
    </tr>
  </tbody>
</table>
</div>




```python
dt[dt.lat.isnull()]
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
      <th>회사명</th>
      <th>자격요건</th>
      <th>우대사항</th>
      <th>혜택 및 복지</th>
      <th>주소</th>
      <th>url</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
output = time.strftime('%y%m%d') +'_wanted_location.csv'
path = os.path.join(os.getcwd(),'output')
dt.to_csv(path+output, index = False)
```


```python
dt_m = dt.drop_duplicates('주소').reset_index(drop=True, )


map = folium.Map([37.518486, 127.024383], zoom_start=11)
#                 tiles='Stamen WaterColor')

from folium.plugins import MarkerCluster
marker_cluster = MarkerCluster().add_to(map)

for row in dt_m.index:
    lat = dt_m.lat[row]
    lng = dt_m.lng[row]
    folium.CircleMarker(
        location = [lat, lng],
        color='', fill=True, fill_color='#044275',
        radius=15,
        popup=(dt['회사명'][row])
    ).add_to(marker_cluster)


map.save('wanted_data_scientist.html')
map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF84ODE0ZjM2ZTFmMGU0MDliOGEwY2ZlZDM1ZjEyYTQwMyB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvbGVhZmxldC5tYXJrZXJjbHVzdGVyLzEuMS4wL2xlYWZsZXQubWFya2VyY2x1c3Rlci5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL2xlYWZsZXQubWFya2VyY2x1c3Rlci8xLjEuMC9NYXJrZXJDbHVzdGVyLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9sZWFmbGV0Lm1hcmtlcmNsdXN0ZXIvMS4xLjAvTWFya2VyQ2x1c3Rlci5EZWZhdWx0LmNzcyIvPgo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwXzg4MTRmMzZlMWYwZTQwOWI4YTBjZmVkMzVmMTJhNDAzIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKICAgICAgICAgICAgdmFyIG1hcF84ODE0ZjM2ZTFmMGU0MDliOGEwY2ZlZDM1ZjEyYTQwMyA9IEwubWFwKAogICAgICAgICAgICAgICAgIm1hcF84ODE0ZjM2ZTFmMGU0MDliOGEwY2ZlZDM1ZjEyYTQwMyIsCiAgICAgICAgICAgICAgICB7CiAgICAgICAgICAgICAgICAgICAgY2VudGVyOiBbMzcuNTE4NDg2LCAxMjcuMDI0MzgzXSwKICAgICAgICAgICAgICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3LAogICAgICAgICAgICAgICAgICAgIHpvb206IDExLAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfMDNmZWU0ZjcyZWEzNDBmMmI0ZmZmZWNhM2QzNzZiMzggPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIkRhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzg4MTRmMzZlMWYwZTQwOWI4YTBjZmVkMzVmMTJhNDAzKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFya2VyX2NsdXN0ZXJfMDc0OTUxZDJiYTA0NGJlYWE1YmM4ZDkxZjg2ZmY0ODggPSBMLm1hcmtlckNsdXN0ZXJHcm91cCgKICAgICAgICAgICAgICAgIHt9CiAgICAgICAgICAgICk7CiAgICAgICAgICAgIG1hcF84ODE0ZjM2ZTFmMGU0MDliOGEwY2ZlZDM1ZjEyYTQwMy5hZGRMYXllcihtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzJhOTQ4NmY1MjU0NDlmZjhkMWYwMGJmNDMwMzhhOWYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFszNy40OTk3OTQsIDEyNy4wMzUwMjE0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYWU3ZThjMWUzMTk2NGVkNjk2Y2YzNDI1NzNlMTFkYmIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzBhZmIwYTBjZmY5NzRkNWZiMzEwYmJmYmNkNTg4YzRhID0gJChgPGRpdiBpZD0iaHRtbF8wYWZiMGEwY2ZmOTc0ZDVmYjMxMGJiZmJjZDU4OGM0YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+66Gc7JWE7Luo7ISk7YyFPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2FlN2U4YzFlMzE5NjRlZDY5NmNmMzQyNTczZTExZGJiLnNldENvbnRlbnQoaHRtbF8wYWZiMGEwY2ZmOTc0ZDVmYjMxMGJiZmJjZDU4OGM0YSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMzJhOTQ4NmY1MjU0NDlmZjhkMWYwMGJmNDMwMzhhOWYuYmluZFBvcHVwKHBvcHVwX2FlN2U4YzFlMzE5NjRlZDY5NmNmMzQyNTczZTExZGJiKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yODE5ZmE2YTU5YjU0YTc2OGI2ODQ4ZDhmMzIzM2JiZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM2LjM5MTc0MDUsIDEyNy40MDcxMTg0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNWVmYjc4NmE1OTk3NDczNzljNjI5YmRkYjQwYjQzZWYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2I3NWRmNzk4MDgwODRlMDI5NzIxOTlmOWQ3YzIwZGNkID0gJChgPGRpdiBpZD0iaHRtbF9iNzVkZjc5ODA4MDg0ZTAyOTcyMTk5ZjlkN2MyMGRjZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7ZSM656c7JWE7J20PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzVlZmI3ODZhNTk5NzQ3Mzc5YzYyOWJkZGI0MGI0M2VmLnNldENvbnRlbnQoaHRtbF9iNzVkZjc5ODA4MDg0ZTAyOTcyMTk5ZjlkN2MyMGRjZCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMjgxOWZhNmE1OWI1NGE3NjhiNjg0OGQ4ZjMyMzNiYmUuYmluZFBvcHVwKHBvcHVwXzVlZmI3ODZhNTk5NzQ3Mzc5YzYyOWJkZGI0MGI0M2VmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81MjlmNTM1NGFiMzA0OGU0OGJkNGU1ZWMyOGRlMzY4MSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjUwNTM3NjYsIDEyNy4wMzQzNTQ2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYWI5MGJiN2Y5MjkxNGJiOGE4MWUzNmMxZTMwYmU4YjcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzZmMjlmMWNhNjE2ZjQyYzE4ZGI2MjA5NWVmZWQzNjA1ID0gJChgPGRpdiBpZD0iaHRtbF82ZjI5ZjFjYTYxNmY0MmMxOGRiNjIwOTVlZmVkMzYwNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7IWA66CJ7Yq47Iqk7YOAPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2FiOTBiYjdmOTI5MTRiYjhhODFlMzZjMWUzMGJlOGI3LnNldENvbnRlbnQoaHRtbF82ZjI5ZjFjYTYxNmY0MmMxOGRiNjIwOTVlZmVkMzYwNSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNTI5ZjUzNTRhYjMwNDhlNDhiZDRlNWVjMjhkZTM2ODEuYmluZFBvcHVwKHBvcHVwX2FiOTBiYjdmOTI5MTRiYjhhODFlMzZjMWUzMGJlOGI3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82MjUyZWVmZGY5ZWI0NTlmODIxYWU0YTQ5ZGI0ZTRlYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjQwMDI0NTYsIDEyNy4xMDQ3ODM3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfODE4OTY5NjUxNzZkNDUyNmJkNmNlZWM5MTA1MTliZGEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzAzODQ2MmYxZGI3YzQ4MmRhYTMwYmRhNzY2YTNmODFjID0gJChgPGRpdiBpZD0iaHRtbF8wMzg0NjJmMWRiN2M0ODJkYWEzMGJkYTc2NmEzZjgxYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7YGQ7ZS97IqkPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzgxODk2OTY1MTc2ZDQ1MjZiZDZjZWVjOTEwNTE5YmRhLnNldENvbnRlbnQoaHRtbF8wMzg0NjJmMWRiN2M0ODJkYWEzMGJkYTc2NmEzZjgxYyk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNjI1MmVlZmRmOWViNDU5ZjgyMWFlNGE0OWRiNGU0ZWEuYmluZFBvcHVwKHBvcHVwXzgxODk2OTY1MTc2ZDQ1MjZiZDZjZWVjOTEwNTE5YmRhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82OTU0MmQwYjYxZDk0M2ZmYjEyM2NhMWFhNDkxZjQzYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjQ4NDI4NjYsIDEyNy4xMTY5ODc4XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTczMGEwZjY3YWYyNGY3OWFjMmU0ZjRhOGM2YmU4ZjIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzljYzE5ODJjNGE5YzRhNmZhNDI4YzZlMDUyMDBiMzQ0ID0gJChgPGRpdiBpZD0iaHRtbF85Y2MxOTgyYzRhOWM0YTZmYTQyOGM2ZTA1MjAwYjM0NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7ZSM656Y7Yuw7Ja0PC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzU3MzBhMGY2N2FmMjRmNzlhYzJlNGY0YThjNmJlOGYyLnNldENvbnRlbnQoaHRtbF85Y2MxOTgyYzRhOWM0YTZmYTQyOGM2ZTA1MjAwYjM0NCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNjk1NDJkMGI2MWQ5NDNmZmIxMjNjYTFhYTQ5MWY0M2EuYmluZFBvcHVwKHBvcHVwXzU3MzBhMGY2N2FmMjRmNzlhYzJlNGY0YThjNmJlOGYyKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xZjEzYzYzYjA2NTc0MjA2YmIwZjVkYTZmMTNkYmVlNiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjQ5NjU0NDIsIDEyNy4wMjQ3NzI1XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNTg4MmRmNjMxMTI5NGY1YWE4NTU3MDQ1MTUzNGMzMTcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2ZlMWZhODA4ZTc2ODQ3ZmZiZjZhZGNhMjg1YWFkZGJkID0gJChgPGRpdiBpZD0iaHRtbF9mZTFmYTgwOGU3Njg0N2ZmYmY2YWRjYTI4NWFhZGRiZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7ZmU7ZW0KOuyhOuTnOu3sCk8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNTg4MmRmNjMxMTI5NGY1YWE4NTU3MDQ1MTUzNGMzMTcuc2V0Q29udGVudChodG1sX2ZlMWZhODA4ZTc2ODQ3ZmZiZjZhZGNhMjg1YWFkZGJkKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8xZjEzYzYzYjA2NTc0MjA2YmIwZjVkYTZmMTNkYmVlNi5iaW5kUG9wdXAocG9wdXBfNTg4MmRmNjMxMTI5NGY1YWE4NTU3MDQ1MTUzNGMzMTcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzgwNzRkNGY2MTUwZDRlNTlhNGQ4YWZjNmU3MzUzOTg3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbMzcuNDk2Nzk0MiwgMTI3LjAyNTcxNjhdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMDQ0Mjc1IiwgImZpbGxPcGFjaXR5IjogMC4yLCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiAxNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcmtlcl9jbHVzdGVyXzA3NDk1MWQyYmEwNDRiZWFhNWJjOGQ5MWY4NmZmNDg4KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85MjNkZmM4ZDcyNzI0N2RmYjc0MzMzMGVjNjAyZmJlNCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYTE0NThmYzJkYjYyNDE3MDlkNGJmNTFjYjZmZmFmYjQgPSAkKGA8ZGl2IGlkPSJodG1sX2ExNDU4ZmMyZGI2MjQxNzA5ZDRiZjUxY2I2ZmZhZmI0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij7rp6Trk5zsl4U8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTIzZGZjOGQ3MjcyNDdkZmI3NDMzMzBlYzYwMmZiZTQuc2V0Q29udGVudChodG1sX2ExNDU4ZmMyZGI2MjQxNzA5ZDRiZjUxY2I2ZmZhZmI0KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl84MDc0ZDRmNjE1MGQ0ZTU5YTRkOGFmYzZlNzM1Mzk4Ny5iaW5kUG9wdXAocG9wdXBfOTIzZGZjOGQ3MjcyNDdkZmI3NDMzMzBlYzYwMmZiZTQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNhMjA5MjlhYmY0MDRkMWNhYTFmMjZiYzZjNTQxMmQ1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbMzcuNDYyNDk0LCAxMjcuMDM2ODgzMV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMwNDQyNzUiLCAiZmlsbE9wYWNpdHkiOiAwLjIsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDE1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFya2VyX2NsdXN0ZXJfMDc0OTUxZDJiYTA0NGJlYWE1YmM4ZDkxZjg2ZmY0ODgpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2ZiZGNhNzU1ZmQ1NDQ1Zjk4MTIyMThjZjdmYWM1MGQ3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF81ZDc2MGUyNDVjNzk0NTIwOWZkNThiNzYyYzUxZmFjOSA9ICQoYDxkaXYgaWQ9Imh0bWxfNWQ3NjBlMjQ1Yzc5NDUyMDlmZDU4Yjc2MmM1MWZhYzkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPuuEpOyYpOyCrO2UvOyXlOyKpChOZW9zYXBpZW5jZSk8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZmJkY2E3NTVmZDU0NDVmOTgxMjIxOGNmN2ZhYzUwZDcuc2V0Q29udGVudChodG1sXzVkNzYwZTI0NWM3OTQ1MjA5ZmQ1OGI3NjJjNTFmYWM5KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8zYTIwOTI5YWJmNDA0ZDFjYWExZjI2YmM2YzU0MTJkNS5iaW5kUG9wdXAocG9wdXBfZmJkY2E3NTVmZDU0NDVmOTgxMjIxOGNmN2ZhYzUwZDcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Y0ZWZiOTI0ZTg2NTQ1NGU5ZmFmNGM4ZDQ0ZjNiNzg2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbMzcuNDkwNDA5Nzk5OTk5OTksIDEyNy4wMDUxNzYzXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYjZmYjU1YWEyMDEyNDBjYmI0NjVlMjk2ZTE4OTYwMTMgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2VkYjI1YmNhYjJiNzQ4NTFiOTQyYTI1MmE2NTAwZGUyID0gJChgPGRpdiBpZD0iaHRtbF9lZGIyNWJjYWIyYjc0ODUxYjk0MmEyNTJhNjUwMGRlMiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+64Sk7Jik7IKs7ZS87JeU7IqkKE5lb3NhcGllbmNlKTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9iNmZiNTVhYTIwMTI0MGNiYjQ2NWUyOTZlMTg5NjAxMy5zZXRDb250ZW50KGh0bWxfZWRiMjViY2FiMmI3NDg1MWI5NDJhMjUyYTY1MDBkZTIpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2Y0ZWZiOTI0ZTg2NTQ1NGU5ZmFmNGM4ZDQ0ZjNiNzg2LmJpbmRQb3B1cChwb3B1cF9iNmZiNTVhYTIwMTI0MGNiYjQ2NWUyOTZlMTg5NjAxMykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNTJlY2RlOTcwZjc3NGE3ZGIzNDljOWE5NzE3YWFjMzIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFszNy41OTIzNjE1LCAxMjcuMDQ5MTU0Ml0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMwNDQyNzUiLCAiZmlsbE9wYWNpdHkiOiAwLjIsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDE1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFya2VyX2NsdXN0ZXJfMDc0OTUxZDJiYTA0NGJlYWE1YmM4ZDkxZjg2ZmY0ODgpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzdlMTliMzIwZjI2ZjQzZDFiOTdlZmJhZjA2MDIxOTgxID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF84ODg5Y2QyN2ZiYTI0NDRkYmM4NjYzZTk4ZjI4YjA5NSA9ICQoYDxkaXYgaWQ9Imh0bWxfODg4OWNkMjdmYmEyNDQ0ZGJjODY2M2U5OGYyOGIwOTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPuy9lOq3uOuEpeyKpOy9lOumrOyVhDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF83ZTE5YjMyMGYyNmY0M2QxYjk3ZWZiYWYwNjAyMTk4MS5zZXRDb250ZW50KGh0bWxfODg4OWNkMjdmYmEyNDQ0ZGJjODY2M2U5OGYyOGIwOTUpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzUyZWNkZTk3MGY3NzRhN2RiMzQ5YzlhOTcxN2FhYzMyLmJpbmRQb3B1cChwb3B1cF83ZTE5YjMyMGYyNmY0M2QxYjk3ZWZiYWYwNjAyMTk4MSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNGZlYzNkNTQwNWQxNGRlMWEwOWM1ZWIyM2FlODY1MDggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFszNy41MDk1Njg3LCAxMjcuMDU3OTY3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOGMwNTI5NWQ2ZTkzNDIyMjkwYjEwMGQyNWEzYjllYmIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzYwZmJjNmI2ODgxZDRhZGFhYjQ4YTZiNGFjYjQ1NTc2ID0gJChgPGRpdiBpZD0iaHRtbF82MGZiYzZiNjg4MWQ0YWRhYWI0OGE2YjRhY2I0NTU3NiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7J2066eI6rOg7JuN7IqkPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzhjMDUyOTVkNmU5MzQyMjI5MGIxMDBkMjVhM2I5ZWJiLnNldENvbnRlbnQoaHRtbF82MGZiYzZiNjg4MWQ0YWRhYWI0OGE2YjRhY2I0NTU3Nik7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNGZlYzNkNTQwNWQxNGRlMWEwOWM1ZWIyM2FlODY1MDguYmluZFBvcHVwKHBvcHVwXzhjMDUyOTVkNmU5MzQyMjI5MGIxMDBkMjVhM2I5ZWJiKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82NGU3MWY5MmEwMTU0ZmNlYWVlODczYTZhZGNhYzZhMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjUwNDI5NTcsIDEyNy4wNjMwNTI3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOWMzN2JhZGVhYTY4NDlmN2E5ZjY3NmVhNDM4NGJkOTYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzgyZjY0ZmMwZGI2NjRjZTFhMjYxYjIwN2ZlZDgzZjlhID0gJChgPGRpdiBpZD0iaHRtbF84MmY2NGZjMGRiNjY0Y2UxYTI2MWIyMDdmZWQ4M2Y5YSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+7J2066eI6rOg7JuN7IqkPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzljMzdiYWRlYWE2ODQ5ZjdhOWY2NzZlYTQzODRiZDk2LnNldENvbnRlbnQoaHRtbF84MmY2NGZjMGRiNjY0Y2UxYTI2MWIyMDdmZWQ4M2Y5YSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNjRlNzFmOTJhMDE1NGZjZWFlZTg3M2E2YWRjYWM2YTIuYmluZFBvcHVwKHBvcHVwXzljMzdiYWRlYWE2ODQ5ZjdhOWY2NzZlYTQzODRiZDk2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mN2VjZjlhNmQzNzQ0YWRjYmY3ODRlMGU5MTkzMTlmYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjU1MDQwMzQsIDEyNy4wNDc5MDMxXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMjA3MzcwZjVhNmU2NDk5YmEyODRhM2RkMGI0ZjA1ZTAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzEzMTgxZWE4MjUzODQyMTI5NmQ2MTkyZmQ4MGIyMWI5ID0gJChgPGRpdiBpZD0iaHRtbF8xMzE4MWVhODI1Mzg0MjEyOTZkNjE5MmZkODBiMjFiOSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+66S87J2065OcKFJpaWlkKTwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yMDczNzBmNWE2ZTY0OTliYTI4NGEzZGQwYjRmMDVlMC5zZXRDb250ZW50KGh0bWxfMTMxODFlYTgyNTM4NDIxMjk2ZDYxOTJmZDgwYjIxYjkpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2Y3ZWNmOWE2ZDM3NDRhZGNiZjc4NGUwZTkxOTMxOWZhLmJpbmRQb3B1cChwb3B1cF8yMDczNzBmNWE2ZTY0OTliYTI4NGEzZGQwYjRmMDVlMCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYmQxZTRkNzYxNTA2NDFiZTg3YzJkNGRiZWI3MGY2ZGUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFszNy41MDA5NDUyLCAxMjcuMDM2MTI3OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMwNDQyNzUiLCAiZmlsbE9wYWNpdHkiOiAwLjIsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDE1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFya2VyX2NsdXN0ZXJfMDc0OTUxZDJiYTA0NGJlYWE1YmM4ZDkxZjg2ZmY0ODgpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2RiYmFjMjZhZWI4NDRlNWViMDJiZDkwMmViNTZiM2NmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF80OGE0MjVmODA4Y2Y0MjNkOGIxZWRhMTgzMmE5ZTI5YSA9ICQoYDxkaXYgaWQ9Imh0bWxfNDhhNDI1ZjgwOGNmNDIzZDhiMWVkYTE4MzJhOWUyOWEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPuu4lOujqOuLtzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9kYmJhYzI2YWViODQ0ZTVlYjAyYmQ5MDJlYjU2YjNjZi5zZXRDb250ZW50KGh0bWxfNDhhNDI1ZjgwOGNmNDIzZDhiMWVkYTE4MzJhOWUyOWEpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2JkMWU0ZDc2MTUwNjQxYmU4N2MyZDRkYmViNzBmNmRlLmJpbmRQb3B1cChwb3B1cF9kYmJhYzI2YWViODQ0ZTVlYjAyYmQ5MDJlYjU2YjNjZikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOWU0NjIxZGIyMjY2NDcwNmJkZWIyNzZkYTNhNWUzMzQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFszNy40OTcwNzIsIDEyNy4wMjg1Nzc3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMzZhZGQ3NDI2OTc3NDhhMTgxYmM0YmY1MzExODE2NmMgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzliOWQzODE5NTRkODRjZjJhNjJlZWU1MWFkMzY2MTZkID0gJChgPGRpdiBpZD0iaHRtbF85YjlkMzgxOTU0ZDg0Y2YyYTYyZWVlNTFhZDM2NjE2ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+642U6riw7ZSE7YyF7Lu07Y2864uIPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzM2YWRkNzQyNjk3NzQ4YTE4MWJjNGJmNTMxMTgxNjZjLnNldENvbnRlbnQoaHRtbF85YjlkMzgxOTU0ZDg0Y2YyYTYyZWVlNTFhZDM2NjE2ZCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfOWU0NjIxZGIyMjY2NDcwNmJkZWIyNzZkYTNhNWUzMzQuYmluZFBvcHVwKHBvcHVwXzM2YWRkNzQyNjk3NzQ4YTE4MWJjNGJmNTMxMTgxNjZjKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wM2I0ZjM4ZjcxZTE0MmQ1YjM4Y2JlZDFmZGFmMzdlYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzM3LjUxNDY0NzQsIDEyNy4wMzIwNDY4XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzA0NDI3NSIsICJmaWxsT3BhY2l0eSI6IDAuMiwgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogMTUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXJrZXJfY2x1c3Rlcl8wNzQ5NTFkMmJhMDQ0YmVhYTViYzhkOTFmODZmZjQ4OCk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNGY3M2UxOTY0ODUyNDVkZmIxOTNiYzdjZjRkZjljNjcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzNhZTAwNjMzYWZlYzRhYTc5Y2FlYTFiYmQ4NmM0MjA2ID0gJChgPGRpdiBpZD0iaHRtbF8zYWUwMDYzM2FmZWM0YWE3OWNhZWExYmJkODZjNDIwNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+642U6riw7ZSE7YyF7Lu07Y2864uIPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzRmNzNlMTk2NDg1MjQ1ZGZiMTkzYmM3Y2Y0ZGY5YzY3LnNldENvbnRlbnQoaHRtbF8zYWUwMDYzM2FmZWM0YWE3OWNhZWExYmJkODZjNDIwNik7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMDNiNGYzOGY3MWUxNDJkNWIzOGNiZWQxZmRhZjM3ZWEuYmluZFBvcHVwKHBvcHVwXzRmNzNlMTk2NDg1MjQ1ZGZiMTkzYmM3Y2Y0ZGY5YzY3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
add_table=pd.DataFrame(dt[['회사명','주소']])

DF_seoul=pd.DataFrame(add_table['주소'].str.contains('서울'))
new_seoul=DF_seoul.replace(True,'서울').replace(False,'경기')
new_seoul=pd.concat([add_table['회사명'],new_seoul],axis=1)

#plt.figure(figsize=(12, 3))
font_location = r"c:/Windows/fonts/malgun.ttf"
font_name = font_manager.FontProperties(fname=font_location).get_name()
matplotlib.rc('font', family=font_name)
matplotlib.rc('font', size=10)

new_seoul['주소'].value_counts().plot(kind='bar', rot=0) #, colors=['slateblue','darkslateblue']
plt.title('회사 분포')
plt.xlabel('지역')
plt.ylabel('개수')

plt.show()
```


![png](output_58_0.png)

