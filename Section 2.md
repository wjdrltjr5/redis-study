# Redis 설치

## MAC

## Window

## 데이터 저장/조회/삭제

키/벨류 형태
저장

```
$ SET lecture inflearn-redis
```

조회

```
$ GET lecture
```

삭제

```
$ DEL lecture
```

## 실행/종료

실행

```
//레디스 서버 실행후
$ redis-cli
```

종료

```
$ exit
```

## CRUD

현재의 키값들 모두 확인 O(N)

```
$ keys *
```

해당 키 검색하기 sql %느낌

```
$ keys *k*
```

키이름 변경

```
$ rename 키이름 바꿀키이름
```

flushall 모든 데이터(key와 value)를 삭제

```
$ flushall
```

key를 이용한 해당 레코드 삭제

```
$ del key
```

key를 이용한 value 수정

```
$ getset key value

```

key를 이용해 value에 append 작업

```
append key 추가할value
```
