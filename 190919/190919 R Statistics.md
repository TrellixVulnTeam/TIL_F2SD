# 190919 R Statistics

## I. 검정방법

### 1. 적합도 검정

- 데이터의 크기가 일정 수 이상이라면 데이터가 정규 분포를 따르는지 확인

- 데이터가 특정 분포를 따르는지 살펴보기 위해 분할표를 만들고,  카이 제곱 검정(Chi Squared Test)을 사용할 수 있다.

```R
# MASS::survey 데이터를 사용해 글씨를 왼손으로 쓰는 사람과 오른손으로 쓰는 사람의 비율이 30% : 70%인지 여부를 분석

> table(survey$W.Hnd)
> chisq.test(table(survey$W.Hnd), p=c(.3, .7))
#p-value < 0.05이므로 글씨를 왼손으로 쓰는 사람과 오른손으로 쓰는 사람의 비가 30% : 70%라는 귀무가설을 기각한다.
```

- 샤피로 윌크 검정Shapiro-Wilk Test
  - 표본이 정규 분포로부터 추출된 것인지 테스트하기 위한 방법이다. 

- shapiro.test( ) 함수를 사용

- 귀무가설은 주어진 데이터가 정규 분포로부터의 표본이라는 것이다.

![1568869057107](190919%20R%20Statistics.assets/1568869057107.png)

```R
> shapiro.test(rnorm(1000))

	Shapiro-Wilk normality test

data:  rnorm(1000)
W = 0.99938, p-value = 0.9896
```



### 2. 콜모고로프 스미르노프 검정(K-S Test)

- 데이터의 누적 분포 함수와 비교하고자 하는 분포의 누적 분포 함수(Cumulative Distribution Function) 간의 최대 거리를 통계량으로 사용하는 가설 검정 방법 

![1568869142177](190919%20R%20Statistics.assets/1568869142177.png)

![1568869145888](190919%20R%20Statistics.assets/1568869145888.png)

```R
#정규 분포를 따르는 두 난수 데이터 간에 분포가 동일한지를 살펴본 예
> ks.test(rnorm(100), rnorm(100))

	Two-sample Kolmogorov-Smirnov test

data:  rnorm(100) and rnorm(100)
D = 0.11, p-value = 0.5806
alternative hypothesis: two-sided
# p-value > 0.05이므로 두 난수가 같은 분포라는 귀무가설을 기각할 수 없다


> ks.test(rnorm(100), runif(100))

	Two-sample Kolmogorov-Smirnov test

data:  rnorm(100) and runif(100)
D = 0.61, p-value < 2.2e-16
alternative hypothesis: two-sided
# p-value가 0.05보다 작아 서로 다른 분포로 판단되었다.


# K-S 검정을 사용해 데이터가 평균 0, 분산 1인 정규 분포로부터 뽑은 표본인지 확인
> ks.test(rnorm(1000), "pnorm", 0, 1)

	One-sample Kolmogorov-Smirnov test

data:  rnorm(1000)
D = 0.030119, p-value = 0.3245
alternative hypothesis: two-sided
#귀무가설을 기각할 수 없어, 주어진 rnorm(100)은 평균 0, 분산 1인 정규 분포로부터의 표본이라고 결론
```



### 3. Q-Q도(Q-Q Plot)

- qqnorm(y) : 주어진 데이터와 정규 확률분포를 비교하는 Q-Q도를 그린다.
- qqplot(x, y) : 두 데이터 셋에 대한 Q-Q도를 그린다.
- qqline(y, distribution=qnorm) : 데이터와 분포를 비교해 이론적으로 성립해야 하는 직선 관계를 그린다.

```R
> x <- rnorm(1000, mean=10, sd=1)
> qqnorm(x)
> qqline(x, lty=2)
# 직선 관계 성립
```

![1](190919%20R%20Statistics.assets/1.JPG)

![2](190919%20R%20Statistics.assets/2.JPG)



```R
> x <- runif(1000)
> qqnorm(x)
> qqline(x, lwd=2)
# 직선 관계가 성립한다 할지라도 데이터의 출처 및 데이터가 정규성을 따를 이유에 대한 고민이 항상 필요하다
```

![3](190919%20R%20Statistics.assets/3.JPG)

![4](190919%20R%20Statistics.assets/4.JPG)

```R
# 정규 확률 그림이 아닌 분포에 대해서는 qqplot( ) 함수를 사용한다
> qqplot(runif(1000, min=1, max=10), 1:10)
```

![5](190919%20R%20Statistics.assets/5.JPG)



----

## II. 추정과 검정

- 추론 통계 분석  

  - 모집단으로부터 추출된 표본으로부터 모수와 관련된 통계량(statistics)들의 값을 계산하고, 이것을 이용하여 모집단의 특성(모수)을 알아내는 과정

  - 모집단으로부터 추출한 표본으로부터 얻은 정보를 이용하여 모집단의 특성을 나타내는 값을 확률적으로 추측하는 추정(estimation)
  - 유의수준과 표본의 검정 통계량을 비교하여 통계적 가설의 진위를 입증하는 가설 검정(hypothesis testing)



### 1. 점추정과 구간추정

#### 1) 점추정

- 하나의 값을 제시하여 모수의 참값을 추측하는 방법
  - 추정값과 모수의 참값 사이의 오차범위 제공 안함
  - 점 추정은 하나의 값과 표본에 의한 검정 통계량을 직접 비교하여 일치하면 귀무가설이 기각되지만, 일치하지 않으면 귀무가설이 채택된다. 
  - 점 추정 방식에 의한 가설 검정은 귀무가설의 기각률이 낮다고 볼 수 있다. 또한 검정 통계량과 모수의 참값 사이의 오차범위를 확인 할 수 없다.

#### 2) 구간추정

- 하한값과 상한값의 구간을 지정 모수의 참값을 추정하는 방법

  - 추정값과 모수의 참값 사이의 오차범위 제공
  - 구간 추정 방식으로 가설을 검정할 경우 오차범위에 의해서 결정된 하한값과 상한값의 신뢰구간과 검정 통계량을 비교하여 가설을 검정 – 추론 통계 분석에서는 구간 추정 방식을 더 많이 이용, 오차범위는 모표준편차가 알려지지 않은 경우 표본의 표준편차(S)를 이용하여 추정한다.

- 모평균의 구간 추정 

  - 모집단으로부터 추출된 표본으로부터 모수와 관련된 통계량(statistics)들의 값을 계산하고, 이것을 이용하여 모집단의 특성(모수)을 알아내는 과정
  - 모집단으로부터 추출한 표본으로부터 얻은 정보를 이용하여 모집단의 특성을 나타내는 값을 확률적으로 추측하는 추정(estimation)
  - 유의수준과 표본의 검정 통계량을 비교하여 통계적 가설의 진위를 입증하는 가설 검정(hypothesis testing)
  - 표본평균(X ̅)을 이용하여 모평균(μ)에 대한 신뢰도 95% 신뢰구간을 추정하는 방법

  ![1568872283631](190919%20R%20Statistics.assets/1568872283631.png)

  - 신뢰구간의 길이는 n의 제곱에 반비례,  모표준편차σ에 비례한다.

  ![1568872302438](190919%20R%20Statistics.assets/1568872302438.png)

  - 모표준편차 σ의 값이 알려지지 않았을 때는 표본의 크기인 n이 충분히 클때 (n ≥ 30)에는 표존 표준편차 S을 사용한다.

```R
##########표본의 통계량으로 모집단의 평균 구간 추정  ##############
# 우리나라 전체 중학교 2학년 남학생의 평균 키를 알아보기 위해서 중학교 2학년 남학생 10,000명 대상으로 키를 조사한 결과  표본평균 (𝑋 ̅) 은 165.1cm이고 표본 표준편차(S)는 2cm 였다.
# 전체 표본 크기(N) : 10,000명
# 표본평균 (X) : 165.1cm
# 표본 표준편차(S) : 2cm

> N <- 10000
> X <- 165.1
> S <- 2
> low <- X-1.96 * S/sqrt(N)   #신뢰구간의 하한값
> high <- X+1.96 * S/sqrt(N)   #신뢰구간의 상한값

> low;high
[1] 165.0608
[1] 165.1392
# 신뢰구간 165.0608< μ(165.1) < 165.1392 에 평균 이 포함

# 신뢰구간으로 표본 오차 구하기
# 하한값-평균신장 , 상한값-평균신장 값을 백분율로 적용
> (low-X)  * 100
[1] -3.92
> (high-X) * 100
[1] 3.92
# 신뢰구간의 표본 오차는 ±3.92

# 해석 : 우리나라  중학교 2학년 남학생의 평균 신장이 95%신뢰수준에서 표본 오차는 ±3.92 범위에서 평균 165.1cm로 조사되었다면 실제 평균키는 165.0608 ~165.1392 사이에 나타날 수 있다.
```



- 표본 오차

  - 표본이 모집단의 특성과 정확히 일치하지 않아서 발생하는 확률의 차이
  - 신뢰구간의 하한값에서 평균을 빼고, 상한값에서 평균을 뺀 값을 백분율로 적용

- 모 비율의 구간 추정  

  - 모비율(p) : 모집단에서 어떤 사건에 대한 비율  예) 제품의 불량율, 대선 후보 지지율 
  - 모비율 추정 : 모집단으로부터 임의추출한 표본에서 어떤 사건에 대한 비율인 표본비율(p ̂)을 이용하여 모비율을 추정
  - 표본비율을 이용하여 모비율에 대한 신뢰도 95% 신뢰구간을 추정하는 방법

  ![1568872379395](190919%20R%20Statistics.assets/1568872379395.png)

```R
##########표본의 비율로부터 모집단의 비율 구간 추정  ###########
# A반도체 회사의 사원을 대상으로 임의 추출한 150명을 조사한 결과 90명이 여자 사원이다
# 표본 크기(n) : 150 
> n <- 150
# 표본비율(????)  : 90/150 = 0.6
> p <- 90/150

# 전체 여자 사원 비율 (모비율) 
> p-1.96 * sqrt(p*(1-p)/n)
[1] 0.5216
> p+1.96 * sqrt(p*(1-p)/n)
[1] 0.6784

#모집단의 비율 구간은 다음과 같습니다.  
0.596864 ≤ 모비율(P) ≤ 0.603136
```



### 2. 단일 집단 검정

- 한 개의 집단과 기존 집단과의 비율 차이 검정은 기술 통계량으로 빈도수에 대한 비율에 의미가 있으며, 평균 차이 검정은 표본 평균에 의미가 있다.

#### 1) 단일 집단 비율 검정   

- 단일 집단의 비율이 어떤 특정한 값과 같은지를 검정하는 방법

- 데이터 전처리 (이상치와 결측치 제거) -> 기술통계량(빈도분석) -> binom.test() -> 검정통계량 분석

  - 이항분포 비율검정

    - 명목척도의 비율을 바탕으로 binom.test()를 이용하여 이항분포의 양측 검정을 통해서 검정 통계량을 구한 후 이를 이용하여 가설을 검정
    - 이항분포는 이산변량이며, 그래프는 좌우대칭인 종 모양의 곡선 형태를 갖는다.

    ![1568872772294](190919%20R%20Statistics.assets/1568872772294.png)

    - alternative=“two.sided”은 양측 검정을 의미
    - conf.level=0.95는 95% 신뢰수준을 의미
    - 귀무가설이 모평균=상수 일때와 모 평균 = 상수가 아닐때 양측가설 검정을 수행하고 방향성이 있는 경우 단측가설 검정을 수행
    - alternative=“greater”은 방향성을 갖는 연구가설을 검정할 경우 이용된다.

- 비율 차이 검정 통계량을 바탕으로 귀무가설의 기각 여부를 결정한다

```R
##################단일 집단 비율 검정 실습###################
# 2016년도 114 전화번호 안내고객을 대상으로 불만을 갖는 고객은 20%였다. 이를 개선하기 위해서 2017년 CS 교육을 실시한 후 150명 고객을 대상으로 조사한 결과 14명이 불만을 가지고 있었다. 기존 20%보다 불만율이 낮아졌다고 할 수 있는가?

# 연구가설(H1) : 기존 2016년도 고객 불만율과 2017년 CS 교육 후 불만율에 차이가 있다.
# 귀무가설(H0) : 기존 2016년도 고객 불만율과 2017년 CS 교육 후 불만율에 차이가 없다.

> data <- read.csv("./data/one_sample.csv", header=TRUE)
> head(data)
  no gender survey time
1  1      2      1  5.1
2  2      2      0  5.2
3  3      2      1  4.7
4  4      2      1  4.8
5  5      2      1  5.0
6  6      2      1  5.4

> str(data)
'data.frame':	150 obs. of  4 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ gender: int  2 2 2 2 2 2 2 2 2 1 ...
 $ survey: int  1 0 1 1 1 1 1 1 0 1 ...
 $ time  : num  5.1 5.2 4.7 4.8 5 5.4 NA 5 4.4 4.9 ...


# 변수 : 번호, 성별, 만족도(명목척도), 시간
> x <- data$survey
> summary(x) 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
 0.0000  1.0000  1.0000  0.9067  1.0000  1.0000 

> length(x)   # 결측치 확인
[1] 150

> table(x)   # 0:불만족, 1:만족
x
  0   1 
 14 136 

> library(prettyR)
> freq(x)   # 결측치 확인
Frequencies for x 
        1    0   NA
      136   14    0
%    90.7  9.3    0 
%!NA 90.7  9.3 


# 만족도(명목척도)의 비율을 바탕으로 binom.test() 이항분포 양측검정 수행 -> 검정 통계량 -> 가설 검정
> binom.test(c(136, 14), p=0.8)
	Exact binomial test
data:  c(136, 14)
number of successes = 136, number of trials = 150, p-value =
0.0006735
alternative hypothesis: true probability of success is not equal to 0.8
95 percent confidence interval:
 0.8483615 0.9480298
sample estimates:
probability of success 
             0.9066667 


> binom.test(c(136, 14), p=0.8,  alternative="two.sided", conf.level=0.95)
	Exact binomial test
data:  c(136, 14)
number of successes = 136, number of trials = 150, p-value =
0.0006735
alternative hypothesis: true probability of success is not equal to 0.8
95 percent confidence interval:
 0.8483615 0.9480298
sample estimates:
probability of success 
             0.9066667 

# 해석 : p-value 유의확률 0.0006735로 유의수준 0.05보다 작기 때문에 기존 만족률(80%)과 차이가 있다. 기존 만족률(80%)보다 크다 혹은 작다의 방향성을 제시하지 않음


# 방향성을 갖는 단측 검정 수행 (더 큰 비율인가? 검정)
> binom.test(c(136, 14), p=0.8,  alternative="greater", conf.level=0.95)

	Exact binomial test

data:  c(136, 14)
number of successes = 136, number of trials = 150, p-value =
0.0003179
alternative hypothesis: true probability of success is greater than 0.8
95 percent confidence interval:
 0.8579426 1.0000000
sample estimates:
probability of success 
             0.9066667 

# 해석 : p-value 유의확률 0.0003179로 유의수준 0.05보다 작기 때문에 기존 만족률 80% 이상의 효과를 얻을 수 있다. CS 교육후에 고객의 불만율은 낮아졌다고 할 수 있다. 귀무가설은 기각이고 연구가설이 채택되므로 CS교육에 효과가 있다.


# 불만율 기준 비율 검정
> binom.test(c(14, 136), p=0.2)

	Exact binomial test

data:  c(14, 136)
number of successes = 14, number of trials = 150, p-value =
0.0006735
alternative hypothesis: true probability of success is not equal to 0.2
95 percent confidence interval:
 0.05197017 0.15163853
sample estimates:
probability of success 
            0.09333333 


> binom.test(c(14, 136), p=0.2,  alternative="two.sided", conf.level=0.95)

	Exact binomial test

data:  c(14, 136)
number of successes = 14, number of trials = 150, p-value = 0.0006735
alternative hypothesis: true probability of success is not equal to 0.2
95 percent confidence interval:
 0.05197017 0.15163853
sample estimates:
probability of success 
            0.09333333 
# 해석 : p-value 유의확률 0.0006735로 유의수준 0.05보다 작기 때문에 기존 불만족률(20%)과 차이가 있다.


> binom.test(c(14, 136), p=0.2,  alternative="less", conf.level=0.95)

	Exact binomial test

data:  c(14, 136)
number of successes = 14, number of trials = 150, p-value = 0.0003179
alternative hypothesis: true probability of success is less than 0.2
95 percent confidence interval:
 0.0000000 0.1420574
sample estimates:
probability of success 
            0.09333333 
# 해석 : p-value 유의확률 0.0003179로 유의수준 0.05보다 작기 때문에 기존 불만족률 20% 보다 낮다.
```



#### 2) 단일 집단 평균 검정 (단일표본 T 검정)

- 단일집단의 평균이 어떤 특정한 집단의 평균과 차이가 있는지를 검정하는 방법
- 데이터 전처리 (이상치와 결측치 제거) -> 기술통계량(평균) -> 정규분포( shapiro.test()) -> t.test() 또는 wilcox.test()  -> 검정통계량 분석
- 모수 검정인 경우 T검정을 수행
- 비모수 검정인 경우 Wilcox 검정을 수행



- 평균 검정 통계량의 특징
  - 비율척도 같은 수치 기반 데이터에 의미가 있다.
  - 분포의 중심 위치를 나타내는 대표값의 성격을 가지며, 정규분포에서 도수분포곡선이 평균값을 중으로 하여 좌우 대칭인 종 모양을 형성한다.
  - 집단 간의 평균에 차이가 있는지를 검정하는 용도로 사용된다.

- 정규분포 검정
  - 단일표본 평균 차이 검정을 하기 전에 데이터의 분포형태가 정규분포 인지를 먼저 검정해야 한다.
  - 정규분포 검정은 stats 패키지에서 제공하는 shapiro.test() 이용
  - 검정 결과가 유의수준 0.05보다 큰 경우 정규분포로 본다.

- 평균 차이 검정
  - 모집단에서 추출한 표본 데이터의 분포형태가 정규분포 형태를 보인다면, T-검정을 수행
  - T-검정은 모집단의 평균값을 검정하는 방법으로 stats 패키지에서 제공하는 t.test() 함수 이용

  ```R
  t.test(x, y = NULL, alternative = c(“two.sided”, “less”, “greater”),
            mu = 0 , paired=FALSE, var.equal = FALSE,  conf.level=0.95, …)
  # alternative  속성 -  양측 검정과 단측 검정
  # conf.level  속성 – 신뢰수준
  # mu 속성 – 비교할 기존 모집단의 평균값을 지정
  ```

  - stats 패키지에서 제공하는 qt()를 이용하면, 귀무가설의 임계값을 확인할 수 있다. 
  - qt() 함수에서 p-value와 자유도(df)를 인수로 지정하여 함수를 실행하면 귀무가설을 기각할 수 있는 임계값을 얻을 수 있다.

```R
########### 단일집단의 평균 검정 (단일표본 T검정) ##############
# 국내에서 생산된 노트북 평균 사용시간이 5.2시간으로 파악된 상황에서 A회사에서 생산된 노트북 평균 사용시간과 차이가 있는지를 검정하기 위해서 A 회사 노트북 150대를 랜덤으로 선정하여 검정을 실시한다. 국내에서 생산된 노트북 평균 사용시간이 5.2시간으로 파악된 상황에서 A회사에서 생산된 노트북 평균 사용시간과 차이가 있는지를 검정하기 위해서 A회사 노트북 150대를 랜덤으로 선정하여 검정을 실시한다.
# 연구가설(H1) : 국내에서 생산된 노트북과 A회사에서 생산된 노트북의 평균 사용시간에 차이가 있다.
# 귀무가설(H0) : 국내에서 생산된 노트북과 A회사에서 생산된 노트북의 평균 사용시간에 차이가 없다.

> data <- read.csv("./data/one_sample.csv", header=TRUE)
> head(data)
  no gender survey time
1  1      2      1  5.1
2  2      2      0  5.2
3  3      2      1  4.7
4  4      2      1  4.8
5  5      2      1  5.0
6  6      2      1  5.4

> str(data) 
'data.frame':	150 obs. of  4 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ gender: int  2 2 2 2 2 2 2 2 2 1 ...
 $ survey: int  1 0 1 1 1 1 1 1 0 1 ...
 $ time  : num  5.1 5.2 4.7 4.8 5 5.4 NA 5 4.4 4.9 ...

> x <- data$time
> summary(x)  # 결측치 확인
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NAs 
  3.000   5.000   5.500   5.557   6.200   7.900      41 

> length(x)   
[1] 150

> mean(x, na.rm=T)  # NA 제외 평균  
[1] 5.556881

> x1 <- na.omit(x)  # NA 제외 평균 
> mean(x1) 
[1] 5.556881


# 정규분포 검정
> shapiro.test(x1)
	Shapiro-Wilk normality test
data:  x1
W = 0.99137, p-value = 0.7242  

# 해석 : 검정 통계량 p-value 값은 0.7242 로 유의수준 0.05보다 크기 때문에 x1 객체의 데이터 분포는 정규분포를 따른다고 할 수 있다.

> hist(x1)  
```

![10](190919%20R%20Statistics.assets/10.JPG)

```R
#stats패키지에서 정규성 검정 - qqnorm(), qqline()는 정규분포 시각화
> install.packages("stats")
> library(stats)
> qqnorm(x1)
> qqline(x1, lty=1, col='blue')
```

![11](190919%20R%20Statistics.assets/11.JPG)

![12](190919%20R%20Statistics.assets/12.JPG)

```R
# 모수 검정 : T-검정 
> t.test(x1, mu=5.2)  # x1객체와 기존 모집단의 평균 5.2시간 비교
	One Sample t-test
data:  x1
t = 3.9461, df = 108, p-value = 0.0001417
alternative hypothesis: true mean is not equal to 5.2
95 percent confidence interval:
 5.377613 5.736148
sample estimates:
mean of x 
 5.556881 

> t.test(x1, mu=5.2, alter="two.side", conf.level=0.95)
	One Sample t-test
data:  x1
t = 3.9461, df = 108, p-value = 0.0001417
alternative hypothesis: true mean is not equal to 5.2
95 percent confidence interval:
 5.377613 5.736148
sample estimates:
mean of x 
 5.556881 

#### 해석 : 검정 통계량 p-value 값은 0.0001417 로 유의수준 0.05보다 작기 때문에 귀무가설 기각
# 귀무가설 : 차이가 없다  0.05보다 크면 채택, 0.05보다 작으면 기각
# 연구가설 : 차이가 있다 
# 국내에서 생산된 노트북 평균 사용시간과  A회사에서 생산된 노트북의 평균 사용시간의 차이가 있다. x1의 평균은 5.55688(점추정)는 신뢰구간에 포함되고 A회사에서 생산된 노트북의 평균 사용시간 5.2는 신뢰구간을 벗어나므로 귀무가설이 기각된다.(신뢰구간은 귀무가설의 채택역의 의미가 있으므로)


# 방향성을 갖는 단측 검정
> t.test(x1, mu=5.2, alter="greater", conf.level=0.95)
	One Sample t-test
data:  x1
t = 3.9461, df = 108, p-value = 7.083e-05
alternative hypothesis: true mean is greater than 5.2
95 percent confidence interval:
 5.406833      Inf
sample estimates:
mean of x 
 5.556881 
# 해석 : 검정 통계량 p-value 값은 7.083e-6로 유의수준 0.05보다 매우 작기 때문에 귀무가설을 채택
# 귀무가설 임계값 계산 qt(p-value, df): 귀무가설을 기각할 수 있는 임계값  qt(7.083e-6, 108) 계산 결과는 -3.946073 (임계값은 절대값) 임계값 3.946073 이상이면 귀무가설을 기각할 수 있다. t 검정 통계량 3.9461이므로 귀무가설을 기각할 수 있다
```



### 3. 두 집단 검정

- 데이터 전처리 -> 두 집단 subset 생성  -> prop.test() -> 검정통계량 분석

- 단일표본 이항분포 비율검정은 binom.test() 를 이용

- 독립표본 이항분포 비율검정은 prop.test()를 이용

- 비율 차이 검정 통계량을 바탕으로 귀무가설의 기각 여부를 결정

#### 1) 두 집단 비율 차이 검정

- 명목 척도의 비율을 바탕으로 prop.test() 함수를 이용하여 두 집단 간 이항분포의 양측 검정을 통해서 검정 통계량을 구한 후  이를 이용하여 가설을 검정한다.

```R
prop.test(x, n, p = NULL, alternative = c(“two.sided”, “less”, “greater”),
             conf.level=0.95, correct=TRUE)
```

```R
############## 두 집단 비율 차이 검정 실습 ################
# IT 교육센터에서 PT를 이용한 프리젠테이션 교육방법과 실시간 코딩 교육방법을 적용하여 교육을 실시하였다. 2시간 교육방법 중 더 효과적인 교육방법을 조사하기 위해서 교육생 300명을 대상으로 설문을 실시하였다. 
# 연구가설(H1) : 두 가지 교육방법에 따라 교육생의 만족률에 차이가 있다.
# 귀무가설(H0) : 두 가지 교육방법에 따라 교육생의 만족률에 차이가 없다.

> data <- read.csv("./data/two_sample.csv", header=TRUE)
> head(data)
  no gender method survey score
1  1      1      1      1   5.1
2  2      1      1      0   5.2
3  3      1      1      1   4.7
4  4      2      1      0   4.8
5  5      1      1      1   5.0
6  6      1      1      1   5.4

> str(data) # method교육방법 , 관측치
'data.frame':	300 obs. of  5 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ gender: int  1 1 1 2 1 1 2 1 1 1 ...
 $ method: int  1 1 1 1 1 1 1 1 1 1 ...
 $ survey: int  1 0 1 0 1 1 0 1 1 0 ...
 $ score : num  5.1 5.2 4.7 4.8 5 5.4 NA 5 4.4 4.9 ...

> x <- data$method
> y <- data$survey

> length(x)   # NA 없음
[1] 300
> length(y)   # NA 없음
[1] 300

> table(x)
x
  1   2 
150 150 

> table(y)
y
  0   1 
 55 245 

# 교차 분석을 위한 분할표 생성
> table(x, y, useNA="ifany")   # 결측치까지 출력(useNA="ifany")
   y
x     0   1
  1  40 110
  2  15 135

# 두 집단의 만족도 차이 비율 검정
# prop.test ('pt교육 만족도와 코딩교육 만족도', '교육방법에 대한 변량-시행횟수')
> prop.test(c(110, 135), c(150, 150))
	2-sample test for equality of proportions with continuity
	correction
data:  c(110, 135) out of c(150, 150)
X-squared = 12.824, df = 1, p-value = 0.0003422
alternative hypothesis: two.sided
95 percent confidence interval:
 -0.25884941 -0.07448392
sample estimates:
   prop 1    prop 2 
0.7333333 0.9000000 


> prop.test(c(110, 135), c(150, 150), alter="two.sided", conf.level=0.95)
	2-sample test for equality of proportions with continuity
	correction
data:  c(110, 135) out of c(150, 150)
X-squared = 12.824, df = 1, p-value = 0.0003422
alternative hypothesis: two.sided
95 percent confidence interval:
 -0.25884941 -0.07448392
sample estimates:
   prop 1    prop 2 
0.7333333 0.9000000 

# 해석 : 검정 통계량 p-value 값은 0.0003422로 유의수준 0.05보다 작기 때문에 두 교육방법 간의 만족도에 차이가 있다. (연구가설 채택) 검정 통계량 X-squared로 가설 검정을 수행하면 df 1일때 기각역은 3.84이고 X-squared 12.82..가 더 크기 때문에 귀무가설을 기각할 수 있다.
```



#### 2) 두 집단 평균 검정(독립표본 T 검정)

- 두 집단을 대상으로 평균 차이 검정을 통해서 두 집단의 평균이 같은지 또는 다른지를 검정

- 데이터 전처리 -> 두집단 subset 작성 -> 기술 통계량(평균)  ->  동질성분포 (var.test()) -> t.test() 또는 wilcox.text() -> 검정통계량 분석
- 동질성 검정
  - 모집단에서 추출된 표본을 대사으로 분산의 동질성 검정은 stats 패키지에서 제공하는 var.test() 함수를 이용할 수 있다.
  - 검정 결과가 유의수준 0.05보다 큰 경우 두 집단 간 분포의 모양이 동질하다고 할 수 있다.
  - 분산의 동질성 검정은 등분산 가정과 등분산 가정되지 않음(이분산)에 따라서 결과가 달라진다.
  - 등분산은 모집단에서 추출된 표본이 균등하게 추출된 경우이고, 이분산은 추출된 표본이 특정 계층으로 편중되어 추출되는 경우이다.

```R
################# 두 집단 평균 검정(독립표본 T검정)##################
# IT 교육센터에서 PT를 이용한 프리젠테이션 교육방법과 실시간 코딩 교육방법을 적용하여 1개월동안 교육받은 교육생 각 150명을 대상으로 실기시험을  실시하였다. 집단간 실기시험의 평균에 차이가 있는가를 검정한다.
# 연구가설(H1) : 교육방법에 따른 두 집단 간 실기시험의 평균에 차이가 있다.
# 귀무가설(H0) : 교육방법에 따른 두 집단 간 실기시험의 평균에 차이가 없다.

> data <- read.csv("./data/two_sample.csv", header=TRUE)
> head(data)
  no gender method survey score
1  1      1      1      1   5.1
2  2      1      1      0   5.2
3  3      1      1      1   4.7
4  4      2      1      0   4.8
5  5      1      1      1   5.0
6  6      1      1      1   5.4

> str(data) #method교육방법 , score(점수)
'data.frame':	300 obs. of  5 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ gender: int  1 1 1 2 1 1 2 1 1 1 ...
 $ method: int  1 1 1 1 1 1 1 1 1 1 ...
 $ survey: int  1 0 1 0 1 1 0 1 1 0 ...
 $ score : num  5.1 5.2 4.7 4.8 5 5.4 NA 5 4.4 4.9 ...

> summary(data)  #score의 NA 개수 : 73개
       no             gender         method        survey           score      
 Min.   :  1.00   Min.   :1.00   Min.   :1.0   Min.   :0.0000   Min.   :3.000  
 1st Qu.: 75.75   1st Qu.:1.00   1st Qu.:1.0   1st Qu.:1.0000   1st Qu.:5.100  
 Median :150.50   Median :1.00   Median :1.5   Median :1.0000   Median :5.600  
 Mean   :150.50   Mean   :1.42   Mean   :1.5   Mean   :0.8167   Mean   :5.685  
 3rd Qu.:225.25   3rd Qu.:2.00   3rd Qu.:2.0   3rd Qu.:1.0000   3rd Qu.:6.300  
 Max.   :300.00   Max.   :2.00   Max.   :2.0   Max.   :1.0000   Max.   :8.000  
                                                                NAs   :73   


# 두 집단 subset 생성 
> result <- subset(data, !is.na(score), c(method, score)) # 조건과 select할 변수
> head(result)
  method score
1      1   5.1
2      1   5.2
3      1   4.7
4      1   4.8
5      1   5.0
6      1   5.4

> length(result)
[1] 2

> a <- subset(result, method==1)
> b <- subset(result, method==2)
> ascore <- a$score
> bscore <- b$score


# 기술통계량(평균)
> length(ascore) 
[1] 109
> length(bscore)  
[1] 118
> mean(ascore)    
[1] 5.556881
> mean(bscore)   
[1] 5.80339

# 분산의 동질성 검정 
> var.test(ascore, bscore)
	F test to compare two variances
data:  ascore and bscore
F = 1.2158, num df = 108, denom df = 117, p-value = 0.3002
alternative hypothesis: true ratio of variances is not equal to 1
95 percent confidence interval:
 0.8394729 1.7656728
sample estimates:
ratio of variances 
          1.215768 

# 해석 : 검정 통계량 p-value 값은 0.3002 로 유의수준 0.05보다 크기 때문에 두 집단 간의 분포 형태가 동일하다고 볼 수 있다.


# 두 집단 평균 차이 검정
> t.test(ascore, bscore)
	Welch Two Sample t-test
data:  ascore and bscore
t = -2.0547, df = 218.19, p-value = 0.0411
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -0.48296687 -0.01005133
sample estimates:
mean of x mean of y 
 5.556881  5.803390 

> t.test(ascore, bscore, alter="two.sided", conf.int=TRUE, conf.level=0.95)
	Welch Two Sample t-test
data:  ascore and bscore
t = -2.0547, df = 218.19, p-value = 0.0411
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -0.48296687 -0.01005133
sample estimates:
mean of x mean of y 
 5.556881  5.803390 

# 해석 : 검정 통계량 p-value 값은 0.0411로 유의수준 0.05보다 작기 때문에 두 집단간의 평균에 차이가 있다.


# 단측 가설 검정 (ascore가 기준으로 비교 -> ascore보다 bscore가 더 큰지 여부)
> t.test(ascore, bscore, alter="greater", conf.int=TRUE, conf.level=0.95)
	Welch Two Sample t-test
data:  ascore and bscore
t = -2.0547, df = 218.19, p-value = 0.9794
alternative hypothesis: true difference in means is greater than 0
95 percent confidence interval:
 -0.4446915        Inf
sample estimates:
mean of x mean of y 
 5.556881  5.803390 
# 해석 : 검정 통계량 p-value 값은 0.9794로 유의수준 0.05보다 크기 때문에 pt교육보다 코딩교육의 실기 점수 평균이 더 크다.


> t.test(ascore, bscore, alter="less", conf.int=TRUE, conf.level=0.95)
	Welch Two Sample t-test
data:  ascore and bscore
t = -2.0547, df = 218.19, p-value = 0.02055
alternative hypothesis: true difference in means is less than 0
95 percent confidence interval:
        -Inf -0.04832672
sample estimates:
mean of x mean of y 
 5.556881  5.803390 
# 해석 : 검정 통계량 p-value 값은 0.02055로 유의수준 0.05보다 작기 때문에 pt교육이 코딩교육의 실기 점수 평균이 더 낮다.
```



#### 3) 대응 두 집단 평균 검정(대응 표본 T 검정)

- 대응 표본 평균검정(Paired Samples t-test)은 동일한 표본을 대상으로 측정된 두 변수의 평균 차이를 검정하는  분석 방법

- 일반적으로  사전검사와 사후 검사의 평균 차이를 검증할 때 많이 이용
  - 예) 교수법 프로그램을 적용하기 전 학생들의 학습력과 교수법 프로그램 적용한 후 학생들의 학습력에 차이가 있는지를 검정

- 데이터 전처리 -> 전과 후  subset 작성 -> 기술 통계량(평균)  ->  동질성분포 (var.test()) -> t.test() 또는 wilcox.text() -> 검정통계량 분석

```R
###############대응 두 집단 평균 검정(대응 표본 T검정)################
# A 교육센터에서 교육생 100명을 대상으로 교수법 프로그램 적용 전에 실기시험을 실시한 후 1개월 동안 동일한 교육생에게 교수법 프로그램을 적용한 후 실기시험을 실시한 점수와 평균에 차이가 있는지를 검정한다.
# 연구가설(H1) : 교수법 프로그램을 적용하기 전 학생들의 학습력과 교수법 프로그램 적용한 후 학생들의 학습력에 차이가 있다.
# 귀무가설(H0) : 교수법 프로그램을 적용하기 전 학생들의 학습력과 교수법 프로그램 적용한 후 학생들의 학습력에 차이가 없다.

> data <- read.csv("./data/paired_sample.csv", header=TRUE)
> head(data)
  no before after
1  1    5.1   6.3
2  2    5.2   6.3
3  3    4.7   6.5
4  4    4.8   5.9
5  5    5.0   6.5
6  6    5.4   7.3

> str(data)  #before, after 컬럼, 관측치 100개
'data.frame':	100 obs. of  3 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ before: num  5.1 5.2 4.7 4.8 5 5.4 5 5 4.4 4.9 ...
 $ after : num  6.3 6.3 6.5 5.9 6.5 7.3 5.9 6.2 6 7.2 ...

> result <- subset(data, !is.na(after), c(before, after))
> length(result$after)   # 결측치 4개 제외됨
[1] 96

> x<-result$before
> y<-result$after

> mean(x) 
[1] 5.16875
> mean(y)  
[1] 6.220833


# 동질성 검정
> var.test(x, y, paired=TRUE)
	F test to compare two variances
data:  x and y
F = 1.0718, num df = 95, denom df = 95, p-value = 0.7361
alternative hypothesis: true ratio of variances is not equal to 1
95 percent confidence interval:
 0.7151477 1.6062992
sample estimates:
ratio of variances 
          1.071793 
# 해석 : 검정 통계량 p-value 값은  0.7361로 유의수준 0.05보다 크기 때문에 두 집단간의 분포형태가 동일하다.


# 평균 차이 검정
> t.test(x, y, paired=TRUE, alter="two.sided", conf.int=TRUE, conf.level=0.95)
	Paired t-test
data:  x and y
t = -13.642, df = 95, p-value < 2.2e-16
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -1.205184 -0.898983
sample estimates:
mean of the differences 
              -1.052083 
# 해석 : 검정 통계량 p-value 값은  2.2e-16 로 유의수준 0.05보다 매우 작기 때문에 두 집단간의 평균에 차이가 있다.


> t.test(x, y, paired=TRUE, alter="less", conf.int=TRUE, conf.level=0.95)
	Paired t-test
data:  x and y
t = -13.642, df = 95, p-value < 2.2e-16
alternative hypothesis: true difference in means is less than 0
95 percent confidence interval:
       -Inf -0.9239849
sample estimates:
mean of the differences 
              -1.052083 
# 해석 : 검정 통계량 p-value 값은 2.2e-16 로 유의수준 0.05보다 매우 작기 때문에 x 집단 평균이 y집단의 평균보다 작다고 할 수 있다.
```



### 4. 세 집단 검정

- 세 집단 검정 
  - 비율 차이 검정은 기술 통계량으로 빈도수에 대한 비율에 의미가 있으며, 세 집단의 평균 차이 검정은 분산 분석이라고 한다.

#### 1) 세 집단 비율검정

- 데이터 전처리 -> 세 집단  subset 작성 -> prop.test()  -> 검정통계량 분석

```R
# IT 교육센터에서 3가지 교육방법을 적용하여 교육을 실시하였다. 3가지 교육방법 중 더 효과적인 교육방법을 조사하기 위해서 교육생 150명을 대상으로 설문을 실시하였다. 
# 연구가설(H1) : 세 가지 교육방법에 따른 집단 간 만족률에 차이가 있다.
# 귀무가설(H0) : 세 가지 교육방법에 따른 집단 간 만족률에 차이가 없다.

> data <- read.csv("./data/three_sample.csv", header=TRUE)
> head(data)
  no method survey score
1  1      1      1   3.2
2  2      2      0    NA
3  3      3      1   4.7
4  4      1      0    NA
5  5      2      1   7.8
6  6      3      1   5.4

> str(data) # method(교육방법: 1,2,3), 관측치 150
'data.frame':	150 obs. of  4 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ method: int  1 2 3 1 2 3 1 2 3 1 ...
 $ survey: int  1 0 1 0 1 1 0 0 1 0 ...
 $ score : num  3.2 NA 4.7 NA 7.8 5.4 NA 8.4 4.4 2.8 ...
 
> method <- data$method
> survey <- data$survey

> table(method, useNA="ifany")
method
 1  2  3 
50 50 50 

> table(method, survey, useNA="ifany") # 교차분할표표
      survey
method  0  1
     1 16 34
     2 13 37
     3 11 39

# prop.test((교육방법에 대한 만족 빈도수), (변량의 길이))
> prop.test(c(34, 37, 39), c(50, 50, 50), alter="two.sided", conf.level=0.95)

	3-sample test for equality of proportions without
	continuity correction

data:  c(34, 37, 39) out of c(50, 50, 50)
X-squared = 1.2955, df = 2, p-value = 0.5232
alternative hypothesis: two.sided
sample estimates:
prop 1 prop 2 prop 3 
  0.68   0.74   0.78 
# 해석 : 검정 통계량 p-value의 값은 0.5232로 유의수준 0.05보다 크므로 세 교육방법 간의 만족도에 차이가 없다. (귀무가설 채택) X-squared 검정 통계량 1.2955는 df 2의 기각값은 5.991보다 작기 때문에 귀무가설을 기각할 수 없다. 만족도 비율 68%, 74%, 78%
```



#### 2) 분산 분석 (F 검정)

> 분산분석(ANOVA  Analysis)-  T 검정과 동일하게 평균에 의한 차이 검정 방법

- 두 집단 이상의 평균 차이를 검정
  - 예) 의학연구 분야에서 개발된 3가지 치료제가 있다고 가정할 때, 이 3가지 치료제의 효과에 차이가 있는지를 검정

- 분산분석은 가설 검정을 위해 F 분포를 따른 F 통계량을  검정 통계량으로 사용하기 때문에 F검정이라고 한다.

- 데이터 전처리 -> 각 집단 subset 작성 -> 기술 통계량(평균)  ->  동질성분포 (barlett.test()) -> aov() 또는 kruskal.test() -> TukeyHSD()
- 분산분석에서 집단 간의 동질성 여부를 검정하기 위해서는 bartlett.test()를 이용
- 집단 간의 분포가 동질한 경우 분산분석을 수행하는  aov() 함수를 이용
- 비모수 검정 방법인 kruskal.test() 를 이용하여 분석을 수행하고, 마지막으로 TukeyHSD() 함수를 이용하여 사후 검정을 수행한다

```R
#### 세 집단의 평균 차이 분석 ####
# 연구가설(H1) : 교육방법에 따른 세 집단 간 실기시험의 평균에 차이가 있다.
# 귀무가설(H0) : 교육방법에 따른 세 집단 간 실기시험의 평균에 차이가 없다.
# 세 가지 교육방법을 적용하여 1개월 동안 교육받은 교육생 각 50명씩을 대상으로 실기시험을 실시하였다. 세 집단 간 실기시험의 평균에 차이가 있는지를 검정한다.

> data <- read.csv("./data/three_sample.csv", header=TRUE)
> head(data)
  no method survey score
1  1      1      1   3.2
2  2      2      0    NA
3  3      3      1   4.7
4  4      1      0    NA
5  5      2      1   7.8
6  6      3      1   5.4
> str(data) # method, score
'data.frame':	150 obs. of  4 variables:
 $ no    : int  1 2 3 4 5 6 7 8 9 10 ...
 $ method: int  1 2 3 1 2 3 1 2 3 1 ...
 $ survey: int  1 0 1 0 1 1 0 0 1 0 ...
 $ score : num  3.2 NA 4.7 NA 7.8 5.4 NA 8.4 4.4 2.8 ...


> data <- subset(data, !is.na(score), c(method, score)) # 결측치 제거
> head(data)
  method score
1      1   3.2
3      3   4.7
5      2   7.8
6      3   5.4
8      2   8.4
9      3   4.4

# 이상치 존재 확인(데이터 분포 현황 분석)
> plot(data$score) # 산점도
> barplot(data$score)
> mean(data$score)  # 돌출된 애들이 이상치임
[1] 14.44725
```

![6](190919%20R%20Statistics.assets/6.JPG)

![7](190919%20R%20Statistics.assets/7.JPG)

```R
> length(data$score)
[1] 91  # 이상치 제거 전 데이터 개수

> data2 <- subset(data, score<=14)  # 평균을 기준으로 이상치 제거
> length(data2$score)
[1] 88  # 이상치 제거 후 데이터 개수

> x <- data2$score
> boxplot(x) # 정제가 끝난 데이터를 박스플롯으로 확인
```

![8](190919%20R%20Statistics.assets/8.JPG)

```R
# 교육방법에 따른 세 집단의 subset 생성
> data2$method2[data2$method==1] <- "방법1"
> data2$method2[data2$method==2] <- "방법2"
> data2$method2[data2$method==3] <- "방법3"

# 교육방법에 따른 빈도수
> table(data2$method2)
방법1 방법2 방법3 
   31    27    30 
> mCnt <- table(data2$method2)
> mAvg <- tapply(data2$score, data2$method2, mean)

# 데이터 프레임 생성
> df <- data.frame(교육방법=mCnt, 성적=mAvg)
> df
      교육방법.Var1 교육방법.Freq     성적
방법1         방법1            31 4.187097
방법2         방법2            27 6.800000
방법3         방법3            30 5.610000

# 세 집단의 동질성 검정
# bartlett.test(종속변수 ~ 독립변수, data=데이터셋)
> bartlett.test(score ~ method, data=data2)

	Bartlett test of homogeneity of variances

data:  score by method
Bartletts K-squared = 3.3157, df = 2, p-value = 0.1905

#### 해석 : 검정 통계량 p-value의 값은 0.1905로 유의수준 0.05보다 크므로 세 집단간 분포 형태가 동질적이라고 볼 수 있다.



# 세 집단의 평균 차이 검정 : (분포 형태가 동질적이기 때문에) aov() 함수 사용
# aov(종속변수~독립변수, data = 데이터셋)
> result <- aov(score~method2, data=data2)
> names(result) # 검정결과에 해당하는 측정변수명 확인
 [1] "coefficients"  "residuals"     "effects"       "rank"          "fitted.values"
 [6] "assign"        "qr"            "df.residual"   "contrasts"     "xlevels"      
[11] "call"          "terms"         "model"

> summary(result)
            Df Sum Sq Mean Sq F value   Pr(>F)    
method2      2  99.37   49.68   43.58 9.39e-14 ***
Residuals   85  96.90    1.14                     
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

#### 해석 : 검정 통계량 p-value의 값은 9.39e-14로 유의수준 0.05보다 매우 작기 떄문에 세 집단간 실기시험 평균에 차이가 있다고 볼 수 있다. (연구가설 채택)
# F 검정 통계량 43.58이 -1.96 ~ +1.96 범위인 귀무가설의 채택역에 해당되지 않으므로, 귀무가설을 기각할 수 있다. (연구가설 채택)


# 세 집단의 평균의 차에 대한 비교 => 사후검정 수행
> TukeyHSD(result) # 분산분석 결과로 사후 검정
  Tukey multiple comparisons of means
    95% family-wise confidence level

Fit: aov(formula = score ~ method2, data = data2)

$method2
                 diff        lwr        upr     p adj
방법2-방법1  2.612903  1.9424342  3.2833723 0.0000000
방법3-방법1  1.422903  0.7705979  2.0752085 0.0000040
방법3-방법2 -1.190000 -1.8656509 -0.5143491 0.0001911

# diff 폭(평균의 차이)의 크기
> plot(TukeyHSD(result))
# lwr, upr은 신뢰구간의 하한값과 상한값
# 방법2와 방법1의 차이가 가장 크다.

# 결론 : 교육방법에 따른 세 집단 간의 실기시험 평균에 차이가 있으며, 세 교육방법에 따른 분석 결과 방법2와 방법1의 차이가 가장 크다.
```

![9](190919%20R%20Statistics.assets/9.JPG)





### 5. 연습문제

#### 1) 연습문제 1

```R
# Q> 중소기업에서 생산한 HDTV 판매율을 높이기 위해서 프로모션을 진행한 결과 기존 구매비율 보다 15% 향상되었는지를 각 단계별로 분석을 수행하여 검정하시오.
# 연구가설(H1) : 기존 구매비율과 차이가 있다.
# 귀무가설(H0) : 기존 구매비율과 차이가 없다.
# 조건) 구매여부 변수 : buy (1: 구매하지 않음, 2: 구매)

### 단계 1: 데이터 불러오기
> hdtv <- read.csv("./data/hdtv.csv", header=TRUE)

### 단계2: 빈도수와 비율 계산
> summary(hdtv)
    user.id           buy     
 Min.   : 1.00   Min.   :1.0  
 1st Qu.:13.25   1st Qu.:1.0  
 Median :25.50   Median :1.0  
 Mean   :25.50   Mean   :1.2  
 3rd Qu.:37.75   3rd Qu.:1.0  
 Max.   :50.00   Max.   :2.0  
> length(hdtv$buy) # 50개
[1] 50

> install.packages('prettyR')
> library(prettyR) # freq() 함수 사용

> freq(hdtv$buy) # 1:40, 2:10
Frequencies for hdtv$buy 
        1    2   NA
       40   10    0
%      80   20    0 
%!NA   80   20 

> table(hdtv$buy)
 1  2 
40 10 

> table(hdtv$buy, useNA="ifany") # NA 빈도수 표시
 1  2 
40 10 


### 단계3: 가설검정
> binom.test(c(10,40), p=0.15) #15% 비교 -> p-value = 0.321
	Exact binomial test
data:  c(10, 40)
number of successes = 10, number of trials = 50, p-value = 0.321
alternative hypothesis: true probability of success is not equal to 0.15
95 percent confidence interval:
 0.1003022 0.3371831
sample estimates:
probability of success 
                   0.2 

> binom.test(c(10,40), p=0.15, alternative="two.sided", conf.level=0.95)
	Exact binomial test
data:  c(10, 40)
number of successes = 10, number of trials = 50, p-value = 0.321
alternative hypothesis: true probability of success is not equal to 0.15
95 percent confidence interval:
 0.1003022 0.3371831
sample estimates:
probability of success 
                   0.2 
# 해설: 귀무가설 채택 => 기존 구매비율(15%)과 차이가 없다.


# 방향성이 있는 단측가설 검정
> binom.test(c(10,40), p=0.15, alternative="greater", conf.level=0.95)
	Exact binomial test
data:  c(10, 40)
number of successes = 10, number of trials = 50, p-value = 0.2089
alternative hypothesis: true probability of success is greater than 0.15
95 percent confidence interval:
 0.1127216 1.0000000
sample estimates:
probability of success 
                   0.2 
# p-value=0.2089

> binom.test(c(10,40), p=0.15, alternative="less", conf.level=0.95) 
	Exact binomial test
data:  c(10, 40)
number of successes = 10, number of trials = 50, p-value = 0.8801
alternative hypothesis: true probability of success is less than 0.15
95 percent confidence interval:
 0.0000000 0.3155961
sample estimates:
probability of success 
                   0.2 
# p-value =0.8801
# 해설 : 방향성이 있는 단측가설은 모두 기각된다.


# 11% 기준 : 방향성이 있는 연구가설 검정
> binom.test(c(10,40), p=0.11, alternative="greater", conf.level=0.95)
	Exact binomial test
data:  c(10, 40)
number of successes = 10, number of trials = 50, p-value =
0.04345
alternative hypothesis: true probability of success is greater than 0.11
95 percent confidence interval:
 0.1127216 1.0000000
sample estimates:
probability of success 
                   0.2 
# p-value=0.04345
# 해설 : 구매비율은 11%을 넘지 못한다.
```



#### 2) 연습문제 2

```R
# Q> 우리나라 전체 중학교 2학년 여학생 평균 키가 148.5cm로 알려져 있는 상태에서 A중학교 2학년 전체 500명을 대상으로 10%인 50명을 표본으로 선정하여 표본평균신장을 계산하고 모집단의 평균과 차이가 있는지를 각 단계별로 분석을 수행하여 검정하시오.
# 연구가설 : 모집단의 평균 신장과 표본평균신장은 차이가 있다.
# 귀무가설 : 모집단의 평균 시장과 표본평균신장은 차이가 없다.

> height <- read.csv("./data/student_height.csv", header=TRUE)
> head(height)
  sudent.id height
1         1    148
2         2    150
3         3    149
4         4    144
5         5    152
6         6    150

### 단계1 : 기술 통계량 평균 계산
> x <- height$height
> summary(x)
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  140.0   147.0   150.0   149.4   151.0   165.0 
# 평균 : 149.4 => 결측치가 없으므로 그대로 사용!


### 단계2 : 정규성 검정
> shapiro.test(x)
	Shapiro-Wilk normality test
data:  x
W = 0.88711, p-value = 0.0001853
# 해석 : 검정 통계량 p-value 값은 0.0001853으로 유의수준 0.05보다 작기 때문에 x객체의 데이터 분포는 정규분포를 따르지 않는다고 할 수 있다.


### 단계3 : 가설 검정
> t.test(x, mu=148.5)
	One Sample t-test
data:  x
t = 1.577, df = 49, p-value = 0.1212
alternative hypothesis: true mean is not equal to 148.5
95 percent confidence interval:
 148.2531 150.5469
sample estimates:
mean of x 
    149.4 

> t.test(x, mu=148.5, alter="two.side", conf.level=0.95)
	One Sample t-test
data:  x
t = 1.577, df = 49, p-value = 0.1212
alternative hypothesis: true mean is not equal to 148.5
95 percent confidence interval:
 148.2531 150.5469
sample estimates:
mean of x 
    149.4 

# 해석1 : 검정 통계량 p-value 값은 0.1212 로 유의수준 0.05보다 크기 때문에 귀무가설 채택
# 해석2 : x의 평균은 149.4(점추정)는 신뢰구간에 포함되고 모집단의 평균신장 148.5도 신뢰구간에 포함되므로 귀무가설이 채택된다.
```



#### 3) 연습문제 3

```R
# Q> 대학에 진학한 남학생과 여학생을 대상으로 진학한 대학에 대해서 만족도에 차이가 있는가를 검정하시오. (두 집단 비율 차이 검정)
# 연구가설 : 성별에 따라 진학한 대학 만족도에 차이가 있다.
# 귀무가설 : 성별에 따라 진학한 대학 만족도에 차이가 없다.
> sample <- read.csv("./data/two_sample.csv")
> head(sample)
  no gender method survey score
1  1      1      1      1   5.1
2  2      1      1      0   5.2
3  3      1      1      1   4.7
4  4      2      1      0   4.8
5  5      1      1      1   5.0
6  6      1      1      1   5.4
 
> x <- sample$gender
> y <- sample$survey
> length(x) # NA 없음
[1] 300
> length(y) # NA 없음
[1] 300

> table(x)
x
  1   2 
174 126 
> table(y)
y
  0   1 
 55 245 

# 교차분석을 위한 교차분할표 생성
> table(x, y, useNA="ifany")
   y
x     0   1
  1  36 138
  2  19 107

# 두 집단의 만족도 차이 비율 검정
> prop.test(c(138, 107), c(174, 126))
	2-sample test for equality of proportions with continuity
	correction
data:  c(110, 135) out of c(150, 150)
X-squared = 12.824, df = 1, p-value = 0.0003422
alternative hypothesis: two.sided
95 percent confidence interval:
 -0.25884941 -0.07448392
sample estimates:
   prop 1    prop 2 
0.7333333 0.9000000 


> prop.test(c(138, 107), c(174, 126))
	2-sample test for equality of proportions with continuity
	correction
data:  c(138, 107) out of c(174, 126)
X-squared = 1.1845, df = 1, p-value = 0.2765
alternative hypothesis: two.sided
95 percent confidence interval:
 -0.14970179  0.03749599
sample estimates:
   prop 1    prop 2 
0.7931034 0.8492063 

> prop.test(c(138, 107), c(174, 126), alter="two.sided", conf.level=0.95)
	2-sample test for equality of proportions with continuity
	correction
data:  c(138, 107) out of c(174, 126)
X-squared = 1.1845, df = 1, p-value = 0.2765
alternative hypothesis: two.sided
95 percent confidence interval:
 -0.14970179  0.03749599
sample estimates:
   prop 1    prop 2 
0.7931034 0.8492063 
# 해석 : 검정 통계량 p-value 값은 0.2765로 유의수준 0.05보다 크기 때문에 성별에 따른 대학진학 만족도에 차이가 없다. (귀무가설 채택) 검정 통계량 X-squared로 가설 검정을 수행하면 df가 1일때 기각역은 3.84이고 X-squared 1.1845가 더 작기 때문에 귀무가설을 기각할 수 없다.
```



#### 4) 연습문제 4

```R
# Q> 교육방법에 따라 시험성적에 차이가 있는지 검정하시오.(두 집단 평균 차이 검정)

> data <- read.csv("./data/twomethod.csv")
> head(data)
  id method score
1  1      1    27
2  2      1     5
3  3      1    21
4  4      1    NA
5  5      1    14
6  6      1    23

> summary(data) # 결측치 6개
       id           method          score      
 Min.   : 1.0   Min.   :1.000   Min.   : 5.00  
 1st Qu.:16.5   1st Qu.:1.000   1st Qu.:18.00  
 Median :32.0   Median :2.000   Median :26.00  
 Mean   :32.0   Mean   :1.619   Mean   :24.28  
 3rd Qu.:47.5   3rd Qu.:2.000   3rd Qu.:31.00  
 Max.   :63.0   Max.   :2.000   Max.   :45.00  
                                NAs   :6  


# 결측치 제거
> result <- subset(data, !is.na(score), c(method, score))
> head(result)
  method score
1      1    27
2      1     5
3      1    21
5      1    14
6      1    23
7      1    20

> a <- subset(result, method==1)
> b <- subset(result, method==2)
> ascore <- a$score
> bscore <- b$score


# 기술통계량(평균)
> length(ascore)
[1] 22
> length(bscore)
[1] 35
> mean(ascore)
[1] 16.40909
> mean(bscore)
[1] 29.22857

# 분산의 동질성 검정 

> var.test(ascore, bscore)

	F test to compare two variances

data:  ascore and bscore
F = 1.0648, num df = 21, denom df = 34, p-value = 0.8494
alternative hypothesis: true ratio of variances is not equal to 1
95 percent confidence interval:
 0.502791 2.427170
sample estimates:
ratio of variances 
           1.06479 
# 해석 : 검정 통계량 p-value 값은 0.8494 로 유의수준 0.05보다 매우 크기 때문에 두 집단 간의 분포 형태가 동일하다고 볼 수 있다.


# 두 집단 평균 차이 검정
> t.test(ascore, bscore)

	Welch Two Sample t-test

data:  ascore and bscore
t = -5.6056, df = 43.705, p-value = 1.303e-06
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -17.429294  -8.209667
sample estimates:
mean of x mean of y 
 16.40909  29.22857 

> t.test(ascore, bscore, alter="two.sided", conf.int=TRUE, conf.level=0.95)

	Welch Two Sample t-test

data:  ascore and bscore
t = -5.6056, df = 43.705, p-value = 1.303e-06  # 0.000001303
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -17.429294  -8.209667
sample estimates:
mean of x mean of y 
 16.40909  29.22857 
# 해석 : 검정 통계량 p-value 값은 1.303e-06으로 유의수준 0.05보다 매우 작기 때문에 두 집단간의 평균에 차이가 크다.


# 단측 가설 검정 (ascore를 기준으로 비교 -> ascore보다 bscore가 더 큰지 여부)
> t.test(ascore, bscore, alter="greater", conf.int=TRUE, conf.level=0.95)

	Welch Two Sample t-test

data:  ascore and bscore
t = -5.6056, df = 43.705, p-value = 1
alternative hypothesis: true difference in means is greater than 0
95 percent confidence interval:
 -16.66255       Inf
sample estimates:
mean of x mean of y 
 16.40909  29.22857 
# 해석 : 검정 통계량 p-value 값은 1로 유의수준 0.05보다 훨씬 크기 때문에 방법1보다 방법2인 경우 시험성적 평균이 더 크다.


> t.test(ascore, bscore, alter="less", conf.int=TRUE, conf.level=0.95)

	Welch Two Sample t-test

data:  ascore and bscore
t = -5.6056, df = 43.705, p-value = 6.513e-07
alternative hypothesis: true difference in means is less than 0
95 percent confidence interval:
      -Inf -8.976413
sample estimates:
mean of x mean of y 
 16.40909  29.22857 
# 해석 : 검정 통계량 p-value 값은 6.513e-07로 유의수준 0.05보다 매우 작기 때문에 방법1인 경우 방법2인 경우보다 시험 성적 평균이 더 낮다.
```



## III. 요인분석, 상관관계 분석

- 요인 분석  : 변수들의 상관성을 바탕으로 변수를 정제하여 상관관계 분석이나 회귀분석에 설명변수(독립변수)로 사용된다.

- 상관 분석  : 요인분석 과정에서 변수들이 상관관계를 분석하여 변수 간의 관련성을 분석하는데 이용

- 회귀 분석  : 인과관계를 분석하는데 중요한 자료를 제공



### 1. 요인 분석 (Factor Analysis)

- 다수의 변수를 대상으로 변수간의 관계를 분석하여 공통차원으로 축약하는 통계기법

- 데이터를 축소하는 변수의 정제 과정

- 여러 가지 항목들을 비슷한 항목으로 묶는 것으로 여러 변수 사이에 존재하는 상호관계를 분석하여 타당성을 검정하고, 공통으로 속해있는 차원이나 요인들을 밝혀냄으로써 변수를 축소하는 과정

#### 1) 요인분석 방법

- 탐색적 요인분석 – 요인분석을 할 때 사전에 어던 변수끼리 묶어야 한다는 전제를 두지 않고 분석하는 방법

- 확인적 요인 분석 – 사전에 묶일 것으로 기대되는 항목끼리 묶여 지는지를 조사하는 방법  

#### 2) 타당성  

- 측정 도구가 측정하고자 하는 것을 정확히 측정할 수 있는 정도를 의미

- 통계량 검정 이전에 구성 타당성(Construct validity)  검증을 위해서 요인분석을 실시한다

#### 3) 요인 분석 전제조건 

- 하위요인으로 구성되는 데이터 셋이 준비되어 있어야 한다

- 분석에 사용되는 변수는 등간척도나 비율척도이어야 하며, 표본의 크기는 최소 50개 이상이 바람직하다

- 요인분석은 상관관계가 높은 변수들까리 그룹화하는 것이므로 변수 간의 상관관계가 매우 낮다면 (보통 ±3 이하) 그  자료는 요인분석에 적합하지 않다

#### 4) 요인 분석 (Factor Analysis) 목적

- 자료의 요약 : 변인을 몇 개의 공통된 변인으로 묶음

- 변인 구조 파악 : 변인들의 상호관계 파악(독립성등)

- 불필요한 변인 제거 : 중요도가 떨어진 변수 제거

- 측정 도구 타당성 검증 : 변인들이 동일한 요인으로 묶이는지 확인  

#### 5) 요인 분석에 대한 활용 방안 

- 측정 도구가 정확히 측정했는지를 알아보기 위해서 측정 변수들이 동일한 요인으로 묶이는지를 검정한다(타당성 검정)

- 변수들의 상관관계가 높은 것끼리 묶어서 변수를 정제한다. (변수 축소)
- 변수의 중요도를 나타내는 요인적재량이 0.4 미만이면 설명력이 부족한 요인으로 판단하여 제거한다 (변수 제거)

- 요인분석에서 얻어지는 결과를 이용하여 상관분석이나 회귀분석의 설명변수로 활용한다.

#### 6) 공통요인으로 변수 정제

- 특정 항목으로 묶여 지는데 사용되는 요인수 결정은 주성분 분석방법과 상관계수 행렬을 이용한 초기 고유값을 이용한다.

- 단계1] 주성분 분석 : 변동량(분산)에 영향을 주는 주요 성분을 분석하는 방법으로 요인분석에서 사용될 요인의 개수를 결정하는데 주로 이용된다.

- 단계2] 고유값으로 요인수 분석 : 고유값이란 어떤 행렬로부터 유도되는 실수값을 의미한다. 일반적으로 변화량의 (총분산)을 기준으로 요인수를 결정하는데 이용된다. 
  - 상관계수 행렬을 대상으로 초기 고유값으로 요인수 분석
  - eigen()은 상관계수 행렬을 대상으로 초기 고유값과 고유벡터를 계산하는 함수이다.	

```R
########## 공통요인으로 변수를 정제하는 요인분석 ##########
# 6개 과목 (s1~s6)
# 점수벡터 (5점 만점, 척도:5)
> s1 <- c(1, 2, 1, 2, 3, 4, 2, 3, 4, 5) # 자연과학
> s2 <- c(1, 3, 1, 2, 3, 4, 2, 4, 3, 4) # 물리화학
> s3 <- c(2, 3, 2, 3, 2, 3, 5, 3, 4, 2) # 인문사회
> s4 <- c(2, 4, 2, 3, 2, 3, 5, 3, 4, 1) # 신문방송
> s5 <- c(4, 5, 4, 5, 2, 1, 5, 2, 4, 3) # 응용수학
> s6 <- c(4, 3, 4, 4, 2, 1, 5, 2, 4, 2) # 추론통계
> name <-1:10  # 각 과목의 문제 이름

# 데이터프레임 생성
> subject <- data.frame(s1, s2, s3, s4, s5, s6)
> str(subject)
'data.frame':	10 obs. of  6 variables:
 $ s1: num  1 2 1 2 3 4 2 3 4 5
 $ s2: num  1 3 1 2 3 4 2 4 3 4
 $ s3: num  2 3 2 3 2 3 5 3 4 2
 $ s4: num  2 4 2 3 2 3 5 3 4 1
 $ s5: num  4 5 4 5 2 1 5 2 4 3
 $ s6: num  4 3 4 4 2 1 5 2 4 2

# 주성분 분석으로 요인수 구하기
> help(prcomp)  #주성분 분석 수행 함수
> pc <- prcomp(subject) # scale = TRUE
> summary(pc) # 변동량으로 볼때 주성분 요인수는 2개로 가정할 수 있다.
Importance of components:
                         PC1    PC2     PC3     PC4     PC5     PC6
Standard deviation     2.389 1.5532 0.87727 0.56907 0.19315 0.12434
Proportion of Variance 0.616 0.2603 0.08305 0.03495 0.00403 0.00167
Cumulative Proportion  0.616 0.8763 0.95936 0.99431 0.99833 1.00000
> plot(pc)
```

![13](190919%20R%20Statistics.assets/13.JPG)

```R
# 고유값으로 요인수 분석
> en <- eigen(cor(subject))  # $values : 고유값, $vectors : 고유벡터
> names(en)
[1] "values"  "vectors"

> en$vectors
           [,1]         [,2]        [,3]       [,4]        [,5]        [,6]
[1,] -0.4062499 -0.351093036  0.63460534  0.3149622  0.45699508  0.03041553
[2,] -0.4319311 -0.400526644  0.11564711 -0.4422216 -0.57042232  0.34452594
[3,]  0.2542077 -0.628807884 -0.06984072  0.3339036 -0.35389906 -0.54622817
[4,]  0.3017115 -0.566028650 -0.37734321 -0.2468016  0.50326085  0.36333366
[5,]  0.4763815  0.008436692  0.58035475 -0.6016209  0.05643527 -0.26654314
[6,]  0.5155637  0.021286661  0.31595023  0.4133867 -0.28995329  0.61559319

> en$values # $values : 고유값(스칼라) 보기 
[1] 3.44393944 1.88761725 0.43123968 0.19932073 0.02624961 0.01163331

> plot(en$values, type="o") # 고유값(스칼라) 보기
# 고유값이 급격하게 감소하는 1~3 변수 3개를 주성분 변수로 가정할 수 있다.
```

![14](190919%20R%20Statistics.assets/14.JPG)



#### 7) 변수간의 상관관계 분석과 요인분석

- 단계1] 상관관계 분석 – 변수 간의 상관성으로 공통요인 추출
- 단계2] 회전법– 요인축을 기본으로 사용한다.

```R
factanal(dataset, factors=요인수, rotation=“요인회전법“ )
scores =“ 요인점수 계산 방법＂
```

- 요인분석결과에서 만약 p-value 값이 0.05 미만이면 요인수가 부족하다는 의미로 요인수를 늘려서 다시 분석을 수행해야 한다.

- Uniqueness 항목은 유효성을 판단하여 제시한 값으로 통상 0.05이하이면 유효한 것으로 본다.
- Loading 항목은 요인 적재값(Loadings)를 보여주는 항목으로 각 변수와 해당 요인 간의 상관관계계수를 제시한다.

- 요인 적재값(요인 부하량)이 통상 +0.4 이상이면 유의하다고 볼 수 있다.
- +0.4 미만이면 설명력이 부족한 요인(중요도가 낮은 변수)으로 판단할 수 있다.
- SS loadings 항목은 각 요인 적재값의 제곱의 합을 제시한 값으로 각 요인의 설명력을 보여준다.
- Proportion Var 항목은 설명된 요인의 분산 비율로 각 요인이 차지하는 설명력의 비율이다.
- Cumulative Var 항목은 누적 분산 비율로 요인의 분산 비율을 누적하여 제시한 값으로 정보손실이 너무 크면 요인분석의 의미가 없어진다.

- 베리맥스 요인회전법

  - 요인분석을 실시하면 요인행렬이 구해지는데, 이 행렬은 어떤 변수들이 어떤 요인에 의해 높게 관계 되어 있는지를 보여주지 않는다.
  - 따라서 요인축의 회전을 통해서 특정 변수가 어떤 요인과 관계가 있는지를 나타내주어야 한다.
  - 요인회전법은 직각회전과 사각회전 방식이 있다.

  - 직각회전 방식인 베리멕스(varimax)는 요인행렬의 열(Column)에 위치한 변수들의 분산 합계가 최대화되도록 요인 적재량 +1, -1, 0에 가깝도록 해주는 회전법으로 각 요인 간의 상관관계가 없다고 자정한 경우 사용되는 방법이다.

```R
# 베리맥스 요인 회전법
> result <- factanal(subject, factors=2, rotation="varimax")
> result

Call:
factanal(x = subject, factors = 2, rotation = "varimax")

Uniquenesses:
   s1    s2    s3    s4    s5    s6 
0.250 0.015 0.005 0.136 0.407 0.107 

Loadings:
   Factor1 Factor2
s1  0.862         
s2  0.988         
s3          0.997 
s4 -0.115   0.923 
s5 -0.692   0.338 
s6 -0.846   0.421 

               Factor1 Factor2
SS loadings      2.928   2.152
Proportion Var   0.488   0.359
Cumulative Var   0.488   0.847

Test of the hypothesis that 2 factors are sufficient.
The chi square statistic is 11.32 on 4 degrees of freedom.
The p-value is 0.0232 
# 베리맥스 요인 회전법을 적용하여 요인 분석 수행 p-value는 0.0232로 유의수준 0.05보다 적기 때문에 요인수로 부족하다는 의미


> result <- factanal(subject, factors=3, rotation="varimax" , scores="regression")
> result

Call:
factanal(x = subject, factors = 3, scores = "regression", rotation = "varimax")

Uniquenesses:
   s1    s2    s3    s4    s5    s6 
0.005 0.056 0.051 0.005 0.240 0.005 

Loadings:
   Factor1 Factor2 Factor3
s1 -0.379           0.923 
s2 -0.710   0.140   0.649 
s3  0.236   0.931   0.166 
s4  0.120   0.983  -0.118 
s5  0.771   0.297  -0.278 
s6  0.900   0.301  -0.307 

               Factor1 Factor2 Factor3
SS loadings      2.122   2.031   1.486
Proportion Var   0.354   0.339   0.248
Cumulative Var   0.354   0.692   0.940

The degrees of freedom for the model is 0 and the fit was 0.7745
```

- 요인점수를 이용한 요인적재량 시각화

  - 요인분석에서 요인점수는 각 관측치(표준화된 값)와 요인 간의 관계를 통해서 구해진 점수를 의미한다. 관측치의 길이와 동일하다.

  - 요인 점수를 얻기 위해서는 scores 속성(scores=“regression” : 회귀분석으로 요인점수 계산)을 지정해야 한다.

- 요인별 변수 묶기

  - 요인분석을 통해서 각 요인에 속하는 입력변수들을 묶어서 파생변수를 생성할 수 있는데 이러한 파생변수는 상관분석이나 회귀분석에서 독립변수로 사용할 수 있다. 

  - 파생 변수는 가독성과 설득력이 가장 높은 산술평균 방식을 적용하여 생성할 수 있다.

  - 단계1 ] 요인별 파생변수를 대상으로 데이터프레임 생성

  - 단계2 ] 요인별 산술평균 계산

  - 단계3 ] 상관관계 분석