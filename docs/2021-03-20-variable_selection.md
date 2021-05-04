---
# layout: post
title: "Mediating/Moderating Effect(매개/조절효과)"
categories:
 - statistical analysis
tags: 
 - statistical analysis
 - regression
 - variable selection
 - forward selection
 - backward elimination
 - stepwise selection
# share: true 
comments: true 
toc: true
toc_label: 'Contents'
toc_sticky: true
read_time: false
---

## Variable Selection

- 회귀분석을 진행할 때 다중공선성이 발생한 경우 이를 보완하기 위한 방법으로 변수선택을 진행한다.
- 변수 선택법의 종류로는 세 가지가 있다.
    - Forward Selection
    - Backward Elimination
    - Stepwise Selection
- 변수 선택의 기준은 Correlation이다.
    - 단순회귀모형에서는 correlation을, 다중선형회귀모형에서는 multiple correlation을 사용한다.
    - Correlation을 측정하는 방법은 두 가지가 있다.
        - Pearson Correlation: R^2(결정계수)는 피어슨 상관계수의 제곱과 동일하다.
        - Spearman's Rank-Order Correlation: 자료가 서열척도로 구성된 경우, 혹은 등간척도인 경우 변수간 관련성을 측정할 수 있다.
    - Partial Correlation은 다음과 같은 과정으로 측정한다.

        $$Y=\alpha + \beta_1X_1+\beta_2X_2+\beta_3X_3+\epsilon \\ When \ X_3 \ is \ added \ to \ the \ model,  \ the \ variation \ explained \ by \ X_3 \ is \ as \ follows: \\ SSR(X_3|X_1,X_2) = SSR(X_1,X_2,X_3)-SSR(X_1, X_2) \\ So \ the \ proportion \ of \ the \ variation \ marginally \ explained \ by \ X_3 \ is \ SSR(X_3|X_1,X_2)/SSE(X_1,X_2). \\ This \ proportion \ is \ called \ a \ partial \ determination \ coefficient \ between \ X_3 \ and \ Y \ controlling \ for \ X_1 \ and \ X_2. \\ And \ partial \ coefficient \ is \ defined \ as \ following. \\ \gamma_{y3|1,2}=(-)\frac{\sqrt{SSR(X_3|X_1,X_2)}}{\sqrt{SSE(X_1,X_2)}}, \ where \ the \ sign \ is \ as \ same \ as \ it \ of \ corresponding \ regression \ coefficient.$$