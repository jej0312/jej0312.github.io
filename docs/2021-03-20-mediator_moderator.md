---
# layout: post
title: "Mediating/Moderating Effect(매개/조절효과)"
categories:
 - statistical analysis
tags: 
 - statistical analysis
 - regression
 - mediating effect
 - moderating effect
# share: true 
comments: true 
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
---

### 조절효과

![moderator](/img/moderator.png)

- 조절효과는 독립변수가 어떤 변수와 합쳐졌을 때 해당 변수가 종속변수의 방향 또는 크기에 영향을 주는지를 살펴보는 것이다.
    - 방향의 변화: 회귀계수가 + ↔ - 변화
    - 크기의 변화: 회귀계수의 절대값이 커지거나 작아짐
- 조절효과는 선형관계를 확인하는 회귀분석에서 비선형관계를 알 수 있는 방법이다.
- 예시는 다음과 같다.
    - Y(종속변수): 지출, X(독립변수): 수입, D(조절변수): 성별
    - 성별을 더미변수로 변환(0:남자, 1:여자)할 경우 조절효과의 회귀식은 다음과 같이 표현할 수 있다.

        $$Y = \beta_0 + \beta_1X+\beta_2D+\beta_3X\times D+ \epsilon$$

        - 기본 회귀식에 더미변수를 추가함으로써, 성별에 따라 절편이 변화한다. (기울기는 유지)
        - 독립변수와 조절변수의 곱을 교호작용(interaction term)이라 함. 교호작용 변수를 추가함으로써, 성별에 따라 기울기가 변화한다. (따라서 \beta_3의 방향을 확인하여 얼마나 유의하게 증가/감소하는지 확인)

            $$Y = \beta_0 + \beta_1X+ \epsilon , \ For \ male \ (D=0) \\ Y= (\beta_0+\beta_2) + (\beta_1 + \beta_3)X+ \epsilon, \ For \ female \ (D=1) $$

            $$Y = (\beta_0+\beta_2) + (\beta_1 + \beta_3)X+ \epsilon$$

    - \beta_2가 유의하다는 것은 여성의 평균 지출금액이 남성보다 유의하게 많다는 것을 의미한다.
    - \beta_3이 양수이며 유의하다는 것은 여성과 남성의 수입이 동일하게 1씩 증가할 경우 여성의 지출이 더 많이 증가한다는 것을 의미한다.
- 검증 방식은 다음과 같은 3단계로 이루어진다.
    1. XZ(interation term)을 계산한다.
    2. X, Z, XZ에 대한 다중선형회귀를 추정한다.
    3. XZ의 회귀계수가 유의미한지를 평가한다.
- 즉, X→Y, D→Y, XD→Y를 입증해야 한다.
- 관련 논문: Baron, R.M., & Kenny, D. A. (1986). The moderator-mediator variable distinction in social psychological research: Conceptual, strategic, and statistical considerations. Journal of personality and social psychology, 51(6).

### 매개효과

![mediator](/img/mediator.png)

- 매개효과는 내가 설정한 독립변수가 어떤 변수를 통했을 때 종속변수에 미치는 영향을 살펴보는 것이다.
- 매개의 종류에 따라 독립변수의 영향이 종속변수에 다다르지 못하는 경우가 있어 유용하다.
- 검증 방식은 크게 다음과 같은 4단계로 이루어진다.
    1. X → M 효과 추정: X를 독립변수로, M을 종속변수로 하여 단순 회귀분석 실시한다. ⇒ a(간접효과) 추정
    2. 전체 효과 추정: X를 독립변수, Y를 종속변수로 하여 단순 회귀분석 실시. 이 때 산출되는 회귀계수는 직간접 효과를 합한 전체 효과($a \times b + c$) 반영한다. ⇒ c(직접효과) 추정
    3. M→Y, X→Y 효과 추정: X, M을 독립변수로, Y를 종속변수로 하여 다중회귀분석 실시한다. ⇒ b 추정
    4. 매개효과 추정: a와 b를 곱하면 M의 매개효과가 산출된다. (a와 b는 통계적으로 유의미해야 한다.) 
- 즉, X→M, X→Y, X+M→Y를 입증해야 한다.
    - X는 M에 대해, X는 Y에 대해, M이 추가된 상황에서 M은 무조건 유의해야 한다.
    - M을 포함했을 때 X가 Y에 주는 영향이 0일 경우(X가 유의하지 않을 경우) 완전매개, 영향이 줄어들 경우(X 역시 유의할 경우) 부분매개라 한다.
- Sobel test는 간접효과가 유의한지 알아보는 테스트이다. \beta_1과 \beta_1의 표준오차, \beta_3과 \beta_3의 표준오차를 사용하여 t-value를 계산한다.

    $$\frac{\beta_1 \times \beta_3}{\sqrt{(\beta_1^2 \times SE_{\beta_3}^2) \times (\beta_3^2 \times SE_{\beta_1}^2)}} $$

    - 결측치로 인한 샘플 사이즈의 변동에 따라 이론적인 간접효과가 달라질 수 있으며, \beta_1 \times \beta_3의 분포가 불분명하므로 부트스트랩을 통해 문제를 해결할 수 있다.

- 참고문서
    - [https://blog.naver.com/moses3650/221232430018](https://blog.naver.com/moses3650/221232430018)
    - [https://blog.naver.com/moses3650/221239332588](https://blog.naver.com/moses3650/221239332588)
    - [https://learnx.tistory.com/entry/연구모형단순-매개효과-조절효과](https://learnx.tistory.com/entry/%EC%97%B0%EA%B5%AC%EB%AA%A8%ED%98%95%EB%8B%A8%EC%88%9C-%EB%A7%A4%EA%B0%9C%ED%9A%A8%EA%B3%BC-%EC%A1%B0%EC%A0%88%ED%9A%A8%EA%B3%BC)
    - (moderator 실습 in R) [https://advstats.psychstat.org/book/moderation/index.php](https://advstats.psychstat.org/book/moderation/index.php)
    - (실습 in R) [https://ademos.people.uic.edu/Chapter14.html](https://ademos.people.uic.edu/Chapter14.html)