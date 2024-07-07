# Redis 데이터 타입 알아보기

명령어 대소문자 구분안함

## Strings

Strings : 문자열, 숫자(별도의 Integer 타입 없음), serialized object(JSON string)등 저장

string 타입 값 저장

```
$ SET lecture inflearn-redis
```

다수의 string 값 저장

```
$ MSET price 100 language ko
```

다수의 string 값 조회

```
$ MGET lecture price language
```

숫자로 사용되는 값 1증가

```
$ INCR price
```

숫자로 사용되는 값 특정값 만큼 증가

```
$ INCRBY price 10
```

JSON 형식 값 저장

```
$ SET inflearn-redis '{"price" : 100, "language" : "ko"}'
```

콜론으로 키 의미 구분
// 인프런 레디스 강좌의 한국어버전의 가격은 200; 암묵적 약속

```
$ SET inflearn-redis:ko:price 200
```

## Lists

Lists String을 LinkedList로 저장 -> push/pop에 최적화 O(1) Queue / Stack구현에 사용

ex)
큐

```
$ LPUSH queue job1 job2 job3
$ RPOP queue  // job1
```

스택

```
$ LPUSH stack job1 job2 job3
$ LPOP stack   // job3
```

범위 조회 <br>
오른쪽기준은 -3 -2 -1 <br>
왼쪽기준은 0 1 2

```
$ LPUSH queue job1 job2 job3
$ LRANGE queue -2 -1
$ LTRIM queue 0 0 // 0~0만 남김
```

## Sets

Sets Unique string을 저장하는 정렬되지 않은 집합 Set Operation 사용가능

```
$ SADD user:1:fruits apple banana orange orange
$ SMEMBERS user:1:fruits // apple banana orange 출력 중복제거 되어있음
$ SCARD user:1:fruits // 카디널리티 출력 (고유 아이템 갯수)
$ SISMEMBER user:1:fruits banana // 정보를 가지고 있는가 exist느낌

$ SADD user:2:fruits apple lenmon
$ SINTER user:1:fruits user:2:fruits // 교집합 출력
$ SDIFF user:1:fruits user:2:fruits // 1-2 차집합
$ SUNION user:1:fruits user:2:fruits // 합집합
```

## Hashes

Hashes : field-value 구조를 갖는 데이터 타입 다양한 속성을 갖는 객체의 데이터를 저장할때 유용

```
$ HSET lecture name inflearn-redis price 100 language ko

$ HGET lecture name
$ HMGET lecture price language invalid

$ HINCRBY lecture price 10 //숫자형 스트링에만 가능
```

## Sorted Sets

Sorted Sets : Unique string을 연관된 score를 통해 정렬된 집합(Set의기능 + 추가로 score속성 저장) 내부적으로 Skip List + Hash Table로 이루어져 있고 score값에 따라 정렬 유지 값 동일시 사전 편찬순

```
$ ZADD porints 10 TeamA 10 TeamB 50 TeamC   // A,B는 동일 점수지만 선착순을 A가 우선

$ ZRANGE points 0 -1 // 처음부터 끝까지 조회
// TeamA TeamB TeamC 지금은 오름차순

$ ZRANGE points 0 -1 REV WITHSCORES // 역순으로 score점수까지 출력
// TeamC 50 TeamB 10 TeamA 10 //내림차순과 점수(줄바꿔져서 출력 같은줄 x)

$ ZRANKE points TeamA // 인덱스 반환 즉 등수 0등
```

## Streams

Streams append-only log에 consumer groups과 같은 기능을 더한 자료 구조
데이터가 추가만됨 각entiry마다 고유 id가 있어 읽을때 O(1)

Concumer Group을 통해 분산 시스템에서 다수의 concumer가 event처리

```
$ XADD events * action like user_id 1 product_id 1
//유니크 아이디 자동 할당 1번유저가 1번상품의 좋아요를 누른 이벤트 생성

$ XADD events * action like user_id 2 product_id 1
//유니크 아이디 자동 할당 2번유저가 1번상품의 좋아요를 누른 이벤트 생성

$ XRANGE events - +  // 다수의 메시지 조회
$ XDEL events 고유ID(xadd시 출력되는) //처리 완료 ID 삭제하기
```

## Geospatial

Geospatials Indexex : 좌표를 저장하고 검색하는 데이터 타입, 거리 계산 범위 탐색 등 지원

```
$ GEOADD seoul:station
      126.923917 37.556944 hond-dae
      127.027583 37.497928 gang-nam
//서울에 지하철역을 키로 데이터 추가 경도,위도순

$ GEODIST seoul:station hong-dae gang-nam km
//두 값 사이의 거리 (km단위) //11.2561출력 지리는데?
```

## Bitmaps

Bitmaps : 실제 데이터 타입은 아니고 String에 binary operation을 적용한 것

```
//user 로그인 정보 저장하기
$ SETBIT user:log-in:23-01-01 123 1
$ SETBIT user:log-in:23-01-01 456 1
$ SETBIT user:log-in:23-01-02 123 1

$ BITCOUNT user:log-in:23-01-01 // 23-01-01에 로그인한 사람 출력 123 456

$ BITOP AND result user:log-in:23-01-01 user:log-in:23-01-02
// 1일과 2일에 모두 로그인한 유저 결과를 result에 저장
$ GETBIT result 123 // 1출력
$ GETBIT result 456 // 0출력
$ BITCOUNT result // 1출력

```

## HyperLogLog

HyperLogLog : 집합의 cardinality를 추정할 수 있는 확률형 자료구조 정확성을 일부 포기하는 대신 저장공간을 효율적으로 사용(평균 에러 0.81%) 실제 값 저장하지 않기 떄문에 적은 메모리사용 아이템을 다시 출력해야 하는 경우는 사용 x

```
$ PFADD fruits apple orange grape kiwi
$ PFCOUNT fruits //4 출력
$ PFADD fruits apple
$ PFCOUNT fruits //4 출력
// 데이터가 많아지면 오차 발생할 수 있음
```

## BloomFilter

BloomFilter : element가 집합 안에 포함되었는지 확인할 수 있는 자료구조(= membership test) 정확성을 일부 포기하는 대신 저장공간을 효율적으로 사용

false positive : element가 집합에 실제로 표함되지 않은데 포함되었다고 잘못 예측할 때가 있음

HyperLogLog 처럼 실제값을 저장하지 않기 때문에 메모리 적게 사용

사용하기위해 특별한 모듈 필요
