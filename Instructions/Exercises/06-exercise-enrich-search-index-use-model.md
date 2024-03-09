---
lab:
  title: Azure Machine Learning 모델을 사용하여 검색 인덱스 보강
---

# Azure Machine Learning 모델을 사용하여 검색 인덱스 보강

기계 학습의 기능을 사용하여 검색 인덱스를 보강할 수 있습니다. 이렇게 하려면 Azure AI Machine Learning 스튜디오에서 학습된 모델을 사용하여 기계 학습 사용자 지정 기술 세트에서 호출합니다.

이 연습에서는 Azure AI Machine Learning 스튜디오 모델을 만든 다음, 해당 모델을 사용하여 엔드포인트를 학습, 배포, 테스트합니다. 그런 다음, Azure Cognitive Search 서비스를 만들고, 샘플 데이터를 만들고, Azure AI Machine Learning 스튜디오 엔드포인트를 사용하여 인덱스를 보강합니다.

> **참고** 이 연습을 완료하려면 Microsoft Azure 구독이 필요합니다. 구독이 아직 없다면 [https://azure.com/free](https://azure.com/free?azure-portal=true)에서 평가판을 신청할 수 있습니다.
>

## Azure Machine Learning 작업 영역 만들기

검색 인덱스를 보강하기 전에 Azure Machine Learning 작업 영역을 만듭니다. 작업 영역에서는 AI 모델을 빌드하고 사용을 위해 배포하는 데 사용할 수 있는 그래픽 도구인 Azure AI Machine Learning 스튜디오에 액세스할 수 있습니다.

1. [Azure Portal](https://portal.azure.com)에 로그인합니다.
1. **+ 리소스 만들기**를 선택합니다.
1. 기계 학습을 검색한 다음, **Azure Machine Learning**을 선택합니다.
1. **만들기**를 실행합니다.
1. **리소스 그룹**에서 **새로 만들기**를 선택하고 이름을 **aml-for-acs-enrichment**로 지정합니다.
1. 작업 영역 세부 정보 섹션에서 **이름**에 **aml-for-acs-workspace**를 입력합니다.
1. 가까운 지원되는 **지역**을 선택합니다.
1. **스토리지 계정**, **키 자격 증명 모음**, **Application Insight** 및 **컨테이너 레지스트리**의 기본값을 사용합니다.
1. **검토 + 만들기**를 선택합니다.
1. **만들기**를 선택합니다.
1. Azure Machine Learning 작업 영역이 배포될 때까지 기다린 다음, **리소스로 이동**을 선택합니다.
1. 개요 창에서 **스튜디오 시작**을 선택합니다.

## 회귀 학습 파이프라인 만들기

이제 회귀 모델을 만들고 Azure AI Machine Learning 스튜디오 파이프라인을 사용하여 학습합니다. 자동차 가격 데이터에 대해 모델을 학습시킵니다. 한번 학습된 모델은 특성에 따라 자동차 가격을 예측합니다.

1. 홈페이지에서 **디자이너**를 선택합니다.

1. 미리 빌드된 구성 요소 목록에서 **회귀 - 자동차 가격 예측(기본)** 을 선택합니다.

    ![미리 빌드된 회귀 모델 선택을 보여 주는 스크린샷](../media/06-media/select-pre-built-components-new.png)

1. **유효성 확인**을 선택합니다.

1. **그래프 유효성 검사** 창에서 **제출 마법사에서 컴퓨팅 대상 선택** 오류를 선택합니다.

    ![모델을 학습하는 컴퓨팅 인스턴스를 만드는 방법을 보여 주는 스크린샷](../media/06-media/create-compute-instance-new.png)
1. **컴퓨팅 형식 선택** 드롭다운에서 **컴퓨팅 인스턴스**를 선택합니다. 그런 다음 아래에서 **Azure ML 컴퓨팅 인스턴스 만들기**를 선택합니다.
1. **컴퓨팅 이름** 필드에 고유한 이름(예: **학습용 컴퓨팅**)을 입력합니다.
1. **검토 + 생성**를 선택한 다음, **생성**를 선택합니다.

1. **Azure ML 컴퓨팅 인스턴스 선택** 필드의 드롭다운에서 인스턴스를 선택합니다. 프로비전이 완료될 때까지 기다려야 할 수도 있습니다.

1. 다시 **유효성 검사**을 선택하면 파이프라인이 정상적으로 보입니다.

    ![파이프라인이 좋아 보이고 제출 단추가 강조 표시된 스크린샷](../media/06-media/submit-pipeline.png)
1. **파이프라인 작업 설정** 창에서 **기본 사항**을 선택합니다.
1. 실험 이름 아래에서 **새로 만들기**를 선택합니다.
1. **새 실험 이름**에 **linear-regression-training**을 입력합니다.
1. **검토 + 제출**을 선택한 다음 **제출**을 선택합니다.

### 엔드포인트에 대한 유추 클러스터 만들기

파이프라인이 선형 회귀 모델을 학습하는 동안 엔드포인트에 필요한 리소스를 만들 수 있습니다. 이 엔드포인트는 모델에 대한 웹 요청을 처리하기 위해 Kubernetes 클러스터가 필요합니다.

1. 왼쪽에서 **Compute**를 선택합니다.

    ![새 유추 클러스터를 만드는 방법을 보여 주는 스크린샷](../media/06-media/create-inference-cluster-new.png)
1. **Kubernetes 클러스터**를 선택한 다음 **+ 새로 만들기**를 선택합니다.
1. 드롭다운에서 **AksCompute**를 선택합니다.
1. **AksCompute 만들기** 창에서 **새로 만들기**를 선택합니다.
1. **위치**에서 다른 리소스를 만드는 데 사용한 것과 동일한 지역을 선택합니다.
1. VM 크기 목록에서 **Standard_A2_v2**를 선택합니다.
1. **다음**을 선택합니다.
1. **컴퓨팅 이름**에 **aml-acs-endpoint**를 입력합니다.
1. **SSL 구성 사용**을 선택합니다.
1. **리프 도메인**에 **aml-for-acs**를 입력합니다.
1. **만들기**를 실행합니다.

### 학습된 모델 등록

파이프라인 작업이 완료되었어야 합니다. `score.py` 및 `conda_env.yaml` 파일을 다운로드합니다. 그런 다음, 학습된 모델을 등록합니다.

1. 왼쪽에서 **작업**을 선택합니다.

    ![완료된 파이프라인 작업을 보여 주는 스크린샷](../media/06-media/completed-pipeline-new.png)
1. 실험을 선택한 다음 표에서 완료된 작업을 선택합니다(예: **회귀 - 자동차 가격 예측(기본)**). 변경 내용을 저장하라는 메시지가 표시되면 변경 내용에 대해 **삭제**를 선택합니다.
1. 디자이너에서 오른쪽 상단의 **작업 개요**를 선택한 다음 **모델 학습** 노드를 선택합니다.

    ![를 다운로드하는 방법을 보여 주는 스크린샷](../media/06-media/download-score-conda.png)
1. **출력 + 로그** 탭에서 **trained_model_outputs** 폴더를 확장합니다.
1. `score.py` 옆에 있는 더 많은 메뉴(**...**)를 선택한 다음, **다운로드**를 선택합니다.
1. `conda_env.yaml` 옆에 있는 더 많은 메뉴(**...**)를 선택한 다음, **다운로드**를 선택합니다.
1. 탭 상단에서 **+ 모델 등록**을 선택합니다.
1. **작업 출력** 필드에서 **trained_model_outputs** 폴더를 선택합니다. 그런 다음 창 하단에서 **다음**을 선택합니다.
1. 모델 **이름**에 **carevalmodel**을 입력합니다.
1. **설명**에 **자동차 가격 예측을 위한 선형 회귀 모델**을 입력합니다.
1. **다음**을 선택합니다.
1. **등록**을 선택합니다.

### 채점 스크립트를 편집하여 Azure AI 검색에 올바르게 응답

Azure Machine Learning 스튜디오 웹 브라우저의 기본 다운로드 위치에 두 개의 파일을 다운로드했습니다. JSON 요청 및 응답 처리 방법을 변경하려면 score.py 파일을 편집해야 합니다. 텍스트 편집기나 Visual Studio Code와 같은 코드 편집기를 사용할 수 있습니다.

1. 편집기에서 Score.py 파일을 엽니다.
1. 실행 함수의 모든 내용을 바꿉니다.

    ```python
    def run(data):
    data = json.loads(data)
    input_entry = defaultdict(list)
    for row in data:
        for key, val in row.items():
            input_entry[key].append(decode_nan(val))

    data_frame_directory = create_dfd_from_dict(input_entry, schema_data)
    score_module = ScoreModelModule()
    result, = score_module.run(
        learner=model,
        test_data=DataTable.from_dfd(data_frame_directory),
        append_or_result_only=True)
    return json.dumps({"result": result.data_frame.values.tolist()})
    ```

    이 Python 코드를 사용하여 다음을 수행합니다.

    ```python
    def run(data):
        data = json.loads(data)
        input_entry = defaultdict(list)
        
        for key, val in data.items():
                input_entry[key].append(decode_nan(val))
    
        data_frame_directory = create_dfd_from_dict(input_entry, schema_data)
        score_module = ScoreModelModule()
        result, = score_module.run(
            learner=model,
            test_data=DataTable.from_dfd(data_frame_directory),
            append_or_result_only=True)
        output = result.data_frame.values.tolist()
        
        return {
                "predicted_price": output[0][-1]
        }    
    ```

    위의 변경 내용을 통해 모드는 자동차 배열 대신 자동차 특성이 있는 단일 JSON 개체를 받을 수 있습니다.

    다른 변화는 전체 JSON 응답 대신 자동차의 예측 가격만 반환하는 것입니다.
1. 텍스트 편집기에서 변경 내용을 저장합니다.

## 사용자 지정 환경 만들기

다음으로 실시간 엔드포인트에 배포할 수 있도록 사용자 지정 환경을 만듭니다.

1. 탐색 창에서 **환경**을 선택합니다.
1. **사용자 지정 환경** 탭을 선택합니다.
1. **+ 만들기**를 선택합니다.
1. **이름**에 **my-custom-environment**를 입력합니다.
1. **환경 형식 선택** 아래의 *큐레이팅된 환경* 목록에서 최신 **automl-gpu** 버전을 선택합니다.
1. **다음**을 선택합니다.
1. 로컬 컴퓨터에서 이전에 다운로드한 `conda_env.yaml` 파일을 열고 해당 콘텐츠를 복사합니다.
1. 브라우저로 돌아가 사용자 지정 창에서 **conda_dependent.yaml**을 선택합니다.
1. 오른쪽 창에서 해당 콘텐츠를 이전에 복사한 코드로 바꿉니다.
1. **다음**을 선택한 다음, **다음**을 다시 선택합니다.
1. 사용자 지정 환경을 만들려면 **만들기**를 선택합니다.

## 업데이트된 채점 코드를 사용하여 모델 배포 <!--Option for web service deployment is greyed out. Can't go further after trying several different things.-->

이제 유추 클러스터를 사용할 준비가 되었습니다. 또한 Azure Cognitive Search 사용자 지정 기술 세트의 요청을 처리하도록 채점 코드를 편집했습니다. 모델에 대한 엔드포인트를 만들고 테스트해 보겠습니다.

1. 왼쪽에서 **모델**을 선택합니다.
1. 등록한 모델인 **carevalmodel**을 선택합니다.

1. **배포**를 선택한 다음 **실시간 엔드포인트**를 선택합니다.

    ![엔드포인트 선택 창의 스크린샷.](../media/06-media/04-select-endpoint.png)
1. **이름**에 고유한 이름을 입력합니다(예: **car-evaluation-endpoint-1440637584**).
1. **컴퓨팅 형식**에서 **관리**를 선택합니다.
1. **인증 유형**에서 **키 기반 인증**을 선택합니다.
1. **다음**을 선택한 후 **다음**을 선택합니다.
1. **다음**을 다시 선택합니다.
1. **유추를 위한 채점 스크립트 선택** 필드에서 업데이트된 `score.py` 파일을 찾아 선택합니다.
1. **환경 형식 선택** 드롭다운에서 **사용자 지정 환경**을 선택합니다.
1. 목록에서 사용자 지정 환경의 확인란을 선택합니다.
1. **다음**을 선택합니다.
1. 가상 머신에서 **Standard_D2as_v4**를 선택합니다.
1. **인스턴스 수**를 **1**로 설정합니다.
1. **다음**을 선택한 다음, **다음**을 다시 선택합니다.
1. **만들기**를 실행합니다.

모델이 배포될 때까지 기다립니다. 최대 10분이 소요될 수 있습니다. **알림** 또는 Azure Machine Learning 스튜디오 엔드포인트 섹션에서 상태를 확인할 수 있습니다.

### 학습된 모델의 엔드포인트 테스트

1. 왼쪽에서 **엔드포인트**를 선택합니다.
1. **car-evaluation-endpoint**를 선택합니다.
1. **테스트**를 선택하고 **테스트 엔드포인트에 데이터 입력**에 이 JSON 예를 붙여넣습니다.

    ```json
    {
        "symboling": 2,
        "make": "mitsubishi",
        "fuel-type": "gas",
        "aspiration": "std",
        "num-of-doors": "two",
        "body-style": "hatchback",
        "drive-wheels": "fwd",
        "engine-location": "front",
        "wheel-base": 93.7,
        "length": 157.3,
        "width": 64.4,
        "height": 50.8,
        "curb-weight": 1944,
        "engine-type": "ohc",
        "num-of-cylinders": "four",
        "engine-size": 92,
        "fuel-system": "2bbl",
        "bore": 2.97,
        "stroke": 3.23,
        "compression-ratio": 9.4,
        "horsepower": 68.0,
        "peak-rpm": 5500.0,
        "city-mpg": 31,
        "highway-mpg": 38,
        "price": 0.0
    }
    ```

1. **테스트**를 선택하면 다음과 같은 응답이 표시됩니다.

    ```json
    {
        "predicted_price": 5852.823214312815
    }
    ```

1. **사용**을 선택합니다.

    ![REST 엔드포인트 및 기본 키를 복사하는 방법을 보여 주는 스크린샷](../media/06-media/copy-rest-endpoint.png)
1. **REST 엔드포인트**를 복사합니다.
1. **기본 키**

### Azure Machine Learning 모델을 Azure AI 검색과 통합

다음으로, 새 Cognitive Search 서비스를 만들고 사용자 지정 기술 세트를 사용하여 인덱스를 보강합니다.

### 테스트 파일 만들기

1. [Azure Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true)에서 리소스 그룹을 선택합니다.
1. **aml-for-acs-enrichment**를 선택합니다.

    ![Azure Portal에서 스토리지 계정 선택을 보여 주는 스크린샷](../media/06-media/navigate-storage-account.png)
1. 스토리지 계정(예: **amlforacsworks1440637584**)을 선택합니다.
1. **설정**에서 **구성**을 선택합니다. 그런 다음 **Blob 익명 액세스 허용**을 **사용**으로 설정합니다.
1. **저장**을 선택합니다.
1. **데이터 스토리지**에서 **컨테이너**를 선택합니다. 
1. 인덱스 데이터를 저장할 새 컨테이너를 만들고 **+ 컨테이너**를 선택합니다.
1. **새 컨테이너** 창의 **이름**에 **docs-to-search**를 입력합니다.
1. **익명 액세스 수준**에서 **컨테이너(컨테이너 및 Blob에 대한 익명 읽기 권한)** 를 선택합니다.
1. **만들기**를 실행합니다.
1. 만든 **docs-to-search** 컨테이너를 선택합니다.
1. 텍스트 편집기에서 JSON 문서를 만듭니다.

    ```json
    {
      "symboling": 0,
      "make": "toyota",
      "fueltype": "gas",
      "aspiration": "std",
      "numdoors": "four",
      "bodystyle": "wagon",
      "drivewheels": "fwd",
      "enginelocation": "front",
      "wheelbase": 95.7,
      "length": 169.7,
      "width": 63.6,
      "height": 59.1,
      "curbweight": 2280,
      "enginetype": "ohc",
      "numcylinders": "four",
      "enginesize": 92,
      "fuelsystem": "2bbl",
      "bore": 3.05,
      "stroke": 3.03,
      "compressionratio": 9.0,
      "horsepower": 62.0,
      "peakrpm": 4800.0,
      "citympg": 31,
      "highwaympg": 37,
      "price": 0
    }
    ```

    문서를 컴퓨터에 `test-car.json` 확장으로 저장합니다.
1. 포털에서 **업로드**를 선택합니다.
1. **BLOB 업로드** 창에서 **파일 찾아보기**를 선택하고 JSON 문서를 저장한 위치로 이동하여 선택합니다.
1. **업로드**를 선택합니다.

### Azure AI 검색 리소스 만들기

1. Azure Portal 홈 페이지에서 **+ 리소스 만들기**를 선택합니다.
1. **Azure AI 검색**를 검색한 다음 **Azure AI 검색**를 선택합니다.
1. **만들기**를 실행합니다.
1. **리소스 그룹**에서 **aml-for-acs-enrichment**를 선택합니다.
1. 서비스 이름에 고유한 이름을 입력합니다(예: **acs-enriched-1440637584**).
1. **위치**에서 이전에 사용한 것과 동일한 지역을 선택합니다.
1. **검토 + 생성**를 선택한 다음, **생성**를 선택합니다.
1. 리소스가 배포될 때까지 기다린 다음, **리소스로 이동**을 선택합니다.
1. **데이터 가져오기**를 선택합니다.
1. **데이터에 연결** 창에서 **데이터 원본** 필드에 대해 **Azure Blob Storage**를 선택합니다.
1. **데이터 원본 이름**에 **import-docs**를 입력합니다.
1. **구문 분석 모드**에서 **JSON**을 선택합니다.
1. **연결 문자열**에서 **기존 연결 선택**을 선택합니다.
1. 업로드한 스토리지 계정을 선택합니다(예: **amlforacsworks1440637584**).
1. **컨테이너** 창에서 **docs-to-search**를 선택합니다. 
1. **선택**을 선택합니다.
1. **다음: 인식 기술 추가(선택 사항)** 를 선택합니다.

### 인식 기술 추가

1. **보강 추가**를 확장한 다음, **사용자 이름 추출**을 선택합니다.
1. **다음: 대상 인덱스 사용자 지정**을 선택합니다.
1. **+ 필드 추가**를 선택하고 **필드 이름** 목록 하단에 **predicted_price**를 입력합니다.
1. **형식**에서 새 항목에 대해 **Edm.Double**을 선택합니다.
1. 모든 필드에 대해 **조회 가능**을 선택합니다.
1. **만들기**에 대해 **검색 가능**을 선택합니다.
1. **다음: 인덱서 만들기**를 선택합니다.
1. **제출**을 선택합니다.

## 기술 세트에 AML 기술 추가

이제 사용자 이름 보강을 Azure Machine Learning 사용자 지정 기술 세트로 바꿉니다.

1. 개요 창의 **검색 관리**에서 **기술 세트**를 선택합니다.
1. **이름** 아래에서 **azureblob-skillset**를 선택합니다.
1. `EntityRecognitionSkill`에 대한 기술 정의를 이 JSON으로 바꾸고 복사한 엔드포인트와 기본 키 값을 바꿔야 합니다.

    ```json
    "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
    "name": "AMLenricher",
    "description": "AML studio enrichment example",
    "context": "/document",
    "uri": "PASTE YOUR AML ENDPOINT HERE",
    "key": "PASTE YOUR PRIMARY KEY HERE",
    "resourceId": null,
    "region": null,
    "timeout": "PT30S",
    "degreeOfParallelism": 1,
    "inputs": [
      {
        "name": "symboling",
        "source": "/document/symboling"
      },
      {
        "name": "make",
        "source": "/document/make"
      },
      {
        "name": "fuel-type",
        "source": "/document/fueltype"
      },
      {
        "name": "aspiration",
        "source": "/document/aspiration"
      },
      {
        "name": "num-of-doors",
        "source": "/document/numdoors"
      },
      {
        "name": "body-style",
        "source": "/document/bodystyle"
      },
      {
        "name": "drive-wheels",
        "source": "/document/drivewheels"
      },
      {
        "name": "engine-location",
        "source": "/document/enginelocation"
      },
      {
        "name": "wheel-base",
        "source": "/document/wheelbase"
      },
      {
        "name": "length",
        "source": "/document/length"
      },
      {
        "name": "width",
        "source": "/document/width"
      },
      {
        "name": "height",
        "source": "/document/height"
      },
      {
        "name": "curb-weight",
        "source": "/document/curbweight"
      },
      {
        "name": "engine-type",
        "source": "/document/enginetype"
      },
      {
        "name": "num-of-cylinders",
        "source": "/document/numcylinders"
      },
      {
        "name": "engine-size",
        "source": "/document/enginesize"
      },
      {
        "name": "fuel-system",
        "source": "/document/fuelsystem"
      },
      {
        "name": "bore",
        "source": "/document/bore"
      },
      {
        "name": "stroke",
        "source": "/document/stroke"
      },
      {
        "name": "compression-ratio",
        "source": "/document/compressionratio"
      },
      {
        "name": "horsepower",
        "source": "/document/horsepower"
      },
      {
        "name": "peak-rpm",
        "source": "/document/peakrpm"
      },
      {
        "name": "city-mpg",
        "source": "/document/citympg"
      },
      {
        "name": "highway-mpg",
        "source": "/document/highwaympg"
      },
      {
        "name": "price",
        "source": "/document/price"
      }
    ],
    "outputs": [
      {
        "name": "predicted_price",
        "targetName": "predicted_price"
      }
    ]  
    ```

1. **저장**을 선택합니다.

### 출력 필드 매핑 업데이트

1. **개요** 창으로 돌아가서 **인덱서**를 선택한 다음 **azureblob-indexer**를 선택합니다.
1. **인덱서 정의(JSON)** 탭을 선택한 후 **outputFieldMappings** 값을 다음으로 변경합니다.

    ```json
    "outputFieldMappings": [
        {
          "sourceFieldName": "/document/predicted_price",
          "targetFieldName": "predicted_price"
        }
      ]
    ```

1. **저장**을 선택합니다.
1. **다시 설정**을 선택한 다음, **예**를 선택합니다.
1. **실행**을 선택한 다음, **예**를 선택합니다.

## 인덱스 보강 테스트

이제 업데이트된 기술 세트가 인덱스 테스트 자동차 문서에 예측 값을 추가합니다. 이를 테스트하려면 다음 단계를 수행합니다.

1. 검색 서비스의 **개요** 창 상단에 있는 **검색 탐색기**를 선택합니다.
1. **검색**을 선택합니다.
1. 결과 맨 아래로 스크롤합니다.
    ![검색 결과에 추가된 예측 자동차 가격 필드를 보여 주는 스크린샷](../media/06-media/test-results-search-explorer.png)
`predicted_price` 채우기 필드가 표시됩니다.

## 연습 리소스 삭제

연습을 완료했으므로 더 이상 필요하지 않은 모든 리소스를 삭제합니다. Azure 리소스 삭제:

1. **Azure Portal**에서 리소스 그룹을 선택합니다.
1. 필요하지 않은 리소스 그룹을 선택한 다음 **리소스 그룹 삭제**를 선택합니다.
