---
lab:
  title: Azure Synapse Analytics 살펴보기
  ilt-use: Lab
---

# Azure Synapse Analytics 살펴보기

Azure Synapse Analytics는 엔드 투 엔드 데이터 분석을 위한 단일 통합 데이터 분석 플랫폼을 제공합니다. 이 연습에서는 데이터를 수집하고 탐색하는 다양한 방법을 살펴봅니다. 이 연습은 Azure Synapse Analytics의 다양한 핵심 기능에 대한 개략적인 개요로 설계됩니다. 다른 연습에서는 특정 기능을 더 자세히 살펴볼 수 있습니다.

이 연습을 완료하는 데 약 **60**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Synapse Analytics 작업 영역 프로비저닝

Azure Synapse Analytics 작업 영역은 데이터 및 데이터 처리 런타임을 관리하기 위한 중앙 지점을 제공합니다. Azure Portal에서 대화형 인터페이스를 사용하여 작업 영역을 프로비저닝하거나 스크립트 또는 템플릿을 사용하여 작업 영역 및 리소스를 배포할 수 있습니다. 대부분의 프로덕션 시나리오에서는 리소스 배포를 반복 가능한 개발 및 작업(*DevOps*) 프로세스에 통합할 수 있도록 스크립트와 템플릿을 사용하여 프로비저닝을 자동화하는 것이 가장 좋습니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics를 프로비저닝합니다.

1. 웹 브라우저의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
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
    cd dp-203/Allfiles/labs/01
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요! 또한 암호에는 로그인 이름의 일부 또는 전부가 포함될 수 없습니다.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 20분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics란?](https://docs.microsoft.com/azure/synapse-analytics/overview-what-is) 문서를 검토합니다.

## Synapse Studio 살펴보기

Synapse Studio는 Azure Synapse Analytics 작업 영역에서 리소스를 관리하고 작업할 수 있는 웹 기반 포털입니다.

1. 설치 스크립트 실행이 완료되면 Azure Portal에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 이 리소스 그룹에 Synapse 작업 영역, 데이터 레이크에 대한 스토리지 계정, Apache Spark 풀, Data Explorer 풀, 전용 SQL 풀이 포함되어 있는지 확인합니다.
2. Synapse 작업 영역을 선택하고 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio를 엽니다. Synapse Studio는 Synapse Analytics 작업 영역에서 사용할 수 있는 웹 기반 인터페이스입니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할, 다음과 같은 다양한 페이지가 표시됩니다.

    ![리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용하는, 확장된 Synapse Studio 메뉴를 보여 주는 이미지](./images/synapse-studio.png)

4. **데이터** 페이지를 보고 데이터 원본을 포함하는 다음과 같은 두 개의 탭이 있는지 확인합니다.
    - 작업 영역에 정의된 데이터베이스를 포함하는 **작업 영역** 탭(전용 SQL 데이터베이스 및 Data Explorer 데이터베이스 포함)
    - 작업 영역에 연결된 데이터 원본이 포함된 **연결됨** 탭(Azure Data Lake 스토리지 포함)

5. 현재 비어 있는 **개발** 페이지를 봅니다. 여기서 데이터 처리 솔루션을 개발하는 데 사용되는 스크립트 및 기타 자산을 정의할 수 있습니다.
6. 또한 비어 있는 **통합** 페이지를 봅니다. 이 페이지를 사용하여 데이터 수집 및 통합 자산을 관리합니다(예: 데이터 원본 간에 데이터를 전송하고 변환하는 파이프라인).
7. **모니터링** 페이지를 봅니다. 여기서는 실행되는 데이터 처리 작업을 확인하고 기록을 볼 수 있습니다.
8. **관리** 페이지를 봅니다. 여기서는 Azure Synapse 작업 영역에서 사용되는 풀, 런타임, 기타 자산을 관리합니다. **분석 풀** 섹션에서 각 탭을 보고 작업 영역에 다음 풀이 포함되어 있는지 확인합니다.
    - **SQL 풀**:
        - **기본 제공**: 요청 시 SQL 명령을 사용하여 데이터 레이크에서 데이터를 탐색하거나 처리할 수 있는 *서버리스* SQL 풀입니다.
        - **sql*xxxxxxx***: 관계형 데이터 웨어하우스 데이터베이스를 호스트하는 *전용* SQL 풀입니다.
    - **Apache Spark 풀**:
        - **spark*xxxxxxx***: 요청 시 Scala 또는 Python과 같은 프로그래밍 언어를 사용하여 데이터 레이크에서 데이터를 탐색하거나 처리할 수 있습니다.

## 파이프라인을 사용하여 데이터 수집

Azure Synapse Analytics로 수행할 수 있는 대표적인 핵심 작업은 분석을 위해 데이터를 다양한 소스에서 작업 영역으로 전송하는(필요하다면 변환하는) 파이프라인을 정의하는 일입니다.

### 데이터 복사 작업을 사용하여 파이프라인 만들기

1. Synapse Studio의 **홈** 페이지에서 **수집**을 선택하여 **데이터 복사** 도구를 엽니다.
2. 데이터 복사 도구의 **속성** 단계에서 **기본 제공 복사 작업**과 **지금 한 번 실행**이 선택되어 있는지 확인하고 **다음 >** 을 클릭합니다.
3. **원본** 단계의 **데이터 세트** 하위 단계에서 다음 설정을 선택합니다.
    - **데이터 유형**: 모두
    - **연결**: *새 연결을 만들고, 표시되는 **연결된 서비스** 창에 있는 **일반 프로토콜** 탭에서 **HTTP**를 선택합니다. 다음과 같은 * 설정을 사용하여 데이터 파일에 대한 연결을 계속합니다.
        - **이름**: Products
        - **설명**: HTTP를 통한 제품 목록
        - **통합 런타임을 통해 연결**: AutoResolveIntegrationRuntime
        - **기준 URL**: `https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/01/adventureworks/products.csv`
        - **서버 인증서 유효성 검사**: 사용
        - **인증 유형**: 익명
4. 연결이 생성되면 **원본 데이터 저장소** 페이지에서 다음 설정이 선택되었는지 확인한 다음, **다음 >** 을 선택합니다.
    - **상대 URL**: 비워 둡니다.
    - **요청 메서드**: GET
    - **추가 헤더**: 비워 둡니다.
    - **이진 복사**: 선택 <u>취소</u>
    - **요청 시간 초과**: 비워 둡니다.
    - **최대 동시 연결 수**: 비워 둡니다.
5. **원본** 단계의 **구성** 하위 단계에서 **데이터 미리 보기**를 선택하여 파이프라인이 수집할 제품 데이터의 미리 보기를 확인한 다음 미리 보기를 닫습니다.
6. 데이터 미리 보기가 끝나면 **파일 형식 설정** 페이지에서 다음 설정이 선택되어 있는지 확인한 다음, **다음 >** 을 선택합니다.
    - **파일 형식**: DelimitedText
    - **열 구분 기호**: 쉼표(,)
    - **행 구분 기호**: 줄 바꿈(\n)
    - **첫 번째 행을 머리글로**: 선택됨
    - **압축 형식**: 없음
7. **대상** 단계의 **데이터 세트** 하위 단계에서 다음 설정을 선택합니다.
    - **대상 유형**: Azure Data Lake Storage Gen 2
    - **연결**: *데이터 레이크 저장소에 대한 기존 연결을 선택합니다(작업 영역 생성 시 생성됨).*
8. 연결을 선택한 다음 **대상/데이터 세트** 단계에서 다음 설정이 선택되어 있는지 확인하고 **다음 >** 을 선택합니다.
    - **폴더 경로**: files/product_data
    - **파일 이름**: products.csv
    - **복사 동작**: 없음
    - **최대 동시 연결 수**: 비워 둡니다.
    - **블록 크기(MB)**: 비워 둡니다.
9. **대상** 단계의 **구성** 하위 단계에 있는 **파일 형식 설정** 페이지에서 다음 속성이 선택되어 있는지 확인합니다. **다음 >** 을 선택합니다.
    - **파일 형식**: DelimitedText
    - **열 구분 기호**: 쉼표(,)
    - **행 구분 기호**: 줄 바꿈(\n)
    - **파일에 헤더 추가**: 선택됨
    - **압축 형식**: 없음
    - **파일당 최대 행**: 비워 둡니다.
    - **파일 이름 접두사**: 비워 둡니다.
10. **설정** 단계에서 다음 설정을 입력하고 **다음 >** 을 클릭합니다.
    - **작업 이름**: 제품 복사
    - **작업 설명** 제품 데이터 복사
    - **내결함성**: 비워 둡니다.
    - **로깅 사용**: 선택 <u>취소됨</u>
    - **로깅 준비**: 선택 <u>취소됨</u>
11. **검토 및 마침** 단계의 **검토** 하위 단계에서 요약을 읽고 **다음 >** 을 클릭합니다.
12. **배포** 단계에서 파이프라인이 배포될 때까지 기다린 후 **마침**을 클릭합니다.
13. Synapse Studio에서 **모니터** 페이지를 선택하고 **파이프라인 실행** 탭에서 **제품 복사** 파이프라인이 **성공** 상태로 완료될 때까지 기다립니다(파이프라인 실행 페이지에서 **&#8635; 새로 고침** 단추를 사용하여 상태를 새로 고칠 수 있음).
14. **통합** 페이지를 보고 **제품 복사**라는 파이프라인이 포함되어 있는지 확인합니다.

### 수집된 데이터 보기

1. **데이터** 페이지에서 **연결된** 탭을 선택하고 Synapse 작업 영역에 대한 **files** 파일 스토리지가 표시될 때까지 **synapse *xxxxxxx*(기본) datalake* * 컨테이너 계층 구조를 확장합니다. 그런 다음, 파일 스토리지를 선택하여 다음처럼 **products.csv**라는 파일이 포함된 **product_data**라는 폴더가 이 위치에 복사되었는지 확인합니다.

    ![Synapse 작업 영역의 파일 스토리지가 있는 Azure Data Lake Storage 계층 구조가 확장된 Synapse Studio를 보여 주는 이미지](./images/product_files.png)

2. **products.csv** 데이터 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 수집된 데이터를 봅니다. 그런 다음, 미리 보기를 닫습니다.

## 서버리스 SQL 풀을 사용하여 데이터 분석

작업 영역에 일부 데이터를 수집했으므로, 이제 Synapse Analytics를 사용하여 데이터를 쿼리하고 분석할 수 있습니다. 데이터를 쿼리하는 가장 일반적인 방법은 SQL를 사용하는 것입니다. Synapse Analytics에서는 서버리스 SQL 풀을 사용하여 데이터 레이크의 데이터에 대해 SQL 코드를 실행할 수 있습니다.

1. Synapse Studio에서 Synapse 작업 영역의 파일 스토리지에 있는 **products.csv** 파일을 마우스 오른쪽 단추로 클릭하고, **새 SQL 스크립트**를 가리킨 다음 **상위 100개 행**을 선택합니다.
2. 열리는 **SQL 스크립트 1** 창에서, 생성된 SQL 코드를 검토합니다. 이 코드는 다음과 같은 형식이어야 합니다.

    ```SQL
    -- This is auto-generated code
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0'
        ) AS [result]
    ```

    이 코드는 가져온 텍스트 파일에서 행 집합을 열고 데이터의 처음 행 100개를 검색합니다.

3. **연결 대상** 목록에서 **기본 제공**이 선택되어 있는지 확인합니다. 작업 영역을 사용하여 만든 기본 제공 SQL 풀을 의미합니다.
4. 도구 모음에서 **&#9655; 실행** 단추를 사용하여 SQL 코드를 실행하고 결과를 검토합니다. 결과는 다음과 같은 형식이어야 합니다

    | C1 | C2 | C3 | C4 |
    | -- | -- | -- | -- |
    | ProductID | ProductName | 범주 | ListPrice |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... |

5. 결과는 C1, C2, C3 및 C4라는 열 4개로 구성됩니다. 결과의 첫 번째 행에는 데이터 필드의 이름이 포함됩니다. 이 문제를 해결하려면 다음과 같이 OPENROWSET 함수에 HEADER_ROW = TRUE 매개 변수를 추가한 다음(datalakexxxxxxx를 데이터 레이크 스토리지 계정의 이름으로 바꿈), 쿼리를 다시 실행합니다.

    ```SQL
    SELECT
        TOP 100 *
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    ```

    이제 결과는 다음과 같은 형식이 됩니다.

    | ProductID | ProductName | 범주 | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... |

6. 다음과 같이 쿼리를 수정합니다(datalakexxxxxxx를 데이터 레이크 스토리지 계정의 이름으로 대체).

    ```SQL
    SELECT
        Category, COUNT(*) AS ProductCount
    FROM
        OPENROWSET(
            BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
            FORMAT = 'CSV',
            PARSER_VERSION='2.0',
            HEADER_ROW = TRUE
        ) AS [result]
    GROUP BY Category;
    ```

7. 다음과 같이 각 범주의 숫자 제품을 포함하는 결과 집합을 반환하는, 수정된 쿼리를 실행합니다.

    | 범주 | ProductCount |
    | -- | -- |
    | Bib Shorts | 3 |
    | 자전거 랙 | 1 |
    | ... | ... |

8. **SQL Script 1**의 **속성** 창에서 **이름**을 **범주별 제품 카운트**로 변경합니다. 그런 다음 도구 모음에서 **게시**를 선택하여 스크립트를 저장합니다.

9. **범주별 제품 수** 스크립트 창을 닫습니다.

10. Synapse Studio에서 **개발** 페이지를 선택하고, 게시한 **범주별 제품 수** SQL 스크립트가 저장되어 있는지 확인합니다.

11. **범주별 제품 수** SQL 스크립트를 선택하여 다시 엽니다. 그런 다음 스크립트가 **기본 제공** SQL 풀에 연결되어 있는지 확인하고 실행하여 제품 수를 검색합니다.

12. **결과** 창에서 **차트** 보기를 선택하고 차트에 대해 다음 설정을 선택합니다.
    - **차트 종류**: 열
    - **범주 열**: 범주
    - **범례(계열) 열**: ProductCount
    - **범례 위치**: 아래쪽 - 가운데
    - **범례(계열) 레이블**: 비워 둡니다.
    - **범례(계열) 최소 값**: 비워 둡니다.
    - **범례(계열) 최대**: 비워 둡니다.
    - **범주 레이블**: 비워 둡니다.

    생성되는 차트는 다음과 유사합니다.

    ![제품 개수 차트 보기를 보여 주는 이미지](./images/column-chart.png)

## Spark 풀을 사용하여 데이터 분석

SQL는 구조화된 데이터 세트를 쿼리하는 데 자주 사용하는 언어지만, 많은 데이터 분석가는 Python 같은 언어를 이용해 데이터를 쉽게 탐색하고 분석할 수 있도록 준비하고 있습니다. Azure Synapse Analytics에서는 Apache Spark 기반의 분산 데이터 처리 엔진을 사용하는 Spark 풀에서 Python(및 기타) 코드를 실행할 수 있습니다.

1. Synapse Studio에서 **products.csv** 파일이 포함된 이전에 연 **파일** 탭이 더 이상 열려 있지 않으면 **데이터** 페이지에서 **product_data** 폴더를 찾습니다. 그런 다음 **products.csv**를 마우스 오른쪽 단추로 클릭하고 **새 Notebook**을 가리킨 다음 **에 로드**를 선택합니다.
2. 열리는 **Notebook 1** 창의 **연결 대상** 목록에서 **sparkxxxxxxx** Spark 풀을 선택하고 **언어**를 **PySpark(Python)** 로 설정합니다.
3. Notebook에서 첫 번째(이자 유일한) 셀의 코드를 검토합니다. 이 코드는 다음과 같은 형식이어야 합니다.

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/product_data/products.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. 코드 셀 왼쪽에 있는 **&#9655;** 아이콘을 사용하여 실행한 다음 결과를 기다립니다. Notebook에서 셀을 처음으로 실행하면 Spark 풀이 시작됩니다. 따라서 결과가 반환되려면 1분 정도 걸립니다.
5. 최종적으로 결과는 셀 아래에 표시되며, 다음과 비슷한 형식이 됩니다.

    | _c0_ | _c1_ | _c2_ | _c3_ |
    | -- | -- | -- | -- |
    | ProductID | ProductName | 범주 | ListPrice |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... |

6. *,header=True* 줄의 압축을 풀면(products.csv 파일에 첫 번째 줄에 열 머리글이 있으므로) 코드가 다음과 같이 표시됩니다.

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/product_data/products.csv', format='csv'
    ## If header exists uncomment line below
    , header=True
    )
    display(df.limit(10))
    ```

7. 셀을 다시 실행하여 결과가 다음과 비슷한지 확인합니다.

    | ProductID | ProductName | 범주 | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | ... | ... | ... | ... |

    Spark 풀이 시작된 상태이므로 셀 재실행 시간이 단축됩니다.

8. 결과에 있는 **&#65291; 코드** 아이콘을 사용하여 Notebook에 새 코드 셀을 추가합니다.
9. 빈 새 코드 셀에 다음 코드를 추가합니다.

    ```Python
    df_counts = df.groupby(df.Category).count()
    display(df_counts)
    ```

10. **&#9655;** 아이콘을 클릭하여 새 코드 셀을 실행하고 결과를 검토합니다. 결과는 다음과 비슷한 형식이어야 합니다.

    | 범주 | 개수 |
    | -- | -- |
    | 헤드세트 | 3 |
    | 바퀴 | 14 |
    | ... | ... |

11. 셀의 결과 출력에서 **차트** 보기를 선택합니다. 생성되는 차트는 다음과 유사합니다.

    ![범주 개수 차트 보기를 보여 주는 이미지](./images/bar-chart.png)

12. 아직 표시되지 않으면 도구 모음의 오른쪽 끝에 있는 **속성** 단추( **&#128463;<sub>*</sub>** 와 유사함)를 선택하여 **속성** 페이지를 표시합니다. 그런 다음, **속성** 창에서 Notebook 이름을 **제품 탐색**으로 변경하고 도구 모음의 **게시** 단추를 사용하여 저장합니다.

13. Notebook 창을 닫고 메시지가 표시되면 Spark 세션을 중지합니다. 그런 다음, **개발** 페이지를 보고 Notebook이 저장되었는지 확인합니다.

## 전용 SQL 풀을 사용하여 데이터 웨어하우스 쿼리

지금까지 데이터 레이크에서 파일 기반 데이터를 탐색하고 처리하는 몇 가지 기술을 살펴보았습니다. 대부분의 경우 엔터프라이즈 분석 솔루션은 데이터 레이크를 사용하여 비정형 데이터를 저장하고 준비한 다음, 관계형 데이터 웨어하우스에 로드하여 BI(비즈니스 인텔리전스) 워크로드를 지원할 수 있습니다. Azure Synapse Analytics에서 이러한 데이터 웨어하우스는 전용 SQL 풀에서 구현할 수 있습니다.

1. Synapse Studio의 **관리** 페이지에 있는 **SQL 풀** 섹션에서 **sql*xxxxxxx*** 전용 SQL 풀 행을 선택한 다음, **&#9655;** 아이콘을 사용하여 다시 시작합니다.
2. SQL 풀이 시작될 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다. **&#8635; 새로 고침** 단추를 사용하여 상태를 주기적으로 확인합니다. 준비되면 상태가 **온라인**으로 표시됩니다.
3. SQL 풀이 시작되면 **데이터** 페이지를 선택합니다. 그리고 **작업 영역** 탭에서 **SQL 데이터베이스**를 확장하고 **sql*xxxxxxx***가 나열되었는지 확인합니다(필요한 경우 페이지 왼쪽 위에 있는 **&#8635;** 아이콘을 사용하여 보기를 새로 고침).
4. **sql*xxxxxxx*** 데이터베이스 및 해당 **Tables** 폴더를 확장한 다음, **FactInternetSales** 테이블의 **...** 메뉴에서 **새 SQL 스크립트**를 가리키고 **상위 100개 행 선택**을 선택합니다.
5. 테이블의 처음 100개 판매 트랜잭션을 표시하는 쿼리 결과를 검토합니다. 이 데이터는 설치 스크립트에 의해 데이터베이스에 로드되었으며 전용 SQL 풀과 연결된 데이터베이스에 영구적으로 저장됩니다.
6. SQL 쿼리를 다음 코드로 바꿉니다.

    ```sql
    SELECT d.CalendarYear, d.MonthNumberOfYear, d.EnglishMonthName,
           p.EnglishProductName AS Product, SUM(o.OrderQuantity) AS UnitsSold
    FROM dbo.FactInternetSales AS o
    JOIN dbo.DimDate AS d ON o.OrderDateKey = d.DateKey
    JOIN dbo.DimProduct AS p ON o.ProductKey = p.ProductKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear, d.EnglishMonthName, p.EnglishProductName
    ORDER BY d.MonthNumberOfYear
    ```

7. **&#9655; 실행** 단추를 사용하여 수정된 쿼리를 실행합니다. 이 쿼리는 판매된 각 제품의 수량을 연도 및 월별로 반환합니다.
8. 아직 표시되지 않으면 도구 모음의 오른쪽 끝에 있는 **속성** 단추( **&#128463;<sub>*</sub>** 와 유사함)를 선택하여 **속성** 페이지를 표시합니다. 그런 다음, **속성** 창에서 쿼리 이름을 **제품 판매 집계**로 변경하고 도구 모음의 **게시** 단추를 사용하여 저장합니다.

9. 쿼리 창을 닫은 다음, **개발** 페이지를 확인하여 SQL 스크립트가 저장되었는지 확인합니다.

10. **관리** 페이지에서 **sql*xxxxxxx*** 전용 SQL 풀 행을 선택하고 &#10074;&#10074; 아이콘을 사용하여 일시 중지합니다.

<!--- ## Explore data with a Data Explorer pool

Azure Synapse Data Explorer provides a runtime that you can use to store and query data by using Kusto Query Language (KQL). Kusto is optimized for data that includes a time series component, such as realtime data from log files or IoT devices.

### Create a Data Explorer database and ingest data into a table

1. In Synapse Studio, on the **Manage** page, in the **Data Explorer pools** section, select the **adx*xxxxxxx*** pool row and then use its **&#9655;** icon to resume it.
2. Wait for the pool to start. It can take some time. Use the **&#8635; Refresh** button to check its status periodically. The status will show as **online** when it is ready.
3. When the Data Explorer pool has started, view the **Data** page; and on the **Workspace** tab, expand **Data Explorer Databases** and verify that **adx*xxxxxxx*** is listed (use **&#8635;** icon at the top-left of the page to refresh the view if necessary)
4. In the **Data** pane, use the **&#65291;** icon to create a new **Data Explorer database** in the **adx*xxxxxxx*** pool with the name **sales-data**.
5. In Synapse Studio, wait for the database to be created (a notification will be displayed).
6. Switch to the **Develop** page, and in the **+** menu, add a KQL script. Then, when the script pane opens, in the **Connect to** list, select your **adx*xxxxxxx*** pool, and in the **Database** list, select **sales-data**.
7. In the new script, add the following code:

    ```kusto
    .create table sales (
        SalesOrderNumber: string,
        SalesOrderLineItem: int,
        OrderDate: datetime,
        CustomerName: string,
        EmailAddress: string,
        Item: string,
        Quantity: int,
        UnitPrice: real,
        TaxAmount: real)
    ```

8. On the toolbar, use the **&#9655; Run** button to run the selected code, which creates a table named **sales** in the **sales-data** database you created previously.
9. After the code has run successfully, replace it with the following code, which loads data into the table:

    ```kusto
    .ingest into table sales 'https://raw.githubusercontent.com/microsoftlearning/dp-203-azure-data-engineer/master/Allfiles/labs/01/files/sales.csv' 
    with (ignoreFirstRecord = true)
    ```

10. Run the new code to ingest the data.

> **Note**: In this example, you imported a very small amount of batch data from a file, which is fine for the purposes of this exercise. In reality, you can use Data Explorer to analyze much larger volumes of data; including realtime data from a streaming source such as Azure Event Hubs.

### Use Kusto query language to query the table

1. Switch back to the **Data** page and in the **...** menu for the **sales-data** database, select **Refresh**.
2. Expand the **sales-data** database's **Tables** folder. Then in the **...** menu for the **sales** table, select **New KQL script** > **Take 1000 rows**.
3. Review the generated query and its results. The query should contain the following code:

    ```kusto
    sales
    | take 1000
    ```

    The results of the query contain the first 1000 rows of data.

4. Modify the query as follows:

    ```kusto
    sales
    | where Item == 'Road-250 Black, 48'
    ```

5. Use the **&#9655; Run** button to run the query. Then review the results, which should contain only the rows for sales orders for the *Road-250 Black, 48* product.

6. Modify the query as follows:

    ```kusto
    sales
    | where Item == 'Road-250 Black, 48'
    | where datetime_part('year', OrderDate) > 2020
    ```

7. Run the query and review the results, which should contain only sales orders for *Road-250 Black, 48* made after 2020.

8. Modify the query as follows:

    ```kusto
    sales
    | where OrderDate between (datetime(2020-01-01 00:00:00) .. datetime(2020-12-31 23:59:59))
    | summarize TotalNetRevenue = sum(UnitPrice) by Item
    | sort by Item asc
    ```

9. Run the query and review the results, which should contain the total net revenue for each product between January 1st and December 31st 2020 in ascending order of product name.

10. If it is not already visible, show the **Properties** page by selecting the **Properties** button (which looks similar to **&#128463;<sub>*</sub>**) on the right end of the toolbar. Then in the **Properties** pane, change the query name to **Explore sales data** and use the **Publish** button on the toolbar to save it.

11. Close the query pane, and then view the **Develop** page to verify that the KQL script has been saved.

12. On the **Manage** page, select the **adx*xxxxxxx*** Data Explorer pool row and use its &#10074;&#10074; icon to pause it. --->

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정, SQL 풀, Data Explorer 풀, 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
