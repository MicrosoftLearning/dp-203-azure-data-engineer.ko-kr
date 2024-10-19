---
lab:
  title: 서버리스 SQL 풀을 사용하여 파일 쿼리
  ilt-use: Lab
---

# 서버리스 SQL 풀을 사용하여 파일 쿼리

SQL은 아마도 전 세계에서 데이터 작업에 가장 많이 사용되는 언어일 것입니다. 대부분의 데이터 분석가는 SQL 쿼리를 사용하여 데이터를 검색, 필터링 및 집계하는 데 능숙하며, 가장 일반적으로 관계형 데이터베이스에서 작업을 수행합니다. 조직에서 스케일링 가능한 파일 스토리지를 활용하여 데이터 레이크를 만드는 경우가 늘어나면서 SQL은 여전히 데이터 쿼리를 위해 선호하는 선택지입니다. Azure Synapse Analytics는 서버리스 SQL 풀을 제공하여 SQL 쿼리 엔진을 데이터 스토리지에서 분리하고 구분된 텍스트 및 Parquet와 같은 일반적인 파일 형식의 데이터 파일에 대해 쿼리를 실행하도록 지원합니다.

이 랩을 완료하는 데 약 **40**분이 걸립니다.

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

4. PowerShell 창에서 다음 명령을 수동으로 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp203/Allfiles/labs/02
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics의 서버리스 SQL 풀](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview) 문서를 검토합니다.

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
8. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 포함된 데이터를 확인합니다. 파일에 머리글 행이 포함되어 있지 않으므로 열 머리글을 표시하는 옵션을 선택 취소할 수 있습니다.
9. 미리 보기를 닫은 다음 **&#8593;** 단추를 사용하여 **sales** 폴더로 다시 이동합니다.
10. **sales** 폴더에서 **json** 폴더를 열면 .json 파일에 몇 가지 샘플 판매 주문이 포함되어 있음을 확인할 수 있습니다. 이러한 파일을 미리 보고 판매 주문에 사용되는 JSON 형식을 확인합니다.
11. 미리 보기를 닫은 다음 **&#8593;** 단추를 사용하여 **sales** 폴더로 다시 이동합니다.
12. **sales** 폴더에서 **parquet** 폴더를 열면 각 연도(2019-2021)에 대한 하위 폴더가 포함되어 있고, 각 폴더에는 **orders.snappy.parquet**라는 파일에 해당 연도의 주문 데이터가 포함되어 있음을 확인할 수 있습니다. 
13. **sales** 폴더로 돌아가서 **csv**, **json** 및 **parquet** 폴더를 볼 수 있도록 합니다.

### SQL을 사용하여 CSV 파일 쿼리

1. **csv** 폴더를 선택한 다음, 도구 모음의 **새 SQL 스크립트** 목록에서 **상위 100행 선택**을 선택합니다.
2. **파일 형식** 목록에서 **텍스트 형식**을 선택한 다음 설정을 적용하여 폴더의 데이터를 쿼리하는 새 SQL 스크립트를 엽니다.
3. 생성된 **SQL Script 1**의 **속성** 창에서 이름을 **Sales CSV 쿼리**로 변경하고 결과 설정을 변경하여 **모든 행**을 표시합니다. 그런 다음 도구 모음에서 **게시**를 선택하여 스크립트를 저장하고 도구 모음의 오른쪽 끝에 있는 **속성** 단추( **&#128463;.** 와 유사함)를 사용하여 **속성** 창을 숨깁니다.
4. 생성된 SQL 코드를 검토하면 다음과 유사해야 합니다.

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    이 코드는 OPENROWSET을 사용하여 sales 폴더의 CSV 파일에서 데이터를 읽고 처음 100개 행의 데이터를 검색합니다.

5. **연결 대상** 목록에서 **기본 제공**이 선택되어 있는지 확인합니다. 작업 영역을 사용하여 만든 기본 제공 SQL 풀을 의미합니다.
6. 도구 모음에서 **&#9655; 실행** 단추를 사용하여 SQL 코드를 실행하고 결과를 검토합니다. 결과는 다음과 같은 형식이어야 합니다

    | C1 | C2 | C3 | C4 | C5 | C6 | C7 | C8 | C9 |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO45347 | 1 | 2020-01-01 | Clarence Raji | clarence35@adventure-works.com |Road-650 Black, 52 | 1 | 699.0982 | 55.9279 |
    | , , | , , | , , | , , | , , | , , | , , | , , | , , |

7. 결과는 C1, C2 등의 열로 구성됩니다. 이 예제에서 CSV 파일에는 열 머리글이 포함되지 않습니다. 할당된 일반 열 이름을 사용하거나 서수 위치를 통해 데이터 작업을 할 수 있지만, 테이블 형식 스키마를 정의한다면 데이터를 더 쉽게 이해할 수 있습니다. 이를 위해서는 여기서 보듯이 WITH 절을 OPENROWSET 함수에 추가하고(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정 이름으로 대체), 쿼리를 다시 실행합니다.

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/csv/',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        )
        WITH (
            SalesOrderNumber VARCHAR(10) COLLATE Latin1_General_100_BIN2_UTF8,
            SalesOrderLineNumber INT,
            OrderDate DATE,
            CustomerName VARCHAR(25) COLLATE Latin1_General_100_BIN2_UTF8,
            EmailAddress VARCHAR(50) COLLATE Latin1_General_100_BIN2_UTF8,
            Item VARCHAR(30) COLLATE Latin1_General_100_BIN2_UTF8,
            Quantity INT,
            UnitPrice DECIMAL(18,2),
            TaxAmount DECIMAL (18,2)
        ) AS [result]
    ```

    이제 결과는 다음과 같은 형식이 됩니다.

    | SalesOrderNumber | SalesOrderLineNumber | OrderDate | CustomerName | EmailAddress | 항목 | 수량 | 단가 | TaxAmount |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | SO45347 | 1 | 2020-01-01 | Clarence Raji | clarence35@adventure-works.com |Road-650 Black, 52 | 1 | 699.10 | 55.93 |
    | , , | , , | , , | , , | , , | , , | , , | , , | , , |

8. 스크립트에 변경 내용을 게시한 다음 스크립트 창을 닫습니다.

### SQL을 사용하여 parquet 파일 쿼리

CSV는 사용하기 쉬운 형식이지만, 빅 데이터 처리 시나리오에서는 압축, 인덱싱 및 분할에 최적화된 파일 형식을 사용하는 것이 일반적입니다. 이러한 형식 중 가장 일반적인 형식은 *parquet*입니다.

1. 데이터 레이크에 대한 파일 시스템을 구성하는 **파일** 탭에서 **sales** 폴더로 돌아가서 **csv**, **json** 및 **parquet** 폴더를 볼 수 있도록 합니다.
2. **parquet** 폴더를 선택한 다음, 도구 모음의 **새 SQL 스크립트** 목록에서 **상위 100행 선택**을 선택합니다.
3. **파일 형식** 목록에서 **Parquet 형식**을 선택한 다음 설정을 적용하여 폴더의 데이터를 쿼리하는 새 SQL 스크립트를 엽니다. 스크립트는 다음과 유사합니다.

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/parquet/**',
            FORMAT = 'PARQUET'
        ) AS [result]
    ```

4. 코드를 실행하면 앞에서 탐색한 CSV 파일과 동일한 스키마로 판매 주문 데이터를 반환합니다. 스키마 정보는 parquet 파일에 포함되므로 적절한 열 이름이 결과에 표시됩니다.
5. 다음과 같이 코드를 수정한 다음(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정의 이름으로 대체) 코드를 실행합니다.

    ```sql
    SELECT YEAR(OrderDate) AS OrderYear,
           COUNT(*) AS OrderedItems
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/parquet/**',
            FORMAT = 'PARQUET'
        ) AS [result]
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear
    ```

6. 결과에는 전체 3년 동안의 주문 수가 포함됩니다. BULK 경로에 사용되는 와일드카드는 쿼리 때문에 모든 하위 폴더에서 데이터가 반환됩니다.

    하위 폴더는 parquet 데이터에 파티션을 반영하는데, 이는 시스템의 성능을 최적화하는 데 자주 사용되는 기술로 여러 데이터 파티션을 병렬로 처리할 수 있습니다.** 파티션을 사용하여 데이터를 필터링할 수도 있습니다.

7. 다음과 같이 코드를 수정한 다음(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정의 이름으로 대체) 코드를 실행합니다.

    ```sql
    SELECT YEAR(OrderDate) AS OrderYear,
           COUNT(*) AS OrderedItems
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/parquet/year=*/',
            FORMAT = 'PARQUET'
        ) AS [result]
    WHERE [result].filepath(1) IN ('2019', '2020')
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear
    ```

8. 결과를 검토하면 2019년과 2020년 판매 수만 포함합니다. 이 필터링은 BULK 경로(*year=\**)의 파티션 폴더 값에 대한 와일드카드와 OPENROWSET에서 반환된 결과의 *filepath* 속성(이 경우 별칭 *[result]*)을 기반으로 하는 WHERE 절을 포함함으로써 수행됩니다.

9. 스크립트의 이름을 **Sales Parquet 쿼리**로 지정하고 게시합니다. 그런 다음 스크립트 창을 닫습니다.

### SQL을 사용하여 JSON 파일 쿼리

JSON은 또 다른 인기 있는 데이터 형식이므로 서버리스 SQL 풀에서 .json 파일을 쿼리하는 것이 유용합니다.

1. 데이터 레이크에 대한 파일 시스템을 구성하는 **파일** 탭에서 **sales** 폴더로 돌아가서 **csv**, **json** 및 **parquet** 폴더를 볼 수 있도록 합니다.
2. **json** 폴더를 선택한 다음, 도구 모음의 **새 SQL 스크립트** 목록에서 **상위 100행 선택**을 선택합니다.
3. **파일 형식** 목록에서 **텍스트 형식**을 선택한 다음 설정을 적용하여 폴더의 데이터를 쿼리하는 새 SQL 스크립트를 엽니다. 스크립트는 다음과 유사합니다.

    ```sql
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/json/',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0'
        ) AS [result]
    ```

    스크립트는 JSON이 아닌 쉼표로 구분된 (CSV) 데이터를 쿼리하도록 설계되었으므로 성공적으로 작동하려면 몇 가지 수정을 수행해야 합니다.

4. 다음과 같이 스크립트를 수정합니다(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정의 이름으로 대체).
    - 파서 버전 매개 변수를 제거합니다.
    - 문자 코드 *0x0b*를 사용하여 필드 종결자, 따옴표 붙은 필드 및 행 종결자에 대한 매개 변수를 추가합니다.
    - 데이터의 JSON 행을 NVARCHAR(MAX) 문자열 포함하는 단일 필드로 결과의 형식을 지정합니다.

    ```sql
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/json/',
            FORMAT = 'CSV',
            FIELDTERMINATOR ='0x0b',
            FIELDQUOTE = '0x0b',
            ROWTERMINATOR = '0x0b'
        ) WITH (Doc NVARCHAR(MAX)) as rows
    ```

5. 수정된 코드를 실행하고 결과에 각 주문에 대한 JSON 문서가 포함되어 있음을 확인합니다.

6. 쿼리를 다음과 같이 수정하여(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정의 이름으로 대체), 쿼리가 JSON_VALUE 함수를 사용하여 JSON 데이터에서 개별 필드 값을 추출하도록 합니다.

    ```sql
    SELECT JSON_VALUE(Doc, '$.SalesOrderNumber') AS OrderNumber,
           JSON_VALUE(Doc, '$.CustomerName') AS Customer,
           Doc
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/json/',
            FORMAT = 'CSV',
            FIELDTERMINATOR ='0x0b',
            FIELDQUOTE = '0x0b',
            ROWTERMINATOR = '0x0b'
        ) WITH (Doc NVARCHAR(MAX)) as rows
    ```

7. 스크립트의 이름을 **Sales JSON 쿼리**로 지정하고 게시합니다. 그런 다음 스크립트 창을 닫습니다.

## 데이터베이스의 외부 데이터 액세스

지금까지 SELECT 쿼리에서 OPENROWSET 함수를 사용하여 데이터 레이크의 파일에서 데이터를 검색했습니다. 쿼리는 서버리스 SQL 풀의 **마스터** 데이터베이스 컨텍스트에서 실행되었습니다. 이 방법은 데이터를 처음 탐색하는 데는 좋지만 더 복잡한 쿼리를 만들려는 경우 Synapse SQL의 *PolyBase* 기능을 사용하여 외부 데이터 위치를 참조하는 데이터베이스에 개체를 만드는 것이 더 효과적일 수 있습니다.

### 외부 데이터 원본 만들기

데이터베이스에서 외부 데이터 원본을 정의하여 파일을 저장하는 데이터 레이크 위치를 참조하는 데 사용할 수 있습니다.

1. Synapse Studio **개발** 페이지의 **+** 메뉴에서 **SQL 스크립트**를 선택합니다.
2. 새 스크립트 창에서 다음 코드(*datalakexxxxxxx*를 데이터 레이크 스토리지 계정의 이름으로 대체)를 추가하여 새 데이터베이스를 만들고 외부 데이터 원본을 추가합니다.

    ```sql
    CREATE DATABASE Sales
      COLLATE Latin1_General_100_BIN2_UTF8;
    GO;

    Use Sales;
    GO;

    CREATE EXTERNAL DATA SOURCE sales_data WITH (
        LOCATION = 'https://datalakexxxxxxx.dfs.core.windows.net/files/sales/'
    );
    GO;
    ```

3. 스크립트 속성을 수정하여 이름을 **Sales DB 만들기**로 변경하고 게시합니다.
4. 스크립트가 **기본 제공** SQL 풀 및 **마스터** 데이터베이스에 연결되어 있는지 확인하고 실행합니다.
5. **데이터** 페이지로 다시 전환하고 Synapse Studio의 오른쪽 위에 있는 **&#8635;** 단추를 사용하여 페이지를 새로 고칩니다. 그런 다음 **데이터** 창의 **작업 영역** 탭을 보면 이제 **SQL 데이터베이스** 목록이 표시됩니다. 이 목록을 확장하여 **Sales** 데이터베이스가 만들어졌는지 확인합니다.
6. **Sales** 데이터베이스, 외부 **리소스** 폴더 및 외부 **데이터 원본** 폴더를 확장하여 만들었던 **sales_data** 외부 데이터 원본을 확인합니다.
7. **Sales** 데이터베이스에 대한 **...** 메뉴에서 **새 SQL 스크립트** > **빈 스크립트**를 선택합니다. 그런 다음 새 스크립트 창에서 다음 쿼리를 입력하고 실행합니다.

    ```sql
    SELECT *
    FROM
        OPENROWSET(
            BULK 'csv/*.csv',
            DATA_SOURCE = 'sales_data',
            FORMAT = 'CSV',
            PARSER_VERSION = '2.0'
        ) AS orders
    ```

    쿼리는 외부 데이터 원본을 사용하여 데이터 레이크에 연결하고 OPENROWSET 함수는 이제 .csv 파일에 대한 상대 경로만 참조하면 됩니다.

8. 다음과 같이 코드를 수정하여 데이터 원본을 사용하여 parquet 파일을 쿼리합니다.

    ```sql
    SELECT *
    FROM  
        OPENROWSET(
            BULK 'parquet/year=*/*.snappy.parquet',
            DATA_SOURCE = 'sales_data',
            FORMAT='PARQUET'
        ) AS orders
    WHERE orders.filepath(1) = '2019'
    ```

### 외부 테이블 만들기

외부 데이터 원본을 사용하면 데이터 레이크의 파일에 더 쉽게 액세스할 수 있지만 SQL을 사용하는 대부분의 데이터 분석가는 데이터베이스의 테이블을 사용하여 작업하는 데 익숙합니다. 다행히 데이터베이스 테이블의 파일에서 행 집합을 캡슐화하는 외부 파일 형식 및 외부 테이블을 정의할 수도 있습니다.

1. SQL 코드를 다음 문으로 바꾸어 CSV 파일에 대한 외부 데이터 형식과 CSV 파일을 참조하는 외부 테이블을 정의하고 실행합니다.

    ```sql
    CREATE EXTERNAL FILE FORMAT CsvFormat
        WITH (
            FORMAT_TYPE = DELIMITEDTEXT,
            FORMAT_OPTIONS(
            FIELD_TERMINATOR = ',',
            STRING_DELIMITER = '"'
            )
        );
    GO;

    CREATE EXTERNAL TABLE dbo.orders
    (
        SalesOrderNumber VARCHAR(10),
        SalesOrderLineNumber INT,
        OrderDate DATE,
        CustomerName VARCHAR(25),
        EmailAddress VARCHAR(50),
        Item VARCHAR(30),
        Quantity INT,
        UnitPrice DECIMAL(18,2),
        TaxAmount DECIMAL (18,2)
    )
    WITH
    (
        DATA_SOURCE =sales_data,
        LOCATION = 'csv/*.csv',
        FILE_FORMAT = CsvFormat
    );
    GO
    ```

2. **데이터** 창에서 **외부 테이블** 폴더를 새로 고치고 확장하고 **Sales** 데이터베이스에 **dbo.orders**라는 테이블이 만들어졌는지 확인합니다.
3. **dbo.orders** 테이블의 **...** 메뉴에서 **새 SQL 스크립트** > **상위 100개 행**을 선택합니다.
4. 생성된 SELECT 스크립트를 실행하고 테이블에서 처음 100개의 데이터 행을 검색하여 데이터 레이크의 파일을 참조하는지 확인합니다.

    >**참고:** 항상 특정 요구 사항 및 사용 사례에 가장 적합한 방법을 선택해야 합니다. 자세한 내용은 [Azure Synapse Analytics에서 서버리스 SQL 풀을 사용하여 OPENROWSET을 사용하는 방법](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-openrowset) 및 [Azure Synapse Analytics에서 서버리스 SQL 풀을 사용하여 외부 스토리지에 액세스](https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-storage-files-overview?tabs=impersonation) 문서에서 확인할 수 있습니다.

## 쿼리 결과 시각화

이제 SQL 쿼리를 사용하여 데이터 레이크의 파일을 쿼리하는 다양한 방법을 살펴보았으므로 이러한 쿼리의 결과를 분석하여 데이터에 대한 인사이트를 얻을 수 있습니다. 인사이트는 차트에서 쿼리 결과를 시각화하면 더 쉽게 파악할 수 있는 경우가 많으며, Synapse Studio 쿼리 편집기에서 통합 차트 기능을 사용하면 쉽게 시각화할 수 있습니다.

1. **개발** 페이지에서 빈 SQL 쿼리를 새로 만듭니다.
2. 스크립트가 **기본 제공** SQL 풀 및 **Sales** 데이터베이스에 연결되어 있는지 확인합니다.
3. 다음 SQL 코드를 입력하고 실행합니다.

    ```sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + TaxAmount) AS GrossRevenue
    FROM dbo.orders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
    ```

4. **결과** 창에서 **차트**를 선택하고 만든 차트를 봅니다. 꺾은선형 차트여야 합니다.
5. 꺾은선형 차트가 2019년부터 2021년까지 3년 동안의 수익 추세를 표시하도록 **범주** 열을 **OrderYear**로 변경합니다.

    ![연도별 수익을 보여 주는 꺾은선형 차트](./images/yearly-sales-line.png)

6. **차트 종류**를 **세로 막대형**으로 전환하여 연간 수익을 세로 막대형 차트로 표시합니다.

    ![연도별 수익을 보여 주는 세로 막대형 차트](./images/yearly-sales-column.png)

7. 쿼리 편집기에서 차트 기능을 실험해 봅니다. 데이터를 대화형으로 탐색하면서 사용할 수 있는 몇 가지 기본 차트 기능을 제공하며, 보고서에 포함할 이미지로 차트를 저장할 수 있습니다. 그러나 Microsoft Power BI와 같은 엔터프라이즈 데이터 시각화 도구에 비해 기능이 제한됩니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역 및 작업 영역용 스토리지 계정이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.