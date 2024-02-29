---
lab:
  title: Azure Stream Analytics 시작
  ilt-use: Suggested demo
---

# Azure Stream Analytics 시작

이 연습에서는 Azure 구독에서 Azure Stream Analytics 작업을 프로비전하고 이를 사용하여 실시간 이벤트 데이터의 스트림을 쿼리 및 요약하고 결과를 Azure Storage에 저장합니다.

이 연습을 완료하는 데 약 **15**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure 리소스 프로비전

이 연습에서는 시뮬레이션된 판매 트랜잭션 데이터의 스트림을 캡처하고, 처리하고, 결과를 Azure Storage의 Blob 컨테이너에 저장합니다. 스트리밍 데이터를 보낼 수 있는 Azure Event Hubs 네임스페이스와 스트림 처리 결과가 저장될 Azure Storage 계정이 필요합니다.

PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 이러한 리소스를 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 리포지토리를 복제합니다.

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 연습의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp-203/Allfiles/labs/17
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Stream Analytics 설명서에서 [Azure Stream Analytics에 오신 것을 환영합니다](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) 문서를 검토합니다.

## 스트리밍 데이터 원본 보기

실시간 데이터를 처리하는 Azure Stream Analytics 작업을 만들기 전에 쿼리해야 하는 데이터 스트림을 살펴 보겠습니다.

1. 설치 스크립트 실행이 완료되면 Azure Portal을 볼 수 있도록 클라우드 셸 창의 크기를 조정하거나 최소화합니다(나중에 클라우드 셸로 돌아갑니다). 그런 다음 Azure Portal에서, 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 이 리소스 그룹에 Azure Storage 계정 및 Event Hubs 네임스페이스가 포함되어 있음을 확인합니다.

    리소스가 프로비전된 **위치를** 확인합니다. 나중에 동일한 위치에 Azure Stream Analytics 작업을 만듭니다.

2. Cloud Shell 창을 다시 열고 다음 명령을 입력하여 100개의 시뮬레이션된 주문을 Azure Event Hubs에 보내는 클라이언트 앱을 실행합니다.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

3. 전송되는 판매 주문 데이터를 관찰합니다. 각 주문은 제품 ID와 수량으로 구성됩니다. 앱은 1000개의 주문을 보낸 후 종료되며, 1분 정도 걸립니다.

## Azure Stream Analytics 작업 만들기

이제 이벤트 허브에 도착하는 판매 트랜잭션 데이터를 처리하는 Azure Stream Analytics 작업을 만들 준비가 되었습니다.

1. Azure Portal의 **dp203-*xxxxxxx*** 페이지에서 **+ 만들기**를 선택하고 `Stream Analytics job`을 검색합니다. 그런 다음, 다음 속성을 사용하여 **Stream Analytics 작업**을 만듭니다.
    - **기본 사항**:
        - **구독**: ‘Azure 구독’
        - **리소스 그룹**: 기존 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다.
        - **이름**: `process-orders`
        - **지역**: 다른 Azure 리소스가 프로비전되는 지역을 선택합니다.
        - **호스팅 환경**: 클라우드
        - **스트리밍 단위**: 1
    - **스토리지**:
        - **스토리지 계정 추가**: 선택 안 함
    - **태그**:
        - *없음*
2. 배포가 완료될 때까지 기다렸다가 배포된 Stream Analytics 작업 리소스로 이동합니다.

## 이벤트 스트림에 대한 입력 만들기

Azure Stream Analytics 작업은 판매 주문이 기록되는 이벤트 허브에서 입력 데이터를 가져와야 합니다.

1. **process-orders** 개요 페이지에서 **입력 추가**를 선택합니다. 그런 다음, **입력** 페이지에서 **스트림 입력 추가** 메뉴를 사용하여 다음 속성을 사용하여 **Event Hub** 입력을 추가합니다.
    - **입력 별칭**: `orders`
    - **구독에서 Event Hub 선택:** 선택됨
    - **구독**: ‘Azure 구독’
    - **Event Hub 네임스페이스**: **이벤트*xxxxxxx*** Event Hubs 네임스페이스 선택
    - **Event Hub 이름**: 기존 **eventhub*xxxxxxx*** 이벤트 허브를 선택합니다.
    - **Event Hub 소비자 그룹**: 기존 **$Default** 소비자 그룹 선택
    - **인증 모드**: 시스템 할당 관리 ID 만들기
    - **파티션 키**: *비워 둠*
    - **이벤트 직렬화 형식**: JSON
    - **인코딩**: UTF-8
2. 입력을 저장하고 입력이 만들어지는 동안 기다립니다. 여러 알림이 표시됩니다. **연결 테스트 성공** 알림을 기다립니다.

## Blob 저장소에 대한 출력 만들기

집계된 판매 주문 데이터를 Azure Storage Blob 컨테이너에 JSON 형식으로 저장합니다.

1. **process-orders** Stream Analytics 작업에 대한 **출력** 페이지를 봅니다. 그런 다음, **추가** 메뉴에서 다음 속성을 사용하여 **Blob 스토리지/ADLS Gen2** 출력을 추가합니다.
    - **출력 별칭:** `blobstore`
    - **구독에서 Blob 스토리지/ADLS Gen2 선택을 선택**: 선택됨
    - **구독**: ‘Azure 구독’
    - **스토리지 계정**: **store*xxxxxxx*** 스토리지 계정을 선택합니다.
    - **컨테이너**: 기존 **데이터** 컨테이너 선택
    - **인증 모드**: 관리 ID: 시스템 할당
    - **이벤트 직렬화 형식**: JSON
    - **형식**: 줄로 구분
    - **인코딩**: UTF-8
    - **쓰기 모드**: 결과가 도착하면 추가
    - **경로 패턴**: `{date}`
    - **날짜 형식**: YYYY/MM/DD
    - **시간 형식**: *해당 없음*
    - **최소 행**: 20
    - **최대 시간**: 0시간, 1분, 0초
2. 출력을 저장하고 출력이 만들어지는 동안 기다립니다. 여러 알림이 표시됩니다. **연결 테스트 성공** 알림을 기다립니다.

## 쿼리 만들기

이제 Azure Stream Analytics 작업에 대한 입력 및 출력을 정의했으므로 쿼리를 사용하여 입력에서 데이터를 선택, 필터링 및 집계하고 결과를 출력으로 보낼 수 있습니다.

1. **process-orders** Stream Analytics 작업에 대한 **쿼리** 페이지를 봅니다. 그런 다음 입력 미리 보기가 표시될 때까지 잠시 기다립니다(이벤트 허브에서 이전에 캡처한 판매 주문 이벤트에 따라).
2. 입력 데이터에는 클라이언트 앱에서 제출한 메시지의 **ProductID** 및 **Quantity** 필드와 이벤트가 이벤트 허브에 추가된 시기를 나타내는 **EventProcessedUtcTime** 필드를 비롯한 추가 Event Hubs 필드가 포함됩니다.
3. 기본 쿼리를 다음과 같이 수정합니다.

    ```
    SELECT
        DateAdd(second,-10,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [blobstore]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 10)
    HAVING COUNT(*) > 1
    ```

    이 쿼리는 **System.Timestamp**(**EventProcessedUtcTime** 필드 기반)를 사용하여 각 제품 ID의 총 수량이 계산되는 각 10초 *연속*(겹치지 않는 순차적) 창의 시작과 끝을 정의하는지 확인합니다.

4. **&#9655; 쿼리 테스트** 단추를 사용하여 쿼리의 유효성을 검사하고 **테스트 결과** 상태가 **성공**으로 표시되는지 확인합니다(반환되는 행이 없더라도).
5. 쿼리를 저장합니다.

## 스트림 작업 실행

이제 작업을 실행하고 실시간 판매 주문 데이터를 처리할 준비가 되었습니다.

1. **process-orders** Stream Analytics 작업의 **개요** 페이지를 보고 **속성** 탭에서 작업에 대한 **입력**, **쿼리**, **출력**, **함수**를 검토합니다. **입력** 및 **출력** 수가 0인 경우 **개요** 페이지의 **&#8635; 새로 고침** 버튼을 사용해 **주문** 입력 및 **blobstore** 출력을 표시합니다.
2. **&#9655; 시작** 단추를 선택하고 지금 스트리밍 작업을 시작합니다. 스트리밍 작업이 성공적으로 시작되었다는 알림이 표시될 때까지 기다립니다.
3. Cloud Shell 창을 다시 열고 필요한 경우 다시 연결한 후 다음 명령을 다시 실행하여 다른 1000개의 주문을 제출합니다.

    ```
    node ~/dp-203/Allfiles/labs/17/orderclient
    ```

4. 앱이 실행되는 동안 Azure Portal에서 **dp203-*xxxxxxx*** 리소스 그룹 페이지로 돌아가서 **store*xxxxxxxxxxxx*** 스토리지 계정을 선택합니다.
6. 스토리지 계정 블레이드 왼쪽의 창에서 **컨테이너** 탭을 선택합니다.
7. **데이터** 컨테이너를 열고 **&#8635; 새로 고침** 단추를 사용하여 현재 연도의 이름을 가진 폴더가 표시될 때까지 보기를 새로 고칩니다.
8. **데이터** 컨테이너에서 현재 연도의 폴더를 포함하는 폴더 계층 구조를 탐색합니다. 여기에는 월, 일의 하위 폴더가 있습니다.
9. 시간 폴더에서 생성된 파일을 확인합니다. 이름이 **0_xxxxxxxxxxxxxxxx.json**과 비슷해야 합니다.
10. 파일의 **...** 메뉴(파일 세부 정보의 오른쪽)에서 **보기/편집**을 선택하고, 파일 내용을 검토합니다. 파일 내용은 각 10초간의 JSON 기록으로 구성되어야 하며, 다음과 같이 각 제품 ID에 대해 처리된 주문 수가 표시되어야 합니다.

    ```
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":6,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":8,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":5,"Orders":15.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":1,"Orders":16.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":3,"Orders":10.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":2,"Orders":25.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":7,"Orders":13.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":4,"Orders":12.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":10,"Orders":19.0}
    {"StartTime":"2022-11-23T18:16:25.0000000Z","EndTime":"2022-11-23T18:16:35.0000000Z","ProductID":9,"Orders":8.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":6,"Orders":41.0}
    {"StartTime":"2022-11-23T18:16:35.0000000Z","EndTime":"2022-11-23T18:16:45.0000000Z","ProductID":8,"Orders":29.0}
    ...
    ```

11. Azure Cloud Shell 창에서 주문 클라이언트 앱이 완료되기를 기다립니다.
12. Azure Portal에서 파일을 새로 고쳐 생성된 전체 결과 집합을 확인합니다.
13. **dp203-*xxxxxxx*** 리소스 그룹으로 돌아가서 **process-orders** Stream Analytics 작업을 다시 엽니다.
14. Stream Analytics 작업 페이지 상단에서 **&#11036; 중지** 단추를 사용해 작업을 중지하고, 메시지가 표시되면 확인합니다.

## Azure 리소스 삭제

Azure Stream Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
2. Azure Storage, Event Hubs 및 Stream Analytics 리소스를 포함하는 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다.
3. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
4. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분 후에 이 연습에서 만든 리소스가 삭제됩니다.
