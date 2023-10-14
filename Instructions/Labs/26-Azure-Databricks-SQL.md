---
lab:
  title: Azure Databricks에서 SQL Warehouse 사용
  ilt-use: Optional demo
---

# Azure Databricks에서 SQL Warehouse 사용

SQL은 데이터를 쿼리하고 조작하기 위한 업계 표준 언어입니다. 많은 데이터 분석가가 SQL을 사용하여 관계형 데이터베이스의 테이블을 쿼리하여 데이터 분석을 수행합니다. Azure Databricks에는 데이터 레이크의 파일에 관계형 데이터베이스 계층을 제공하기 위해 Spark 및 Delta Lake 기술을 기반으로 하는 SQL 기능이 포함되어 있습니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## 시작하기 전에

Azure Databricks SQL Warehouse를 프로비저닝하려면 하나 이상의 지역에서 관리 수준 액세스 권한과 충분한 할당량이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Databricks 작업 영역 프로비저닝

이 연습에서는 프리미엄 계층 Azure Databricks 작업 영역이 필요합니다.

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
    cd dp-203/Allfiles/labs/26
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).

7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 [Azure Databricks 설명서에서 Azure Databricks의 데이터 웨어하우징이란?](https://learn.microsoft.com/azure/databricks/sql/) 문서를 검토합니다.

## SQL Warehouse 보기 및 시작

1. Azure Databricks 작업 영역 리소스가 배포되면 Azure Portal로 이동합니다.
2. Azure Databricks 작업 영역의 **개요** 페이지에서 **작업 영역 시작** 단추를 사용하여 새 브라우저 탭에서 Azure Databricks 작업 영역을 엽니다. 메시지가 표시되면 로그인합니다.
3. **현재 데이터 프로젝트가 무엇인가요?** 메시지가 표시되면 **마침**을 선택하여 닫습니다. 그런 다음, Azure Databricks 작업 영역 포털을 보고 왼쪽의 사이드바에 작업 범주의 이름이 포함되어 있음을 확인합니다.

    >**팁**: Databricks 작업 영역 포털을 사용하면 다양한 팁과 알림이 표시될 수 있습니다. 이를 해제하고 제공된 지침에 따라 이 연습의 작업을 완료합니다.

1. 사이드바의 **SQL**에서 **SQL 웨어하우스를** 선택합니다.
1. 작업 영역에 **Starter Warehouse**라는 SQL Warehouse가 이미 포함되어 있는지 확인합니다.
1. SQL Warehouse의 **작업**( **&#8285;** ) 메뉴에서 **편집**을 선택합니다. 그런 다음, **클러스터 크기** 속성을 **2X-Small**로 설정하고 변경 내용을 저장합니다.
1. **시작** 단추를 사용하여 SQL Warehouse를 시작합니다(1~2분 정도 걸릴 수 있음).

> **참고**: SQL Warehouse를 시작하지 못하면 Azure Databricks 작업 영역이 프로비저닝된 지역에서 구독 할당량이 부족할 수 있습니다. 자세한 내용은 [필수 Azure vCPU 할당량](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota)을 참조하세요. 이 경우 웨어하우스가 시작되지 않을 때 오류 메시지에 자세히 설명된 대로 할당량 증가를 요청할 수 있습니다. 또는 작업 영역을 삭제하고 다른 지역에 새 작업 영역을 만들 수 있습니다. `./setup.ps1 eastus`와 같이 설치 스크립트에 대한 매개 변수로 지역을 지정할 수 있습니다.

## 데이터베이스 스키마 만들기

1. SQL Warehouse가 *실행* 중일 때 사이드바에서 **SQL 편집기를** 선택합니다.
2. **스키마 브라우저** 창에서 *hive_metastore* 카탈로그에 default라는 데이터베이스가 포함되어 있음을 확인**합니다**.
3. **새 쿼리** 창에 다음 SQL 코드를 입력합니다.

    ```sql
    CREATE SCHEMA adventureworks;
    ```

4. **&#9658;실행(1000)** 단추를 사용하여 SQL 코드를 실행합니다.
5. 코드가 성공적으로 실행되면 **스키마 브라우저** 창에서 창 아래쪽의 새로 고침 단추를 사용하여 목록을 새로 고칩니다. 그런 다음 **hive_metastore** 및 **adventureworks를** 확장하고 데이터베이스가 만들어졌지만 테이블이 없는지 확인합니다.

테이블에 **기본** 데이터베이스를 사용할 수 있지만 분석 데이터 저장소를 빌드할 때는 특정 데이터에 대한 사용자 지정 데이터베이스를 만드는 것이 가장 좋습니다.

## 테이블 만들기

1. [**products.csv**](https://raw.githubusercontent.com/MicrosoftLearning/dp-203-azure-data-engineer/master/Allfiles/labs/26/data/products.csv) 파일을 로컬 컴퓨터에 다운로드하여 **products.csv**저장합니다.
1. Azure Databricks 작업 영역 포털의 사이드바에서 **(+) 새로** 만들기를 선택한 다음 **파일 업로드** 를 선택하고 컴퓨터에 다운로드한 **products.csv** 파일을 업로드합니다.
1. **데이터 업로드** 페이지에서 **adventureworks** 스키마를 선택하고 테이블 이름을 **제품**으로 설정합니다. 그런 다음, 페이지의 왼쪽 아래 모서리에서 **테이블 만들기** 를 선택합니다.
1. 테이블이 만들어지면 세부 정보를 검토합니다.

파일에서 데이터를 가져와 테이블을 만들면 데이터베이스를 쉽게 채울 수 있습니다. Spark SQL을 사용하여 코드를 사용하여 테이블을 만들 수도 있습니다. 테이블 자체는 하이브 메타스토어의 메타데이터 정의이며, 포함된 데이터는 DBFS(Databricks 파일 시스템) 스토리지에 Delta 형식으로 저장됩니다.

## 쿼리 만들기

1. 사이드바에서 **(+) 새로 만들기**를 선택한 다음, **쿼리**를 선택합니다.
2. **스키마 브라우저** 창에서 **hive_metastore** 및 **adventureworks를** 확장하고 제품 테이블이 나열되어 있는지 확인**합니다**.
3. **새 쿼리** 창에 다음 SQL 코드를 입력합니다.

    ```sql
    SELECT ProductID, ProductName, Category
    FROM adventureworks.products; 
    ```

4. **&#9658;실행(1000)** 단추를 사용하여 SQL 코드를 실행합니다.
5. 쿼리가 완료되면 결과 테이블을 검토합니다.
6. 쿼리 편집기의 오른쪽 위에 있는 **저장** 단추를 사용하여 쿼리를 **제품 및 범주**로 저장합니다.

쿼리를 저장하면 나중에 동일한 데이터를 쉽게 다시 검색할 수 있습니다.

## 대시보드 만들기

1. 사이드바에서 **(+) 새로 만들기**를 선택한 다음, **대시보드**를 선택합니다.
2. **새 대시보드** 대화 상자에서 **Adventure Works Products**라는 이름을 입력하고 **저장**을 선택합니다.
3. **Adventure Works Products** 대시보드의 **추가** 드롭다운 목록에서 **시각화**를 선택합니다.
4. **시각화 위젯 추가** 대화 상자에서 **제품 및 범주** 쿼리를 선택합니다. 그런 다음, **새 시각화 만들기**를 선택하고 제목을 **범주별 제품**으로 설정합니다. 그리고 **시각화 만들기**를 선택합니다.
5. 시각화 편집기에서 다음 속성을 설정합니다.
    - **시각화 형식**: 막대형
    - **가로 차트**: 선택됨
    - **Y 열**: 범주
    - **X 열**: 제품 ID: 개수
    - **그룹화 기준**: 범주
    - **범례 배치**: 자동(유연함)
    - **범례 항목 순서**: 기본
    - **스태킹**: 스택
    - **백분율로 값 정규화**: 선택 <u>취소됨</u>
    - **누락 및 NULL 값**: 차트에 표시 안 함

6. 시각화를 저장하고 대시보드에서 봅니다.
7. **편집 완료**를 선택하여 사용자가 볼 수 있는 대시보드를 봅니다.

대시보드는 비즈니스 사용자와 데이터 테이블 및 시각화를 공유하는 좋은 방법입니다. 대시보드를 주기적으로 새로 고치고 구독자에게 메일을 보내도록 예약할 수 있습니다.

## Azure Databricks 리소스 삭제

이제 Azure Databricks에서 SQL Warehouse 탐색을 완료했으므로 불필요한 Azure 비용을 방지하고 구독에서 용량을 확보하려면 만든 리소스를 삭제해야 합니다.

1. Azure Databricks 작업 영역 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. 관리되는 리소스 그룹이 아닌 Azure Databricks 작업 영역이 포함된 리소스 그룹을 선택합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.

    몇 분이 지나면 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
