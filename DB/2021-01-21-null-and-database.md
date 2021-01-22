---
title: NULL과 관계형 데이터 베이스
tags: database
---

# Null이란 무엇인가

NULL은 데이터베이스에만 사용되는 것이 아닌 Java나 C등 프로그래밍 언어를 사용할 때도 흔히 볼 수 있다. 개발자들 사이에서는 null 값이라는 용어로 사용되곤 하는데 이는 잘못된 표현이다. null은 값이 존재하지 않는다는 의미를 나타내는 표기로 null'값'이라는 것은 적절치 않다. 값이 존재하지 않는다는 수학적 의미로 공집합과 혼돈될 수 있지만 엄밀히 말해서 공집합은 요소의 갯수가 0개인 집합이다. 값이 존재하지 않는다는 의미의 null과는 다르다.

sql에서 null은 c와 java에서 사용하는 null 과의 차이가 있는데 일반적인 프로그래밍 언어에서는 a == null 이라는 것으로 null과 값을 비교할 수가 있는데 sql에서는 IS NULL, IS NOT NULL 처럼 IS를 사용하고 비교연산자를 사용하지 않는다. 즉, SQL에서는 NULL을 값으로 취급하지 않는다는 뜻이다.

# 3차 논리(3VL)

NULL의 문제점은 NULL이 될 수 있는 칼럼을 연산할 때 발생한다. 칼럼을 비교하거나 값을 가공해야 하는 일이 발생한다.

NULL은 값이 없다는 것이기 때문에 단순 비교연산자로 비교할 수가 없다.

## NULL은 연산을 망친다

단순 boolean 의 결과가 나오는 연산을 예로 들자면 100 < NULL 은 true도 false도 아닌 NULL이다. 이는 true와 false를 가지려고 한 질의에 혼란을 줄 수가 있다. NULL + 1과 같은 연산도 NULL이라는 결과가 나온다.

## 검색 결과가 의도하지 않은 결과가 될 가능성

NULL이 칼럼에 있다는 것을 의식하지 않고 질의를 수행하게 되면 WHERE에서 NULL과 비교하게 되어 원하지 않는 값이 나오게 될 수 있다. 예를 들어 다음 질의는 age = NULL 인 값을 가져오지 않는다.

```sql
SELECT * FROM users WHERE age <> 20
```

20살이 아닌 사용자를 가져오지만 여기에 NULL은 포함되지 않는다.  여기서 NULL을 포함하려면 다음과 같이 작성해야한다

```sql
SELECT * FROM users WHERE age <> 20 AND age IS NULL
```

## NULL에 의한 제3의 논리값

칼럼에 NULL이 포함되어 있다면 NULL일 때는 어떻게 처리해야하는가? 라는 논리가 필요하다. SELECT가 결과를 반환할 때는 WHERE 절의 조건이 TRUE가 될 때 뿐이다. FALSE나 NULL이면 만족하지 않는다고 판단하게 된다.

NULL은 값이 존재하지 않음을 나타내는 값이다. 그 값은 Unknown이라 할 수 있고 Unknown이 포함된 식을 평가하면 그 값도 Unknown이 되어 버린다. NULL이 포함된 식은 NULL이 되어버린다. NULL이 있는 덕분에 마치 논리값이 3개가 존재하는 것처럼 보인다.

이처럼 TRUE, FALSE, UNKNOWN 세 가지 논리값에 의해서 판정이 이뤄지는 논리 시스템을 3차 논리라고 한다.

## 생각보다 성가신 3VL

NULL의 문제는 3VL을 다루지 않을 수 없다. 3VL은 수치 연산이나 문자열 뿐만 아니라 AND나 OR 연산에도 적용된다. 요소 하나라도 NULL이 있다면 식 전체가 3VL이 되어버릴 것이다. 평가식이 3VL인지 2VL인지에 따라서 SQL의 개발 효율은 크게 바뀐다.

## 3차 논리의 한계

3VL이 논리적으로 잘못된 것은 없지만 SQL에서 3VL을 제대로 표현하기란 쉽지가 않다. 관계형 모델은 현실 세계를 표현하기 위한 모델이므로 3VL로는 현실세계를 제대로 표현할 수가 없다.

### Unknown의 모호함

Unknown이란 현실세계를 표현하기에 어려운 부분이 많다. 예를 들어 눈 앞의 사람의 나이를 몰라도 어느 연령대인지 추측을 할 수가 있다. 애초에 Unknown이란 정보 자체가 없다는 것인데 그것을 현실세계에서 사용하기란 쉽지가 않다.

## NULL은 폐쇄 세계 가정에 반한다

 NULL을 사용하면 안 되는 가장 큰 이유는 관계형 모델을 근본적으로 뒤집는 존재이기 때문이다. 관계형 모델은 폐쇄 세계 가정이라는 가설 위에 이루어져 있다. 이 가설 덕분에 판명된 사실은 릴레이션에 포함돼 있으며, 릴레이션끼리 조합해 연산한 결과 릴레이션도 사실 전부 딱 맞게 포함된다. 따라서 모든 질의가 릴레이션의 연산만으로 해결이 된다.

그러나 NULL이 이 전제를 뒤집는다. NULL은 현재 시점에서 알 수 없는 값이다. 두 개의 테이블을 결합할 때 3VL의 정의에 따라서 연산을 처리하면 키가 NULL인 경우에 그 행은 결과에 포함되지 않을 것이다. 즉 모든 질의를 릴레이션의 연산만으로 해결한다는 전제가 무너져 내리는 것이다.

때문에 NULL은 관계형 모델을 근본적으로 파괴하는 것이다.

## 옵티마이저에 대한 폐해

NULL의 존재가 나쁜 영향을 미치는 것은 데이터 모델이라는 논리적인 측면뿐만 아니다. 쿼리의 실행 계획인 옵티마이저의 구현에도 큰 악영향이 있다. 옵티마이저란 쿼리의 실행이 최적의 성능을 내도록 바꾸어주는 것이다.

NULL이 존재하는 경우 수학 적으로 증명할 수 있는 조합이 많이 줄어든다. 결과적으로 옵티마이저가 아니라 사람이 직접 튜닝을 해주어야한다. 또한 NULL은 인덱스의 제일 앞 또는 제일 뒤로 배치된다. 때문에 행이 늘어날수록 NULL을 위해 스캔하는 시간이 길어진다.

# NULL의 대책

## 테이블을 정규화 한다

NULL을 제거하는 가장 전통적인 방법은 테이블을 정규화하는 것이다. 이미 1NF시에 NULL이 포함돼서는 안된다. 어떤 값이 NULL이라는 것은 아직 그 데이터가 필요하지 않는 다는 의미이다. 때문에 별도의 테이블로 분리하는 것이 좋다.

## 잘못된 NULL 대책

칼럼을 모두 NOT NULL로 정의하는 대신 NULL과 같은 의미가 있는 값으로 정의하는 것이다. 예를 들어 나이를 모르는 사람의 나이를 NULL이 아닌 -1과 같은 값으로 정의한다고 가정했을 때다. 나이가 20이 아닌 사람을 찾고자 한다.

```sql
SELECT * FROM USER WHERE AGE <> 20
```

이와 같은 경우는 -1의 값이 있더라도 잘 처리할 수 있다. 그러나 미성년자를 찾는 쿼리를 작성한다고 가정한다

```sql
SELECT * FROM USER WHERE AGE < 20
```

이는 -1이라는 값을 가진 행도 추출하게 된다. 여기서 -1은 NULL 대신 사용하고 있다. 전혀 관계없는 질의에도 결과가 도출되어버리는 문제가 발생한다. 나이가 불분명하다라는 의미에서 -1을 사용하지만 NULL과의 본질적인 차이가 있는지도 의문이다. NULL을 피하고자 편의상 NULL이 아닌 NULL과 같은 의미가 있는 기본값을 사용하는 것은 더욱 상황을 악화시킬수가 있다.

## COALESCE 함수

정규화를 통해 NULL을 없애는 것도 중요하지만, 그것만으로도 NULL을 완전히 피할수는 없다. SQL에 포함된 식을 평가한 결과가 NULL이 될 수도 있다. 다음과 같은 예가 있다

- 행의 개수가 0개인 행에 SUM이나 AVG와 같은 집계함수를 실행했을 때
- 스칼라 또는 행 서브 쿼리를 실행한 결과 일치하는 행이 없을 때
- OUTER JOIN 실행 시에 일치하는 행이 없을 때
- CASE 식에서 ELSE문이 없고, 어떤 조건도 해당하지 않을 때
- NULLIF 식을 평가한 결과가 NULL일 때

SQL에서 바른 결과를 얻으려면 이러한 결과에서 생기는 NULL에 대해서도 적절히 처리되게 로직을 생각해야한다.

NULL이 발생할 수 있는 경우 COALESCE 함수를 사용할 수 있다. 

```sql
SELECT continent, COALESCE(SUM(population), 0)
FROM countries GROUP BY continent
```

이런식으로 질의 결과가 null일 때 기본값을 설정해두면 된다. 이와 같은 COALESCE 함수의 사용법은 다이나믹 디폴트라고 한다. 한편 NULL을 가진 칼럼에 대해 COALESCE 함수로 기본값을 정의하는 방식의 사용법은 추천하지 않는다. 단순히 칼럼의 기본값을 정의하는 것 뿐이라면 NULL이 가진의미와 다르지 않고 오히려 더 나쁘게 만들어버릴 수 있기 때문이다. COALESCE는 SQL 구조상 어쩔 수 없이 NULL이 될 가능성이 있는 집계함수나 스칼라 서브 쿼리의 기본값으로 이용할 때 의미가 있다.

## 빈문자열 처리

소프트웨어 중에 공백문자열과 NULL을 같은 의미로 보기도 한다. NULL과는 다른 의미이므로 명확히 구분해야 한다. NULL은 값이 없음을 뜻하고 빈 문자열은 길이가 0인 문자열이기 때문이다. 빈 문자열과 NULL이 같은 뜻이라고 정하는 경우 대처 방법은 없다. 다만 응용프로그램이 빈 문자열을 필요로 하는 경우엔 문제가 되지 않는다.

## NULL을 사용해도 좋을 때

DB에서 NULL을 완전히 제거하는 것은 불가능하며 그렇게 해서도 안된다.

SQL은 관계형 모델을 기반으로 하고 있지만 관계형 모델 이외의 데이터도 처리할 수 있는 표현력이 있다. 현실의 세계를 관계형 모델만으로 표현하는 것은 불가능 하므로 SQL의 이러한 성질이 SQL을 모든 상황에 대응할 수 있는 언어로 만들었다.

NULL은 관계형 모델을 뒤집어 버리지만 관계형 모델에 따라서 테이블을 사용하는 경우에 해당한다. 관계형 모델에 적합하지 않은 데이터를 테이블을 사용할 때는 관계형 모델의 원리 원칙을 따를 필요는 없다. 

따라서 NULL을 허용할지 안할지는 대상 테이블이 관계형 모델에 따라 설계된 것인지 아닌지를 명확하게 이해할 필요가 있다.