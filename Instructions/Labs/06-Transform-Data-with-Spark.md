---
lab:
  title: Synapse Analytics에서 Spark를 사용하여 데이터 변환
  ilt-use: Lab
---

# Synapse Analytics에서 Spark를 사용하여 데이터 변환

데이터 *엔지니어*는 종종 Spark Notebook을 데이터를 하나의 형식이나 구조에서 다른 형식이나 구조로 변환하는 *ETL(추출, 변환, 로드)* 또는 *ELT(추출, 로드, 변환)* 작업을 수행하는 기본 도구 중 하나로 사용합니다.

이 연습에서는 Azure Synapse Analytics의 Spark Notebook을 사용하여 파일의 데이터를 변환합니다.

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

3. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp-203 -f
    git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer dp-203
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 연습의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp-203/Allfiles/labs/06
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 대해 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics 핵심 개념의 Apache Spark](https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-concepts) 문서를 검토합니다.

## Spark Notebook을 사용하여 데이터 변환

1. 배포 스크립트가 완료된 후 Azure Portal에서 만든 **dp203-*xxxxxxx*** 리소스 그룹으로 이동하여 이 리소스 그룹에 Synapse 작업 영역, 데이터 레이크에 대한 스토리지 계정, Apache Spark 풀이 포함되어 있는지 확인합니다.
2. Synapse 작업 영역을 선택하고 해당 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio를 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **관리** 페이지에서 **Apache Spark 풀** 탭을 선택하고 **spark*xxxxxxx***와 비슷한 이름의 Spark 풀이 작업 영역에 프로비저닝되었음을 확인합니다.
5. **데이터** 페이지에서 **연결된** 탭을 보고 작업 영역에 **synapse*xxxxxxx*(Primary - datalake*xxxxxxx*)**와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
6. 스토리지 계정을 확장하고 **파일(기본)** 이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
7. **파일** 컨테이너를 선택하고 **data** 및 **synapse**라는 폴더가 포함되어 있는지 확인합니다. synapse 폴더는 Azure Synapse에서 사용되며 **data** 폴더에는 쿼리하려는 데이터 파일이 포함되어 있습니다.
8. **data** 폴더를 열고 3년간의 판매 데이터에 대한 .csv 파일이 포함되어 있는지 확인합니다.
9. 파일을 마우스 오른쪽 단추로 클릭하고 **미리 보기**를 선택하여 포함된 데이터를 확인합니다. 파일에 머리글 행이 포함되어 있으므로 열 머리글을 표시하는 옵션을 선택할 수 있습니다.
10. 미리 보기를 닫습니다. 그런 다음 Allfiles/labs/06/Notebooks에서 [Spark Transform.ipynb**를 다운로드**합니다.](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer/tree/master/Allfiles/labs/06/notebooks)

    > **참고**: Ctrl+a***를 사용한 다음 ctrl+c***를 사용하여 ***이 텍스트를 복사한 다음***, 메모장과 같은 Ctrl+v***를 사용하여 ***도구에 붙여넣은 다음 파일을 사용하여 모든 파일***의 ***파일 형식으로 Spark Transform.ipynb**로 **저장하는 것이 가장 좋습니다. 파일을 클릭한 다음 줄임표(...)를 선택한 다음 다운로드하여 저장한 위치를 기억하여 파일을 다운로드할 수도 있습니다.
    ![GitHub에서 Spark Notebook 다운로드](./images/select-download-notebook.png)

11. **그런 다음 개발** 페이지에서 전자 필기장을 확장**하여** + 가져오기 옵션을 클릭합니다.

    ![Spark Notebook 가져오기](./image/../images/spark-notebook-import.png)
        
12. 방금 다운로드하여 Spark Transfrom.ipynb**으로 **저장한 파일을 선택합니다.
13. Notebook을 **spark*xxxxxxx*** Spark 풀에 연결합니다.
14. Notebook에서 메모를 검토하고 코드 셀을 실행합니다.

    > **참고**: Spark 풀을 시작해야 하므로 첫 번째 코드 셀을 실행하는 데 몇 분 정도 걸립니다. 후속 셀은 더 빠르게 실행됩니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp203-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정, 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
