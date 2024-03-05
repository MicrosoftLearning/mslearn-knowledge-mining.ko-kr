---
lab:
  title: Azure AI 검색을 위한 사용자 지정 기술 만들기
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Azure AI 검색을 위한 사용자 지정 기술 만들기

Azure AI 검색은 AI 기술의 보강 파이프라인을 사용하여 문서에서 AI 생성 필드를 추출하고 검색 인덱스에 포함합니다. 포괄적인 기본 제공 기술 세트를 사용할 수 있습니다. 하지만 기본 제공 기술로는 특정 요구 사항을 충족할 수 없는 경우에는 사용자 지정 기술을 만들 수 있습니다.

이 연습에서는 사용자 지정 기술을 만듭니다. 이 기술은 문서의 개별 단어 빈도를 표로 만들어 가장 많이 사용되는 상위 5개 단어 목록을 생성한 다음 가상의 여행사인 Margie's Travel용 검색 솔루션에 추가합니다.

## Visual Studio Code에서 앱 개발 준비

Visual Studio Code를 사용하여 검색 앱을 개발합니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되었습니다.

> **팁**: **mslearn-knowledge-mining** 리포지토리를 이미 복제한 경우 Visual Studio Code에서 엽니다. 그렇지 않은 경우에는 다음 단계에 따라 개발 환경에 복제합니다.

1. Visual Studio Code 시작
1. 팔레트를 열고(Shift+Ctrl+P) **Git: Clone** 명령을 실행하여 `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` 리포지토리를 로컬 폴더(아무 폴더나 관계없음)에 복제합니다.
1. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.
1. 리포지토리의 C# 코드 프로젝트를 지원하는 추가 파일이 설치되는 동안 기다립니다.

    > **참고**: 빌드 및 디버깅에 필요한 자산을 추가하라는 메시지가 표시되면 **나중에**를 선택합니다.

## Azure 리소스 만들기

> **참고**: 이전에 **[Azure AI 검색 솔루션 만들기](01-azure-search.md)** 연습을 완료했고 구독에 이러한 Azure 리소스가 아직 남아 있는 경우 이 섹션을 건너뛰고 **검색 솔루션 만들기** 섹션부터 시작하면 됩니다. 그렇지 않은 경우에는 아래 단계에 따라 필요한 Azure 리소스를 프로비전합니다.

1. 웹 브라우저에서 `https://portal.azure.com`의 Azure Portal을 열고 Azure 구독과 연결된 Microsoft 계정을 사용하여 로그인합니다.
2. 상단 검색 창에서 *Azure AI 서비스*를 검색하고 **Azure AI 서비스**를 선택한 후 다음 설정을 사용하여 Azure AI 서비스 다중 서비스 계정 리소스를 만듭니다.
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: 리소스 그룹 선택 또는 만들기(제한된 구독을 사용 중이라면 새 리소스 그룹을 만들 권한이 없을 수도 있으므로 제공된 리소스 그룹 사용)**
    - **지역**: *지리적으로 가까운 지역 중에서 선택*
    - **이름**: *고유 이름 입력*
    - **가격 책정 계층**: 표준 S0
1. 배포되면 리소스로 이동하여 **개요** 페이지에서 **구독 ID**와 **위치**를 확인합니다. 후속 단계에서 이러한 값과 리소스 그룹의 이름이 필요합니다. 
1. Visual Studio Code에서 **Labfiles/02-search-skill** 폴더를 확장하고 **setup.cmd**를 선택합니다. 이 배치 스크립트를 사용하여 필요한 Azure 리소스를 만드는 데 필요한 Azure CLI(명령줄 인터페이스)를 실행합니다.
1. **02-search-skill** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.
1. 터미널 창에서 다음 명령을 입력하여 Azure 구독에 대해 인증된 연결을 설정합니다.

    ```powershell
    az login --output none
    ```

8. 메시지가 표시되면 Azure 구독을 선택하거나 로그인합니다. 그런 다음 Visual Studio Code로 돌아가서 로그인 프로세스가 완료될 때까지 기다립니다.
9. 다음 명령을 실행하여 Azure 위치를 나열합니다.

    ```powershell
    az account list-locations -o table
    ```

10. 출력에서 리소스 그룹 위치에 해당하는 **Name** 값을 찾습니다(예를 들어 미국 동부에 해당하는 이름은 *eastus*임).**
11. **setup.cmd** 스크립트에서 구독 ID, 리소스 그룹 이름 및 위치 이름에 대한 적절한 값을 사용하여 **subscription_id**, **resource_group** 및 **location** 변수 선언을 수정합니다. 그런 다음, 변경 사항을 저장합니다.
12. **02-search-skill** 폴더의 터미널에서 다음 명령을 입력하여 스크립트를 실행합니다.

    ```powershell
    ./setup
    ```

    > **참고**: 스크립트 실행이 실패하면 올바른 변수 이름을 적용하여 스크립트를 저장했는지 확인하고 다시 시도합니다.

13. 스크립트가 완료되면 표시되는 출력을 검토하고 Azure 리소스에 대한 다음 정보를 확인합니다(나중에 해당 값이 필요함).
    - 스토리지 계정 이름
    - 스토리지 연결 문자열
    - 검색 서비스 엔드포인트
    - 검색 서비스 관리자 키
    - 검색 서비스 쿼리 키

14. Azure Portal에서 리소스 그룹을 새로 고치고 Azure Storage 계정, Azure AI 서비스 리소스 및 Azure AI 검색 리소스가 포함되어 있는지 확인합니다.

## 검색 솔루션 만들기

이제 필요한 Azure 리소스가 준비되었으므로 다음 구성 요소로 이루어진 검색 솔루션을 만들 수 있습니다.

- Azure Storage 컨테이너의 문서를 참조하는 **데이터 원본**.
- 문서에서 AI 생성 필드를 추출하기 위한 기술 보강 파이프라인을 정의하는 **기술 세트**.
- 검색 가능한 문서 레코드 집합을 정의하는 **인덱스**.
- 데이터 원본에서 문서를 추출하고, 기술을 적용하고, 인덱스를 채우는 **인덱서**.

이 연습에서는 Azure AI 검색 REST 인터페이스를 사용하여 JSON 요청을 제출하여 이러한 구성 요소를 만듭니다.

1. Visual Studio Code의 **02-search-skill** 폴더에서 **create-search** 폴더를 확장하고 **data_source.json**을 선택합니다. 이 파일에는 **margies-custom-data**라는 데이터 원본에 대한 JSON 정의가 포함되어 있습니다.
2. **YOUR_CONNECTION_STRING** 자리 표시자를 다음과 유사한 Azure Storage 계정의 연결 문자열로 바꿉니다.

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    Azure Portal의 스토리지 계정에 대한 **액세스 키** 페이지에서 연결 문자열을 확인할 수 있습니다.**

3. 업데이트된 JSON 파일을 저장하고 닫습니다.
4. **create-search** 폴더에서 **skillset.json**을 엽니다. 이 파일에는 **margies-custom-skillset**이라는 기술 세트에 대한 JSON 정의가 포함되어 있습니다.
5. 기술 세트 정의 상단의 **cognitiveServices** 요소에서 **YOUR_AI_SERVICES_KEY** 자리 표시자를 Azure AI 서비스 리소스의 키 중 하나로 바꿉니다.

    *Azure Portal의 Azure AI 서비스 리소스에 대한 **키 및 엔드포인트** 페이지에서 키를 찾을 수 있습니다.*

6. 업데이트된 JSON 파일을 저장하고 닫습니다.
7. **create-search** 폴더에서 **index.json**을 엽니다. 이 파일에는 **margies-custom-index**라는 인덱스에 대한 JSON 정의가 포함되어 있습니다.
8. 인덱스의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
9. **create-search** 폴더에서 **indexer.json**을 엽니다. 이 파일에는 **margies-custom-indexer**라는 인덱서에 대한 JSON 정의가 포함되어 있습니다.
10. 인덱서의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
11. **create-search** 폴더에서 **create-search.cmd**를 엽니다. 이 배치 스크립트는 cURL 유틸리티를 사용하여 Azure AI 검색 리소스에 대한 REST 인터페이스에 JSON 정의를 제출합니다.
12. **YOUR_SEARCH_URL** 및 **YOUR_ADMIN_KEY** 변수 자리 표시자를 Azure AI 검색 리소스에 대한 **Url** 및 **관리자 키** 중 하나로 바꿉니다.

    *Azure Portal의 Azure AI 검색 리소스에 대한 **개요** 및 **키** 페이지에서 이러한 값을 찾을 수 있습니다.*

13. 업데이트된 배치 파일을 저장합니다.
14. **create-search** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.
15. **create-search** 폴더의 터미널 창에 다음 명령을 입력하여 배치 스크립트를 실행합니다.

    ```powershell
    ./create-search
    ```

16. 스크립트가 완료되면 Azure Portal의 Azure AI 검색 리소스 페이지에서 **인덱서** 페이지를 선택하고 인덱싱 프로세스가 완료될 때까지 기다립니다.

    ***새로 고침**을 선택하여 인덱싱 작업 진행률을 추적할 수 있습니다. 인덱싱이 완료되려면 1분 정도 걸릴 수 있습니다.*

## 인덱스 검색

이제 인덱스가 생성되었으므로 해당 인덱스를 검색할 수 있습니다.

1. Azure AI 검색 리소스 블레이드 상단에서 **검색 탐색기**를 선택합니다.
2. 쿼리 탐색기의 **쿼리 문자열** 상자에 다음 쿼리 문자열을 입력한 후 **검색**을 선택합니다.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    이 쿼리는 *London*이 언급되어 있고, 작성자가 *Reviewer*이며, 긍정적인 **sentiment** 레이블이 있는 모든 문서(즉, 런던을 언급한 긍정적인 내용의 리뷰)에 대한 **url**, **sentiment** 및 **keyphrases**를 검색합니다.

## 사용자 지정 기술용 Azure 함수 만들기

검색 솔루션에는 이전 작업에서 본 감정 점수 및 핵심 구 목록과 같은 문서의 정보로 인덱스를 보강하는 다양한 기본 제공 AI 기술이 포함되어 있습니다.

사용자 지정 기술을 만들면 인덱스를 추가로 보강할 수 있습니다. 예를 들어 각 문서에서 사용 빈도가 가장 높은 단어를 식별하려고 하는데 이 기능을 제공하는 기본 제공 기술은 없는 경우 사용자 지정 기술을 만들면 유용할 수 있습니다.

여기서는 단어 개수 기능을 사용자 지정 기술로 구현하기 위해 원하는 언어로 Azure 함수를 만듭니다.

> **참고**: 이 연습에서는 Azure Portal에서 코드 편집 기능을 사용하여 간단한 Node.JS 함수를 만듭니다. 프로덕션 솔루션에서는 일반적으로 Visual Studio Code 등의 개발 환경을 사용하여 원하는 언어(예: C#, Python, Node.JS 또는 Java)로 함수 앱을 만들고 DevOps 프로세스의 일부로 Azure에 게시합니다.

1. Azure Portal의 **홈** 페이지에서 다음 설정을 사용하여 새 **함수 앱** 리소스를 만듭니다.
    - **구독**: *사용자의 구독*
    - **리소스 그룹**: *Azure AI 검색 리소스와 동일한 리소스 그룹*
    - **함수 앱 이름**: 고유한 이름**
    - **게시**: 코드
    - **런타임 스택**: Node.js
    - **버전**: 18 LTS
    - **지역**: *Azure AI 검색 리소스와 동일한 지역*

2. 배포가 완료될 때까지 기다렸다가 배포된 함수 앱 리소스로 이동합니다.
3. **개요** 페이지에서 **Azure Portal에서 만들기** 옵션을 선택하여 다음 설정으로 새 함수를 만듭니다.
    - **개발 환경 설정**”
        - **개발 환경**: 포털에서 개발
    - **템플릿 선택**”
        - **템플릿**: HTTP 트리거
    - **템플릿 세부 정보**:
        - **새 함수**: wordcount
        - **권한 부여 수준**: 함수
4. *wordcount* 함수가 만들어질 때까지 기다립니다. 그런 다음, 해당 페이지에서 **코드 + 테스트** 탭을 선택합니다.
5. 기본 함수 코드를 다음 코드로 바꿉니다.

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. 함수를 저장한 다음, **테스트/실행** 창을 엽니다.
7. **테스트/실행** 창에서 기존 **본문**을 다음 JSON으로 바꿉니다. 이는 하나 이상의 문서에 대한 데이터가 포함된 레코드가 처리를 위해 제출되는 Azure AI 검색 기술에서 예상하는 스키마를 반영합니다.

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. **실행**을 클릭하고 함수에서 반환하는 HTTP 응답 콘텐츠를 확인합니다. 이는 각 문서에 대한 응답이 반환되는 기술을 사용할 때 Azure AI 검색에서 예상하는 스키마를 반영합니다. 이 경우 문서마다 최대 10개의 용어가 표시되는 빈도에 따라 내림차순으로 응답을 구성합니다.

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. **테스트/실행** 창을 닫고 **wordcount** 함수 블레이드에서 **함수 URL 가져오기**를 클릭합니다. 그런 다음 기본 키의 URL을 클립보드에 복사합니다. 다음 절차에서 이 URL이 필요합니다.

## 검색 솔루션에 사용자 지정 기술 추가

이제검색 솔루션 기술 세트에 함수를 사용자 지정 기술로 포함하고 이 함수가 생성하는 결과를 인덱스의 필드에 매핑해야 합니다. 

1. Visual Studio Code의 **02-search-skill/update-search** 폴더에서 **update-skillset.json** 파일을 엽니다. 여기에는 기술 세트의 JSON 정의가 포함됩니다.
2. 기술 세트 정의를 검토합니다. 여기에는 **get-top-words**라는 이름의 **WebApiSkill** 기술뿐 아니라 이전과 동일한 기술도 포함됩니다.
3. **get-top-words** 기술 정의를 편집하여 **uri** 값을 Azure 함수의 URL(이전 절차에서 클립보드에 복사한 값)로 설정하고 **YOUR-FUNCTION-APP-URL**을 바꿉니다.
4. 기술 세트 정의 상단의 **cognitiveServices** 요소에서 **YOUR_AI_SERVICES_KEY** 자리 표시자를 Azure AI 서비스 리소스의 키 중 하나로 바꿉니다.

    *Azure Portal의 Azure AI 서비스 리소스에 대한 **키 및 엔드포인트** 페이지에서 키를 찾을 수 있습니다.*

5. 업데이트된 JSON 파일을 저장하고 닫습니다.
6. **update-search** 폴더에서 **update-index.json**을 엽니다. 이 파일에는 **margies-custom-index** 인덱스에 대한 JSON 정의가 포함되어 있으며 인덱스 정의 하단에 **top_words**라는 추가 필드가 있습니다.
7. 인덱스의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
8. **update-search** 폴더에서 **update-indexer.json**을 엽니다. 이 파일에는 **top_words** 필드에 대한 추가 매핑과 함께 **margies-custom-indexer**에 대한 JSON 정의가 포함되어 있습니다.
9. 인덱서의 JSON을 검토한 후 내용을 변경하지 않고 파일을 닫습니다.
10. **update-search** 폴더에서 **update-search.cmd**를 엽니다. 이 배치 스크립트는 cURL 유틸리티를 사용하여 업데이트된 JSON 정의를 Azure AI 검색 리소스에 대한 REST 인터페이스에 제출합니다.
11. **YOUR_SEARCH_URL** 및 **YOUR_ADMIN_KEY** 변수 자리 표시자를 Azure AI 검색 리소스에 대한 **Url** 및 **관리자 키** 중 하나로 바꿉니다.

    *Azure Portal의 Azure AI 검색 리소스에 대한 **개요** 및 **키** 페이지에서 이러한 값을 찾을 수 있습니다.*

12. 업데이트된 배치 파일을 저장합니다.
13. **update-search** 폴더를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기**를 선택합니다.
14. **update-search** 폴더의 터미널 창에 다음 명령을 입력하여 배치 스크립트를 실행합니다.

    ```powershell
    ./update-search
    ```

15. 스크립트가 완료되면 Azure Portal의 Azure AI 검색 리소스 페이지에서 **인덱서** 페이지를 선택하고 인덱싱 프로세스가 완료될 때까지 기다립니다.

    ***새로 고침**을 선택하여 인덱싱 작업 진행률을 추적할 수 있습니다. 인덱싱이 완료되려면 1분 정도 걸릴 수 있습니다.*

## 인덱스 검색

이제 인덱스가 생성되었으므로 해당 인덱스를 검색할 수 있습니다.

1. Azure AI 검색 리소스 블레이드 상단에서 **검색 탐색기**를 선택합니다.
2. 검색 탐색기에서 보기를 **JSON 보기**로 변경한 후 다음 검색 쿼리를 제출합니다.

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    이 쿼리는 *Las Vegas*를 언급하는 모든 문서에 대해 **url** 및 **top_words** 필드를 검색합니다.

## 자세한 정보

Azure AI 검색을 위한 사용자 지정 기술 만들기에 대해 자세히 알아보려면 [Azure AI 검색 설명서](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)를 참조하세요.
