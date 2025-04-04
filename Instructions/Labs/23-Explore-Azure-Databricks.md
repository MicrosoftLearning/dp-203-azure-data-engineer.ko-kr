---
lab:
  title: Azure Databricks 탐색
  ilt-use: Suggested demo
---

# Azure Databricks 탐색

Azure Databricks는 인기 있는 오픈 소스 Databricks 플랫폼의 Microsoft Azure 기반 버전입니다.

Azure Synapse Analytics와 마찬가지로 Azure Databricks 작업 영역은 Azure에서 Databricks 클러스터, 데이터 및 리소스를 관리하기 위한 중심점을 제공합니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Databricks 작업 영역 프로비저닝

이 연습에서는 스크립트를 사용하여 새 Azure Databricks 작업 영역을 프로비저닝합니다.

> **팁**: *표준* 또는 *평가판* Azure Databricks 작업 영역이 이미 있는 경우 이 절차를 건너뛰고 기존 작업 영역을 사용할 수 있습니다.

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

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp-203/Allfiles/labs/23
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).

7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Databricks 설명서의 [Azure Databricks란?](https://learn.microsoft.com/azure/databricks/introduction/) 문서를 검토합니다.

## 클러스터 생성

Azure Databricks는 Apache Spark 클러스터를 사용하여 여러 노드에서 병렬로 데이터를 처리하는 분산 처리 플랫폼입니다. 각 클러스터는 작업을 조정하는 드라이버 노드와 처리 작업을 수행하는 작업자 노드로 구성됩니다.

이 연습에서는 랩 환경(리소스가 제한될 수 있음)에서 사용되는 컴퓨팅 리소스를 최소화하기 위해 *단일 노드* 클러스터를 만듭니다. 프로덕션 환경에서는 일반적으로 여러 작업자 노드가 있는 클러스터를 만듭니다.

> **팁**: Azure Databricks 작업 영역에 13.3 LTS 런타임 버전이 있는 클러스터가 이미 있는 경우 해당 클러스터를 사용하여 이 연습을 완료하고 이 절차를 건너뛸 수 있습니다.

1. Azure Portal에서 스크립트로 만든 **dp203-*xxxxxxx*** 리소스 그룹(또는 기존 Azure Databricks 작업 영역이 포함된 리소스 그룹)으로 이동
1. Azure Databricks Service 리소스(설치 스크립트를 사용하여 만든 경우 **databricks*xxxxxxx***라는 이름)를 선택합니다.
1. 작업 영역의 **개요** 페이지에서 **작업 영역 시작** 단추를 사용하여 새 브라우저 탭에서 Azure Databricks 작업 영역을 엽니다. 메시지가 표시되면 로그인합니다.

    > **팁**: Databricks 작업 영역 포털을 사용하면 다양한 팁과 알림이 표시될 수 있습니다. 이를 해제하고 제공된 지침에 따라 이 연습의 작업을 완료하세요.

1. Azure Databricks 작업 영역 포털을 보고 왼쪽 사이드바에 수행할 수 있는 다양한 유형의 작업에 대한 링크가 포함되어 있는 것을 확인합니다.

1. 사이드바에서 **(+) 새로 만들기** 링크를 선택한 다음 **클러스터**를 선택합니다.
1. **새 클러스터** 페이지에서 다음 설정을 사용하여 새 클러스터를 만듭니다.
    - **클러스터 이름**: *사용자 이름*의 클러스터(기본 클러스터 이름)
    - **클러스터 모드**: 단일 노드
    - **액세스 모드**: 단일 사용자(*사용자 계정 선택*)
    - **Databricks 런타임 버전**: 13.3 LTS(Spark 3.4.1, Scala 2.12)
    - **Photon 가속 사용**: 선택됨
    - **노드 형식**: Standard_DS3_v2
    - *30***분 동안 비활성 상태**이면 **종료**

1. 클러스터가 만들어질 때까지 기다립니다. 1~2분 정도 소요됩니다.

> **참고**: 클러스터를 시작하지 못하면 Azure Databricks 작업 영역이 프로비저닝된 지역에서 구독 할당량이 부족할 수 있습니다. 자세한 내용은 [CPU 코어 제한으로 클러스터 생성 방지](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit)를 참조하세요. 이 경우 작업 영역을 삭제하고 다른 지역에 새 작업 영역을 만들 수 있습니다. `./setup.ps1 eastus`와 같이 설치 스크립트에 대한 매개 변수로 지역을 지정할 수 있습니다.

## Spark를 사용하여 데이터 파일 분석

많은 Spark 환경에서와 마찬가지로 Databricks는 Notebook을 사용하여 데이터를 탐색하는 데 사용할 수 있는 메모와 대화형 코드 셀을 결합하도록 지원합니다.

1. 사이드바에서 **(+) 새로 만들기** 링크를 사용하여 **Notebook**을 만듭니다.
1. 기본 Notebook 이름(**Untitled Notebook *[날짜]***)을 **Explore products**로 변경하고 아직 선택하지 않은 경우 **연결** 드롭다운 목록에서 클러스터를 선택합니다. 클러스터가 실행되고 있지 않으면 시작하는 데 1분 정도 걸릴 수 있습니다.
1. [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/23/adventureworks/products.csv) 파일을 로컬 컴퓨터에 다운로드하여 **products.csv**로 저장합니다. 다음으로 **Explore products** Notebook의 **파일** 메뉴에서 **DBFS에 데이터 업로드**를 선택합니다.
1. **데이터 업로드** 대화 상자에서 파일이 업로드될 **DBFS 대상 디렉터리**를 기록해 둡니다. 그런 다음 **파일** 영역을 선택하고 컴퓨터에 다운로드한 **products.csv** 파일을 업로드합니다. 파일이 업로드되면 **다음**을 선택합니다.
1. **Notebook의 파일 액세스** 창에서 샘플 PySpark 코드를 선택하고 클립보드에 복사합니다. 이를 사용하여 파일의 데이터를 데이터 프레임으로 로드합니다. 그런 후 **완료**를 선택합니다.
1. **제품 탐색** Notebook의 빈 코드 셀에 복사한 코드를 붙여넣습니다. 다음과 유사하게 표시됩니다.

    ```python
    df1 = spark.read.format("csv").option("header", "true").load("dbfs:/FileStore/shared_uploads/user@outlook.com/products.csv")
    ```

1. 셀의 오른쪽 위에 있는 **&#9656; 셀 실행** 메뉴 옵션을 사용하여 실행합니다. 메시지가 표시되면 클러스터를 시작하고 연결합니다.
1. 코드에서 실행하는 Spark 작업이 완료되기를 기다립니다. 코드는 업로드한 파일의 데이터에서 **df1**이라는 데이터 프레임 개체를 만들었습니다.
1. 기존 코드 셀에서 **+** 아이콘을 사용하여 새 코드 셀을 추가합니다. 그런 다음 새 셀에서 다음 코드를 입력합니다.

    ```python
    display(df1)
    ```

1. 새 셀의 오른쪽 위에 있는 **&#9656; 셀 실행** 메뉴 옵션을 사용하여 실행합니다. 이 코드는 다음과 유사한 데이터 프레임의 콘텐츠를 표시합니다.

    | ProductID | ProductName | 범주 | ListPrice |
    | -- | -- | -- | -- |
    | 771 | Mountain-100 Silver, 38 | 산악용 자전거 | 3399.9900 |
    | 772 | Mountain-100 Silver, 42 | 산악용 자전거 | 3399.9900 |
    | , , | , , | , , | , , |

1. 결과 표 위에서 **+** 를 선택한 다음 **시각화**를 선택하여 시각화 편집기를 본 후 다음 옵션을 적용합니다.
    - **시각화 형식**: 막대형
    - **X 열**: 범주
    - **Y 열**: 새 열을 추가하고 **ProductID**를 선택합니다. **개수** 집계를 적용합니다.

    시각화를 저장하고 다음과 같이 Notebook에 표시되는지 확인합니다.

    ![범주별 제품 수를 보여 주는 막대형 차트.](./images/databricks-chart.png)

## 테이블 만들기 및 쿼리

많은 데이터 분석은 Python 또는 Scala와 같은 언어를 사용하여 파일의 데이터를 작업하는 것이 쉬운 반면 많은 데이터 분석 솔루션은 관계형 데이터베이스에 구축됩니다. 그 곳에서 데이터가 테이블에 저장되고 SQL을 사용하여 조작됩니다.

1. **제품 탐색** Notebook에서 이전에 실행한 코드 셀의 차트 출력 아래에서 **+** 아이콘을 사용하여 새 셀을 추가합니다.
2. 새 셀에서 다음 코드를 입력하고 실행합니다.

    ```python
    df1.write.saveAsTable("products")
    ```

3. 셀이 완료되면 다음 코드를 사용하여 셀 아래에 새 셀을 추가합니다.

    ```sql
    %sql

    SELECT ProductName, ListPrice
    FROM products
    WHERE Category = 'Touring Bikes';
    ```

4. 여행용 자전거 범주의 제품 이름과 가격을 반환하는 SQL 코드가 포함된 새 셀을 실행합니다.
5. 사이드바에서 **카탈로그** 링크를 선택하고 **products** 테이블이 기본 데이터베이스 스미카에 만들어졌는지 확인합니다(**default**로 명명됨). Spark 코드를 사용하여 데이터 분석가가 데이터를 탐색하고 분석 보고서를 생성하는 데 사용할 수 있는 사용자 지정 데이터베이스 스키마 및 관계형 테이블의 스키마를 만들 수 있습니다.

## Azure Databricks 리소스 삭제

이제 Azure Databricks 탐색을 완료했으므로 불필요한 Azure 비용을 방지하고 구독에서 용량을 확보하려면 만든 리소스를 삭제해야 합니다.

1. Azure Databricks 작업 영역 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. **dp203-*xxxxxxx*** 리소스 그룹(관리되는 리소스 그룹이 아님)을 선택하고 Azure Databricks 작업 영역이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
