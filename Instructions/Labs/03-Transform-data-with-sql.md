---
lab:
  title: 서버리스 SQL 풀을 사용하여 데이터 변환
  ilt-use: Lab
---

# 서버리스 SQL 풀을 사용하여 파일 변환

데이터 *분석가*는 종종 SQL을 사용하여 분석 및 보고를 위해 데이터를 쿼리합니다. 데이터 *엔지니어*는 SQL을 사용하여 데이터를 조작하고 변환할 수도 있습니다. 종종 데이터 수집 파이프라인 또는 ETL(추출, 변환, 로드) 프로세스의 일부로 사용됩니다.

이 연습에서는 Azure Synapse Analytics의 서버리스 SQL 풀을 사용하여 파일의 데이터를 변환합니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Synapse Analytics 작업 영역 프로비저닝

데이터 레이크 스토리지에 액세스할 수 있는 Azure Synapse Analytics 작업 영역이 필요합니다. 기본 제공 서버리스 SQL 풀을 사용하여 데이터 레이크의 파일을 쿼리할 수 있습니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics 작업 영역을 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 연습의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp-203/Allfiles/labs/03
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Synapse SQL을 사용한 CETAS](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-cetas) 문서를 검토합니다.

## 파일의 데이터 쿼리

스크립트는 Azure Synapse Analytics 작업 영역 및 Azure Storage 계정을 프로비저닝하여 데이터 레이크를 호스트한 다음, 일부 데이터 파일을 데이터 레이크에 업로드합니다.

### 데이터 레이크에서 파일 보기

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **데이터** 페이지에서 **연결된** 탭을 보고 작업 영역에 **synapse*xxxxxxx*(Primary - datalake*xxxxxxx*)**와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
5. 스토리지 계정을 확장하고 **파일**이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
6. **파일** 컨테이너를 선택하고 **sales**라는 폴더가 포함되어 있는지 확인합니다. 이 폴더에는 쿼리할 데이터 파일이 포함되어 있습니다.
7. **sales** 폴더와 해당 폴더에 포함된 **cvs** 폴더를 열고 이 폴더에 3년간의 판매 데이터에 대한 .csv 파일이 포함되어 있는지 확인합니다.
8. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 포함된 데이터를 확인합니다. 파일에는 머리글 행이 포함되어 있습니다.
9. 미리 보기를 닫은 다음 **&#8593;** 단추를 사용하여 **sales** 폴더로 다시 이동합니다.

### SQL을 사용하여 CSV 파일 쿼리

1. **csv** 폴더를 선택한 다음, 도구 모음의 **새 SQL 스크립트** 목록에서 **상위 100행 선택**을 선택합니다.
2. **파일 형식** 목록에서 **텍스트 형식**을 선택한 다음 설정을 적용하여 폴더의 데이터를 쿼리하는 새 SQL 스크립트를 엽니다.
3. 생성된 **SQL Script 1**의 **속성** 창에서 이름을 **Sales CSV 파일 쿼리**로 변경하고 결과 설정을 변경하여 **모든 행**을 표시합니다. 그런 다음, 도구 모음에서 **게시**를 선택하여 스크립트를 저장하고 도구 모음의 오른쪽 끝에 있는 **속성** 단추( **&#128463;<sub>*</sub>** 와 유사함)를 사용하여 **속성** 창을 숨깁니다.
4. 생성된 SQL 코드를 검토하면 다음과 유사해야 합니다.

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    이 코드는 OPENROWSET을 사용하여 sales 폴더의 CSV 파일에서 데이터를 읽고 처음 100개 행의 데이터를 검색합니다.

5. 이 경우 데이터 파일에는 첫 번째 행에 열 이름이 포함됩니다. 따라서 여기에 표시된 대로 쿼리를 수정하여 `OPENROWSET` 절에 `HEADER_ROW = TRUE` 매개 변수를 추가합니다(이전 매개 변수 다음에 쉼표 추가를 잊지 마세요).

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/**',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

6. **연결 대상** 목록에서 **기본 제공**이 선택되어 있는지 확인합니다. 작업 영역을 사용하여 만든 기본 제공 SQL 풀을 의미합니다. 그런 다음, 도구 모음에서 **&#9655; 실행** 단추를 사용하여 SQL 코드를 실행하고 결과를 검토합니다. 결과는 다음과 같은 형식이어야 합니다.

    | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | EmailAddress | 항목 | 수량 | 단가 | TaxAmount |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO43701 | 1 | 2019-07-01 | Christy Zhu | christy12@adventure-works.com |Mountain-100 Silver, 44 | 1 | 3399.99 | 271.9992 |
    | ... | ... | ... | ... | ... | ... | ... | ... | ... |

7. 스크립트에 변경 내용을 게시한 다음 스크립트 창을 닫습니다.

## CETAS(CREATE EXTERNAL TABLE AS SELECT) 문을 사용하여 데이터 변환

SQL을 사용하여 파일의 데이터를 변환하고 결과를 다른 파일에 유지하는 간단한 방법은 CETAS(CREATE EXTERAL TABLE AS SELECT) 문을 사용하는 것입니다. 이 문은 쿼리 요청에 따라 테이블을 만들지만 테이블의 데이터는 데이터 레이크에 파일로 저장됩니다. 그런 다음, 변환된 데이터를 외부 테이블을 통해 쿼리하거나 파일 시스템에서 직접 액세스할 수 있습니다(예: 변환된 데이터를 데이터 웨어하우스로 로드하는 다운스트림 프로세스에 포함).

### 외부 데이터 원본 및 파일 형식 만들기

데이터베이스에서 외부 데이터 원본을 정의하여 외부 테이블에 대한 파일을 저장할 데이터 레이크 위치를 참조하는 데 사용할 수 있습니다. 외부 파일 형식을 사용하면 이러한 파일의 형식(예: Parquet 또는 CSV)을 정의할 수 있습니다. 이러한 개체를 사용하여 외부 테이블을 사용하려면 기본 **master** 데이터베이스가 아닌 데이터베이스에서 만들어야 합니다.

1. Synapse Studio **개발** 페이지의 **+** 메뉴에서 **SQL 스크립트**를 선택합니다.
2. 새 스크립트 창에서 다음 코드(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정의 이름으로 대체)를 추가하여 새 데이터베이스를 만들고 외부 데이터 원본을 추가합니다.

    ```sql
    -- Database for sales data
    CREATE DATABASE Sales
      COLLATE Latin1_General_100_BIN2_UTF8;
    GO;
    
    Use Sales;
    GO;
    
    -- External data is in the Files container in the data lake
    CREATE EXTERNAL DATA SOURCE sales_data WITH (
        LOCATION = 'https://datalakexxxxxxx.dfs.core.windows.net/files/'
    );
    GO;
    
    -- Format for table files
    CREATE EXTERNAL FILE FORMAT ParquetFormat
        WITH (
                FORMAT_TYPE = PARQUET,
                DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
            );
    GO;
    ```

3. 스크립트 속성을 수정하여 이름을 **Sales DB 만들기**로 변경하고 게시합니다.
4. 스크립트가 **기본 제공** SQL 풀 및 **마스터** 데이터베이스에 연결되어 있는지 확인하고 실행합니다.
5. **데이터** 페이지로 다시 전환하고 Synapse Studio의 오른쪽 위에 있는 **&#8635;** 단추를 사용하여 페이지를 새로 고칩니다. 그런 다음 **데이터** 창의 **작업 영역** 탭을 보면 이제 **SQL 데이터베이스** 목록이 표시됩니다. 이 목록을 확장하여 **Sales** 데이터베이스가 만들어졌는지 확인합니다.
6. **Sales** 데이터베이스, 외부 **리소스** 폴더 및 외부 **데이터 원본** 폴더를 확장하여 만들었던 **sales_data** 외부 데이터 원본을 확인합니다.

### 외부 테이블 만들기

1. Synapse Studio **개발** 페이지의 **+** 메뉴에서 **SQL 스크립트**를 선택합니다.
2. 새 스크립트 창에서 다음 코드를 추가하여 외부 데이터 원본을 사용하여 CSV 판매 파일에서 데이터를 검색하고 집계합니다. **BULK** 경로가 데이터 원본이 정의된 폴더 위치를 기준으로 하는지 확인합니다.

    ```sql
    USE Sales;
    GO;
    
    SELECT Item AS Product,
           SUM(Quantity) AS ItemsSold,
           ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

3. 스크립트를 실행합니다. 결과는 다음과 유사합니다.

    | 제품 | ItemsSold | NetRevenue |
    | -- | -- | -- |
    | AWC Logo Cap | 1063 | 8791.86 |
    | ... | ... | ... |

4. 다음과 같이 외부 테이블에 쿼리 결과를 저장하도록 SQL 코드를 수정합니다.

    ```sql
    CREATE EXTERNAL TABLE ProductSalesTotals
        WITH (
            LOCATION = 'sales/productsales/',
            DATA_SOURCE = sales_data,
            FILE_FORMAT = ParquetFormat
        )
    AS
    SELECT Item AS Product,
        SUM(Quantity) AS ItemsSold,
        ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
    FROM
        OPENROWSET(
            BULK 'sales/csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0',
            HEADER_ROW = TRUE
        ) AS orders
    GROUP BY Item;
    ```

5. 스크립트를 실행합니다. 이번에는 출력이 없지만 코드는 쿼리 결과에 따라 외부 테이블을 만들었어야 합니다.
6. 스크립트 이름을 **ProductSalesTotals 테이블 만들기**로 지정하고 게시합니다.
7. **데이터** 페이지의 **작업 영역** 탭에서 **Sales** SQL 데이터베이스에 대한 **외부 테이블** 폴더의 콘텐츠를 확인하여 **ProductSalesTotals**라는 새 테이블이 만들어졌는지 확인합니다.
8. **ProductSalesTotals** 테이블의 **...** 메뉴에서 **새 SQL 스크립트** > **상위 100개 행 선택**을 선택합니다. 그런 다음, 결과 스크립트를 실행하고 집계된 제품 판매 데이터를 반환했는지 확인합니다.
9. 데이터 레이크에 대한 파일 시스템이 포함된 **파일** 탭에서 **sales** 폴더의 콘텐츠를 보고(필요한 경우 보기를 새로 고침) 새 **productsales** 폴더가 만들어졌는지 확인합니다.
10. **productsales** 폴더에서 ABC123DE----.parquet과 유사한 이름의 파일이 하나 이상 생성되었는지 확인합니다. 이러한 파일에는 집계된 제품 판매 데이터가 포함됩니다. 이를 증명하기 위해 파일 중 하나를 선택하고 **새 SQL 스크립트** > **상위 100개 행 선택** 메뉴를 사용하여 직접 쿼리할 수 있습니다.

## 저장 프로시저에서 데이터 변환 캡슐화

데이터를 자주 변환해야 하는 경우 저장 프로시저를 사용하여 CETAS 문을 캡슐화할 수 있습니다.

1. Synapse Studio **개발** 페이지의 **+** 메뉴에서 **SQL 스크립트**를 선택합니다.
2. 새 스크립트 창에서 다음 코드를 추가하여 **Sales** 데이터베이스에 연간 매출을 집계하고 결과를 외부 테이블에 저장하는 저장 프로시저를 만듭니다.

    ```sql
    USE Sales;
    GO;
    CREATE PROCEDURE sp_GetYearlySales
    AS
    BEGIN
        -- drop existing table
        IF EXISTS (
                SELECT * FROM sys.external_tables
                WHERE name = 'YearlySalesTotals'
            )
            DROP EXTERNAL TABLE YearlySalesTotals
        -- create external table
        CREATE EXTERNAL TABLE YearlySalesTotals
        WITH (
                LOCATION = 'sales/yearlysales/',
                DATA_SOURCE = sales_data,
                FILE_FORMAT = ParquetFormat
            )
        AS
        SELECT YEAR(OrderDate) AS CalendarYear,
                SUM(Quantity) AS ItemsSold,
                ROUND(SUM(UnitPrice) - SUM(TaxAmount), 2) AS NetRevenue
        FROM
            OPENROWSET(
                BULK 'sales/csv/*.csv',
                DATA_SOURCE = 'sales_data',
                FORMAT = 'CSV',
                PARSER_VERSION = '2.0',
                HEADER_ROW = TRUE
            ) AS orders
        GROUP BY YEAR(OrderDate)
    END
    ```

3. 스크립트를 실행하여 저장 프로시저를 만듭니다.
4. 방금 실행한 코드에서 다음 코드를 추가하여 저장 프로시저를 호출합니다.

    ```sql
    EXEC sp_GetYearlySales;
    ```

5. 방금 추가한 `EXEC sp_GetYearlySales;` 문만 선택하고 **&#9655; 실행** 단추를 사용하여 실행합니다.
6. 데이터 레이크에 대한 파일 시스템이 포함된 **파일** 탭에서 **sales** 폴더의 콘텐츠를 보고(필요한 경우 보기를 새로 고침) 새 **yearlysales** 폴더가 만들어졌는지 확인합니다.
7. **yearlysales** 폴더에서 집계된 연간 판매 데이터가 포함된 parquet 파일이 만들어졌는지 확인합니다.
8. SQL 스크립트로 다시 전환하고, `EXEC sp_GetYearlySales;` 문을 다시 실행하고, 오류가 발생하는지 확인합니다.

    스크립트가 외부 테이블을 삭제하더라도 데이터가 포함된 폴더는 삭제되지 않습니다. 저장 프로시저를 다시 실행하려면(예: 예약된 데이터 변환 파이프라인의 일부로) 이전 데이터를 삭제해야 합니다.

9. **파일** 탭으로 다시 전환하고 **sales** 폴더를 봅니다. 그런 다음, **yearlysales** 폴더를 선택하고 삭제합니다.
10. SQL 스크립트로 다시 전환하고 `EXEC sp_GetYearlySales;` 문을 다시 실행합니다. 이번에는 작업이 성공하고 새 데이터 파일이 생성됩니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역 및 작업 영역용 스토리지 계정이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
