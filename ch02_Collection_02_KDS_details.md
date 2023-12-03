# Kinesis Data Stream 요소들

# I. Kinesis Producers

- 데이터를 보내는 방법
    - Kinesis SDK: 코드를 써서 KDS로 데이터 직접 전송
    - Kinesis Producer Library(KPL): 더 나은 코드 작성 가능, 고급 기능
    - Kinesis Agent: 서버에서 실행되는 리눅스 프로그램, 로그 파일 확보 가능
    - 제 3의 라이브러리: Spark, Log4J, Appenders, Flume, Kafka Connect, Nifi 등


## 1. Kinesis SDK - PutRecords
- 배치를 통해 처리량 늘림 -> HTTP 요청 줄어듦
- 처리량 초과시 대처: **ProvisionedTroughputExceeded**
    - Mb 또는 레코드 수 초과
    - hot shard 체크(파티션 키가 잘못되거나 너무 많은 데이터가 파티션으로 감)
        - 핫 파티션이 되지 않도록 키를 많이 배포
    - 해결방법
        - 2~4초 후 재시도
        - 샤드(스케일) 양 늘림
        - 파티션 키가 좋은 것인지 체크
- 사용 범용성: 모바일 장치에서도 사용가능(안드로이드, iOS 등)
- 사용 사례: 낮은 처리량, 지연 높음, 단순 API, AWS Lambda

## 2. Kinesis Producer Library(KPL)

- C++ or Java 라이브러리
- 고기능의 장기 프로듀서를 만들 때 사용, 자동화 기능 탑재
- 처리량 초과 상황(exception) 시 자동으로 처리 결정
- (더 나은 성능) Synchronous or Asyncrhronous API
- CloudWatch에 모니터링 지표 보냄
- 배치: 처리량은 늘리고 비용은 감소
    - collect: API 호출에 따라 여러 레코드 수집 & 여러 shard 작성
    - aggregate(increased latency): 한 레코드에 여러 레코드 저장 가능
- 압축은 유저가 직접 구현
- CLI로 읽을 수 없음, 읽으려면 KCL이나 특수 라이브러리 필요

## 3. Kinesis Producer Library(KPL) Batching

<div style="text-align:center;">
    <img src="./img/ch02_02_img01.png" width=500/>
</div>

- 키네시스에 7개의 레코드를 보내는데 aggregation & collection 통해 API 콜만 하면 됨
- 지연시간 & 배치 수 간 조절: ex. latency 줄이고 batch 수 늘림, 또는 그 반대

## 4. KPL 라이브러리를 사용하지 않는 경우

- KPL이 RecordMaxBufferedTime를 야기할 때(프로세싱이 지연되어 최대 버퍼링 타임 초과 시)
    - 긴 지연시간을 감수하는 대신 전송되는 데이터에 많은 packing이 포함 -> 효율 & 성능 높음
    - 지연 시간을 허용하지 않는 app의 경우, AWS SDK를 이용해서 PutRecords를 통해 (나머지 데이터는 버리고) 최신 데이터만 선택 해 KDS로 보내도록 함

## 5. Kinesis Agent

- 로그 파일을 모니터링하고 KDS에 데이터를 보냄
- Java 기반 에이전트, KPL 라이브러리 위에 구현됨
- 리눅스 기반 환경에만 설치
- 특징
    - 다양한 디렉토리로부터 작성 가능, 여러 스트림에 작성 가능
    - 디렉토리/로그파일 기반 라우팅
    - 스트림으로 보내기 전에 사전 처리 가능(csv -> json, log -> json 등)
    - CloudWatch에 모니터링 지표 보냄

# II. Kinesis Consumers