# 들어가기 전에
## 데이터 수집의 분류

||실시간 수집|거의 실시간 수집|배치 수집|
|:---:|:---:|:---:|:---:|
|특징|즉각 action|반응형 action|historical action|
|종류|- Kinesis Data Streams(KDS) <br/> - Simple Queue Services(SQS) <br/> - Interet of Things(IoT)|- Kinesis Data Firehose(KDF) <br/> - DataBase Migration Services(DMS)| - Snowball <br/> - Data Pipelne|


# 개요

# 1. KDS 아키텍쳐
- [AWS Docu. 참고 ](https://docs.aws.amazon.com/streams/latest/dev/key-concepts.html)

- KDS 데이터 흐름 

    <div style="text-align:center;">
        <img src="./img/ch02_01_img01.png"/>
    </div>

    - 참고
        - producer: KDS에 끊임없이 데이터 전송
        - consumer: 실시간으로 유입된 데이터 처리하고 처리한 결과물들을 S3, DynamoDB 등의 저장소에 저장
    - Producer -> KDS
        - producer가 record 형태로 데이터를 전송한다. record는 (1) 파티션 키(데이터가 어떤 shard로 할당될지 결정)와 (2) 데이터 블롭으로 구성됨 <br/> 
        데이터 블롭은 데이터 값 자체, 초당 1MB까지 or shard 당 1000개 메세지까지 전송 가능
        - shard(데이터 조각)가 3개라면 초당 3MB 또는 3000개 전송 가능
    - KDS -> Consumer
        - 다양한 대상이 Consumer가 될 수 있고(app, Lambda, Kinesis Data Firehose, Kinesis Data Analytics), consuemr는 데이터 포맷으로 데이터를 전송받을 수 있음
        - KDS로부터 Record를 받음. 
            - (1) 파티션 키  (2) 시퀀스 번호 (3) Data Blob
            - 2Mb/sec per shard

# 2. KDS 특성
- 1 ~ 365일 retention
- 데이터 재처리 가능
- (immuatability) 키네시스에 입력되면 삭제 불가
- (ordering) 동일 파티션을 공유하는 데이터는 동일 shard로 감

# 3. Capacity Mode
- Provision mode: 사용량 아는 경우 사용 유리
    - shard 개수, 크기 등 수동으로 조작 가능, API 사용 가능
    - 샤드당 1Mb/s in
    - 샤드 당 2Mb/s out
- On-demand mode: 사용량 모르는 경우 사용
    - capacity 조정할 필요 없음
    - 기본 capacity 정해져있음, 4Mb/s in 4000 records
    - 30일간 처리량 기반 자동 스케일링
    - 사용량, 시간에 기바난 가격

# 4. KDS 보안
- IAM을 이용하 통제 접근/권한
- HTTPS를 이용한 암호화
- KMS 암호화
- client 측 데이터 암호화/복호화 -> 구현은 어렵지만 보안이 강화됨
- VPC 엔드포인트 사용(인터넷 거치지 않음)

--------

# 참고. 관련 용어
1. **Kinesis Data Stream**
    - 샤드(shards, 파편)가 저장된 데이터 레코드의 시퀀스를 포함하는 데이터 스트림의 집합
    - 샤드는 데이터 스트림의 기본 구성 요소

2. **Data Record**
    - Amazon Kinesis 데이터 스트림에 저장된 기본 데이터 단위
    - 시퀀스 번호, 파티션 키 및 데이터 블롭으로 구성됨
    - 데이터 블롭은 수정 불가능한 바이트 시퀀스 (최대 1MB)로 구성
    - Amazon Kinesis Data Streams는 블롭의 데이터를 수정하거나 해석하지 않음

3. **Capacity Mode**
    - 용량이 관리되고 과금되는 방식을 결정하는 모드
    - 온디맨드 모드: 필요한 처리량을 제공하기 위해 자동으로 샤드를 관리하며 실제 사용량에 따라 과금
    - 프로비저닝 모드: 샤드의 수를 지정해야 하며 시간당 샤드 수에 따라 과금

4. **Retention Period**
    - 데이터 레코드가 스트림에 추가된 후 접근 가능한 시간
    - 기본 보유 기간은 24시간이며 최대 8760시간 (365일)까지 늘릴 수 있으며 최소 24시간까지 줄일 수 있음

5. **Producer**
    - Amazon Kinesis Data Streams에 레코드를 추가하는 역할
    - 예: 웹 서버가 로그 데이터를 스트림에 보내는 것

6. **Consumer**
    - Amazon Kinesis Data Streams에서 레코드를 가져와 처리
    - 소비자는 주로 Amazon Kinesis Data Streams 애플리케이션

7. **Amazon Kinesis Data Streams Application**
    - 스트림의 소비자로 일반적으로 EC2 인스턴스 그룹에서 실행되는 애플리케이션
    - 공유 팬-아웃 소비자 및 향상된 팬-아웃 소비자 두 가지 유형의 소비자가 있음
    - 애플리케이션의 출력은 다른 스트림의 입력 또는 다양한 AWS 서비스로 전송될 수 있음

8. **Shard**
    - 스트림의 고유 식별 데이터 레코드 시퀀스
    - 고정 용량 단위를 제공
    - 읽기에 대해 초당 최대 5 거래 및 쓰기에 대해 초당 최대 1,000 레코드를 지원
    - 스트림의 총 용량은 샤드의 용량의 합계

9. **Partition Key**
    - 스트림 내에서 파티션 키별로 데이터를 그룹화하는 데 사용
    - 데이터 레코드를 여러 샤드로 분할
    - 최대 길이가 256자인 유니코드 문자열
    - MD5 해시 함수를 사용하여 파티션 키를 128비트 정수 값으로 매핑하고 해당하는 데이터 레코드를 샤드에 매핑

10. **Sequence Number**
    - 샤드 내의 파티션 키당 고유한 시퀀스 번호
    - Amazon Kinesis Data Streams가 스트림에 쓰기 후에 할당
    - 동일한 파티션 키에 대해 시간이 지남에 따라 일반적으로 시퀀스 번호가 증가

11. **Kinesis Client Library**
    - 애플리케이션에 편입되어 스트림에서의 데이터 소비를 내고자 하는데 사용
    - 각 샤드에 대해 레코드 프로세서가 실행되고 데이터를 처리하는 것을 보장함