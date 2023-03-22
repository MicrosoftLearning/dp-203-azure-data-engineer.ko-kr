---
lab:
  title: Azure Data Factory를 사용하여 Azure Databricks Notebook 자동화
  ilt-use: Suggested demo
---

# Azure Data Factory를 사용하여 Azure Databricks Notebook 자동화

Azure Databricks의 Notebook을 사용하여 데이터 파일 처리 및 테이블에 데이터 로드와 같은 데이터 엔지니어링 작업을 수행할 수 있습니다. 데이터 엔지니어링 파이프라인의 일부로 이러한 작업을 오케스트레이션해야 하는 경우 Azure Data Factory를 사용할 수 있습니다.

이 연습을 완료하는 데 약 **40**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure 리소스 프로비전

이 연습에서는 스크립트를 사용하여 Azure 구독에서 새 Azure Databricks 작업 영역 및 Azure Data Factory 리소스를 프로비저닝합니다.

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
    cd dp-203/Allfiles/labs/27
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).

7. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 5분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 [Azure Data Factory란?](https://docs.microsoft.com/azure/data-factory/introduction)을 검토합니다.
8. 스크립트가 완료되면 클라우드 셸 창을 닫고 스크립트에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 Azure Databricks 작업 영역 및 Azure Data Factory(V2) 리소스가 포함되어 있는지 확인합니다(리소스 그룹 보기를 새로 고쳐야 할 수도 있음).

## Notebook 가져오기

Azure Databricks 작업 영역에서 Notebook을 만들어 다양한 프로그래밍 언어로 작성된 코드를 실행할 수 있습니다. 이 연습에서는 일부 Python 코드가 포함된 기존 Notebook을 가져옵니다.

1. Azure Portal의 **dp203-*xxxxxxx*** 리소스 그룹에서 **databricks*xxxxxxx*** Azure Databricks 서비스 리소스를 선택합니다.
2. **databricks*xxxxxxx***의 **개요** 페이지에서 **작업 영역 시작** 단추를 사용하여 새 브라우저 탭에서 Azure Databricks 작업 영역을 엽니다. 메시지가 표시되면 로그인합니다.
3. **현재 데이터 프로젝트가 무엇인가요?** 메시지가 표시되면 **마침**을 선택하여 닫습니다. 그런 다음, Azure Databricks 작업 영역 포털을 보고 왼쪽의 사이드바에 수행할 수 있는 다양한 작업에 대한 아이콘이 포함되어 있는지 확인합니다. 사이드바가 확장되어 작업 범주의 이름을 표시합니다.
4. 사이드바를 확장하고 **작업 영역** 탭을 선택합니다. 그런 다음, **사용자** 폴더를 선택하고 **&#8962; *your_user_name*** 폴더의 **&#9662;** 메뉴에서 **가져오기**를 선택합니다.
5. **Notebook 가져오기** 대화 상자에서 **URL**을 선택하고 `https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/raw/master/Allfiles/labs/27/Process-Data.ipynb`에서 Notebook을 가져옵니다.
6. **&#8962; 홈**을 선택한 다음, 방금 가져온 **데이터 처리** Notebook을 엽니다.

    **참고**: 팁이 표시되면 **확인** 단추를 사용하여 닫습니다. 이는 작업 영역 인터페이스를 처음 탐색할 때 표시될 수 있는 향후 팁에 적용됩니다.

7. Notebook의 콘텐츠를 검토합니다. 여기에는 다음과 같은 몇 가지 Python 코드 셀이 포함됩니다.
    - **folder**라는 매개 변수가 전달된 경우 이를 검색합니다(그렇지 않으면 *data*의 기본값 사용).
    - GitHub에서 데이터를 다운로드하고 DBFS(Databricks 파일 시스템)의 지정된 폴더에 저장합니다.
    - Notebook을 종료하고 데이터가 출력으로 저장된 경로를 반환합니다.

    > **팁**: Notebook에는 필요한 데이터 처리 논리가 실제로 포함될 수 있습니다. 이 간단한 예제는 주요 원칙을 표시하도록 설계되었습니다.

## Azure Data Factory와 Azure Databricks 통합 사용

Azure Data Factory 파이프라인에서 Azure Databricks를 사용하려면 Azure Data Factory에 Azure Databricks 작업 영역에 액세스할 수 있는 연결된 서비스를 만들어야 합니다.

### 액세스 토큰 생성

1. Azure Databricks 포털의 왼쪽 위 메뉴 모음에서 사용자 이름을 선택한 다음, 드롭다운에서 **사용자 설정**을 선택합니다.
2. **사용자 설정** 페이지의 **액세스 토큰** 탭에서 **새 토큰 생성**을 선택하고 *Data Factory* 주석과 빈 수명(토큰이 만료되지 않도록)을 사용하여 새 토큰을 생성합니다. **완료***를 선택하기 <u>전에</u> 표시될 때 토큰을 *복사해야 합니다.
3. 복사한 토큰을 텍스트 파일에 붙여넣어 이 연습의 뒷부분에서 편리하게 사용할 수 있도록 합니다.

### Azure Data Factory에서 연결된 서비스 만들기

1. Azure Portal로 돌아가서 **dp203-*xxxxxxx*** 리소스 그룹에서 **adf*xxxxxxx*** Azure Data Factory 리소스를 선택합니다.
2. **개요** 페이지에서 **스튜디오 시작**을 선택하여 Azure Data Factory Studio를 엽니다. 메시지가 표시되면 로그인합니다.
3. Azure Data Factory Studio에서 **>>** 아이콘을 사용하여 왼쪽의 탐색 창을 확장합니다. 그런 다음, **관리** 페이지를 선택합니다.
4. **관리** 페이지의 **연결된 서비스** 탭에서 **+ 새로 만들기**를 선택하여 새 연결된 서비스를 추가합니다.
5. **새 연결된 서비스** 창에서 맨 위에 있는 **컴퓨팅** 탭을 선택합니다. 그런 다음, **Azure Databricks**를 선택합니다.
6. 계속하여 다음 설정을 사용하여 연결된 서비스를 만듭니다.
    - **이름**: AzureDatabricks
    - **설명**: Azure Databricks 작업 영역
    - **통합 런타임을 통해 연결**: AutoResolveInegrationRuntime
    - **계정 선택 방법**: Azure 구독에서
    - **Azure 구독**: *구독 선택*
    - **Databricks 작업 영역**: ***databricksxxxxxxx** 작업 영역 선택*
    - **클러스터 선택**: 새 작업 클러스터
    - **Databrick 작업 영역 URL**: *Databricks 작업 영역 URL로 자동으로 설정*
    - **인증 유형**: 액세스 토큰
    - **액세스 토큰**: *액세스 토큰 붙여넣기*
    - **클러스터 버전**: 10.4 LTS(Scala 2.12, Spark 3.2.1)
    - **클러스터 노드 형식**: Standard_DS3_v2
    - **Python 버전**: 3
    - **작업자 옵션**: 수정됨
    - **작업자**: 1명

## 파이프라인을 사용하여 Azure Databricks Notebook 실행

이제 연결된 서비스를 만들었으므로 파이프라인에서 사용하여 이전에 본 Notebook을 실행할 수 있습니다.

### 파이프라인 만들기

1. Azure Data Factory Studio의 탐색 창에서 **작성자**를 선택합니다.
2. **작성자** 페이지의 **팩토리 리소스** 창에서 **+** 아이콘을 사용하여 **파이프라인**을 추가합니다.
3. 새 파이프라인의 **속성** 창에서 해당 이름을 **Databricks를 사용하여 데이터 처리**로 변경합니다. 그런 다음, 도구 모음의 오른쪽 끝에 있는 **속성** 단추( **&#128463;<sub>*</sub>** 와 유사함)를 사용하여 **속성** 창을 숨깁니다.
4. **작업** 창에서 **Databricks**를 펼치고 **Notebook** 작업을 파이프라인 디자이너 화면으로 끕니다.
5. 새 **Notebook1** 작업을 선택한 상태에서 아래쪽 창에서 다음 속성을 설정합니다.
    - **일반**:
        - **이름**: 데이터 처리
    - **Azure Databricks**:
        - **Databricks 연결된 서비스**: *이전에 만든 **AzureDatabricks** 연결된 서비스 선택*
    - **설정**:
        - **Notebook 경로**: ***Users/your_user_name** 폴더로 이동하여 **데이터 처리** Notebook 선택*
        - **기본 매개 변수**: *값이 **product_data**인 **folder**라는 새 매개 변수 추가*
6. 파이프라인 디자이너 화면 위의 **유효성 검사** 단추를 사용하여 파이프라인의 유효성을 검사합니다. 그런 다음, **모두 게시** 단추를 사용하여 게시(저장)합니다.

### 파이프라인 실행

1. 파이프라인 디자이너 화면 위에서 **트리거 추가**를 선택한 다음, **지금 트리거**를 선택합니다.
2. **파이프라인 실행** 창에서 **확인**을 선택하여 파이프라인을 실행합니다.
3. 왼쪽의 탐색 창에서 **모니터링**을 선택하고 **파이프라인 실행** 탭에서 **Databricks를 사용하여 데이터 처리** 파이프라인을 확인합니다. Spark 클러스터를 동적으로 만들고 Notebook을 실행하기 때문에 실행하는 데 시간이 걸릴 수 있습니다. **파이프라인 실행** 페이지의 **&#8635; 새로 고침** 단추를 사용하여 상태를 새로 고칠 수 있습니다.

    > **참고**: 파이프라인이 실패하면 작업 클러스터를 만들기 위해 Azure Databricks 작업 영역이 프로비저닝된 지역에서 구독 할당량이 부족할 수 있습니다. 자세한 내용은 [CPU 코어 제한으로 클러스터 생성 방지](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit)를 참조하세요. 이 경우 작업 영역을 삭제하고 다른 지역에 새 작업 영역을 만들 수 있습니다. `./setup.ps1 eastus`와 같이 설치 스크립트에 대한 매개 변수로 지역을 지정할 수 있습니다.

4. 실행이 성공하면 해당 이름을 선택하여 실행 세부 정보를 봅니다. 그런 다음, **Databricks를 사용하여 데이터 처리** 페이지의 **작업 실행** 섹션에서 **데이터 처리** 작업을 선택하고 해당 ***출력*** 아이콘을 사용하여 작업에서 출력 JSON을 볼 수 있습니다. 이는 다음과 유사합니다.
    ```json
    {
        "runPageUrl": "https://adb-..../run/...",
        "runOutput": "dbfs:/product_data/products.csv",
        "effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (East US)",
        "executionDuration": 61,
        "durationInQueue": {
            "integrationRuntimeQueue": 0
        },
        "billingReference": {
            "activityType": "ExternalActivity",
            "billableDuration": [
                {
                    "meterType": "AzureIR",
                    "duration": 0.03333333333333333,
                    "unit": "Hours"
                }
            ]
        }
    }
    ```

5. Notebook에서 데이터를 저장한 *경로* 변수인 **runOutput** 값을 확인합니다.

## Azure Databricks 리소스 삭제

이제 Azure Databricks와의 Azure Data Factory 통합 탐색을 완료했으므로 불필요한 Azure 비용을 방지하고 구독에서 용량을 확보하려면 만든 리소스를 삭제해야 합니다.

1. Azure Databricks 작업 영역 및 Azure Data Factory 스튜디오 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. Azure Databricks 및 Azure Data Factory 작업 영역(관리되는 리소스 그룹이 아님)이 포함된 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음 **삭제**를 선택합니다.

    몇 분이 지나면 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
