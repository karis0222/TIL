# 파이썬 타입

# 데이터 타입

- 정수 : int
- 실수 : float
- 문자열 : str
- Boolean : True, False
- Null : None


# 컨테이너 타입
- 리스트(list) : []
  - 순서가 있다.
  - 중복 가능하다.
- 튜플(tuple) : ()
  - 순서가 있다.
  - 중복 가능하다.
  - 불변적이다.
- 딕셔너리(dict) : {key: value}
  - 순서가 없다.
  - 중복을 허용하지 않는다
    - 정확히 말하자면 중복된 키를 가진 요소가 존재할 경우 그 중 하나의 요소만을 가져오며 나머지 요소는 가져올 수 없다
    - 또한 어떤 요소를 가져올 지 알 수 없다. 딕셔너리는 순서가 없는 컨테이너 타입이기 때문.
  - Key: Value 형식.
- 셋(set) : {}
  - 순서가 없다.
  - 중복을 허용하지 않는다.

# 변수
파이썬의 변수는 동적(Dynamic) 타입이다.

```python
a = 10      #int
a = '10'    #str
a = []      #list
```
