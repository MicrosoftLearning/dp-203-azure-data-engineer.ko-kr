---
lab:
  title: 파이프라인에서 Apache Spark Notebook 사용
  ilt-use: Lab
---

# 파이프라인에서 Apache Spark Notebook 사용

이 연습에서는 Apache Spark Notebook을 실행하는 작업이 포함된 Azure Synapse Analytics 파이프라인을 만듭니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure Synapse Analytics 작업 영역 프로비저닝

데이터 레이크 스토리지 및 Spark 풀에 액세스할 수 있는 Azure Synapse Analytics 작업 영역이 필요합니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics 작업 영역을 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    ![Cloud Shell 창이 있는 Azure Portal](./images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. Cloud Shell은 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 —, **&#9723;** , **X** 아이콘을 사용하여 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```powershell
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 연습의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```powershell
    cd dp-203/Allfiles/labs/11
    ./setup.ps1
    ```
    
6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse 파이프라인](https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-performance-pipelines) 문서를 검토합니다.

## 대화형으로 Spark Notebook 실행

Notebook을 사용하여 데이터 변환 프로세스를 자동화하기 전에 나중에 자동화할 프로세스를 더 잘 이해하기 위해 Notebook을 대화형으로 실행하는 것이 유용할 수 있습니다.

1. 스크립트가 완료되면 Azure Portal에서 만든 dp203-xxxxxxx 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 ›› 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 여러 페이지가 표시됩니다.
4. **데이터** 페이지에서 연결됨 탭을 보고 작업 영역에 **synapsexxxxxxx(Primary - datalakexxxxxxx)** 와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
5. 스토리지 계정을 확장하고 **파일(기본)** 이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
6. 파일 컨테이너를 선택하고 여기에 변환하려는 데이터 파일이 포함된 **데이터**라는 폴더가 포함되어 있는지 확인합니다.
7. **data**** 폴더를 열고 포함된 CSV 파일을 봅니다. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 데이터의 샘플을 확인합니다. 완료되면 미리 보기를 닫습니다.
8. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 포함된 데이터를 확인합니다. 파일에 머리글 행이 포함되어 있으므로 열 머리글을 표시하는 옵션을 선택할 수 있습니다.
9. 미리 보기를 닫습니다. 그런 다음 Allfiles/labs/11/Notebooks에서 Spark Transform.ipynb**[를 다운로드**합니다.](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/11/notebooks)

    > **참고**: Ctrl+a***를 사용한 다음 ctrl+c***를 사용하여 ***이 텍스트를 복사한 다음***, 메모장과 같은 Ctrl+v***를 사용하여 ***도구에 붙여넣은 다음 파일을 사용하여 모든 파일***의 ***파일 형식으로 Spark Transform.ipynb**로 **저장하는 것이 가장 좋습니다. GitHub 내에서 파일을 선택한 다음 줄임표를 선택한 다음 다운로드를 선택하여 기억할 수 있는 위치에 저장하는 옵션도 있습니다. 
    ![GitHub에서 Notebook 파일 다운로드](./images/select-download-notebook.png)

10 그런 다음 **개발** 페이지에서 전자 필기장을** 확장**하여 + 가져오기 옵션을 클릭합니다.

    ![Spark Notebook import](./image/../images/spark-notebook-import.png)
        
12. 방금 다운로드하여 Spark Transfrom.ipynb**으로 **저장한 파일을 선택합니다.
13. Notebook을 **spark*xxxxxxx*** Spark 풀에 연결합니다.
14. Notebook에서 메모를 검토하고 코드 셀을 실행합니다.

    > **참고**: Spark 풀을 시작해야 하므로 첫 번째 코드 셀을 실행하는 데 몇 분 정도 걸립니다. 후속 셀은 더 빠르게 실행됩니다.
9. Notebook에 포함된 코드를 검토하고 다음을 확인합니다.
    - 변수를 설정하여 고유한 폴더 이름을 정의합니다.
    - **/data** 폴더에서 CSV 판매 주문 데이터를 로드합니다.
    - 고객 이름을 여러 필드로 분할하여 데이터를 변환합니다.
    - 변환된 데이터를 고유하게 명명된 폴더에 Parquet 형식으로 저장합니다.
10. Notebook 툴바에서 Notebook을 **spark*xxxxxxx*** Spark 풀에 연결한 다음 **&#9655; 모두 실행** 단추를 사용하여 노트북의 모든 코드 셀을 실행합니다.
  
    코드 셀을 실행하기 전에 Spark 세션을 시작하는 데 몇 분 정도 걸릴 수 있습니다.

11. 모든 Notebook 셀이 실행된 후 변환된 데이터가 저장된 폴더의 이름을 기록해 둡니다.
12. **파일** 탭(여전히 열려 있어야 함)으로 전환하고 루트 **파일** 폴더를 봅니다. 필요한 경우 **더보기** 메뉴에서 **새로 고침**을 선택하여 새 폴더를 확인합니다. 그런 다음 이를 열어 Parquet 파일이 포함되어 있는지 확인합니다.
13. 루트 **파일** 폴더로 돌아가 노트북에서 생성된 고유한 이름의 폴더를 선택하고 **새 SQL 스크립트** 메뉴에서 **상위 100개 행 선택**을 선택합니다.
14. **상위 100개 행 선택** 창에서 파일 형식을 **Parquet 형식**으로 설정하고 변경 사항을 적용합니다.
15. 열리는 새 SQL 스크립트 창에서 **&#9655; 실행** 단추를 눌러 SQL 코드를 실행하고 변환된 판매 주문 데이터를 반환하는지 확인합니다.

## 파이프라인에서 Notebook 실행

이제 변환 프로세스를 이해했으므로 파이프라인에서 Notebook을 캡슐화하여 자동화할 준비가 되었습니다.

### 매개 변수 셀 만들기

1. Synapse Studio에서 Notebook이 포함된 **Spark 변환** 탭으로 돌아가 도구 모음의 오른쪽 끝에 있는 **...** 메뉴에서 **출력 지우기**을 선택합니다.
2. 첫 번째 코드 셀(**folderName** 변수를 설정하는 코드 포함)을 선택합니다.
3. 코드 셀의 오른쪽 상단에 있는 팝업 도구 모음의 **...** 메뉴에서 **\[@] 매개변수 셀 토글**을 선택합니다. **매개변수**라는 단어가 셀 오른쪽 하단에 표시되는지 확인합니다.
4. 도구 모음에서 **게시** 단추를 사용하여 변경사항을 저장합니다.

### 파이프라인 만들기

1. Synapse Studio에서 **통합** 페이지를 선택합니다. 그런 다음, **+** 메뉴에서 **파이프라인**을 선택하여 새 파이프라인을 만듭니다.
2. 새 파이프라인의 **속성** 창에서 이름을 **Pipeline1**에서 **판매 데이터 변환**으로 변경합니다. 그런 다음, **속성** 창 위의 **속성** 단추를 사용하여 숨깁니다.
3. **작업** 창에서 **Synapse**를 확장합니다. 그런 다음, 다음과 같이 **Notebook** 작업을 파이프라인 디자인 화면으로 끌어옵니다.

    ![Notebook 작업이 포함된 파이프라인의 스크린샷](images/notebook-pipeline.png)

4. 
5. Notebook 작업의 **일반** 탭에서 이름을 **Spark 변환 실행**으로 변경합니다.
6. Notebook 작업의 **설정** 탭에서 다음 속성을 설정합니다.
    - **Notebook**: **Spark 변환** Notebook을 선택합니다.
    - **기본 매개 변수**: 이 섹션을 확장하고 다음 설정을 사용하여 매개 변수를 정의합니다.
        - **이름**: folderName
        - **형식**: 문자열
        - **값**: **동적 콘텐츠 추가**를 선택하고 매개 변수 값을 *파이프라인 실행 ID* 시스템 변수(`@pipeline().RunId`)로 설정합니다.
    - **Spark 풀**: **spark*xxxxxxx*** 풀을 선택합니다.
    - **실행기 크기**: **소형(vCore 4개, 28GB 메모리)** 을 선택합니다.

    파이프라인 창은 다음과 같이 표시됩니다.

    ![설정이 있는 Notebook 작업이 포함된 파이프라인의 스크린샷.](images/notebook-pipeline-settings.png)

### 파이프라인 게시 및 실행

1. **모두 게시** 단추를 사용하여 파이프라인(및 기타 저장되지 않은 자산)을 게시합니다.
2. 파이프라인 디자이너 창의 위쪽에 있는 **트리거 추가** 메뉴에서 **지금 트리거**를 선택합니다. 그런 다음, **확인**을 선택하여 파이프라인 실행을 확인합니다.

    **참고**: 예약된 시간에 또는 특정 이벤트에 대한 응답으로 파이프라인을 실행하는 트리거를 만들 수도 있습니다.

3. 파이프라인이 실행되기 시작하면 **모니터링** 페이지에서 **파이프라인 실행** 탭을 보고 **판매 데이터 변환** 파이프라인의 상태를 검토합니다.
4. **판매 데이터 변환** 파이프라인을 선택하여 세부 정보를 확인하고 **작업 실행** 창에서 파이프라인 실행 ID를 확인합니다.

    파이프라인을 완료하는 데 5분 이상이 걸릴 수 있습니다. 도구 모음의 **&#8635; 새로 고침** 단추를 사용하여 상태를 확인할 수 있습니다.

5. 파이프라인 실행이 성공하면 **데이터** 페이지에서 **파일** 스토리지 컨테이너로 이동하여 파이프라인 실행 ID로 명명된 새 폴더가 만들어졌고 변환된 판매 데이터에 대한 Parquet 파일이 포함되어 있는지 확인합니다.
   
## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정, 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
