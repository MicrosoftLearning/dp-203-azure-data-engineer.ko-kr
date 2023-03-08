---
lab:
  title: 관계형 데이터 웨어하우스에 데이터 로드
  ilt-use: Lab
---

# 관계형 데이터 웨어하우스에 데이터 로드

이 연습에서는 전용 SQL 풀에 데이터를 로드합니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Synapse Analytics 작업 영역 프로비저닝

데이터 레이크 스토리지 및 데이터 웨어하우스를 호스트하는 전용 SQL 풀에 액세스할 수 있는 Azure Synapse Analytics 작업 영역이 필요합니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics 작업 영역을 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. Cloud Shell은 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 —, **&#9723;** , **X** 아이콘을 사용하여 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 연습의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```powershell
    cd dp-203/Allfiles/labs/09
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(이 옵션은 여러 Azure 구독에 액세스할 수 있는 경우에만 발생함).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 대해 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요!

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics의 전용 SQL 풀에 대한 데이터 로드 전략](https://learn.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading) 문서를 검토합니다.

## 데이터 로드 준비

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요 페이지**의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 ›› 아이콘을 사용하여 메뉴를 확장하면 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 Synapse Studio 내의 다른 페이지가 표시됩니다.
4. **관리** 페이지의 **SQL 풀** 탭에서 이 연습에 대한 데이터 웨어하우스를 호스트하는 **sql*xxxxxxx*** 전용 SQL 풀의 행을 선택하고 **&#9655;** 아이콘을 사용하여 시작합니다. 메시지가 표시되면 다시 시작하려는지 확인합니다.

    풀을 다시 시작하는 데 몇 분 정도 걸릴 수 있습니다. **&#8635; 새로 고침** 단추를 사용하여 상태를 주기적으로 확인할 수 있습니다. 준비되면 상태가 **온라인**으로 표시됩니다. 기다리는 동안 아래 단계를 진행하여 로드할 데이터 파일을 확인합니다.

5. **데이터** 페이지에서 **연결됨** 탭을 보고 작업 영역에 **synapsexxxxxxx(Primary - datalakexxxxxxx)** 와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
6. 스토리지 계정을 확장하고 **파일(기본)** 이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
7. 파일 컨테이너를 선택하고 **데이터**라는 폴더가 포함되어 있는지 확인합니다. 이 폴더에는 데이터 웨어하우스에 로드할 데이터 파일이 포함되어 있습니다.
8. **데이터** 폴더를 열고 고객 및 제품 데이터의 .csv 파일이 포함되어 있는지 확인합니다.
9. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 포함된 데이터를 확인합니다. 파일에 머리글 행이 포함되어 있으므로 열 머리글을 표시하는 옵션을 선택할 수 있습니다.
10. **관리** 페이지로 돌아가 전용 SQL 풀이 온라인인지 확인합니다.

## 데이터 웨어하우스 테이블 로드

Data Warehouse에 데이터를 로드하는 몇 가지 SQL 기반 방법을 살펴보겠습니다.

1. **데이터** 페이지에서 **작업 공간** 탭을 선택합니다.
2. **SQL Database**를 확장하고 **sql*xxxxxxx*** 데이터베이스를 선택합니다. 그런 다음, **...** 메뉴에서 **새 SQL 스크립트** > 
**빈 스크립트**를 선택합니다.

이제 다음 연습의 인스턴스에 연결된 빈 SQL 페이지가 있습니다. 이 스크립트를 사용하여 데이터를 로드하는 데 사용할 수 있는 몇 가지 SQL 기술을 살펴봅니다.

### COPY 문을 사용하여 데이터 레이크에서 데이터 로드

1. SQL 스크립트에서 창에 다음 코드를 입력합니다.

    ```sql
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

2. 도구 모음에서 **&#9655; 실행** 단추를 사용하여 SQL 코드를 실행하고 현재 **StageProduct** 테이블에 **0**개의 행이 있는지 확인합니다.
3. 코드를 다음 COPY 문으로 바꿉니다(**datalake*xxxxxx***을 데이터 레이크 이름으로 변경).

    ```sql
    COPY INTO dbo.StageProduct
        (ProductID, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
    FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/Product.csv'
    WITH
    (
        FILE_TYPE = 'CSV',
        MAXERRORS = 0,
        IDENTITY_INSERT = 'OFF',
        FIRSTROW = 2 --Skip header row
    );


    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

4. 스크립트를 실행하고 결과를 검토합니다. 11개의 행이 **StageProduct** 테이블에 로드되어야 합니다.

    이제 동일한 기술을 사용하여 다른 테이블을 로드해 보겠습니다. 이번에는 발생할 수 있는 오류를 로깅합니다.

5. 스크립트 창의 SQL 코드를 다음 코드로 바꾸고 **datalake*xxxxxx***을 ```FROM``` 및 ```ERRORFILE``` 절 모두에서 데이터 레이크 이름으로 변경합니다.

    ```sql
    COPY INTO dbo.StageCustomer
    (GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
    MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
    SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
    NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
    FROM 'https://datalakexxxxxx.dfs.core.windows.net/files/data/Customer.csv'
    WITH
    (
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 5
    ,FIRSTROW = 2 -- skip header row
    ,ERRORFILE = 'https://datalakexxxxxx.dfs.core.windows.net/files/'
    );
    ```

6. 스크립트를 실행하고 결과 메시지를 검토합니다. 원본 파일에 잘못된 데이터가 있는 행이 포함되어 있으므로 한 행이 거부됩니다. 위의 코드는 최대 **5**개의 오류를 지정하므로 단일 오류로 인해 유효한 행이 로드되지 않아야 합니다. 다음 쿼리를 실행하여 *로드된* 행을 볼 수 있습니다.

    ```sql
    SELECT *
    FROM dbo.StageCustomer
    ```

7. **파일** 탭에서 데이터 레이크의 루트 폴더를 보고 **_rejectedrows**라는 새 폴더가 만들어졌는지 확인합니다(이 폴더가 표시되지 않으면 **자세히** 메뉴에서 **새로 고침**을 선택하여 보기를 새로 고침).
8. **_rejectedrows** 폴더와 폴더에 포함된 날짜 및 시간별 하위 폴더를 열고 이름이 ***QID123_1_2*.Error.Txt** 및 ***QID123_1_2*.Row.Txt**와 유사한 파일이 만들어졌는지 확인합니다. 이러한 각 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 오류 및 거부된 행의 세부 정보를 볼 수 있습니다.

    준비 테이블을 사용하면 데이터를 이동하거나 사용하여 기존 차원 테이블에 추가하거나 upsert하기 전에 데이터의 유효성을 검사하거나 변환할 수 있습니다. COPY 문은 데이터 레이크의 파일에서 준비 테이블로 데이터를 쉽게 로드하는 데 사용할 수 있는 간단하지만 고성능 기술을 제공하며 앞에서 살펴본 것처럼 잘못된 행을 식별하고 리디렉션합니다.

### CTAS(CREATE TABLE AS) 문 사용

1. 스크립트 창으로 돌아가서 포함된 코드를 다음 코드로 바꿉니다.

    ```sql
    CREATE TABLE dbo.DimProduct
    WITH
    (
        DISTRIBUTION = HASH(ProductAltKey),
        CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
        ProductID AS ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.StageProduct;
    ```

2. 스크립트를 실행하여 **ProductAltKey**를 해시 배포 키로 사용하고 클러스터형 columnstore 인덱스가 있는 준비된 제품 데이터에서 **DimProduct**라는 새 테이블을 만듭니다.
4. 다음 쿼리를 사용하여 새 **DimProduct** 테이블의 콘텐츠를 봅니다.

    ```sql
    SELECT ProductKey,
        ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.DimProduct;
    ```

    CTAS(CREATE TABLE AS SELECT) 식에는 다음과 같은 다양한 용도가 있습니다.

    - 쿼리 성능을 향상하기 위해 다른 테이블과 일치하도록 테이블의 해시 키를 재배포합니다.
    - 델타 분석을 수행한 후 기존 값을 기반으로 준비 테이블에 서로게이트 키를 할당합니다.
    - 보고서를 위해 집계 테이블을 빠르게 만듭니다.

### INSERT 문과 UPDATE 문을 결합하여 천천히 변화하는 차원 테이블 로드

**DimCustomer** 테이블은 형식 1 및 형식 2 SCD(느린 변경 차원)를 지원합니다. 여기서 형식 1은 기존 행에 대한 현재 위치 업데이트가 발생하며, 형식 2 변경으로 인해 특정 차원 엔터티 인스턴스의 최신 버전을 나타내는 새 행이 생성됩니다. 이 테이블을 로드하려면 INSERT 문(새 고객을 로드하기 위해) 및 UPDATE 문(형식 1 또는 형식 2 변경을 적용하기 위해)의 조합이 필요합니다.

1. 쿼리 창에서 기존 SQL 코드를 다음 코드로 바꿉니다.

    ```sql
    INSERT INTO dbo.DimCustomer ([GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
    [Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
    [EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
    [DateFirstPurchase],[CommuteDistance])
    SELECT *
    FROM dbo.StageCustomer AS stg
    WHERE NOT EXISTS
        (SELECT * FROM dbo.DimCustomer AS dim
        WHERE dim.CustomerAlternateKey = stg.CustomerAlternateKey);

    -- Type 1 updates (change name, email, or phone in place)
    UPDATE dbo.DimCustomer
    SET LastName = stg.LastName,
        EmailAddress = stg.EmailAddress,
        Phone = stg.Phone
    FROM DimCustomer dim inner join StageCustomer stg
    ON dim.CustomerAlternateKey = stg.CustomerAlternateKey
    WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

    -- Type 2 updates (address changes triggers new entry)
    INSERT INTO dbo.DimCustomer
    SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
    stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
    stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
    stg.DateFirstPurchase,stg.CommuteDistance
    FROM dbo.StageCustomer AS stg
    JOIN dbo.DimCustomer AS dim
    ON stg.CustomerAlternateKey = dim.CustomerAlternateKey
    AND stg.AddressLine1 <> dim.AddressLine1;
    ```

2. 스크립트를 실행하고 출력을 검토합니다.

## 로드 후 최적화 수행

데이터 웨어하우스에 새 데이터를 로드한 후에는 테이블 인덱스를 다시 빌드하고 일반적으로 쿼리되는 열에 대한 통계를 업데이트하는 것이 좋습니다.

1. 스크립트 창의 코드를 다음 코드로 바꿉니다.

    ```sql
    ALTER INDEX ALL ON dbo.DimProduct REBUILD;
    ```

2. 스크립트를 실행하여 **DimProduct** 테이블의 인덱스를 다시 빌드합니다.
3. 스크립트 창의 코드를 다음 코드로 바꿉니다.

    ```sql
    CREATE STATISTICS customergeo_stats
    ON dbo.DimCustomer (GeographyKey);
    ```

4. 스크립트를 실행하여 **DimCustomer** 테이블의 **GeographyKey** 열에 대한 통계를 만들거나 업데이트합니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정, 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
