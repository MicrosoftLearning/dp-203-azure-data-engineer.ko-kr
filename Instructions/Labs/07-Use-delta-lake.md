---
lab:
  title: Azure Synapse Analytics에서 Delta Lake 사용
  ilt-use: Lab
---

# Azure Synapse Analytics에서 Spark와 함께 Delta Lake 사용

Delta Lake는 데이터 레이크 위에 트랜잭션 데이터 스토리지 계층을 빌드하는 오픈 소스 프로젝트입니다. Delta Lake는 일괄 처리 및 스트리밍 데이터 작업 모두에 대한 관계형 의미 체계에 대한 지원을 추가하고 Apache Spark를 사용하여 데이터 레이크의 기본 파일을 기반으로 하는 테이블에서 데이터를 처리하고 쿼리할 수 있는 Lakehouse 아키텍처를 만들 수 있습니다.

이 연습을 완료하는 데 약 **40**분 정도 소요됩니다.

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
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 연습의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp-203/Allfiles/labs/07
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Delta Lake란?](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-what-is-delta-lake) 문서를 검토합니다.

## 델타 테이블 만들기

이 스크립트는 Azure Synapse Analytics 작업 영역 및 Azure Storage 계정을 프로비전하여 데이터 레이크를 호스트한 다음 데이터 파일을 데이터 레이크에 업로드합니다.

### 데이터 레이크에서 데이터 탐색

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **데이터** 페이지에서 **연결된** 탭을 보고 작업 영역에 **synapse*xxxxxxx*(Primary - datalake*xxxxxxx*)**와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
5. 스토리지 계정을 확장하고 **파일**이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
6. **파일** 컨테이너를 선택하고 **products**라는 폴더가 포함되어 있습니다. 이 폴더에는 이 연습에서 작업할 데이터가 포함되어 있습니다.
7. **products 폴더를** 열고 **products.csv라는 파일이 **포함되어 있음을 확인합니다.
8. **products.csv**선택한 다음 도구 모음의 **새 Notebook** 목록에서 **DataFrame에 로드를** 선택합니다.
9. 열리는 **Notebook 1** 창의 **연결 대상** 목록에서 **sparkxxxxxxx** Spark 풀을 선택하고 **언어**를 **PySpark(Python)** 로 설정합니다.
10. Notebook에서 첫 번째(이자 유일한) 셀의 코드를 검토합니다. 이 코드는 다음과 같은 형식이어야 합니다.

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

11. *,header=True* 줄의 압축을 풀면(products.csv 파일에 첫 번째 줄에 열 머리글이 있으므로) 코드가 다음과 같이 표시됩니다.

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/products/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

12. 코드 셀 왼쪽에 있는 **&#9655;** 아이콘을 사용하여 실행한 다음 결과를 기다립니다. Notebook에서 셀을 처음으로 실행하면 Spark 풀이 시작됩니다. 따라서 결과가 반환되려면 1분 정도 걸립니다. 최종적으로 결과는 셀 아래에 표시되며, 다음과 비슷한 형식이 됩니다.

    | ProductID | ProductName | 범주 | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... |

### 델타 테이블에 파일 데이터 로드

1. 첫 번째 코드 셀에서 반환된 결과 아래에서 **+ 코드** 단추를 사용하여 새 코드 셀을 추가합니다. 그런 다음, 새 셀에 다음 코드를 입력하고 실행합니다.

    ```Python
    delta_table_path = "/delta/products-delta"
    df.write.format("delta").save(delta_table_path)
    ```

2. **파일** 탭에서 도구 모음의 **&#8593;** 아이콘을 사용하여 **파일** 컨테이너의 루트로 돌아가고 **delta**라는 새 폴더가 만들어졌는지 확인합니다. 이 폴더와 폴더에 포함된 **products-delta** 테이블을 엽니다. 여기서 데이터가 포함된 parquet 형식 파일이 표시됩니다.

3. **Notebook 1** 탭으로 돌아가서 다른 새 코드 셀을 추가합니다. 그런 다음 새 셀에서 다음 코드를 추가하고 실행합니다.

    ```Python
    from delta.tables import *
    from pyspark.sql.functions import *

    # Create a deltaTable object
    deltaTable = DeltaTable.forPath(spark, delta_table_path)

    # Update the table (reduce price of product 771 by 10%)
    deltaTable.update(
        condition = "ProductID == 771",
        set = { "ListPrice": "ListPrice * 0.9" })

    # View the updated data as a dataframe
    deltaTable.toDF().show(10)
    ```

    데이터가 **DeltaTable** 개체에 로드되고 업데이트됩니다. 쿼리 결과에 반영된 업데이트를 볼 수 있습니다.

4. 다음 코드를 사용하여 다른 새 코드 셀을 추가하고 실행합니다.

    ```Python
    new_df = spark.read.format("delta").load(delta_table_path)
    new_df.show(10)
    ```

    코드는 **DeltaTable** 개체를 통해 변경한 내용이 유지되었는지 확인하여 데이터 레이크의 위치에서 데이터 프레임에 델타 테이블 데이터를 로드합니다.

5. 델타 레이크의 *시간 이동* 기능을 사용하여 이전 버전의 데이터를 보는 옵션을 지정하여 방금 실행한 코드를 다음과 같이 수정합니다.

    ```Python
    new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
    new_df.show(10)
    ```

    수정된 코드를 실행하면 데이터의 원래 버전이 결과에 표시됩니다.

6. 다음 코드를 사용하여 다른 새 코드 셀을 추가하고 실행합니다.

    ```Python
    deltaTable.history(10).show(20, False, True)
    ```

    테이블에 대한 마지막 20개 변경 내용의 기록이 표시됩니다. 두 가지(원래 생성 및 수행한 업데이트)가 있어야 합니다.

## 카탈로그 테이블 만들기

지금까지는 테이블의 기반이 되는 parquet 파일이 포함된 폴더에서 데이터를 로드하여 델타 테이블을 사용했습니다. 데이터를 캡슐화하는 *카탈로그 테이블을* 정의하고 SQL 코드에서 참조할 수 있는 명명된 테이블 엔터티를 제공할 수 있습니다. Spark는 델타 레이크에 대해 다음 두 종류의 카탈로그 테이블을 지원합니다.

- 테이블 데이터가 포함된 parquet 파일의 경로로 정의된 *외부* 테이블입니다.
- *Spark* 풀에 대한 Hive 메타스토어에 정의된 관리 테이블입니다.

### 외부 테이블 만들기

1. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```Python
    spark.sql("CREATE DATABASE AdventureWorks")
    spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
    ```

    이 코드는 **AdventureWorks** 라는 새 데이터베이스를 만든 다음, 이전에 정의한 parquet 파일의 경로를 기반으로 해당 데이터베이스에 **ProductsExternal** 이라는 외부 테이블을 만듭니다. 그런 다음 테이블의 속성에 대한 설명을 표시합니다. **Location** 속성은 지정한 경로입니다.

2. 새 코드 셀을 추가한 다음, 다음 코드를 입력하고 실행합니다.

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsExternal;
    ```

    이 코드는 SQL을 사용하여 컨텍스트를 **AdventureWorks** 데이터베이스(데이터를 반환하지 않음)로 전환한 다음 **ProductsExternal** 테이블(Delta Lake 테이블의 제품 데이터가 포함된 결과 집합을 반환)을 쿼리합니다.

### 관리되는 테이블 만들기

1. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```Python
    df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
    spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
    ```

    이 코드는 원래 **products.csv** 파일에서 로드한 DataFrame을 기반으로 **ProductsManaged**라는 관리되는 테이블을 만듭니다(제품 771 가격을 업데이트하기 전에). 테이블에서 사용하는 parquet 파일의 경로를 지정하지 않습니다. 이 경로는 Hive 메타스토어에서 관리되며 테이블 설명의 **Location** 속성( **files/synapse/workspaces/synapsexxxxxxx/warehouse** 경로)에 표시됩니다.

2. 새 코드 셀을 추가한 다음, 다음 코드를 입력하고 실행합니다.

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM ProductsManaged;
    ```

    이 코드는 SQL을 사용하여 **ProductsManaged** 테이블을 쿼리합니다.

### 외부 테이블과 관리되는 테이블 비교

1. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```sql
    %%sql

    USE AdventureWorks;

    SHOW TABLES;
    ```

    이 코드는 **AdventureWorks** 데이터베이스의 테이블을 나열합니다.

2. 코드 셀을 다음과 같이 수정하고 실행을 추가합니다.

    ```sql
    %%sql

    USE AdventureWorks;

    DROP TABLE IF EXISTS ProductsExternal;
    DROP TABLE IF EXISTS ProductsManaged;
    ```

    이 코드는 메타스토어에서 테이블을 삭제합니다.

3. **파일** 탭으로 돌아가서 **files/delta/products-delta 폴더를** 봅니다. 데이터 파일은 이 위치에 여전히 존재합니다. 외부 테이블을 삭제하면 메타스토어에서 테이블이 제거되었지만 데이터 파일은 그대로 유지됩니다.
4. **파일/synapse/workspaces/synapsexxxxxxx/warehouse** 폴더를 보고 **ProductsManaged** 테이블 데이터에 대한 폴더가 없음을 확인합니다. 관리되는 테이블을 삭제하면 메타스토어에서 테이블이 제거되고 테이블의 데이터 파일도 삭제됩니다.

### SQL을 사용하여 테이블 만들기

1. 새 코드 셀을 추가한 다음, 다음 코드를 입력하고 실행합니다.

    ```sql
    %%sql

    USE AdventureWorks;

    CREATE TABLE Products
    USING DELTA
    LOCATION '/delta/products-delta';
    ```

2. 새 코드 셀을 추가한 다음, 다음 코드를 입력하고 실행합니다.

    ```sql
    %%sql

    USE AdventureWorks;

    SELECT * FROM Products;
    ```

    이전에 변경한 내용을 반영하는 기존 Delta Lake 테이블 폴더에 대해 새 카탈로그 테이블이 만들어졌는지 확인합니다.

## 스트리밍 데이터에 델타 테이블 사용

Delta Lake는 스트리밍 데이터를 지원합니다. 델타 테이블은 Spark 구조적 스트리밍 API를 사용하여 만든 데이터 스트림의 *싱크* 또는 *원본* 일 수 있습니다. 이 예제에서는 시뮬레이션된 IoT(사물 인터넷) 시나리오에서 일부 스트리밍 데이터의 싱크로 델타 테이블을 사용합니다.

1. **Notebook 1** 탭으로 돌아가서 새 코드 셀을 추가합니다. 그런 다음 새 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
    from notebookutils import mssparkutils
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    # Create a folder
    inputPath = '/data/'
    mssparkutils.fs.mkdirs(inputPath)

    # Create a stream that reads data from the folder, using a JSON schema
    jsonSchema = StructType([
    StructField("device", StringType(), False),
    StructField("status", StringType(), False)
    ])
    iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

    # Write some event data to the folder
    device_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"ok"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''
    mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
    print("Source stream created...")
    ```

    *만든 원본 스트림...* 메시지가 인쇄되어 있는지 확인합니다. 방금 실행한 코드는 가상 IoT 디바이스의 판독값을 나타내는 일부 데이터가 저장된 폴더를 기반으로 스트리밍 데이터 원본을 만들었습니다.

2. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
    # Write the stream to a delta table
    delta_stream_table_path = '/delta/iotdevicedata'
    checkpointpath = '/delta/checkpoint'
    deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
    print("Streaming to delta sink...")
    ```

    이 코드는 스트리밍 디바이스 데이터를 델타 형식으로 씁니다.

3. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
    # Read the data in delta format into a dataframe
    df = spark.read.format("delta").load(delta_stream_table_path)
    display(df)
    ```

    이 코드는 스트리밍된 데이터를 델타 형식으로 데이터 프레임으로 읽습니다. 스트리밍 데이터를 로드하는 코드는 델타 폴더에서 정적 데이터를 로드하는 데 사용되는 코드와 다르지 않습니다.

4. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
    # create a catalog table based on the streaming sink
    spark.sql("CREATE TABLE IotDeviceData USING DELTA LOCATION '{0}'".format(delta_stream_table_path))
    ```

    이 코드는 델타 폴더를 기반으로 **IotDeviceData** 라는 카탈로그 테이블을 **기본** 데이터베이스에 만듭니다. 다시 말하지만, 이 코드는 비 스트리밍 데이터에 사용되는 것과 동일합니다.

5. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    이 코드는 스트리밍 원본의 디바이스 데이터를 포함하는 **IotDeviceData** 테이블을 쿼리합니다.

6. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
    # Add more data to the source stream
    more_data = '''{"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"ok"}
    {"device":"Dev1","status":"error"}
    {"device":"Dev2","status":"error"}
    {"device":"Dev1","status":"ok"}'''

    mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    이 코드는 더 가상의 디바이스 데이터를 스트리밍 원본에 씁니다.

7. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```sql
    %%sql

    SELECT * FROM IotDeviceData;
    ```

    이 코드는 이제 스트리밍 원본에 추가된 추가 데이터를 포함해야 하는 **IotDeviceData** 테이블을 다시 쿼리합니다.

8. 새 코드 셀에서 다음 코드를 추가하고 실행합니다.

    ```python
    deltastream.stop()
    ```

    이 코드는 스트림을 중지합니다.

## 서버리스 SQL 풀에서 델타 테이블 쿼리

spark 풀 외에도 Azure Synapse Analytics에는 기본 제공 서버리스 SQL 풀이 포함되어 있습니다. 이 풀의 관계형 데이터베이스 엔진을 사용하여 SQL을 사용하여 델타 테이블을 쿼리할 수 있습니다.

1. **파일** 탭에서 **파일/델타** 폴더로 이동합니다.
2. **products-delta 폴더를** 선택하고 도구 모음의 **새 SQL 스크립트** 드롭다운 목록에서 **TOP 100 행 선택을** 선택합니다.
3. **상위 100개 행 선택** 창의 **파일 형식** 목록에서 **델타 형식**을 선택한 다음 **적용**을 선택합니다.
4. 생성된 SQL 코드를 검토합니다. 이 코드는 다음과 같습니다.

    ```sql
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/delta/products-delta/',
            FORMAT = 'DELTA'
        ) AS [result]
    ```

5. **&#9655; 실행** 아이콘을 사용하여 스크립트를 실행하고 결과를 검토합니다. 다음과 유사하게 표시됩니다.

    | ProductID | ProductName | 범주 | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3059.991 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... |

    서버리스 SQL 풀을 사용하여 Spark를 사용하여 만든 델타 형식 파일을 쿼리하고 결과를 보고 또는 분석에 사용하는 방법을 보여 줍니다.

6. 쿼리를 다음 SQL 코드로 바꿉다.

    ```sql
    USE AdventureWorks;

    SELECT * FROM Products;
    ```

7. 코드를 실행하고 서버리스 SQL 풀을 사용하여 Spark 메타스토어로 정의된 카탈로그 테이블에서 Delta Lake 데이터를 쿼리할 수도 있습니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정, 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
