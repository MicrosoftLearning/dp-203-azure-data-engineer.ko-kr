---
lab:
  title: Azure Databricks에서 Delta Lake 사용
  ilt-use: Optional demo
---

# Azure Databricks에서 Delta Lake 사용

Delta Lake는 데이터 레이크 위에 Spark에 대한 트랜잭션 데이터 스토리지 계층을 빌드하는 오픈 소스 프로젝트입니다. Delta Lake는 일괄 처리 및 스트리밍 데이터 작업 모두에 대한 관계형 의미 체계에 대한 지원을 추가하고 Apache Spark를 사용하여 데이터 레이크의 기본 파일을 기반으로 하는 테이블에서 데이터를 처리하고 쿼리할 수 있는 Lakehouse 아키텍처를 만들 수 있습니다.

이 연습을 완료하는 데 약 **40**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Databricks 작업 영역 프로비저닝

이 연습에서는 스크립트를 사용하여 새 Azure Databricks 작업 영역을 프로비저닝합니다.

> **팁**: *표준* 또는 *Trial* Azure Databricks 작업 영역이 이미 있는 경우 이 절차를 건너뛸 수 있습니다.

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
    cd dp-203/Allfiles/labs/25
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).

7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Databricks 설명서의 [Delta 기술 소개](https://learn.microsoft.com/azure/databricks/introduction/delta-comparison) 문서를 검토합니다.

## 클러스터 생성

Azure Databricks는 Apache Spark 클러스터를 사용하여 여러 노드에서 병렬로 데이터를 처리하는 분산 처리 플랫폼입니다. 각 클러스터는 작업을 조정하는 드라이버 노드와 처리 작업을 수행하는 작업자 노드로 구성됩니다.

> **팁**: Azure Databricks 작업 영역에 13.3 LTS 런타임 버전이 있는 클러스터가 이미 있는 경우 해당 클러스터를 사용하여 이 연습을 완료하고 이 절차를 건너뛸 수 있습니다.

1. Azure Portal에서 스크립트로 만든 **dp203-*xxxxxxx*** 리소스 그룹(또는 기존 Azure Databricks 작업 영역이 포함된 리소스 그룹)으로 이동
1. Azure Databricks Service 리소스(설치 스크립트를 사용하여 만든 경우 **databricks*xxxxxxx***라는 이름)를 선택합니다.
1. 작업 영역의 **개요** 페이지에서 **작업 영역 시작** 단추를 사용하여 새 브라우저 탭에서 Azure Databricks 작업 영역을 엽니다. 메시지가 표시되면 로그인합니다.

    > **팁**: Databricks 작업 영역 포털을 사용하면 다양한 팁과 알림이 표시될 수 있습니다. 이를 해제하고 제공된 지침에 따라 이 연습의 작업을 완료합니다.

1. Azure Databricks 작업 영역 포털을 보고 왼쪽의 사이드바에 수행할 수 있는 다양한 작업에 대한 아이콘이 포함되어 있는지 확인합니다.

1. **(+) 새로 만들기** 작업을 선택한 다음, **클러스터**를 선택합니다.
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

## Notebook을 사용하여 델타 레이크 탐색

이 연습에서는 Notebook의 코드를 사용하여 Azure Databricks의 델타 레이크를 탐색합니다.

1. 작업 영역에 대한 Azure Databricks 작업 영역 포털의 왼쪽 사이드바에서 **작업 영역**을 선택합니다. 그런 다음, **&#8962; 홈** 폴더를 선택합니다.
1. 페이지 위쪽의 사용자 이름 옆에 있는 **&#8942;** 메뉴에서 **가져오기**를 선택합니다. 그런 다음 **가져오기** 대화 상자에서 **URL**을 선택하고 `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/25/Delta-Lake.ipynb`에서 Notebook을 가져오기
1. Notebook을 클러스터에 연결하고 포함된 지침을 따릅니다. 포함된 셀을 실행하여 델타 레이크 기능을 탐색합니다.

## Azure Databricks 리소스 삭제

이제 Azure Databricks에서 Delta Lake 탐색을 완료했으므로 불필요한 Azure 비용을 방지하고 구독에서 용량을 확보하려면 만든 리소스를 삭제해야 합니다.

1. Azure Databricks 작업 영역 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. **dp203-*xxxxxxx*** 리소스 그룹(관리되는 리소스 그룹이 아님)을 선택하고 Azure Databricks 작업 영역이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
