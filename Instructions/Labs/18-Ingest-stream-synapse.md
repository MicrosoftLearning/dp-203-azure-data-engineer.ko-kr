---
lab:
  title: Azure Stream Analytics 및 Azure Synapse Analytics를 사용하여 실시간 데이터 수집
  ilt-use: Lab
---

# Azure Stream Analytics 및 Azure Synapse Analytics를 사용하여 실시간 데이터 수집

데이터 분석 솔루션에는 종종 데이터 *스트림*을 수집하고 처리해야 하는 요구 사항이 포함됩니다. 스트림 처리는 스트림이 일반적으로 *무한*하다는 점에서 일괄 처리와 다릅니다. 즉, 고정 간격이 아닌 영구적으로 처리해야 하는 데이터의 연속 원본입니다.

Azure Stream Analytics는 Azure Event Hubs 또는 Azure IoT Hub 같은 스트리밍 원본의 데이터 스트림에서 작동하는 *쿼리*를 정의하는 데 사용할 수 있는 클라우드 서비스를 제공합니다. Azure Stream Analytics 쿼리를 사용하여 추가 분석을 위해 데이터 저장소에 직접 데이터 스트림을 수집하거나 임시 창을 기반으로 데이터를 필터링, 집계 및 요약할 수 있습니다.

이 연습에서는 Azure Stream Analytics를 사용하여 온라인 소매 애플리케이션에서 생성될 수 있는 것과 같은 판매 주문 데이터의 스트림을 처리합니다. 주문 데이터는 Azure Stream Analytics 작업이 데이터를 읽고 Azure Synapse Analytics로 수집하는 Azure Event Hubs로 전송됩니다.

이 연습을 완료하는 데 약 **45**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure 리소스 프로비전

이 연습에서는 데이터 레이크 스토리지 및 전용 SQL 풀에 액세스할 수 있는 Azure Synapse Analytics 작업 영역이 필요합니다. 스트리밍 주문 데이터를 보낼 수 있는 Azure Event Hubs 네임스페이스도 필요합니다.

PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 이러한 리소스를 프로비전합니다.

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
    cd dp-203/Allfiles/labs/18
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요!

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 15분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Stream Analytics 설명서에서 [Azure Stream Analytics에 오신 것을 환영합니다](https://learn.microsoft.com/azure/stream-analytics/stream-analytics-introduction) 문서를 검토합니다.

## 전용 SQL 풀로 스트리밍 데이터 수집

먼저 Azure Synapse Analytics 전용 SQL 풀의 테이블에 데이터 스트림을 직접 수집해 보겠습니다.

### 스트리밍 소스 및 데이터베이스 테이블 보기

1. 설치 스크립트 실행이 완료되면 Cloud Shell 창을 최소화합니다(나중에 다시 돌아가게 됨). 그런 다음 Azure Portal에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 이 리소스 그룹에 Azure Synapse 작업 영역, 데이터 레이크에 대한 스토리지 계정, 전용 SQL 풀 및 Event Hubs 네임스페이스가 포함되어 있음을 확인합니다.
2. Synapse 작업 영역을 선택하고 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio를 엽니다. Synapse Studio는 Synapse Analytics 작업 영역에서 사용할 수 있는 웹 기반 인터페이스입니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **관리** 페이지의 **SQL 풀** 섹션에서 **sql*xxxxxxx*** 전용 SQL 풀 행을 선택한 다음, **&#9655;** 아이콘을 사용하여 다시 시작합니다.
5. SQL 풀이 시작될 때까지 기다리는 동안 Azure Portal에 포함된 브라우저 탭으로 다시 전환하고 Cloud Shell 창을 다시 엽니다.
6. Cloud Shell 창에서 다음 명령을 입력하여 100개의 시뮬레이션된 주문을 Azure Event Hubs에 보내는 클라이언트 앱을 실행합니다.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

7. 전송되는 주문 데이터를 관찰합니다. 각 주문은 제품 ID와 수량으로 구성됩니다.
8. 주문 클라이언트 앱이 완료되면 Cloud Shell 창을 최소화하고 Synapse Studio 브라우저 탭으로 다시 전환합니다.
9. Synapse Studio **관리** 페이지에서 전용 SQL 풀의 상태가 **온라인**인지 확인하고 **데이터** 페이지로 전환한 다음, **작업 영역** 창에서 **SQL 데이터베이스**, **sql*xxxxxxx*** SQL 풀 및 **테이블**을 확장하여 **dbo.FactOrder** 테이블을 확인합니다.
10. **dbo.FactOrder** 테이블의 **...** 메뉴에서 **새 SQL 스크립트** > **상위 100 행 선택**을 선택하고 결과를 검토합니다. 테이블에 **OrderDateTime**, **ProductID** 및 **수량**에 대한 열이 포함되어 있지만 현재 데이터 행이 없는지 확인합니다.

### 주문 데이터를 수집하는 Azure Stream Analytics 작업 만들기

1. Azure Portal을 포함하는 브라우저 탭으로 다시 전환하고 **db000-*xxxxxxx*** 리소스 그룹이 프로비전된 지역을 기록해 둡니다. <u>동일한 지역</u>에 Stream Analytics 작업을 만듭니다.
2. **홈**페이지에서 **+ 리소스 만들기**를 선택하고 `Stream Analytics job`을 검색합니다. 그런 다음, 다음 속성을 사용하여 **Stream Analytics 작업**을 만듭니다.
    - **기본 사항**:
        - **구독**: ‘Azure 구독’
        - **리소스 그룹**: 기존 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다.
        - **이름**: `ingest-orders`
        - **지역**: Synapse Analytics 작업 영역이 프로비전되는 <u>동일한 지역</u>을 선택합니다.
        - **호스팅 환경**: 클라우드
        - **스트리밍 단위**: 1
    - **스토리지**:
        - **스토리지 계정에서 비공개 데이터 보호**: 선택됨
        - **구독**: ‘Azure 구독’
        - **스토리지 계정 유형**: 스토리지 계정
        - **스토리지**: **datalake*xxxxxxx*** 스토리지 계정을 선택합니다.
        - **인증 모드:** 연결 문자열
    - **태그**:
        - *없음*
3. 배포가 완료될 때까지 기다렸다가 배포된 Stream Analytics 작업 리소스로 이동합니다.

### 이벤트 데이터 스트림에 대한 입력 만들기

1. **ingest-orders** 개요 페이지에서 **입력 추가**를 선택합니다. 그런 다음, **입력** 페이지에서 **스트림 입력 추가** 메뉴를 사용해 다음 속성을 가진 **Event Hub** 입력을 추가합니다.
    - **입력 별칭**: `orders`
    - **구독에서 Event Hub 선택:** 선택됨
    - **구독**: ‘Azure 구독’
    - **Event Hub 네임스페이스**: **이벤트*xxxxxxx*** Event Hubs 네임스페이스 선택
    - **Event Hub 이름**: 기존 **eventhub*xxxxxxx*** 이벤트 허브를 선택합니다.
    - **Event Hub 소비자 그룹**: 기존 **$Default** 소비자 그룹 선택
    - **인증 모드**: 시스템 할당 관리 ID 만들기
    - **파티션 키**: *공백으로 둠*
    - **이벤트 직렬화 형식**: JSON
    - **인코딩**: UTF-8
2. 입력을 저장하고 입력이 만들어지는 동안 기다립니다. 여러 알림이 표시됩니다. **연결 테스트 성공** 알림을 기다립니다.

### SQL 테이블에 대한 출력 만들기

1. **ingest-orders** Stream Analytics 작업에 대한 **출력** 페이지를 봅니다. 그런 다음 **추가** 메뉴에서 다음 속성을 사용해 **Azure Synapse Analytics** 출력을 추가합니다.
    - **출력 별칭:** `FactOrder`
    - **구독에서 Azure Synapse Analytics 선택:** 선택됨
    - **구독**: ‘Azure 구독’
    - **데이터베이스**: **sql*xxxxxxx*(synapse*xxxxxxx *)* * 데이터베이스 선택
    - **인증 모드**: SQL 서버 인증
    - **사용자 이름**: SQLUser
    - **암호**: *설치 스크립트를 실행할 때 SQL 풀에 대해 지정한 암호*
    - **테이블**: `FactOrder`
2. 출력을 저장하고 출력이 만들어지는 동안 기다립니다. 여러 알림이 표시됩니다. **연결 테스트 성공** 알림을 기다립니다.

### 이벤트 스트림을 수집하는 쿼리 만들기

1. **ingest-orders** Stream Analytics 작업에 대한 **쿼리** 페이지를 봅니다. 그런 다음 입력 미리 보기가 표시될 때까지 잠시 기다립니다(이벤트 허브에서 이전에 캡처한 판매 주문 이벤트에 따라).
2. 입력 데이터에는 클라이언트 앱에서 제출한 메시지의 **ProductID** 및 **Quantity** 필드와 이벤트가 이벤트 허브에 추가된 시기를 나타내는 **EventProcessedUtcTime** 필드를 비롯한 추가 Event Hubs 필드가 포함됩니다.
3. 기본 쿼리를 다음과 같이 수정합니다.

    ```
    SELECT
        EventProcessedUtcTime AS OrderDateTime,
        ProductID,
        Quantity
    INTO
        [FactOrder]
    FROM
        [orders]
    ```

    이 쿼리는 입력(이벤트 허브)에서 필드를 가져와 출력(SQL 테이블)에 직접 쓰는지 확인합니다.

4. 쿼리를 저장합니다.

### 주문 데이터 수집을 위해 스트리밍 작업 실행

1. **ingest-orders** Stream Analytics 작업의 **개요** 페이지를 보고 **속성** 탭에서 작업에 대한 **입력**, **쿼리**, **출력**, **함수**를 검토합니다. **입력** 및 **출력** 수가 0인 경우 **개요** 페이지의 **&#8635; 새로 고침** 버튼을 사용하여 **주문** 입력 및 **FactTable** 출력을 표시합니다.
2. **&#9655; 시작** 버튼을 선택하고 지금 스트리밍 작업을 시작합니다. 스트리밍 작업이 성공적으로 시작되었다는 알림이 표시될 때까지 기다립니다.
3. Cloud Shell 창을 다시 열고 다음 명령을 다시 실행하여 또 다른 100개의 주문을 제출합니다.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4. 주문 클라이언트 앱이 실행되는 동안 Synapse Studio 브라우저 탭으로 전환하고 이전에 실행한 쿼리를 확인하여 **dbo.FactOrder** 테이블에서 TOP 100 행을 선택합니다.
5. **&#9655; 실행** 버튼을 사용해 쿼리를 다시 실행하고 테이블에 이벤트 스트림의 주문 데이터가 포함되어 있는지 확인합니다(그렇지 않은 경우 잠시 기다렸다가 쿼리를 다시 실행). Stream Analytics 작업은 작업이 실행되고 순서 이벤트가 이벤트 허브로 전송되는 한 새 이벤트 데이터를 테이블로 모두 푸시합니다.
6. **관리** 페이지에서 **sql*xxxxxxx*** 전용 SQL 풀을 일시 중지합니다(불필요한 Azure 요금을 방지하기 위해).
7. Azure Portal이 포함된 브라우저 탭으로 돌아가 Cloud Shell 창을 최소화합니다. 그런 다음 **&#128454; 정지** 버튼을 사용해 Stream Analytics 작업을 중지하고 Stream Analytics 작업이 성공적으로 중지되었다는 알림을 기다립니다.

## 데이터 레이크의 스트리밍 데이터 요약

지금까지 Stream Analytics 작업을 사용하여 스트리밍 소스에서 SQL 테이블로 메시지를 수집하는 방법을 살펴보았습니다. 이제 Azure Stream Analytics를 사용하여 임시 창을 통해 데이터를 집계하는 방법을 살펴보겠습니다. 이 경우 5초마다 판매되는 각 제품의 총 수량을 계산합니다. 또한 데이터 레이크 Blob 저장소에 CSV 형식으로 결과를 작성하여 작업에 다른 종류의 출력을 사용하는 방법도 살펴봅니다.

### 주문 데이터를 집계하는 Azure Stream Analytics 작업 만들기

1. Azure Portal **홈**페이지에서 **+ 리소스 만들기**를 선택하고 `Stream Analytics job`을 검색합니다. 그런 다음, 다음 속성을 사용하여 **Stream Analytics 작업**을 만듭니다.
    - **기본 사항**:
        - **구독**: ‘Azure 구독’
        - **리소스 그룹**: 기존 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다.
        - **이름**: `aggregate-orders`
        - **지역**: Synapse Analytics 작업 영역이 프로비전되는 <u>동일한 지역</u>을 선택합니다.
        - **호스팅 환경**: 클라우드
        - **스트리밍 단위**: 1
    - **스토리지**:
        - **스토리지 계정에서 비공개 데이터 보호**: 선택됨
        - **구독**: ‘Azure 구독’
        - **스토리지 계정 유형**: 스토리지 계정
        - **스토리지**: **datalake*xxxxxxx*** 스토리지 계정을 선택합니다.
        - **인증 모드:** 연결 문자열
    - **태그**:
        - *없음*

2. 배포가 완료될 때까지 기다렸다가 배포된 Stream Analytics 작업 리소스로 이동합니다.

### 원시 주문 데이터에 대한 입력 만들기

1. **aggregate-orders** 개요 페이지에서 **입력 추가**를 선택합니다. 그런 다음, **입력** 페이지에서 **스트림 입력 추가** 메뉴를 사용해 다음 속성을 가진 **Event Hub** 입력을 추가합니다.
    - **입력 별칭**: `orders`
    - **구독에서 Event Hub 선택:** 선택됨
    - **구독**: ‘Azure 구독’
    - **Event Hub 네임스페이스**: **이벤트*xxxxxxx*** Event Hubs 네임스페이스 선택
    - **Event Hub 이름**: 기존 **eventhub*xxxxxxx*** 이벤트 허브를 선택합니다.
    - **Event Hub 소비자 그룹**: 기존 **$Default** 소비자 그룹 선택
    - **인증 모드**: 시스템 할당 관리 ID 만들기
    - **파티션 키**: *공백으로 둠*
    - **이벤트 직렬화 형식**: JSON
    - **인코딩**: UTF-8
2. 입력을 저장하고 입력이 만들어지는 동안 기다립니다. 여러 알림이 표시됩니다. **연결 테스트 성공** 알림을 기다립니다.

### 데이터 레이크 저장소에 대한 출력 만들기

1. **aggregate-orders** Stream Analytics 작업에 대한 **출력** 페이지를 봅니다. 그런 다음, **추가** 메뉴에서 다음 속성을 사용하여 **Blob 스토리지/ADLS Gen2** 출력을 추가합니다.
    - **출력 별칭:** `datalake`
    - **구독에서 Blob 스토리지/ADLS Gen2 선택을 선택**: 선택됨
    - **구독**: ‘Azure 구독’
    - **스토리지 계정**: **datalake*xxxxxxx*** 스토리지 계정을 선택합니다.
    - **컨테이너**: 기존 **파일** 컨테이너 선택
    - **인증 모드:** 연결 문자열
    - **이벤트 직렬화 형식**: CSV - 콤마(,)
    - **인코딩**: UTF-8
    - **경로 패턴**: `{date}`
    - **날짜 형식**: YYYY/MM/DD
    - **시간 형식**: *해당 없음*
    - **최소 행**: 20
    - **최대 시간**: 0시간, 1분, 0초
2. 출력을 저장하고 출력이 만들어지는 동안 기다립니다. 여러 알림이 표시됩니다. **연결 테스트 성공** 알림을 기다립니다.

### 이벤트 데이터를 집계하는 쿼리 만들기

1. **aggregate-orders** Stream Analytics 작업에 대한 **쿼리** 페이지를 봅니다.
2. 기본 쿼리를 다음과 같이 수정합니다.

    ```
    SELECT
        DateAdd(second,-5,System.TimeStamp) AS StartTime,
        System.TimeStamp AS EndTime,
        ProductID,
        SUM(Quantity) AS Orders
    INTO
        [datalake]
    FROM
        [orders] TIMESTAMP BY EventProcessedUtcTime
    GROUP BY ProductID, TumblingWindow(second, 5)
    HAVING COUNT(*) > 1
    ```

    이 쿼리는 **System.Timestamp**(**EventProcessedUtcTime** 필드 기반)를 사용하여 각 제품 ID의 총 수량이 계산되는 각 5초 *연속*(겹치지 않는 순차적) 창의 시작과 끝을 정의하는지 확인합니다.

3. 쿼리를 저장합니다.

### 스트리밍 작업을 실행하여 주문 데이터 집계

1. **aggregate-orders** Stream Analytics 작업의 **개요** 페이지를 보고 **속성** 탭에서 작업에 대한 **입력**, **쿼리**, **출력**, **함수**를 검토합니다. **입력** 및 **출력** 수가 0인 경우 **개요** 페이지의 **&#8635; 새로 고침** 버튼을 사용해 **주문** 입력 및 **데이터 레이크** 출력을 표시합니다.
2. **&#9655; 시작** 버튼을 선택하고 지금 스트리밍 작업을 시작합니다. 스트리밍 작업이 성공적으로 시작되었다는 알림이 표시될 때까지 기다립니다.
3. Cloud Shell 창을 다시 열고 다음 명령을 다시 실행하여 다음과 같이 또 다른 100개의 주문을 제출합니다.

    ```
    node ~/dp-203/Allfiles/labs/18/orderclient
    ```

4.  주문 앱이 완료되면 Cloud Shell 창을 최소화합니다. 그런 다음, Synapse Studio 브라우저 탭으로 전환하고 **데이터** 페이지의 **연결됨** 탭에서 **Azure Data Lake Storage Gen2** > **synapse*xxxxxxx*(기본 - datalake*xxxxxxx *)* *를 확장하고 **파일(기본)** 컨테이너를 선택합니다.
5. **파일** 컨테이너가 비어 있는 경우 잠시 기다린 다음, **&#8635; 새로 고침**을 사용해 보기를 새로 고칩니다. 결국 현재 연도로 명명된 폴더를 표시해야 합니다. 그러면 월 및 일에 대한 폴더가 포함됩니다.
6. 연도의 폴더를 선택하고 **새 SQL 스크립트** 메뉴에서 **상위 100개 행 선택**을 선택합니다. 그런 다음, **파일 형식**을 **텍스트 형식**으로 설정하고 설정을 적용합니다.
7. 열린 쿼리 창에서 여기에 표시된 대로 쿼리를 수정하여 `HEADER_ROW = TRUE` 매개 변수를 추가합니다.

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/2023/**',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

8. **&#9655; 실행** 버튼을 사용해 SQL 쿼리를 실행하고 결과를 확인합니다. 이는 5초 동안 주문된 각 제품의 수량을 표시합니다.
8. Azure Portal이 포함된 브라우저 탭으로 돌아가 **&#128454; 중지** 버튼을 사용해 Stream Analytics 작업을 중지하고 Stream Analytics 작업이 성공적으로 중지되었다는 알림을 기다립니다.

## Azure 리소스 삭제

Azure Stream Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Azure Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. Azure Synapse, Event Hubs 및 Stream Analytics 리소스를 포함한 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다(관리되는 리소스 그룹 아님).
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분 후에 이 연습에서 만든 리소스가 삭제됩니다.
