---
lab:
  title: Azure Databricks에서 Spark 사용
  ilt-use: Lab
---

# Azure Databricks에서 Spark 사용

Azure Databricks는 인기 있는 오픈 소스 Databricks 플랫폼의 Microsoft Azure 기반 버전입니다. Azure Databricks는 Apache Spark를 기반으로 하며 파일의 데이터 작업을 포함하는 데이터 엔지니어링 및 분석 작업을 위한 스케일링 성능이 뛰어난 솔루션을 제공합니다. Spark의 이점 중 하나는 Java, Scala, Python, SQL을 포함한 다양한 프로그래밍 언어를 지원한다는 점입니다. Spark는 데이터 정리 및 조작, 통계 분석 및 기계 학습, 데이터 분석 및 시각화를 포함한 데이터 처리 워크로드를 위한 매우 유연한 솔루션입니다.

이 연습을 완료하는 데 약 **45**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Databricks 작업 영역 프로비저닝

이 연습에서는 스크립트를 사용하여 새 Azure Databricks 작업 영역을 프로비저닝합니다.

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
    cd dp-203/Allfiles/labs/24
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).

7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Databricks 설명서의 [Databricks 데이터 과학 및 엔지니어링이란?](https://docs.microsoft.com/azure/databricks/scenarios/what-is-azure-databricks-ws) 문서를 검토합니다.

## 클러스터 생성

Azure Databricks는 Apache Spark 클러스터를 사용하여 여러 노드에서 병렬로 데이터를 처리하는 분산 처리 플랫폼입니다. 각 클러스터는 작업을 조정하는 드라이버 노드와 처리 작업을 수행하는 작업자 노드로 구성됩니다.

> **참고**: 이 연습에서는 랩 환경(리소스가 제한될 수 있음)에서 사용되는 컴퓨팅 리소스를 최소화하기 위해 *단일 노드* 클러스터를 만듭니다. 프로덕션 환경에서는 일반적으로 여러 작업자 노드가 있는 클러스터를 만듭니다.

1. Azure Portal에서 실행한 스크립트에서 만든 **dp203-*xxxxxxx*** 리소스 그룹을 찾습니다.
2. **databricks*xxxxxxx*** Azure Databricks 서비스 리소스를 선택합니다.
3. **databricks*xxxxxxx***의 **개요** 페이지에서 **작업 영역 시작** 단추를 사용하여 새 브라우저 탭에서 Azure Databricks 작업 영역을 엽니다. 메시지가 표시되면 로그인합니다.
4. **현재 데이터 프로젝트가 무엇인가요?** 메시지가 표시되면 **마침**을 선택하여 닫습니다. 그런 다음, Azure Databricks 작업 영역 포털을 보고 왼쪽의 사이드바에 수행할 수 있는 다양한 작업에 대한 아이콘이 포함되어 있는지 확인합니다. 사이드바가 확장되어 작업 범주의 이름을 표시합니다.
5. **(+) 새로 만들기** 작업을 선택한 다음, **클러스터**를 선택합니다.

    **참고**: 팁이 표시되면 **확인** 단추를 사용하여 닫습니다. 이는 작업 영역 인터페이스를 처음 탐색할 때 표시될 수 있는 향후 팁에 적용됩니다.

6. **새 클러스터** 페이지에서 다음 설정을 사용하여 새 클러스터를 만듭니다.
    - **클러스터 이름**: *사용자 이름*의 클러스터(기본 클러스터 이름)
    - **클러스터 모드**: 단일 노드
    - **액세스 모드**(*메시지가 표시되는 경우*): 단일 사용자
    - **Databricks 런타임 버전**: 10.4 LTS(Scala 2.12, Spark 3.2.1)
    - **Photon 가속 사용**: 선택 취소됨
    - **노드 형식**: Standard_DS3_v2
    - *30***분 동안 비활성 상태**이면 **종료**

7. 클러스터가 만들어질 때까지 기다립니다. 1~2분 정도 소요됩니다.

> **참고**: 클러스터를 시작하지 못하면 Azure Databricks 작업 영역이 프로비저닝된 지역에서 구독 할당량이 부족할 수 있습니다. 자세한 내용은 [CPU 코어 제한으로 클러스터 생성 방지](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit)를 참조하세요. 이 경우 작업 영역을 삭제하고 다른 지역에 새 작업 영역을 만들 수 있습니다. `./setup.ps1 eastus`와 같이 설치 스크립트에 대한 매개 변수로 지역을 지정할 수 있습니다.

## Notebook을 사용하여 데이터 살펴보기

많은 Spark 환경에서와 마찬가지로 Databricks는 Notebook을 사용하여 데이터를 탐색하는 데 사용할 수 있는 메모와 대화형 코드 셀을 결합하도록 지원합니다.

1. 왼쪽의 사이드바를 확장하고 **작업 영역** 탭을 선택합니다. 그런 다음, **사용자** 폴더를 선택하고 **&#8962; *your_user_name*** 폴더의 **&#9662;** 메뉴에서 **가져오기**를 선택합니다.
2. **Notebook 가져오기** 대화 상자에서 **URL**을 선택하고 `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/24/Databricks-Spark.ipynb`에서 Notebook을 가져옵니다.
3. **&#8962; 홈**을 선택한 다음, 방금 가져온 **Databricks Spark** Notebook을 엽니다.
4. Notebook이 ***사용자 이름*의 클러스터**에 연결되어 있는지 확인하고 포함된 지침을 따릅니다. 파일의 데이터를 탐색하기 위해 포함된 셀을 실행합니다.

## Azure Databricks 리소스 삭제

이제 Azure Databricks 탐색을 완료했으므로 불필요한 Azure 비용을 방지하고 구독에서 용량을 확보하려면 만든 리소스를 삭제해야 합니다.

1. Azure Databricks 작업 영역 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. **dp203-*xxxxxxx*** 리소스 그룹(관리되는 리소스 그룹이 아님)을 선택하고 Azure Databricks 작업 영역이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
