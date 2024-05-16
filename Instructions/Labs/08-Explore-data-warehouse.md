---
lab:
  title: 관계형 데이터 웨어하우스 탐색
  ilt-use: Suggested demo
---

# 관계형 데이터 웨어하우스 탐색

Azure Synapse Analytics는 데이터 레이크의 파일 기반 데이터 분석뿐만 아니라 대규모 관계형 데이터 웨어하우스와 이를 로드하는 데 사용되는 데이터 전송 및 변환 파이프라인을 포함하여 엔터프라이즈 데이터 웨어하우징을 지원하는 확장 가능한 집합 기능을 기반으로 합니다. 이 랩에서는 Azure Synapse Analytics의 전용 SQL 풀을 사용하여 관계형 데이터 웨어하우스에 데이터를 저장하고 쿼리하는 방법을 살펴봅니다.

이 랩을 완료하는 데 약 **45**분이 걸립니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Synapse Analytics 작업 영역 프로비저닝

Azure Synapse Analytics 작업 영역은 데이터 및 데이터 처리 런타임을 관리하기 위한 중앙 지점을 제공합니다. Azure Portal에서 대화형 인터페이스를 사용하여 작업 영역을 프로비저닝하거나 스크립트 또는 템플릿을 사용하여 작업 영역 및 리소스를 배포할 수 있습니다. 대부분의 프로덕션 시나리오에서는 리소스 배포를 반복 가능한 개발 및 작업(*DevOps*) 프로세스에 통합할 수 있도록 스크립트와 템플릿을 사용하여 프로비저닝을 자동화하는 것이 가장 좋습니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics를 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp500 -f
    git clone https://github.com/MicrosoftLearning/DP-500-Azure-Data-Analyst dp500
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp500/Allfiles/03
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요!

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 15분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics의 전용 SQL 풀이란?](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is) 문서를 검토합니다.

## 데이터 웨어하우스 스키마 살펴보기

이 랩에서 데이터 웨어하우스는 Azure Synapse Analytics의 전용 SQL 풀에서 호스트됩니다.

### 전용 SQL 풀 시작

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp500-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **관리** 페이지에서 **SQL 풀** 탭이 선택되어 있는지 확인한 다음, **sql*xxxxxxx*** 전용 SQL 풀을 선택하고 **&#9655;** 아이콘을 사용하여 시작합니다. 메시지가 표시되면 다시 시작하려는지 확인합니다.
5. SQL 풀이 다시 시작될 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다. **&#8635; 새로 고침** 단추를 사용하여 상태를 주기적으로 확인합니다. 준비되면 상태가 **온라인**으로 표시됩니다.

### 데이터베이스의 테이블 보기

1. Synapse Studio에서 **데이터** 페이지를 선택하고 **작업 영역** 탭이 선택되어 있고 **SQL 데이터베이스** 범주가 포함되어 있는지 확인합니다.
2. **SQL 데이터베이스**, **sql*xxxxxxx*** 풀, 해당 **Tables** 폴더를 확장하여 데이터베이스의 테이블을 확인합니다.

    관계형 데이터 웨어하우스는 일반적으로 팩트 테이블과 차원 테이블로 구성된 스키마를 기반으로 합니다.**** 테이블은 팩트 테이블의 숫자 메트릭이 차원 테이블이 나타내는 엔터티의 특성에 의해 집계되는 분석 쿼리에 최적화되어 있습니다. 예를 들어 제품, 고객, 날짜 등을 기준으로 인터넷 판매 수익을 집계할 수 있습니다.
    
3. **dbo.FactInternetSales** 테이블 및 해당 **Columns** 폴더를 확장하여 관련 테이블의 열을 확인합니다. 대부분의 열은 차원 테이블의 행을 참조하는 키입니다.** 다른 값은 분석을 위한 숫자 값(측정값)입니다.**
    
    키는 팩트 테이블이 각 차원 테이블과 직접 관련된 경우 팩트 테이블을 별표 스키마에서 하나 이상의 차원 테이블과 연결하기 위해 사용됩니다.(가운데에 팩트 테이블이 있는 다중 뾰족한 “별”을 형성).**

4. **dbo.DimPromotion** 테이블의 열을 보고 테이블의 각 행을 고유하게 식별하는 고유한 **PromotionKey**가 있는지 확인합니다. 또한 **AlternateKey**도 포함합니다.

    일반적으로 데이터 웨어하우스의 데이터는 하나 이상의 트랜잭션 원본에서 가져왔습니다. 대체 키는 원본에서 이 엔터티의 인스턴스에 대한 비즈니스 식별자를 반영하지만, 일반적으로 데이터 웨어하우스 차원 테이블의 각 행을 고유하게 식별하기 위해 고유한 숫자 서로게이트 키가 생성됩니다.**** 이 방법의 이점 중 하나는 데이터 웨어하우스가 서로 다른 시점에 동일한 엔터티의 여러 인스턴스를 포함할 수 있다는 것입니다(예: 주문 시 고객의 주소를 반영하는 동일한 고객의 레코드).

5. **dbo.DimProduct**의 열을 보고 **dbo.DimProductSubcategory** 테이블을 참조하는 **ProductSubcategoryKey** 열이 포함되어 있는지 확인하고, 이어서 **dbo.DimProductCategory** 테이블을 참조하는 **ProductCategoryKey** 열이 포함되어 있는지 확인합니다.

    경우에 따라 차원이 부분적으로 여러 관련 테이블로 정규화되어 하위 범주 및 범주로 그룹화될 수 있는 제품과 같은 다양한 수준의 세분성을 허용합니다. 이로 인해 간단한 별표가 눈송이 스키마로 확장되고 중앙 팩트 테이블이 차원 테이블과 관련되어 추가 차원 테이블과 관련됩니다.**

6. **dbo.DimDate** 테이블의 열을 보고 요일, 일, 월, 연도, 일 이름, 월 이름 등을 포함하여 날짜의 다양한 임시 특성을 반영하는 여러 열이 포함되어 있는지 확인합니다.

    데이터 웨어하우스의 시간 차원은 일반적으로 팩트 테이블의 측정값을 집계하려는 세분성(차원의 조직이라고도 함)의 가장 작은 임시 단위에 대한 행을 포함하는 차원 테이블로 구현됩니다.** 이 경우 측정값을 집계할 수 있는 가장 낮은 조직은 개별 날짜이며 테이블에는 첫 번째 날짜부터 데이터에서 참조된 마지막 날짜까지의 각 날짜에 대한 행이 포함됩니다. **DimDate** 테이블의 특성을 사용하면 분석가가 일관된 임시 특성 집합을 사용하여 팩트 테이블의 날짜 키를 기준으로 측정값을 집계할 수 있습니다(예: 주문 날짜를 기준으로 월별 주문 보기). **FactInternetSales** 테이블에는 **DimDate** 테이블과 관련된 세 가지 키인 **OrderDateKey**, **DueDateKey**, **ShipDateKey**가 포함되어 있습니다.

## 데이터 웨어하우스 테이블 쿼리

이제 데이터 웨어하우스 스키마의 더 중요한 측면 중 일부를 살펴보았으므로 테이블을 쿼리하고 일부 데이터를 검색할 준비가 되었습니다.

### 쿼리 팩트 및 차원 테이블

관계형 데이터 웨어하우스의 숫자 값은 여러 특성에서 데이터를 집계하는 데 사용할 수 있는 관련 차원 테이블이 있는 팩트 테이블에 저장됩니다. 이 디자인은 대부분의 관계형 데이터 웨어하우스 쿼리 관련 테이블(JOIN 절 사용)에서 데이터 집계 및 그룹화(집계 함수 및 GROUP BY 절 사용)가 포함됨을 의미합니다.

1. **데이터** 페이지에서 **sql*xxxxxxx*** SQL 풀을 선택하고 해당 **...** 메뉴에서 **새 SQL 스크립트** > **빈 스크립트**를 선택합니다.
2. 새 **SQL 스크립트 1** 탭이 열리면 **속성** 창에서 스크립트 이름을 **인터넷 판매량 분석**으로 변경하고 **쿼리당 결과 설정**을 변경하여 모든 행을 반환합니다. 그런 다음, 도구 모음의 **게시** 단추를 사용하여 스크립트를 저장하고 스크립트 창을 볼 수 있도록 도구 모음의 오른쪽 끝에 있는 **속성** 단추(**&#128463;.** 유사함)를 사용하여 **속성** 창을 닫습니다.
3. 빈 스크립트에 다음 코드를 추가합니다.

    ```sql
    SELECT  d.CalendarYear AS Year,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY Year;
    ```

4. **&#9655; 실행** 단추를 사용하여 스크립트를 실행하고 결과를 검토합니다. 그러면 매년 인터넷 판매량 합계가 표시됩니다. 이 쿼리는 인터넷 판매 팩트 테이블을 주문 날짜에 따라 시간 차원 테이블에 조인하고 팩트 테이블의 판매 금액 측정값을 차원 테이블의 월 특성별로 집계합니다.

5. 다음과 같이 쿼리를 수정하여 시간 차원의 월 특성을 추가한 다음, 수정된 쿼리를 실행합니다.

    ```sql
    SELECT  d.CalendarYear AS Year,
            d.MonthNumberOfYear AS Month,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
    ORDER BY Year, Month;
    ```

    시간 차원의 특성을 사용하면 팩트 테이블의 측정값을 여러 계층적 수준(이 경우 연도 및 월)으로 집계할 수 있습니다. 이는 데이터 웨어하우스의 일반적인 패턴입니다.

6. 다음과 같이 쿼리를 수정하여 월을 제거하고 집계에 두 번째 차원을 추가한 다음, 실행하여 결과를 확인합니다(각 지역의 연간 인터넷 판매량 합계 표시).

    ```sql
    SELECT  d.CalendarYear AS Year,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY d.CalendarYear, g.EnglishCountryRegionName
    ORDER BY Year, Region;
    ```

    지리란 고객 차원을 통한 인터넷 판매 팩트 테이블과 관련된 눈송이 차원입니다.** 따라서 지역별로 인터넷 판매량을 집계하려면 쿼리에 두 개의 조인이 필요합니다.

7. 쿼리를 수정하고 다시 실행하여 다른 눈송이 차원을 추가하고 제품 범주별로 연간 지역 판매량을 집계합니다.

    ```sql
    SELECT  d.CalendarYear AS Year,
            pc.EnglishProductCategoryName AS ProductCategory,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    JOIN DimProduct AS p ON i.ProductKey = p.ProductKey
    JOIN DimProductSubcategory AS ps ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
    JOIN DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
    GROUP BY d.CalendarYear, pc.EnglishProductCategoryName, g.EnglishCountryRegionName
    ORDER BY Year, ProductCategory, Region;
    ```

    이번에는 제품 범주의 눈송이 차원에 제품, 하위 범주, 범주 간의 계층적 관계를 반영하기 위해 세 개의 조인이 필요합니다.

8. 스크립트를 게시하여 저장합니다.

### 순위 함수 사용

대량의 데이터를 분석할 때 또 다른 일반적인 요구 사항은 데이터를 파티션별로 그룹화하고 특정 메트릭에 따라 파티션에서 각 엔터티의 순위를 결정하는 것입니다.**

1. 기존 쿼리에서 다음 SQL을 추가하여 국가/지역 이름에 따라 파티션을 통해 2022년 판매액 값을 검색합니다.

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            ROW_NUMBER() OVER(PARTITION BY g.EnglishCountryRegionName
                              ORDER BY i.SalesAmount ASC) AS RowNumber,
            i.SalesOrderNumber AS OrderNo,
            i.SalesOrderLineNumber AS LineItem,
            i.SalesAmount AS SalesAmount,
            SUM(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            AVG(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionAverage
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    WHERE d.CalendarYear = 2022
    ORDER BY Region;
    ```

2. 새 쿼리 코드만 선택하고 **&#9655; 실행** 단추를 사용하여 실행합니다. 다음으로, 다음 표와 유사하게 표시되는 결과를 검토합니다.

    | 지역 | RowNumber | OrderNo | LineItem | SalesAmount | RegionTotal | RegionAverage |
    |--|--|--|--|--|--|--|
    |오스트레일리아|1|SO73943|2|2.2900|2172278.7900|375.8918|
    |오스트레일리아|2|SO74100|4|2.2900|2172278.7900|375.8918|
    |...|...|...|...|...|...|...|
    |오스트레일리아|5779|SO64284|1|2443.3500|2172278.7900|375.8918|
    |캐나다|1|SO66332|2|2.2900|563177.1000|157.8411|
    |캐나다|2|SO68234|2|2.2900|563177.1000|157.8411|
    |...|...|...|...|...|...|...|
    |캐나다|3568|SO70911|1|2443.3500|563177.1000|157.8411|
    |프랑스|1|SO68226|3|2.2900|816259.4300|315.4016|
    |프랑스|2|SO63460|2|2.2900|816259.4300|315.4016|
    |...|...|...|...|...|...|...|
    |프랑스|2588|SO69100|1|2443.3500|816259.4300|315.4016|
    |독일|1|SO70829|3|2.2900|922368.2100|352.4525|
    |독일|2|SO71651|2|2.2900|922368.2100|352.4525|
    |...|...|...|...|...|...|...|
    |독일|2617|SO67908|1|2443.3500|922368.2100|352.4525|
    |영국|1|SO66124|3|2.2900|1051560.1000|341.7484|
    |영국|2|SO67823|3|2.2900|1051560.1000|341.7484|
    |...|...|...|...|...|...|...|
    |영국|3077|SO71568|1|2443.3500|1051560.1000|341.7484|
    |미국|1|SO74796|2|2.2900|2905011.1600|289.0270|
    |미국|2|SO65114|2|2.2900|2905011.1600|289.0270|
    |...|...|...|...|...|...|...|
    |미국|10051|SO66863|1|2443.3500|2905011.1600|289.0270|

    이러한 결과에 대해 다음과 같은 사실을 확인합니다.

    - 각 판매 주문 품목에 대한 행이 있습니다.
    - 행은 판매가 이루어진 지역에 따라 파티션으로 구성됩니다.
    - 각 지리적 파티션 내의 행은 판매액 순서(가장 낮은 금액에서 가장 높은 금액으로)로 번호가 매겨집니다.
    - 각 행에는 품목 판매액과 지역별 총 판매액 및 평균 판매액이 포함됩니다.

3. 기존 쿼리에서 다음 코드를 추가하여 GROUP BY 쿼리 내에서 창 함수를 적용하고 총 판매액에 따라 각 지역의 도시 순위를 지정합니다.

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            g.City,
            SUM(i.SalesAmount) AS CityTotal,
            SUM(SUM(i.SalesAmount)) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            RANK() OVER(PARTITION BY g.EnglishCountryRegionName
                        ORDER BY SUM(i.SalesAmount) DESC) AS RegionalRank
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY g.EnglishCountryRegionName, g.City
    ORDER BY Region;
    ```

4. 새 쿼리 코드만 선택하고 **&#9655; 실행** 단추를 사용하여 실행합니다. 그런 다음, 결과를 검토하고 다음을 확인합니다.
    - 결과에는 지역별로 그룹화된 각 도시에 대한 행이 포함됩니다.
    - 각 도시에 대한 총 판매액(개별 판매액 합계)이 계산됩니다.
    - 지역별 총 판매액(지역에 있는 각 도시의 판매액 합계)은 지역 파티션을 기준으로 계산됩니다.
    - 지역 파티션 내의 각 도시 순위는 도시당 총 판매액을 내림차순으로 정렬하여 계산됩니다.

5. 업데이트된 스크립트를 게시하여 변경 내용을 저장합니다.

> **팁**: ROW_NUMBER 및 RANK는 Transact-SQL에서 사용할 수 있는 순위 함수의 예입니다. 자세한 내용은 Transact-SQL 언어 설명서의 [순위 함수](https://docs.microsoft.com/sql/t-sql/functions/ranking-functions-transact-sql) 참조를 참조하세요.

### 대략적인 개수 검색

매우 많은 양의 데이터를 탐색할 경우 쿼리를 실행하는 데 상당한 시간과 리소스가 소요될 수 있습니다. 데이터 분석에는 절대적으로 정확한 값이 필요하지 않은 경우가 많습니다. 대략적인 값만 비교해도 충분할 수 있습니다.

1. 기존 쿼리에서 다음 코드를 추가하여 각 연도의 판매 주문 수를 검색합니다.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        COUNT(DISTINCT i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

2. 새 쿼리 코드만 선택하고 **&#9655; 실행** 단추를 사용하여 실행합니다. 그런 다음, 반환되는 출력을 검토합니다.
    - 쿼리 아래의 **결과** 탭에서 각 연도의 주문 수를 확인합니다.
    - **메시지** 탭에서 쿼리의 총 실행 시간을 봅니다.
3. 쿼리를 다음과 같이 수정하여 각 연도의 대략적인 개수를 반환합니다. 그런 다음, 쿼리를 다시 실행합니다.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        APPROX_COUNT_DISTINCT(i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

4. 반환되는 출력을 검토합니다.
    - 쿼리 아래의 **결과** 탭에서 각 연도의 주문 수를 확인합니다. 이러한 값은 이전 쿼리에서 검색한 실제 개수의 2% 이내여야 합니다.
    - **메시지** 탭에서 쿼리의 총 실행 시간을 봅니다. 이전 쿼리보다 짧아야 합니다.

5. 스크립트를 게시하여 변경 내용을 저장합니다.

> **팁**: 자세한 내용은 [APPROX_COUNT_DISTINCT](https://docs.microsoft.com/sql/t-sql/functions/approx-count-distinct-transact-sql) 함수 문서를 참조하세요.

## 과제 - 재판매인 판매 분석

1. **sql*xxxxxxx*** SQL 풀의 빈 스크립트를 새로 만들고 **재판매인 판매 분석**이라는 이름으로 저장합니다.
2. 스크립트에서 SQL 쿼리를 만들어 **FactResellerSales** 팩트 테이블 및 관련 차원 테이블을 기반으로 다음 정보를 찾습니다.
    - 회계 연도 및 분기당 판매된 항목의 총 수량입니다.
    - 판매한 직원과 관련된 회계 연도, 분기, 판매 지역별 판매 항목의 총 수량입니다.
    - 제품 범주별로 회계 연도, 분기, 판매 지역당 판매된 항목의 총 수량입니다.
    - 해당 연도의 총 판매액을 기준으로 회계 연도당 각 판매 지역 순위입니다.
    - 각 판매 지역의 대략적인 연간 판매 주문 수입니다.

    > **팁**: Synapse Studio에서 **개발** 페이지의 **솔루션** 스크립트와 쿼리를 비교합니다.

3. 쿼리를 실험하여 데이터 웨어하우스 스키마의 나머지 테이블을 참고용으로 살펴봅니다.
4. 완료되면 **관리** 페이지에서 **sql*xxxxxxx*** 전용 SQL 풀을 일시 중지합니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp500-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정 및 작업 영역용 전용 SQL 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp500-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
