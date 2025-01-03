---
lab:
  title: Azure Databricks에서 SQL Warehouse 사용
  ilt-use: Optional demo
---

SQL은 데이터를 쿼리하고 조작하기 위한 업계 표준 언어입니다. 많은 데이터 분석가가 SQL을 사용하여 관계형 데이터베이스의 테이블을 쿼리하여 데이터 분석을 수행합니다. Azure Databricks에는 데이터 레이크의 파일에 관계형 데이터베이스 계층을 제공하기 위해 Spark 및 Delta Lake 기술을 기반으로 하는 SQL 기능이 포함되어 있습니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## Azure Databricks 작업 영역 프로비저닝

> **팁**: *프리미엄* 또는 *평가판* Azure Databricks 작업 영역이 이미 있는 경우 이 절차를 건너뛰고 기존 작업 영역을 사용할 수 있습니다.

이 연습에서는 스크립트를 사용하여 새 Azure Databricks 작업 영역을 프로비저닝합니다. 이 스크립트는 이 연습에 필요한 컴퓨팅 코어에 대한 충분한 할당량이 있는 Azure 구독에서 *프리미엄* 계층 Azure Databricks 작업 영역 리소스를 만들려고 시도하고, 사용자 계정에 Azure Databricks 작업 영역 리소스를 만들기 위한 구독에 충분한 권한이 있다고 가정합니다. 할당량 또는 권한 부족으로 인해 스크립트가 실패하는 경우 [Azure Portal에서 대화형으로 Azure Databricks 작업 영역을 만들 수 ](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace)있습니다.

1. 웹 브라우저의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 Cloud Shell 창 왼쪽 상단에 있는 드롭다운 메뉴를 사용하여 이를 ***PowerShell***로 변경합니다.

3. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
    ```

5. 리포지토리가 복제된 후 다음 명령을 입력하여 사용 가능한 지역에서 Azure Databricks 작업 영역을 프로비전하는 **setup.ps1** 스크립트를 실행합니다.

    ```
    ./mslearn-databricks/setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Databricks 설명서의 [Azure Databricks의 데이터 웨어하우징이란?](https://learn.microsoft.com/azure/databricks/sql/) 문서를 검토합니다.

## SQL Warehouse 보기 및 시작

1. Azure Databricks 작업 영역 리소스가 배포되면 Azure Portal로 이동합니다.
1. Azure Databricks 작업 영역의 **개요** 페이지에서 **작업 영역 시작** 단추를 사용하여 새 브라우저 탭에서 Azure Databricks 작업 영역을 엽니다. 메시지가 표시되면 로그인합니다.

    > **팁**: Databricks 작업 영역 포털을 사용하면 다양한 팁과 알림이 표시될 수 있습니다. 이를 해제하고 제공된 지침에 따라 이 연습의 작업을 완료합니다.

1. Azure Databricks 작업 영역 포털을 보고 왼쪽의 사이드바에 작업 범주의 이름이 포함되어 있는지 확인합니다.
1. 사이드바의 **SQL**아래에서 **SQL Warehouse**를 선택합니다.
1. 작업 영역에 **Starter Warehouse**라는 SQL Warehouse가 이미 포함되어 있는지 확인합니다.
1. SQL Warehouse의 **작업**( **&#8285;** ) 메뉴에서 **편집**을 선택합니다. 그런 다음, **클러스터 크기** 속성을 **2X-Small**로 설정하고 변경 내용을 저장합니다.
1. **시작** 단추를 사용하여 SQL Warehouse를 시작합니다(1~2분 정도 걸릴 수 있음).

> **참고**: SQL Warehouse를 시작하지 못하면 Azure Databricks 작업 영역이 프로비저닝된 지역에서 구독 할당량이 부족할 수 있습니다. 자세한 내용은 [필수 Azure vCPU 할당량](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota)을 참조하세요. 이 경우 웨어하우스가 시작되지 않을 때 오류 메시지에 자세히 설명된 대로 할당량 증가를 요청할 수 있습니다. 또는 작업 영역을 삭제하고 다른 지역에 새 작업 영역을 만들 수 있습니다. `./setup.ps1 eastus`와 같이 설치 스크립트에 대한 매개 변수로 지역을 지정할 수 있습니다.

## 데이터베이스 스키마 만들기

1. SQL Warehouse가 *실행 중*인 경우 사이드바에서 **SQL 편집기**를 선택합니다.
2. **스키마 브라우저** 창에서 *hive_metastore* 범주에 이름이 **default**인 데이터베이스가 포함되어 있는지 확인합니다.
3. **새 쿼리** 창에 다음 SQL 코드를 입력합니다.

    ```sql
   CREATE DATABASE retail_db;
    ```

4. **&#9658;Run (1000)** 단추를 사용하여 SQL 코드를 실행합니다.
5. 코드가 성공적으로 실행되면 **스키마 브라우저** 창에서 상단에 있는 새로 고침 단추를 사용하여 목록을 새로 고칩니다. 그런 다음 **hive_metastore** 및 **retail_db**를 확장하고 데이터베이스가 만들어졌지만 테이블이 없는지 확인합니다.

테이블에 **기본** 데이터베이스를 사용할 수 있지만 분석 데이터 저장소를 빌드할 때는 특정 데이터에 대한 사용자 지정 데이터베이스를 만드는 것이 가장 좋습니다.

## 테이블 만들기

1. [`products.csv`](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/products.csv)파일을 로컬 컴퓨터에 다운로드하여 **products.csv**로 저장합니다.
1. Azure Databricks 작업 영역 포털의 사이드바에서 **새로 만들기(+)** 를 선택한 다음, **데이터**를 선택합니다.
1. **데이터 추가** 페이지에서 **테이블 만들기 또는 수정**을 선택하고 다운로드한 **products.csv** 파일을 컴퓨터에 업로드합니다.
1. **파일 업로드 페이지의 테이블 만들기 또는 수정**에서 **retail_db** 스키마를 선택하고 테이블 이름을 **제품**으로 설정합니다. 그런 다음, 페이지의 오른쪽 아래 모서리에서 **테이블 만들기**를 선택합니다.
1. 테이블이 만들어지면 세부 정보를 검토합니다.

파일에서 데이터를 가져와 테이블을 만들면 데이터베이스를 쉽게 채울 수 있습니다. Spark SQL을 사용하여 코드를 사용하여 테이블을 만들 수도 있습니다. 테이블 자체는 하이브 메타스토어의 메타데이터 정의이며, 포함된 데이터는 DBFS(Databricks 파일 시스템) 스토리지에 Delta 형식으로 저장됩니다.

## 대시보드 만들기

1. 사이드바에서 **(+) 새로 만들기**를 선택한 다음, **대시보드**를 선택합니다.
2. 새 대시보드 이름을 선택하고 `Retail Dashboard`로 변경합니다.
3. **데이터** 탭에서 **SQL에서 만들기**를 선택하고 다음 쿼리를 사용합니다.

    ```sql
   SELECT ProductID, ProductName, Category
   FROM retail_db.products; 
    ```

4. **실행**을 선택한 다음 제목 없는 데이터 세트의 이름을 `Products and Categories`로 바꿉니다.
5. **캔버스** 탭을 선택한 다음 **시각화 추가**를 선택합니다.
6. 시각화 편집기에서 다음 속성을 설정합니다.
    
    - **데이터 세트**: 제품 및 범주
    - **시각화 형식**: 막대형
    - **X축**: COUNT(ProductID)
    - **Y축**: 범주

7. **게시**를 선택하여 사용자가 볼 수 있는 대시보드를 확인합니다.

대시보드는 비즈니스 사용자와 데이터 테이블 및 시각화를 공유하는 좋은 방법입니다. 대시보드를 주기적으로 새로 고치고 구독자에게 메일을 보내도록 예약할 수 있습니다.

## 정리

Azure Databricks 포털의 **SQL Warehouse** 페이지에서 SQL Warehouse 를 선택하고 **&#9632; 중지**를 선택하여 종료합니다.

이제 Azure Databricks 탐색을 완료했으므로 불필요한 Azure 비용을 방지하고 구독에서 용량을 확보하기 위해 만든 리소스를 삭제할 수 있습니다.
