---
lab:
  title: SQL용 Azure Synapse Link 사용
  ilt-use: Suggested demo
---

# SQL용 Azure Synapse Link 사용

SQL용 Azure Synapse Link를 사용하면 SQL Server 또는 Azure SQL Database의 트랜잭션 데이터베이스를 Azure Synapse Analytics의 전용 SQL 풀과 자동으로 동기화할 수 있습니다. 이 동기화를 사용하면 원본 운영 데이터베이스에서 쿼리 오버헤드를 발생시키지 않고도 Synapse Analytics에서 대기 시간이 낮은 분석 워크로드를 수행할 수 있습니다.

이 연습을 완료하는 데 약 **35**분 정도 소요됩니다.

## 시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## Azure 리소스 프로비전

이 연습에서는 Azure SQL Database 리소스의 데이터를 Azure Synapse Analytics 작업 영역으로 동기화합니다. 먼저 스크립트를 사용하여 Azure 구독에서 이러한 리소스를 프로비저닝합니다.

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
    cd dp-203/Allfiles/labs/15
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure SQL Database에 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요!

8. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 15분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다. 기다리는 동안 Azure Synapse Analytics 설명서에서 [SQL용 Azure Synapse Link란?](https://docs.microsoft.com/azure/synapse-analytics/synapse-link/sql-synapse-link-overview) 문서를 검토합니다.

## Azure SQL Database 구성

Azure SQL Database에 Azure Synapse Link를 설정하려면 먼저 필요한 구성 설정이 Azure SQL Database 서버에 적용되었는지 확인해야 합니다.

1. [Azure Portal](https://portal.azure.com)에서 설치 스크립트에서 만든 **dp203-*xxxxxxx*** 리소스 그룹을 찾아 **sqldb*xxxxxxxx*** Azure SQL 서버를 선택합니다.

    > **참고**: Azure SQL 서버 리소스 **sqldb*xxxxxxxx***) 및 Azure Synapse Analytics 전용 SQL 풀(** sql*xxxxxxxx***)을 혼합하지 않도록 주의하세요.

2. Microsoft SQL Server 리소스 페이지의 왼쪽 창에 있는 **보안** 섹션(아래쪽 근처)에서 **ID**를 선택합니다. 그런 다음, **시스템 할당 관리 ID**에서 **상태** 옵션을 **켜기**로 설정합니다. 그런 다음, **&#128427; 저장** 아이콘을 사용하여 구성 변경 내용을 저장합니다.

    ![Azure Portal의 Azure SQL 서버 ID 페이지 스크린샷](./images/sqldb-identity.png)

3. 왼쪽 창의 **보안** 섹션에서 **네트워킹**을 선택합니다. 그런 다음, **방화벽 규칙**에서 **Azure 서비스 및 리소스가 이 서버에 액세스하도록 허용**하는 예외를 선택합니다.

4. **&#65291; 방화벽 규칙 추가** 단추를 사용하여 다음 설정으로 새 방화벽 규칙을 추가합니다.

    | 규칙 이름 | 시작 IP | 종료 IP |
    | -- | -- | -- |
    | AllClients | 0.0.0.0 | 255.255.255.255 |

    > **참고**: 이 규칙은 인터넷에 연결된 컴퓨터에서 서버에 액세스할 수 있도록 허용합니다. 연습을 간소화하기 위해 이 기능을 사용할 수 있지만 프로덕션 시나리오에서는 데이터베이스를 사용해야 하는 네트워크 주소로만 액세스를 제한해야 합니다.

5. **저장** 단추를 사용하여 구성 변경 내용을 저장합니다.

    ![Azure Portal의 Azure SQL 서버 네트워킹 페이지 스크린샷](./images/sqldb-network.png)

## 트랜잭션 데이터베이스 살펴보기

Azure SQL 서버는 **AdventureWorksLT**라는 샘플 데이터베이스를 호스트합니다. 이 데이터베이스는 운영 애플리케이션 데이터에 사용되는 트랜잭션 데이터베이스를 나타냅니다.

1. Azure SQL 서버에 대한 **개요** 페이지의 아래쪽에서 **AdventureWorksLT** 데이터베이스를 선택합니다.
2. **AdventureWorksLT** 데이터베이스 페이지에서 **쿼리 편집기** 탭을 선택하고 다음 자격 증명으로 SQL 서버 인증을 사용하여 로그인합니다.
    - SQLUser **로그인**
    - **암호**: *설정 스크립트를 실행할 때 지정한 암호입니다.*
3. 쿼리 편집기가 열리면 **테이블** 노드를 확장하고 데이터베이스의 테이블 목록을 봅니다. **SalesLT** 스키마에 테이블이 포함됩니다(예: **SalesLT.Customer**).

## Azure Synapse Link 구성

이제 Synapse Analytics 작업 영역에서 SQL용 Azure Synapse Link를 구성할 준비가 되었습니다.

### 전용 SQL 풀 시작

1. Azure Portal에서 Azure SQL 데이터베이스에 대한 쿼리 편집기를 닫고(변경 내용 삭제) **dp203-*xxxxxxx*** 리소스 그룹의 페이지로 돌아갑니다.
2. **synapse*xxxxxxx*** Synapse 작업 영역을 열고 해당 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio를 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 여러 페이지가 표시됩니다.
4. **관리** 페이지의 **SQL 풀** 탭에서 **sql*xxxxxxx*** 전용 SQL 풀에 대한 행을 선택하고 **&#9655;** 아이콘을 사용하여 시작합니다. 메시지가 표시되면 다시 시작하려는지 확인합니다.
5. SQL 풀이 다시 시작될 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다. **&#8635; 새로 고침** 단추를 사용하여 상태를 주기적으로 확인할 수 있습니다. 준비되면 상태가 **온라인**으로 표시됩니다.

### 대상 스키마 만들기

1. Synapse Studio의 **데이터** 페이지에 있는 **작업 영역** 탭에서 **SQL 데이터베이스**를 확장하고 **sql*xxxxxxx*** 데이터베이스를 선택합니다.
2. **sql*xxxxxxx*** 데이터베이스에 대한 **...** 메뉴에서 **새 SQL 스크립트** > **빈 스크립트**를 선택합니다.
3. **SQL Script 1** 창에서 다음 SQL 코드를 입력하고 **&#9655; 실행** 단추를 사용하여 실행합니다.

    ```sql
    CREATE SCHEMA SalesLT;
    GO
    ```

4. 쿼리가 성공적으로 완료될 때까지 기다립니다. 이 코드는 전용 SQL 풀에 대한 데이터베이스에 **SalesLT**라는 스키마를 만들어 Azure SQL 데이터베이스에서 해당 이름의 스키마에 있는 테이블을 동기화할 수 있도록 합니다.

### 링크 연결 만들기

1. Synapse Studio의 **통합** 페이지에 있는 **&#65291;** 드롭다운 메뉴에서 **링크 연결**을 선택합니다. 그런 다음, 다음 설정을 사용하여 새 연결된 연결을 만듭니다.
    - **원본 형식**: Azure SQL 데이터베이스
    - **원본 연결된 서비스**: 다음 설정을 사용하여 새 연결된 서비스를 추가합니다(새 탭이 열림).
        - **이름**: SqlAdventureWorksLT
        - **설명**: AdventureWorksLT 데이터베이스에 연결
        - **통합 런타임을 통해 연결**: AutoResolveIntegrationRuntime
        - **연결 문자열**: 선택됨
        - **Azure 구독에서**: 선택됨
        - **Azure 구독**: *Azure 구독 선택*
        - **서버 이름**: ***sqldbxxxxxxx** Azure SQL 서버 선택*
        - **데이터베이스 이름**: AdventureWorksLT
        - **인증 유형**: SQL 인증
        - **사용자 이름**: SQLUser
        - **암호**: *설정 스크립트를 실행할 때 지정한 암호*

        *계속하기 전에 **연결 테스트** 옵션을 사용하여 연결 설정이 올바른지 확인하세요! 그런 다음, **만들기**를 클릭합니다.*

    - **원본 테이블**: 다음 테이블을 선택합니다.
        - **SalesLT.Customer**
        - **SalesLT.Product**
        - **SalesLT.SalesOrderDetail**
        - **SalesLT.SalesOrderHeader**

        *계속해서 다음 설정을 구성합니다.*

    > **참고**: 일부 대상 테이블은 사용자 지정 데이터 형식을 사용하거나 원본 테이블의 데이터가 *클러스터형 columnstore 인덱스*의 기본 구조 유형과 호환되지 않기 때문에 오류를 표시합니다.

    - **대상 풀**: ***sqlxxxxxxx** 전용 SQL 풀 선택*

        *계속해서 다음 설정을 구성합니다.*

    - **링크 연결 이름**: sql-adventureworkslt-conn
    - **코어 수**: 4개(+ 드라이버 코어 4개)

2. 생성된 **sql-adventureworkslt-conn** 페이지에서 생성된 테이블 매핑을 봅니다. **속성** 단추( **&#128463;<sub>*</sub>** 와 비슷하게 표시됨)를 사용하여 **속성** 창을 숨기고 모든 것을 더 쉽게 볼 수 있습니다. 

3. 다음과 같이 테이블 매핑에서 구조 유형을 수정합니다.

    | 원본 테이블 | 대상 테이블 | 배포 유형 | 배포 열 | 구조 유형 |
    |--|--|--|--|--|
    | SalesLT.Customer **&#8594;** | \[SalesLT] . \[Customer] | 라운드 로빈 | - | 클러스터형 columnstore 인덱스 |
    | SalesLT.Product **&#8594;** | \[SalesLT] . \[Product] | 라운드 로빈 | - | 힙 |
    | SalesLT.SalesOrderDetail **&#8594;** | \[SalesLT] . \[SalesOrderDetail] | 라운드 로빈 | - | 클러스터형 columnstore 인덱스 |
    | SalesLT.SalesOrderHeader **&#8594;** | \[SalesLT] . \[SalesOrderHeader] | 라운드 로빈 | - | 힙 |

4. 생성된 **sql-adventureworkslt-conn** 페이지의 맨 위에 있는 **&#9655; 시작** 단추를 사용하여 동기화를 시작합니다. 메시지가 표시되면 **확인**을 선택하여 링크 연결을 게시하고 시작합니다.
5. 연결을 시작한 후 **모니터링** 페이지에서 **링크 연결** 탭을 보고 **sql-adventureworkslt-conn** 연결을 선택합니다. **&#8635; 새로 고침** 단추를 사용하여 상태를 주기적으로 업데이트할 수 있습니다. 초기 스냅샷 복사 프로세스를 완료하고 복제를 시작하는 데 몇 분 정도 걸릴 수 있습니다. 그 후에는 원본 데이터베이스 테이블의 모든 변경 내용이 동기화된 테이블에서 자동으로 재생됩니다.

### 복제된 데이터 보기

1. 테이블 상태가 **실행 중**으로 변경된 후 **데이터** 페이지를 선택하고 오른쪽 위에 있는 **&#8635;** 아이콘을 사용하여 보기를 새로 고칩니다.
2. **작업 영역** 탭에서 **SQL 데이터베이스**, **sql*xxxxxxx*** 데이터베이스, 해당 **Tables** 폴더를 확장하여 복제된 테이블을 봅니다.
3. **sql*xxxxxxx*** 데이터베이스에 대한 **...** 메뉴에서 **새 SQL 스크립트** > **빈 스크립트**를 선택합니다. 그런 다음, 새 스크립트 페이지에서 다음 SQL 코드를 입력합니다.

    ```sql
    SELECT  oh.SalesOrderID, oh.OrderDate,
            p.ProductNumber, p.Color, p.Size,
            c.EmailAddress AS CustomerEmail,
            od.OrderQty, od.UnitPrice
    FROM SalesLT.SalesOrderHeader AS oh
    JOIN SalesLT.SalesOrderDetail AS od 
        ON oh.SalesOrderID = od.SalesOrderID
    JOIN  SalesLT.Product AS p 
        ON od.ProductID = p.ProductID
    JOIN SalesLT.Customer as c
        ON oh.CustomerID = c.CustomerID
    ORDER BY oh.SalesOrderID;
    ```

4. **&#9655; 실행** 단추를 사용하여 스크립트를 실행하고 결과를 봅니다. 쿼리는 원본 데이터베이스가 아닌 전용 SQL 풀의 복제된 테이블에 대해 실행되므로 비즈니스 애플리케이션에 영향을 주지 않고 분석 쿼리를 실행할 수 있습니다.
5. 완료되면 **관리** 페이지에서 **sql*xxxxxxx*** 전용 SQL 풀을 일시 중지합니다.

## Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. 이 연습의 시작 부분에서 설치 스크립트에서 만든 **dp203-*xxxxxxx*** 리소스 그룹을 선택합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp203-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분 후에 리소스 그룹 및 여기에 포함된 리소스가 삭제됩니다.
