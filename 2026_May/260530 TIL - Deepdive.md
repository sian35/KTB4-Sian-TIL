3주차 딥다이브
## 고급 데이터 변환 및 처리를 위한 Pandas 기능(GroupBy 등) 및 기법을 설명하시오.

### 1. 병합 (concat, merge)
#### 1) 연결 
##### ```pd.concat([df1, df2, ...], axis=연결할 방향)```
주어진 데이터프레임을 단순히 ***위/아래 또는 좌/우***로 연결
axis=0 : 세로 방향
axis=1 : 가로 방향
##### 예시 [출처](https://pandas.pydata.org/docs/user_guide/merging.html)
```
df1 = pd.DataFrame(
    {
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
        "C": ["C0", "C1", "C2", "C3"],
        "D": ["D0", "D1", "D2", "D3"],
    },
    index=[0, 1, 2, 3],
)

df2 = pd.DataFrame(
    {
        "A": ["A4", "A5", "A6", "A7"],
        "B": ["B4", "B5", "B6", "B7"],
        "C": ["C4", "C5", "C6", "C7"],
        "D": ["D4", "D5", "D6", "D7"],
    },
    index=[4, 5, 6, 7],
)

df3 = pd.DataFrame(
    {
        "A": ["A8", "A9", "A10", "A11"],
        "B": ["B8", "B9", "B10", "B11"],
        "C": ["C8", "C9", "C10", "C11"],
        "D": ["D8", "D9", "D10", "D11"],
    },
    index=[8, 9, 10, 11],
)

frames = [df1, df2, df3]

result = pd.concat(frames)
```
#### 2) 병합
##### `pd.merge(left, right, on='기준 열', how='병합 방식')`
두 개 이상의 데이트프레임을 ***공통된 열 또는 인덱스***를 기준으로 병합
##### 예시 [출처](https://pandas.pydata.org/docs/user_guide/merging.html)
```
left = pd.DataFrame(
    {
        "key": ["K0", "K1", "K2", "K3"],
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
    }
)

right = pd.DataFrame(
    {
        "key": ["K0", "K1", "K2", "K3"],
        "C": ["C0", "C1", "C2", "C3"],
        "D": ["D0", "D1", "D2", "D3"],
    }
)

result = pd.merge(left, right, on="key")

result

Out[47]: 
  key   A   B   C   D
0  K0  A0  B0  C0  D0
1  K1  A1  B1  C1  D1
2  K2  A2  B2  C2  D2
3  K3  A3  B3  C3  D3
```
#### 연결(concat)과 병합(merge) 차이점
| 구분         | 차이점                                                |
| ---------- | -------------------------------------------------- |
| `merge()`  | 공통 열을 기준으로 병합 수행. 일치하는 데이터를 병합하고 일치하지 않으면 `NaN` 발생 |
| `concat()` | 병합할 키 없이 단순히 데이터를 연결하여 합침                          |

-----
### 2. 사용자 함수 적용 (User-Defined Functions)

#### 1) `df.map('적용할 함수')`
is used specifically to apply **element-wise** UDFs

예시 [출처](https://pandas.pydata.org/docs/user_guide/user_defined_functions.html)
```
temperature_celsius = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def to_fahrenheit(value):
    return value * (9 / 5) + 32

temperature_celsius.map(to_fahrenheit)

Out[6]: 
    NYC  Los Angeles
0  57.2         71.6
1  69.8         82.4
2  73.4         87.8
```
to_fahrenheit 함수가 6번(element 6개) 호출됨
#### 2) `df.apply('적용할 함수')`
allows you to apply UDFs for a **whole column or row**

예시 [출처](https://pandas.pydata.org/docs/user_guide/user_defined_functions.html)
```
temperature_celsius = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def to_fahrenheit(column):
    return column * (9 / 5) + 32

temperature_celsius.apply(to_fahrenheit)

Out[9]: 
    NYC  Los Angeles
0  57.2         71.6
1  69.8         82.4
2  73.4         87.8
```
to_fahrenheit 함수가 2번(column 2개) 호출됨

예시2
```
temperature = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def normalize(column):
    return column / column.mean()

temperature.apply(normalize)

Out[12]: 
        NYC  Los Angeles
0  0.724138     0.814815
1  1.086207     1.037037
2  1.189655     1.148148
```
normalize 함수는 계산을 위해서 컬럼 단위로 평균을 구해야 함. 따라서 element 별로 함수를 호출하면 안되기 때문에 apply를 사용해야 한다. 

----
### 3. 그룹화 (Groupby)
#### 동작 단계
- Split : .groupby() 를 사용하여 컬럼 조건에 따라 독립된 그룹으로 나눈다.
- Apply : 나뉘어진 독립된 그룹에 그룹별 함수를 적용하는 단계이다.
- Combine : 독립된 그룹별로 함수가 적용된 결과를 다시 하나의 테이블로 합친다.

##### `df.groupby("컬럼 명")`

예시 [출처](https://pandas.pydata.org/docs/user_guide/groupby.html)
```
df2 = pd.DataFrame({"X": ["B", "B", "A", "A"], "Y": [1, 2, 3, 4]})

df2.groupby(["X"]).sum()
Out[20]: 
   Y
X   
A  7
B  3

df2.groupby(["X"], sort=False).sum()

Out[21]: 
   Y
X   
B  3
A  7
```

##### 나뉘어진 그룹 조회
```
df3 = pd.DataFrame({"X": ["A", "B", "A", "B"], "Y": [1, 4, 3, 2]})

df3.groupby("X").get_group("A")

Out[23]: 
   X  Y
0  A  1
2  A  3

df3.groupby(["X"]).get_group(("B",))

Out[24]: 
   X  Y
1  B  4
3  B  2
```

---
### 4. 피벗 (Pivot)

##### 1) `df.pivot(index='행으로 설정할 열', columns='열로 설정할 열', values='값으로 설정할 열')`
- 데이터의 열을 기준으로 재구성해주는 메서드
- 중복된 인덱스와 열을 허용하지 않음. 

예시 [출처](https://pandas.pydata.org/docs/user_guide/reshaping.html)
```
data = {
   "value": range(12),
   "variable": ["A"] * 3 + ["B"] * 3 + ["C"] * 3 + ["D"] * 3,
   "date": pd.to_datetime(["2020-01-03", "2020-01-04", "2020-01-05"] * 4)
}

df = pd.DataFrame(data)

pivoted = df.pivot(index="date", columns="variable", values="value")

pivoted

Out[4]: 
variable    A  B  C   D
date                   
2020-01-03  0  3  6   9
2020-01-04  1  4  7  10
2020-01-05  2  5  8  11
```

##### 2)`df.pivot_table(index='행으로 설정할 열', columns='열로 설정할 열', values='값으로 설정할 열', aggfunc='집계 함수', fill_value='결측값을 대체할 값', margins='집계 결과에 총계를 추가할지 여부')`
##### `pd.pivot_table(data,index='행으로 설정할 열', columns='열로 설정할 열', values='값으로 설정할 열' ...)`

- 더 복잡한 피벗 테이블을 생성하는 데 사용됨. 
- 집계 함수(aggfunc)를 지원한다.
	- aggfunc: 데이터가 중복될 경우 사용할 집계 함수.  

예시 [출처](https://pandas.pydata.org/docs/reference/api/pandas.pivot_table.html)
```
df = pd.DataFrame(
    {
        "A": ["foo", "foo", "foo", "foo", "foo", "bar", "bar", "bar", "bar"],
        "B": ["one", "one", "one", "two", "two", "one", "one", "two", "two"],
        "C": [
            "small",
            "large",
            "large",
            "small",
            "small",
            "large",
            "small",
            "small",
            "large",
        ],
        "D": [1, 2, 2, 3, 3, 4, 5, 6, 7],
        "E": [2, 4, 5, 5, 6, 6, 8, 9, 9],
    }
)

table = pd.pivot_table(
    df, values="D", index=["A", "B"], columns=["C"], aggfunc="sum"
)

table

C        large  small
A   B
bar one    4.0    5.0
    two    7.0    6.0
foo one    4.0    1.0
    two    NaN    6.0
```

##### `pivot()` 과 `pivot_table()`의 차이점
| 차이점       | `pivot()`                  | `pivot_table()`             |
| --------- | -------------------------- | --------------------------- |
| 집계 함수 지원  | 집계 함수 제공하지 않음.             | 집계 함수 지원                    |
| 중복 데이터 처리 | 중복된 인덱스-열 쌍을 허용하지 않음       | 집계 함수를 이용해 중복데이터를 처리 가능     |
| 결측 값 처리   | 결측값이 자동으로 처리되지 않고 NaN으로 남음 | fill_value를 지정하여 결측값을 처리 가능 |
| 다중 값 처리   | 하나의 열만 값으로 변환              | 다중 값(여러 열을 값으로) 처리 가능       |

----
### 5. 범주형 데이터 처리
#### one-hot encoding
![[Pasted image 20260530142026.png]]
##### `pd.get_dummies(data)`
범주형 데이터를 가변수(dummy variable)로 변환해준다.
각 카테고리는 1과 0이 포함된 고유한 열을 갖는다.

>[!note] 가변수(dummy variable)?
데이터를 수치형 데이터로 변환해줄 때 서로 간의 관계성이 생길 수 있기 때문에 관계성이 없는 가변수로 변환해주는 과정이 데이터 전처리에서 필요하다.

사용 이유 : 범주형 데이터를 수치형으로 변환해주기 위하여

----
### 6. 날짜/시간 처리
#### 1) Timestamp와 DatetimeIndex
특정 시점을 나타내는 날짜와 시간을 표현
##### `pd.Timestamp('나노 초')`

##### `pd.to_datetime(arg, format='원하는 날짜 포맷')`

```
pd.to_datetime('2019-1-1 12')

Timestamp('2019-01-01 12:00:00')
```

여러 컬럼에서 날짜 정보를 모아서 datetime으로 반환도 가능
예시 [출처](https://wikidocs.net/172646)
```
df = pd.DataFrame({"year": [2015, 2016], "month": [2, 3], "day": [4, 5]})

pd.to_datetime(df)

0   2015-02-04
1   2016-03-05
dtype: datetime64[us]
```

to_datetime() 에 날짜들의 리스트를 넣을 경우 범위를 반환하는데 DatetimeIndex라는 클래스로 배열이 생기고 데이터 타입이 datetime 이다.
```
pd.to_datetime(['2018-1-1', '2019-1-2'])

DatetimeIndex(['2018-01-01', '2019-01-02'], dtype='datetime64[ns]', freq=None)
```

##### `pd.date_range('2019-01', '2019-02')`
- 특정 기간의 날짜를 자동 생성한다. (2019-01-01 ~ 2019-02-01)

```
DatetimeIndex(['2019-01-01', '2019-01-02', '2019-01-03', '2019-01-04',
               '2019-01-05', '2019-01-06', '2019-01-07', '2019-01-08',
               '2019-01-09', '2019-01-10', '2019-01-11', '2019-01-12',
               '2019-01-13', '2019-01-14', '2019-01-15', '2019-01-16',
               '2019-01-17', '2019-01-18', '2019-01-19', '2019-01-20',
               '2019-01-21', '2019-01-22', '2019-01-23', '2019-01-24',
               '2019-01-25', '2019-01-26', '2019-01-27', '2019-01-28',
               '2019-01-29', '2019-01-30', '2019-01-31', '2019-02-01'],
              dtype='datetime64[ns]', freq='D')

```
#### 2) Period와 PeriodIndex
기간을 포괄하는 개념
##### 기본적으로 월 단위로 인식됨. 
`pd.Period('2019-05')`
`Period('2019-05', 'M')`

##### 일 단위로 자동 생성됨
`pd.Period('2019-05', freq='D')`
`Period('2019-05-01', 'D')`

##### `pd.period_range('2019-01', '2019-02', freq='D')`
str 타입의 두 개 이상 날짜 데이터를 전달하면 **PeriodIndex 클래스**로 묶인다.

```
PeriodIndex(['2019-01-01', '2019-01-02', '2019-01-03', '2019-01-04',
             '2019-01-05', '2019-01-06', '2019-01-07', '2019-01-08',
             '2019-01-09', '2019-01-10', '2019-01-11', '2019-01-12',
             '2019-01-13', '2019-01-14', '2019-01-15', '2019-01-16',
             '2019-01-17', '2019-01-18', '2019-01-19', '2019-01-20',
             '2019-01-21', '2019-01-22', '2019-01-23', '2019-01-24',
             '2019-01-25', '2019-01-26', '2019-01-27', '2019-01-28',
             '2019-01-29', '2019-01-30', '2019-01-31', '2019-02-01'],
            dtype='period[D]', freq='D')

```

