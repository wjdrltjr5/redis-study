# Redis 특수 명령어

## 데이터 만료

Expiration : 데이터를 특정시간 이후에 만료 시키는 기능

TTL(Time To Live) : 데이터가 유효한 시간(초단위)

특징 - 데이터 조회 요청시에 만료된 데이터는 조회되지 않음
데이터가 만료되자마자 삭제하지 않고, 만료로 표시했다가 백그라운드에서 주기적으로 삭제

```
$ SET greeting hello  // greeting : hello
$ TTL greeting // -1 -> 데이터 만료 설정 x
$ EXPIRE greeting 10 // 10초후 만료 설정
$ TTL greeting  // 남은시간 출력 초과후에는 -2
$ GET greeting // 초과전에는 hello 초과후 nil출력

$ SETEX greeting 10 hello   // 만료시간까지 같이 설정
```

## NX/XX

NX : 해당 key가 존재하지 않는 경우에만 SET

XX : 해당 key가 이미 존재하는 경우에만 SET

Null Reply : SET이 동작하지 않는 경우 (nil)응답

```
$ SET greeting hello NX //됨
$ SET greeting hello XX //됨

$ SET invalid abcd XX // 실패 nil
$ SET greeting hello NX // 실패 nil
```

## Pub/Sub

Pub/Sub : Publisher와 Subscriber가 서로 알지 못해도 통신이 가능하도록 decoupling된 패턴 Publisher는 Subscriber에게 직접 메시지를 보내지 않고 Channel에 Publish Subscriber는 관심이 있는 Channel을 필요에 따라 Subscribe하며 메시지 수신

메시지가 보관되는 Stream과 달리 Pub/Sub은 Subscribe 하지 않을 때 발행된 메시지 수신 불가.

```
//구독
SUBSCRIBE ch:order ch:payment //1
//구독했다는 관련정보 출력 //1.1
"message"
"ch:order"
"new-order"

"message"
"ch:payment"
"new-payment"


```

```
PUBLISH ch:order new-order //2
PUBLISH ch:payment new-payment //3
```

## Pipeline

Pipeline : 다수의 commands를 한번에 요청하여 네트워크 성능을 향상 시키는 기술
Round-Trip Times 횟수 최소화 대부분의 클라이언트 라이브러리에서 지원

Round-Trip Times : Request/Response 모델에서 발생하는 네트워크 지연 시간

다수의 명령어 개별적 트랜잭션 x

## Transaction

Transaction : 다수의 명령어를 하나의 트랜잭션으로 처리 -> ACID
하나의 트랜잭션이 처리되는 동안 다른 클라이언트의 요청이 중간에 끼어들 수 없음

```
$ MULTI //트랜잭션 시작 TX표시로 트랜잭션 적용 확인 가능
...
$ DISCARD //롤백
$ EXEC // 커밋
```
