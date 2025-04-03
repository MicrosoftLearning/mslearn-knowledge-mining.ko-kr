---
lab:
  title: 푸시 API를 사용하여 인덱스에 추가
---

# 푸시 API를 사용하여 인덱스에 추가

Azure AI 검색 인덱스를 만들고 C# 코드를 사용하여 해당 인덱스에 문서를 업로드하는 방법을 탐색하려고 합니다.

이 연습에서는 기존 C# 솔루션을 복제하고 실행하여 문서를 업로드할 최적의 일괄 처리 크기를 계산합니다. 그런 다음, 이 일괄 처리 크기를 사용하고 스레드 접근 방식을 사용하여 문서를 효과적으로 업로드합니다.

> **참고** 이 연습을 완료하려면 Microsoft Azure 구독이 필요합니다. 구독이 아직 없다면 [https://azure.com/free](https://azure.com/free?azure-portal=true)에서 평가판을 신청할 수 있습니다.

## Azure 리소스 설정

시간을 절약하려면 이 Azure Resource Manager 템플릿을 선택하여 나중에 연습에서 필요한 리소스를 만듭니다.

1. [Azure 에 리소스 배포](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json) - 이 링크를 선택하여 Azure AI 리소스를 만듭니다.
    ![Azure에 리소스를 배포할 때 표시되는 옵션의 스크린샷](../media/07-media/deploy-azure-resources.png)
1. **리소스 그룹**에서 **새로 만들기**를 선택하고, 이름을 **cog-search-language-exe**로 지정합니다.
1. **지역**에서 가까운 [지원되는 지역](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability)을 선택합니다.
1. **리소스 접두사**는 전역적으로 고유해야 하며, 임의의 숫자 및 소문자 접두사를 입력합니다(예: **acs118245**).
1. **위치**에서 위에서 선택한 동일한 지역을 선택합니다.
1. **검토 + 만들기**를 선택합니다.
1. **만들기**를 선택합니다.
1. 배포가 완료되면 **리소스 그룹으로 이동**을 선택하여 만든 모든 리소스를 확인합니다.

    ![배포된 모든 Azure 리소스를 보여 주는 스크린샷.](../media/07-media/azure-resources-created.png)

## Azure AI 검색 서비스 REST API 정보 복사

1. 리소스 목록에서 만든 검색 서비스를 선택합니다. 위의 예에서는 **acs118245-search-service**입니다.
1. 검색 서비스 이름을 텍스트 파일에 복사합니다.

    ![검색 서비스의 키 섹션 스크린샷.](../media/07-media/search-api-keys-exercise-version.png)
1. 왼쪽에서 **키**를 선택한 다음, **기본 관리자 키**를 동일한 텍스트 파일에 복사합니다.

## 리포지토리를 Cloud Shell에서 복제합니다.

Azure Portal에서 Cloud Shell을 사용하여 코드를 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: 최근에 **mslearn-knowledge-mining** 리포지토리를 이미 복제했다면 이 작업을 건너뛸 수 있습니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Azure Portal에서 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 버튼으로 Azure Portal에서 새 Cloud Shell을 생성하고 ***PowerShell*** 환경을 선택합니다. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    > **팁**: CloudShell에 명령을 붙여넣을 때, 출력이 화면 버퍼의 많은 부분을 차지할 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. PowerShell 창에서 다음 명령을 입력하여 이 연습이 포함된 GitHub 리포지토리를 복제합니다.

    ```
    rm -r mslearn-knowledge-mining -f
    git clone https://github.com/microsoftlearning/mslearn-knowledge-mining mslearn-knowledge-mining
    ```

1. 리포지토리가 복제된 후 애플리케이션 코드 파일이 포함된 폴더로 이동합니다.  

    ```
   cd mslearn-knowledge-mining/Labfiles/07-exercise-add-to-index-use-push-api/OptimizeDataIndexing
    ```

## 애플리케이션 설정

1. `ls` 명령을 사용하면 **OptimizeDataIndexing** 폴더의 내용을 볼 수 있습니다. 구성 설정을 위한 `appsettings.json` 파일이 포함되어 있습니다.

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code appsettings.json
    ```

    코드 편집기에서 파일이 열립니다.

    ![appsettings.json 파일의 내용을 보여 주는 스크린샷.](../media/07-media/update-app-settings.png)

1. 검색 서비스 이름 및 기본 관리 키를 붙여넣습니다.

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    설정 파일은 위와 유사해야 합니다.
   
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.
1. 터미널에서 `dotnet run`을(를) 입력하고 **Enter** 키를 누릅니다.

    ![예외가 있는 VS Code에서 실행되는 앱을 보여 주는 스크린샷.](../media/07-media/debug-application.png)

    출력에 따르면 이 경우 가장 성능이 좋은 배치 크기는 900개 문서이며 전송 속도(MB/초)가 가장 높습니다.
   
    >**참고**: 전송 속도 값은 스크린샷에 표시된 값과 다를 수 있습니다. 그러나 성능이 가장 뛰어난 배치 크기는 여전히 동일해야 합니다. 

## 코드를 편집하여 스레딩, 백오프, 재시도 전략 구현

스레드를 사용하여 문서를 검색 인덱스에 업로드하도록 앱을 변경할 준비가 된 주석 처리된 코드가 있습니다.

1. 다음 명령을 입력하여 클라이언트 응용 프로그램의 코드 파일을 엽니다:

    ```
   code Program.cs
    ```

1. 38행과 39행을 다음과 같이 주석으로 처리합니다.

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. 41~49줄의 주석 처리를 제거합니다.

    ```csharp
    long numDocuments = 100000;
    DataGenerator dg = new DataGenerator();
    List<Hotel> hotels = dg.GetHotels(numDocuments, "large");

    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    일괄 처리 크기와 스레드 수를 제어하는 ​​코드는 `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`입니다. 일괄 처리 크기는 1,000이고 스레드는 8개입니다.

    ![편집된 모든 코드를 보여 주는 스크린샷.](../media/07-media/thread-code-ready.png)

    코드는 위와 같아야 합니다.

1. 변경 내용을 저장합니다.
1. 터미널을 선택한 다음, 아직 종료하지 않았다면 아무 키나 눌러 실행 중인 프로세스를 종료합니다.
1. 터미널에서 `dotnet run`을 실행합니다.

    앱은 8개의 스레드를 시작한 다음, 각 스레드가 콘솔에 새 메시지 쓰기를 마치면 다음을 수행합니다.

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    100,000개의 문서가 업로드되면 앱에서 요약이 작성됩니다(완료하는 데 시간이 걸릴 수 있음).

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

`TestBatchSizesAsync` 프로시저의 코드를 탐색하여 코드가 일괄 처리 크기 성능을 테스트하는 방법을 확인합니다.

`IndexDataAsync` 프로시저의 코드를 탐색하여 코드가 스레딩을 관리하는 방법을 확인합니다.

`ExponentialBackoffAsync`의 코드를 탐색하여 코드가 지수 백오프 재시도 전략을 구현하는 방법을 확인합니다.

문서가 Azure Portal의 인덱스로 추가되었는지 검색하고 확인할 수 있습니다.

![100000개의 문서가 포함된 검색 인덱스를 보여 주는 스크린샷.](../media/07-media/check-search-service-index.png)

## 정리

연습을 완료했으므로 더 이상 필요하지 않은 모든 리소스를 삭제합니다. 머신에 복제된 코드로 시작합니다. 그런 다음, Azure 리소스를 삭제합니다.

1. **Azure Portal**에서 리소스 그룹을 선택합니다.
1. 이 연습을 위해 만든 리소스 그룹을 선택합니다.
1. **리소스 그룹 삭제**를 선택합니다. 
1. 삭제를 확인한 다음 **삭제**를 선택합니다.
1. 필요하지 않은 리소스를 선택한 다음, **삭제**를 선택합니다.
