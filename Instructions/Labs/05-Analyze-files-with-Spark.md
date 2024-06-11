---
lab:
  title: Spark를 사용하여 데이터 레이크에서 데이터 분석
  ilt-use: Suggested demo
---
# Spark를 사용하여 데이터 레이크에서 데이터 분석

Apache Spark는 분산 데이터 처리를 위한 오픈 소스 엔진으로, Data Lake Storage에서 대량의 데이터를 탐색, 처리, 분석하는 데 널리 사용됩니다. Spark는 Microsoft Azure 클라우드 플랫폼의 Azure HDInsight, Azure Databricks, Azure Synapse Analytics를 비롯한 많은 데이터 플랫폼 제품에서 처리 옵션으로 사용할 수 있습니다. Spark의 이점 중 하나는 Java, Scala, Python, SQL을 포함한 다양한 프로그래밍 언어를 지원한다는 점입니다. Spark는 데이터 정리 및 조작, 통계 분석 및 기계 학습, 데이터 분석 및 시각화를 포함한 데이터 처리 워크로드를 위한 매우 유연한 솔루션입니다.

이 랩을 완료하는 데 약 **45**분이 걸립니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Synapse Analytics 작업 영역 프로비저닝

데이터 레이크 스토리지에 액세스할 수 있는 Azure Synapse Analytics 작업 영역과 데이터 레이크에서 파일을 쿼리하고 처리하는 데 사용할 수 있는 Apache Spark 풀이 필요합니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics 작업 영역을 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp203 -f
    git clone  https://github.com/MicrosoftLearning/Dp-203-azure-data-engineer dp203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp203/Allfiles/labs/05
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics의 Apache Spark](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview) 문서를 검토합니다.

## 파일의 데이터 쿼리

스크립트는 Azure Synapse Analytics 작업 영역 및 Azure Storage 계정을 프로비저닝하여 데이터 레이크를 호스트한 다음, 일부 데이터 파일을 데이터 레이크에 업로드합니다.

### 데이터 레이크에서 파일 보기

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp500-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **관리** 페이지에서 **Apache Spark 풀** 탭을 선택하고 **spark*xxxxxxx***와 비슷한 이름의 Spark 풀이 작업 영역에 프로비저닝되었음을 확인합니다. 나중에 이 Spark 풀을 사용하여 작업 영역의 데이터 레이크 스토리지에 있는 파일에서 데이터를 로드하고 분석할 수 있습니다.
5. **데이터** 페이지에서 **연결된** 탭을 보고 작업 영역에 **synapse*xxxxxxx*(Primary - datalake*xxxxxxx*)**와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
6. 스토리지 계정을 확장하고 **파일**이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
7. **파일** 컨테이너를 선택하고 **sales** 및 **synapse**라는 폴더가 포함되어 있는지 확인합니다. **synapse** 폴더는 Azure Synapse에서 사용되며 **sales** 폴더에는 쿼리하려는 데이터 파일이 포함되어 있습니다.
8. **sales** 폴더와 해당 폴더에 포함된 **orders** 폴더를 열고 **orders** 폴더에 3년간의 판매 데이터에 대한 .csv 파일이 포함되어 있는지 확인합니다.
9. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 포함된 데이터를 확인합니다. 파일에 머리글 행이 포함되어 있지 않으므로 열 머리글을 표시하는 옵션을 선택 취소할 수 있습니다.

### Spark를 사용하여 데이터 탐색

1. **orders** 폴더에서 파일을 선택한 다음 도구 모음의 **새 Notebook** 목록에서 **DataFrame에 로드**를 선택합니다. 데이터 프레임은 테이블 형식 데이터 세트를 나타내는 Spark의 구조체입니다.
2. 열려 있는 새 **Notebook 1** 탭의 **연결 대상** 목록에서 Spark 풀(**spark*xxxxxxx***)을 선택합니다. 그런 다음 **&#9655; 모두 실행** 단추를 사용하여 Notebook의 모든 셀을 실행합니다(현재는 하나만 있습니다!).

    이 세션에서 Spark 코드를 실행한 것은 이번이 처음이기 때문에 Spark 풀이 시작되어야 합니다. 이는 세션의 첫 번째 실행이 몇 분 정도 걸릴 수 있음을 의미합니다. 후속 실행은 더 빨라질 것입니다.

3. Spark 세션이 초기화되기를 기다리는 동안 생성된 코드를 검토합니다. 이 코드는 다음과 유사하게 표시됩니다.

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/2019.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. 코드 실행이 완료되면 Notebook의 셀 아래에 있는 출력을 검토합니다. **_c0**, **_c1**, **_c2** 등 형식의 자동 열 이름을 가진 선택한 파일의 처음 10개의 행을 보여 줍니다.
5. **spark.read.load** 함수가 폴더에 있는 <u>모든</u> CSV 파일에서 데이터를 읽고 **display** 함수가 처음 100개의 행을 표시하도록 코드를 수정합니다. 코드는 다음과 같아야 합니다(데이터 레이크 저장소의 이름과 일치하는 *datalakexxxxxxx* 사용).

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv'
    )
    display(df.limit(100))
    ```

6. 코드 셀의 왼쪽에 있는 **&#9655;** 단추를 사용하여 해당 셀만 실행하고 결과를 검토합니다.

    이제 데이터 프레임에는 모든 파일의 데이터가 포함되지만 열 이름은 유용하지 않습니다. Spark는 “schema-on-read” 접근 방식을 사용하여 포함된 데이터를 기반으로 열에 적합한 데이터 형식을 결정하려고 시도하며, 머리글 행이 텍스트 파일에 있는 경우 열 이름을 식별하는 데 사용할 수 있습니다(**load** 함수에서 **header=True** 매개 변수 지정). 또는 데이터 프레임의 명시적 스키마를 정의할 수 있습니다.

7. 열 이름 및 데이터 형식을 포함하는 데이터 프레임의 명시적 스키마를 정의하려면 다음과 같이 코드를 수정합니다(*datalakexxxxxxx* 대체). 셀의 코드를 다시 실행합니다.

    ```Python
    %%pyspark
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv', schema=orderSchema)
    display(df.limit(100))
    ```

8. 결과에 있는 **+ 코드** 단추를 사용하여 Notebook에 새 코드 셀을 추가합니다. 그런 다음 새 셀에서 다음 코드를 추가하여 데이터 프레임의 스키마를 표시합니다.

    ```Python
    df.printSchema()
    ```

9. 새 셀을 실행하고 데이터 프레임 스키마가 정의한 **orderSchema**와 일치하는지 확인합니다. **printSchema** 함수는 자동으로 유추된 스키마가 있는 데이터 프레임을 사용할 때 유용할 수 있습니다.

## 데이터 프레임에서 데이터 분석

Spark의 **dataframe** 개체는 Python의 Pandas 데이터 프레임과 유사하며 포함된 데이터를 조작, 필터링, 그룹화, 분석하는 데 사용할 수 있는 다양한 함수를 포함합니다.

### 데이터 프레임 필터링

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    customers = df['CustomerName', 'Email']
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

2. 새 코드 셀을 실행하고 결과를 검토합니다. 다음 세부 정보를 살펴봅니다.
    - 데이터 프레임에서 작업을 수행하면 그 결과는 새 데이터 프레임이 됩니다(이 경우 **df** 데이터 프레임에서 열의 특정 하위 집합을 선택하여 새 **customers** 데이터 프레임이 생성됨).
    - 데이터 프레임은 포함된 데이터를 요약하고 필터링하는 데 사용할 수 있는 **count** 및 **distinct** 함수를 제공합니다.
    - `dataframe['Field1', 'Field2', ...]` 구문은 열의 하위 집합을 정의하는 간단한 방법입니다. **select** 메서드를 사용할 수도 있으므로 위 코드의 첫 번째 줄을 `customers = df.select("CustomerName", "Email")`과 같이 작성할 수 있습니다.

3. 다음과 같이 코드를 수정합니다.

    ```Python
    customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

4. 수정된 코드를 실행하여 *Road-250 Red, 52* 제품을 구매한 고객을 확인합니다. 한 함수의 출력이 다음 함수의 입력이 되도록 여러 함수를 함께 “연결”할 수 있습니다. 이 경우에 **select** 메서드에서 만든 데이터 프레임은 필터링 조건을 적용하는 데 사용되는 **where** 메서드의 소스 데이터 프레임이 됩니다.

### 데이터 프레임에서 데이터 집계 및 그룹화

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    productSales = df.select("Item", "Quantity").groupBy("Item").sum()
    display(productSales)
    ```

2. 추가한 코드 셀을 실행하면 결과에 제품별로 그룹화된 주문 수량의 합계가 표시됩니다. **groupBy** 메서드는 행을 *Item*별로 그룹화하고 후속 **sum** 집계 함수는 나머지 모든 숫자 열(이 경우 *Quantity*)에 적용됩니다.

3. Notebook에 다른 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
    display(yearlySales)
    ```

4. 추가한 코드 셀을 실행하면 결과에 연간 판매 주문 수가 표시되는지 확인합니다. **select** 메서드에는 *OrderDate* 필드의 연도 구성 요소를 추출하는 SQL **year** 함수가 포함되어 있으며, **alias** 메서드를 사용하여 추출된 연도 값에 열 이름을 할당합니다. 그런 다음 데이터를 파생된 *Year* 열로 그룹화하고 각 그룹의 행 수를 계산한 후, 마지막으로 **orderBy** 메서드를 사용하여 결과 데이터 프레임을 정렬합니다.

## Spark SQL을 사용하여 데이터 쿼리

앞에서 보았듯이 데이터 프레임 개체의 네이티브 메서드를 사용하면 데이터를 매우 효과적으로 쿼리하고 분석할 수 있습니다. 그러나 많은 데이터 분석가들에게는 SQL 구문을 사용하는 것이 더 편리합니다. Spark SQL은 SQL 문을 실행하거나 관계형 테이블에서 데이터를 유지하는 데 사용할 수 있는 Spark의 SQL 언어 API입니다.

### PySpark 코드에서 Spark SQL 사용

Azure Synapse Studio Notebook의 기본 언어는 Spark 기반 Python 런타임인 PySpark입니다. 이 런타임 내에서 **spark.sql** 라이브러리를 사용하여 Python 코드 내에 Spark SQL 구문을 포함하고 테이블 및 뷰와 같은 SQL 구문으로 작업할 수 있습니다.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    df.createOrReplaceTempView("salesorders")

    spark_df = spark.sql("SELECT * FROM salesorders")
    display(spark_df)
    ```

2. 셀을 실행하고 결과를 검토합니다. 다음과 같은 부분을 확인하세요.
    - 이 코드는 **df** 데이터 프레임의 데이터를 **salesorders**라는 임시 뷰로 유지합니다. Spark SQL은 임시 뷰 또는 지속 테이블을 SQL 쿼리의 원본으로 사용할 수 있도록 지원합니다.
    - 그런 다음 **spark.sql** 메서드를 사용하여 **salesorders** 뷰에 대해 SQL 쿼리를 실행합니다.
    - 쿼리 결과는 데이터 프레임에 저장됩니다.

### 셀에서 SQL 코드를 실행합니다.

PySpark 코드가 포함된 셀에 SQL 문을 포함시키는 것이 유용하겠지만 데이터 분석가는 SQL에서 직접 작업하려는 경우가 많습니다.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```sql
    %%sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
    FROM salesorders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
    ```

2. 셀을 실행하고 결과를 검토합니다. 다음과 같은 부분을 확인하세요.
    - 셀의 시작 부분에 있는 `%%sql` 줄(*magic*이라고 함)은 Spark SQL 언어 런타임을 사용하여 PySpark 대신 이 셀에서 코드를 실행해야 함을 나타냅니다.
    - SQL 코드는 이전에 PySpark를 사용하여 만든 **salesorder** 뷰를 참조합니다.
    - SQL 쿼리의 출력은 셀 아래에 결과로 자동으로 표시됩니다.

> **참고**: Spark SQL 및 데이터 프레임에 대한 자세한 내용은 [Spark SQL 설명서](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html)를 참조하세요.

## Spark를 사용하여 데이터 시각화

그림은 속담처럼 천 단어의 가치가 있으며 차트는 종종 천 행의 데이터보다 낫습니다. Azure Synapse Analytics의 Notebook에는 데이터 프레임 또는 Spark SQL 쿼리에서 표시되는 데이터의 기본 제공 차트 보기가 포함되어 있지만 포괄적인 차트 작성을 위해 설계되지는 않았습니다. 그러나 **matplotlib** 및 **seaborn**과 같은 Python 그래픽 라이브러리를 사용하여 데이터 프레임의 데이터에서 차트를 만들 수 있습니다.

### 결과를 차트로 보기

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```sql
    %%sql
    SELECT * FROM salesorders
    ```

2. 코드를 실행하고 이전에 만든 **salesorders** 뷰에서 데이터를 반환하는지 살펴봅니다.
3. 셀 아래의 결과 섹션에서 **보기** 옵션을 **테이블**에서 **차트**로 변경합니다.
4. 차트의 오른쪽 위에 있는 **보기 옵션** 단추를 사용하여 차트의 옵션 창을 표시합니다. 그런 다음, 옵션을 다음과 같이 설정하고 **적용**을 선택합니다.
    - **차트 종류**: 가로 막대형 차트
    - **키**: Item
    - **값**: Quantity
    - **계열 그룹**: *비워 둠*
    - **집계**: Sum
    - **누적**: *선택되지 않음*

5. 차트가 다음과 유사한지 확인합니다.

    ![총 주문 수량별 제품의 가로 막대형 차트](./images/notebook-chart.png)

### **matplotlib** 시작

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                    SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
                FROM salesorders \
                GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
                ORDER BY OrderYear"
    df_spark = spark.sql(sqlQuery)
    df_spark.show()
    ```

2. 코드를 실행하고 연간 수익을 포함하는 Spark 데이터 프레임을 반환하는지 살펴봅니다.

    데이터를 차트로 시각화하려면 먼저 **matplotlib** Python 라이브러리를 사용하여 시작합니다. 이 라이브러리는 다른 많은 라이브러리의 기반이 되는 핵심 그리기 라이브러리이며 차트를 만드는 데 많은 유연성을 제공합니다.

3. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 추가합니다.

    ```Python
    from matplotlib import pyplot as plt

    # matplotlib requires a Pandas dataframe, not a Spark one
    df_sales = df_spark.toPandas()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

    # Display the plot
    plt.show()
    ```

4. 셀을 실행하고 결과를 검토합니다. 이는 매년 총 수익과 함께 세로 막대형 차트로 구성됩니다. 이 차트를 생성하는 데 사용되는 코드의 다음 기능을 확인합니다.
    - **matplotlib** 라이브러리에는 *Pandas* 데이터 프레임이 필요하므로 Spark SQL 쿼리에서 반환되는 *Spark* 데이터 프레임을 이 형식으로 변환해야 합니다.
    - **matplotlib** 라이브러리의 핵심은 **pyplot** 개체입니다. 이것은 대부분의 그리기 기능의 기초입니다.
    - 기본 설정을 사용하면 사용자 지정할 수 있는 상당한 범위가 있는 사용 가능한 차트가 생성됩니다.

5. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

6. 코드 셀을 다시 실행하고 결과를 확인합니다. 이제 차트에 좀 더 많은 정보가 포함되어 있습니다.

    플롯은 기술적으로 **그림**과 함께 포함됩니다. 이전 예제에서는 암시적으로 그림이 작성되었지만 명시적으로 그림을 작성할 수도 있습니다.

7. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a Figure
    fig = plt.figure(figsize=(8,3))

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

8. 코드 셀을 다시 실행하고 결과를 확인합니다. 그림에 따라 플롯의 모양과 크기가 결정됩니다.

    그림에는 각각 고유의 축에 여러 개의 하위 플롯이 포함될 수 있습니다.**

9. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a figure for 2 subplots (1 row, 2 columns)
    fig, ax = plt.subplots(1, 2, figsize = (10,4))

    # Create a bar plot of revenue by year on the first axis
    ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
    ax[0].set_title('Revenue by Year')

    # Create a pie chart of yearly order counts on the second axis
    yearly_counts = df_sales['OrderYear'].value_counts()
    ax[1].pie(yearly_counts)
    ax[1].set_title('Orders per Year')
    ax[1].legend(yearly_counts.keys().tolist())

    # Add a title to the Figure
    fig.suptitle('Sales Data')

    # Show the figure
    plt.show()
    ```

10. 코드 셀을 다시 실행하고 결과를 확인합니다. 그림에는 코드에 지정된 하위 플롯이 포함되어 있습니다.

> **참고**: matplotlib를 사용하여 그리는 방법에 대한 자세한 내용은 [matplotlib 설명서](https://matplotlib.org/)를 참조하세요.

### **seaborn** 라이브러리 사용

**matplotlib**를 사용하면 여러 종류의 복잡한 차트를 만들 수 있지만 최상의 결과를 얻으려면 몇 가지 복잡한 코드가 필요할 수 있습니다. 이러한 이유로 수년 동안 복잡성을 추상화하고 기능을 향상시키기 위해 matplotlib의 기반으로 많은 새로운 라이브러리가 구축되었습니다. 그러한 라이브러리 중 하나가 **seaborn**입니다.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    import seaborn as sns

    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

2. 코드를 실행하고 seaborn 라이브러리를 사용하여 가로 막대형 차트를 표시하는지 살펴봅니다.
3. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Set the visual theme for seaborn
    sns.set_theme(style="whitegrid")

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

4. 코드를 실행하고 seaborn을 사용하면 플롯에 일관된 색 테마를 설정할 수 있습니다.

5. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

6. 코드를 실행하여 연간 수익을 꺾은선형 차트로 확인합니다.

> **참고**: seaborn을 사용한 그리기에 대한 자세한 내용은 [ 설명서](https://seaborn.pydata.org/index.html)를 참조하세요.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp500-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정 및 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp500-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
