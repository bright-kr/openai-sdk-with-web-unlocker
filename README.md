# 고성능을 위한 Web Unlocker와 OpenAI Agents SDK 통합 방법

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 OpenAI의 Agents SDK를 Web Unlocker API와 결합하여 웹사이트에서 데이터를 가져오고 처리하는 방식으로 Python에서 견고한 AI 에이전트를 생성하는 방법을 설명합니다.

- [OpenAI Agents SDK란?](#what-is-openai-agents-sdk)
- [이 AI 에이전트 접근 방식의 주요 과제](#major-challenges-with-this-ai-agent-approach)
- [Agents SDK를 Web Unlocker API와 통합하기](#integrating-agents-sdk-with-a-web-unlocker-api)
  - [단계 #1: 프로젝트 설정](#step-1-project-setup)
  - [단계 #2: 프로젝트 의존성 설치 및 시작하기](#step-2-install-the-projects-dependencies-and-get-started)
  - [단계 #3: 환경 변수 읽기 설정](#step-3-set-up-environment-variables-reading)
  - [단계 #4: OpenAI Agents SDK 설정](#step-4-set-up-openai-agents-sdk)
  - [단계 #5: Web Unlocker API 설정](#step-5-set-up-web-unlocker-api)
  - [단계 #6: 웹 페이지 콘텐츠 추출 함수 생성](#step-6-create-the-web-page-content-extraction-function)
  - [단계 #7: 데이터 모델 정의](#step-7-define-the-data-models)
  - [단계 #8: 에이전트 로직 초기화](#step-8-initialize-the-agent-logic)
  - [단계 #9: 실행 루프 구현](#step-9-implement-the-execution-loop)
  - [단계 #10: 전체 구성 통합](#step-10-put-it-all-together)
  - [단계 #11: AI 에이전트 테스트](#step-11-test-the-ai-agent)

## What Is OpenAI Agents SDK?

[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)는 OpenAI가 만든 오픈 소스 Python 라이브러리입니다. 개발자가 에이전트 기반 AI 애플리케이션을 간단하고 효율적이며 프로덕션 환경에 적합한 방식으로 구축할 수 있도록 지원합니다. 이 라이브러리는 OpenAI의 이전 실험 프로젝트인 [Swarm](https://github.com/openai/swarm)을 정제한 버전입니다.

OpenAI Agents SDK는 최소한의 추상화로 여러 필수 구성 요소를 제공합니다:

- **Agents**: 특정 지침과 도구가 결합된 LLM로, 작업을 실행합니다
- **Handoffs**: 필요할 때 에이전트가 다른 에이전트에게 작업을 전달할 수 있게 합니다
- **Guardrails**: 에이전트 입력을 검증하여 예상 형식이나 요구 사항을 충족하는지 확인합니다

이러한 핵심 요소와 Python의 범용성이 결합되어, 에이전트와 도구 간의 정교한 상호작용을 쉽게 만들 수 있습니다.

또한 SDK에는 내장 트레이싱 기능이 있어 에이전트 워크플로를 시각화하고, 문제를 해결하며, 평가할 수 있습니다. 특정 사용 사례를 위한 모델 파인튜닝도 지원합니다.

## Major Challenges with This AI Agent Approach

대부분의 AI 에이전트는 콘텐츠를 추출하거나 페이지 요소와 상호작용하는 등 웹 페이지에서의 작업을 자동화하는 것을 목표로 합니다. 즉, 본질적으로 Web을 프로그래밍 방식으로 탐색해야 합니다.

AI 모델 자체의 잠재적 오해석을 넘어, 이러한 에이전트가 마주하는 가장 큰 [장애물](https://brightdata.co.kr/blog/web-data/anti-scraping-techniques)은 웹사이트의 방어 메커니즘을 처리하는 일입니다. 많은 사이트가 앤チボット 및 アンチスクレイピング 기술을 구현하여 AI 에이전트를 제한하거나 오도할 수 있기 때문입니다. 특히 오늘날에는 [anti-AI CAPTCHA와 고도화된 봇 탐지 시스템이 점점 더 보편화](https://hackernoon.com/ai-agent-browsers-are-failing-and-its-not-just-because-of-captchas)되고 있어 더욱 중요합니다.

이러한 장애물을 극복하려면, 에이전트의 웹 탐색 역량을 강화하기 위해 [Bright Data의 Web Unlocker API](https://brightdata.co.kr/products/web-unlocker)와 같은 솔루션과 통합해야 합니다. 이 도구는 인터넷에 연결되는 어떤 HTTP 클라이언트나 솔루션( AI 에이전트 포함)과도 함께 동작하는 웹 언락 게이트웨이 역할을 합니다. 어떤 웹페이지에서든 깨끗하고 차단되지 않은 HTML을 제공합니다. 더 이상 CAPTCHA, IP 제약, 접근 불가 콘텐츠에 막히지 않습니다.

## Integrating Agents SDK with a Web Unlocker API

이 안내 섹션에서는 OpenAI Agents SDK를 Bright Data의 Web Unlocker API와 통합하여 다음을 수행할 수 있는 AI 에이전트를 구성하는 방법을 확인합니다:

1. 어떤 웹 페이지에서든 텍스트 요약 생성
2. e-commerce 웹사이트에서 구조화된 상품 정보 획득
3. 뉴스 기사에서 핵심 세부 정보 수집

이를 위해 에이전트는 OpenAI Agents SDK에 Web Unlocker API를 모든 웹 페이지 콘텐츠를 획득하는 메커니즘으로 사용하도록 지시합니다. 콘텐츠를 확보한 뒤, 에이전트는 각 작업에 필요한 형태로 데이터를 추출하고 포맷팅하는 AI 로직을 적용합니다.

> **면책 조항**:
> 
> 위에서 언급한 세 가지 사용 사례는 단지 예시입니다. 여기서 제시하는 방법론은 에이전트의 동작을 커스터마이즈함으로써 다양한 다른 시나리오로 확장할 수 있습니다.

최적의 성능을 위해 OpenAI Agents SDK와 Bright Data의 Web Unlocker API를 사용하여 Python에서 AI スクレイピング 에이전트를 개발하려면 다음 지침을 따르십시오.

### Prerequisites

이 튜토리얼을 시작하기 전에 다음 사항을 준비했는지 확인하십시오:

- 컴퓨터에 Python 3 이상 설치
- 활성 Bright Data 계정
- 활성 OpenAI 계정
- HTTP リクエスト에 대한 기본 이해
- Pydantic 모델에 대한 약간의 친숙함
- AI 에이전트 기능에 대한 일반적인 이해

### Step #1: Project Setup

먼저 시스템에 Python 3이 설치되어 있는지 확인하십시오. 설치되어 있지 않다면 [Python 다운로드](https://www.python.org/downloads/) 후 운영 체제에 맞는 설치 지침을 따르십시오.

터미널을 실행하고 スクレイピング 에이전트 프로젝트를 위한 새 디렉터리를 생성하십시오:

```sh
mkdir openai-sdk-agent
```

`openai-sdk-agent` 디렉터리에는 Python 기반, Agents SDK로 구동되는 에이전트의 모든 코드가 포함됩니다.

프로젝트 디렉터리로 이동한 다음 [가상 환경](https://docs.python.org/3/library/venv.html)을 설정하십시오:

```sh
cd openai-sdk-agent
python -m venv venv
```

선호하는 Python IDE에서 프로젝트 디렉터리를 여십시오. [Python extension이 설치된 Visual Studio Code](https://code.visualstudio.com/docs/languages/python) 또는 [PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/#section=windows)을 권장합니다.

`openai-sdk-agent` 디렉터리 안에 `agent.py`라는 새 Python 파일을 생성하십시오. 이제 디렉터리 구조는 다음과 같이 보일 것입니다:

![The file structure of the AI agent project](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/The-file-structure-of-the-AI-agent-project.png)

현재 `scraper.py`는 비어 있는 Python 스크립트이지만, 곧 원하는 AI 에이전트 로직을 포함하게 됩니다.

IDE의 터미널에서 가상 환경을 활성화하십시오. Linux 또는 macOS의 경우 다음 명령을 실행하십시오:

```sh
./env/bin/activate
```

Windows에서는 다음을 실행하십시오:

```powershell
env/Scripts/activate
```

### Step #2: Install the Project's Dependencies and Get Started

이 프로젝트는 다음 Python 라이브러리를 사용합니다:

- [`openai-agents`](https://openai.github.io/openai-agents-python/): OpenAI Agents SDK로, Python에서 AI 에이전트를 생성하는 데 사용합니다.
- [`requests`](https://requests.readthedocs.io/en/latest/): Bright Data의 Web Unlocker API에 연결하고 AI 에이전트가 처리할 웹 페이지의 HTML 콘텐츠를 가져오는 데 사용합니다. 자세한 내용은 [Python Requests 라이브러리 완전 정복](https://brightdata.co.kr/blog/web-data/python-requests-guide) 가이드를 참고하십시오.
- [`pydantic`](https://docs.pydantic.dev/latest/): 구조화된 출력 모델을 정의하여, 에이전트가 명확하고 검증된 형식으로 데이터를 반환하도록 합니다.
- [`markdownify`](https://python.langchain.com/docs/integrations/document_transformers/markdownify/): 원시 HTML 콘텐츠를 깔끔한 Markdown으로 변환합니다. (곧 그 이점을 설명합니다.)
- [`python-dotenv`](https://github.com/theskumar/python-dotenv): `.env` 파일에서 환경 변수를 로드합니다. 여기에 OpenAI 및 Bright Data의 자격 증명을 저장합니다.

활성화된 가상 환경에서 다음으로 모두 설치하십시오:

```sh
pip install requests pydantic openai-agents openai-agents markdownify python-dotenv
```

이제 `scraper.py`를 다음 import 및 async 보일러플레이트 코드로 설정하십시오:

```python
import asyncio
from agents import Agent, RunResult, Runner, function_tool
import requests
from pydantic import BaseModel
from markdownify import markdownify as md
from dotenv import load_dotenv 

# AI agent logic...

async def run():
    # Call the async AI agent logic...

if __name__ == "__main__":
    asyncio.run(run())
```

### Step #3: Set Up Environment Variables Reading

프로젝트 디렉터리에 `.env` 파일을 생성하십시오:

![Adding a .env file to your project](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/Adding-a-.env-file-to-your-project.png)

이 파일에는 API 키 및 시크릿 토큰과 같은 환경 변수를 저장합니다. `.env` 파일에서 환경 변수를 로드하려면 `dotenv` 패키지의 `load_dotenv()`를 사용하십시오:

```python
load_dotenv()
```

이제 [`os.getenv()`](https://docs.python.org/3/library/os.html#os.getenv)를 사용하여 특정 환경 변수에 다음과 같이 접근할 수 있습니다:

```python
os.getenv("ENV_NAME")
```

Python 표준 라이브러리의 [`os`](https://docs.python.org/3/library/os.html)를 import하는 것을 잊지 마십시오:

```python
import os
```

### Step #4: Set Up OpenAI Agents SDK

OpenAI Agents SDK를 사용하려면 유효한 OpenAI API 키가 필요합니다. 아직 생성하지 않았다면 [OpenAI 공식 가이드](https://help.openai.com/en/articles/4936850-where-do-i-find-my-openai-api-key)를 따라 API 키를 생성하십시오.

키를 발급받은 후 `.env` 파일에 다음과 같이 추가하십시오:

```python
OPENAI_API_KEY="<YOUR_OPENAI_KEY>"
```

`<YOUR_OPENAI_KEY>` 플레이스홀더를 실제 키로 반드시 교체하십시오.

추가 설정은 필요하지 않습니다. `openai-agents` SDK는 `OPENAI_API_KEY` 환경 변수에서 API 키를 자동으로 가져오도록 설계되어 있습니다.

### Step #5: Set Up Web Unlocker API

아직 계정이 없다면 [Bright Data 계정을 생성](https://brightdata.co.kr/?hs_signup=1)하십시오. 이미 있다면 [로그인](https://brightdata.co.kr/cp/start)하십시오.

다음으로, [Bright Data 공식 Web Unlocker 문서](https://docs.brightdata.com/scraping-automation/web-unlocker/quickstart)를 참고하여 API 토큰을 획득하십시오. 또는 아래 단계를 따르십시오.

Bright Data "User Dashboard" 페이지에서 "Get proxy products" 옵션을 선택하십시오:

![Clicking the "Get proxy products" option](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/Clicking-the-Get-proxy-products-option.png)

제품 테이블에서 "unblocker"라고 표시된 행을 찾은 다음 클릭하십시오:

![Clicking the "unblocker" row](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/Clicking-the-unblocker-row.png)

"unlocker" 페이지에서 클립보드 아이콘을 사용하여 API 토큰을 복사하십시오:

![Copying the API token](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/Copying-the-API-token.png)

또한 우측 상단의 토글이 "On"으로 전환되어 Web Unlocker 제품이 활성화되어 있는지 확인하십시오.

"Configuration" 탭에서 최적의 효과를 위해 다음 옵션이 활성화되어 있는지 확인하십시오:

- [Premium domains](https://docs.brightdata.com/scraping-automation/web-unlocker/configuration#premium-domains)
- [CAPTCHA Solver](https://brightdata.co.kr/products/web-unlocker/captcha-solver)

![Making sure that the premium options for effectiveness are enabled](https://media.brightdata.com/2025/04/Making-sure-that-the-premium-options-for-effectiveness-are-enabled.png)

`.env` 파일에 다음 환경 변수를 추가하십시오:

```python
BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN="<YOUR_BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN>"
```

플레이스홀더를 실제 API 토큰으로 교체하십시오.

### Step #6: Create the Web Page Content Extraction Function

다음 작업을 수행하는 `get_page_content()` 함수를 생성하십시오:

1. `BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN` 환경 변수를 읽습니다
2. `requests`를 사용하여 [제공된 URL로 Bright Data의 Web Unlocker API에 리クエスト를 전송](https://docs.brightdata.com/scraping-automation/web-unlocker/send-your-first-request)합니다
3. API가 반환한 원시 HTML을 가져옵니다
4. HTML을 Markdown으로 변환하고 반환합니다

위 로직을 다음과 같이 구현하십시오:

```python
@function_tool
def get_page_content(url: str) -> str:
    """
    Retrieves the HTML content of a given web page using Bright Data's Web Unlocker API,
    bypassing anti-bot protections. The response is converted from raw HTML to Markdown
    for easier and cheaper processing.

    Args:
        url (str): The URL of the web page to scrape.

    Returns:
        str: The Markdown-formatted content of the requested page.
    """

    # Read the Bright Data's Web Unlocker API token from the envs
    BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN = os.getenv("BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN")

    # Configure the Web Unlocker API call
    api_url = "https://api.brightdata.com/request"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN}"
    }
    data = {
        "zone": "unblocker",
        "url": url,
        "format": "raw"
    }

    # Make the call to Web Uncloker to retrieve the unblocked HTML of the target page
    response = requests.post(api_url, headers=headers, data=json.dumps(data))

    # Extract the raw HTML response
    html = response.text

    # Convert the HTML to markdown and return it
    markdown_text = md(html)

    return markdown_text
```

**참고 1**: 함수는 반드시 [`@function_tool`](https://openai.github.io/openai-agents-python/tools/)로 어노테이션되어야 합니다. 이 특수 데코레이터는 OpenAI Agents SDK에 이 함수가 에이전트가 특정 작업을 수행하기 위한 도구로 사용할 수 있음을 알립니다. 이 경우 해당 함수는 에이전트가 처리할 웹 페이지 콘텐츠를 가져오는 데 활용할 수 있는 “엔진” 역할을 합니다.

**참고 2**: `get_page_content()` 함수는 입력 타입을 명시적으로 선언해야 합니다.  
이를 생략하면 다음과 같은 오류가 발생합니다: `Error getting response: Error code: 400 - {'error': {'message': "Invalid schema for function 'get_page_content': In context=('properties', 'url'), schema must have a 'type' key.``"`

원시 HTML을 Markdown으로 변환하면 HTML이 매우 장황하고 scripts, styles, metadata 같은 불필요한 요소를 자주 포함하기 때문에 성능 효율과 비용 효율이 향상됩니다. AI 에이전트는 이러한 콘텐츠가 필요하지 않습니다. 에이전트에 텍스트, 링크, 이미지 등 핵심 요소만 필요하다면 Markdown이 훨씬 더 깔끔하고 компакт한 표현을 제공합니다.

특히 HTML-to-Markdown 변환은 입력 크기를 최대 99%까지 줄일 수 있으며, 다음 두 가지를 절감합니다:

- 토큰( OpenAI 모델 사용 비용 절감)
- 처리 시간(모델이 작은 입력에서 더 빠르게 동작)

자세한 내용은 “[Why Are the New AI Agents Choosing Markdown Over HTML?](https://hackernoon.com/why-are-the-new-ai-agents-choosing-markdown-over-html)” 문서를 참고하십시오.

### Step #7: Define the Data Models

정상적으로 동작하려면 OpenAI SDK 에이전트는 출력 데이터의 예상 구조를 정의하기 위해 Pydantic 모델이 필요합니다. 우리가 만들 에이전트는 다음 세 가지 출력 중 하나를 반환할 수 있음을 기억하십시오:

1.  페이지 요약
2.  상품 정보
3.  뉴스 기사 정보

이에 대응하는 세 가지 [Pydantic 모델](https://docs.pydantic.dev/latest/concepts/models/)을 정의해 보겠습니다:

```python
class Summary(BaseModel):
    summary: str

class Product(BaseModel):
    name: str
    price: Optional[float] = None
    currency: Optional[str] = None
    ratings: Optional[int] = None
    rating_score: Optional[float] = None

class News(BaseModel):
    title: str
    subtitle: Optional[str] = None
    authors: Optional[List[str]] = None
    text: str
    publication_date: Optional[str] = None
```

**참고**: `Optional`을 사용하면 에이전트가 더 범용적이고 유연해집니다. 모든 페이지가 스키마에 정의된 모든 데이터를 포함하지는 않으므로, 이러한 유연성은 필드가 누락될 때 오류를 방지하는 데 도움이 됩니다.

[`typing`](https://docs.python.org/3/library/typing.html)에서 `Optional`과 `List`를 import하는 것을 잊지 마십시오:

```python
from typing import Optional, List
```

### Step #8: Initialize the Agent logic

`openai-agents` SDK의 [`Agent`](https://platform.openai.com/docs/guides/agents) 클래스를 사용하여 세 가지 특화 에이전트를 정의하십시오:

```python
summarization_agent = Agent(
    name="Text Summarization Agent",
    instructions="You are a content summarization agent that summarizes the input text.",
    tools=[get_page_content],
    output_type=Summary,
)

product_info_agent = Agent(
    name="Product Information Agent",
    instructions="You are a product parsing agent that extracts product details from text.",
    tools=[get_page_content],
    output_type=Product,
)

news_info_agent = Agent(
    name="News Information Agent",
    instructions="You are a news parsing agent that extracts relevant news details from text.",
    tools=[get_page_content],
    output_type=News,
)
```

각 에이전트는 다음을 수행합니다:

1. 의도한 기능을 설명하는 명확한 지침 문자열을 포함합니다. OpenAI Agents SDK는 이를 사용해 에이전트의 동작을 가이드합니다.
2. `get_page_content()`를 도구로 사용하여 입력 데이터(즉, 웹 페이지 콘텐츠)를 가져옵니다.
3. 앞서 정의한 Pydantic 모델(`Summary`, `Product`, `News`) 중 하나로 출력을 반환합니다.

사용자 요청을 적절한 특화 에이전트로 자동 라우팅하기 위해, 상위 레벨 에이전트를 정의하십시오:

```python
routing_agent = Agent(
    name="Routing Agent",
    instructions=(
        "You are a high-level decision-making agent. Based on the user's request, "
        "hand off the task to the appropriate agent."
    ),
    handoffs=[summarization_agent, product_info_agent, news_info_agent],
) 
```

이 에이전트는 `run()` 함수에서 쿼리하여 AI 에이전트 로직을 구동하는 데 사용합니다.

### Step #9: Implement the Execution Loop

`run()` 함수에서 다음 루프를 추가하여 AI 에이전트 로직을 실행하십시오:

```python
# Keep iterating until the use type "exit"
while True:
    # Read the user's request
    request = input("Your request -> ")
    # Stops the execution if the user types "exit"
    if request.lower() in ["exit"]:
        print("Exiting the agent...")
        break
    # Read the page URL to operate on
    url = input("Page URL -> ")

    # Routing the user's request to the right agent
    output = await Runner.run(routing_agent, input=f"{request} {url}")
    # Conver the agent's output to a JSON string
    json_output = json.dumps(output.final_output.model_dump(), indent=4)
    print(f"Output -> \n{json_output}\n\n")
```

이 루프는 사용자 입력을 지속적으로 감시하며, 각 요청을 적절한 에이전트(요약, 상품, 뉴스)로 라우팅하여 처리합니다. 사용자 쿼리와 대상 URL을 결합하고, 로직을 실행한 뒤, [`json`](https://docs.python.org/3/library/json.html)을 사용해 구조화된 결과를 JSON 형식으로 표시합니다. 다음으로 import하십시오:

```python
import json
```

### Step #10: Put It All Together

이제 `scraper.py` 파일에는 다음 내용이 포함되어야 합니다:

```python
import asyncio
from agents import Agent, RunResult, Runner, function_tool
import requests
from pydantic import BaseModel
from markdownify import markdownify as md
from dotenv import load_dotenv
import os
from typing import Optional, List
import json

# Load the environment variables from the .env file
load_dotenv()

# Define the Pydantic output models for your AI agent
class Summary(BaseModel):
    summary: str

class Product(BaseModel):
    name: str
    price: Optional[float] = None
    currency: Optional[str] = None
    ratings: Optional[int] = None
    rating_score: Optional[float] = None

class News(BaseModel):
    title: str
    subtitle: Optional[str] = None
    authors: Optional[List[str]] = None
    text: str
    publication_date: Optional[str] = None

@function_tool
def get_page_content(url: str) -> str:
    """
    Retrieves the HTML content of a given web page using Bright Data's Web Unlocker API,
    bypassing anti-bot protections. The response is converted from raw HTML to Markdown
    for easier and cheaper processing.

    Args:
        url (str): The URL of the web page to scrape.

    Returns:
        str: The Markdown-formatted content of the requested page.
    """

    # Read the Bright Data's Web Unlocker API token from the envs
    BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN = os.getenv("BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN")

    # Configure the Web Unlocker API call
    api_url = "https://api.brightdata.com/request"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {BRIGHT_DATA_WEB_UNLOCKER_API_TOKEN}"
    }
    data = {
        "zone": "unblocker",
        "url": url,
        "format": "raw"
    }

    # Make the call to Web Uncloker to retrieve the unblocked HTML of the target page
    response = requests.post(api_url, headers=headers, data=json.dumps(data))

    # Extract the raw HTML response
    html = response.text

    # Convert the HTML to markdown and return it
    markdown_text = md(html)

    return markdown_text

# Define the individual OpenAI agents
summarization_agent = Agent(
    name="Text Summarization Agent",
    instructions="You are a content summarization agent that summarizes the input text.",
    tools=[get_page_content],
    output_type=Summary,
)

product_info_agent = Agent(
    name="Product Information Agent",
    instructions="You are a product parsing agent that extracts product details from text.",
    tools=[get_page_content],
    output_type=Product,
)

news_info_agent = Agent(
    name="News Information Agent",
    instructions="You are a news parsing agent that extracts relevant news details from text.",
    tools=[get_page_content],
    output_type=News,
)

# Define a high-level routing agent that delegates tasks to the appropriate specialized agent
routing_agent = Agent(
    name="Routing Agent",
    instructions=(
        "You are a high-level decision-making agent. Based on the user's request, "
        "hand off the task to the appropriate agent."
    ),
    handoffs=[summarization_agent, product_info_agent, news_info_agent],
)

async def run():
    # Keep iterating until the use type "exit"
    while True:
        # Read the user's request
        request = input("Your request -> ")
        # Stops the execution if the user types "exit"
        if request.lower() in ["exit"]:
            print("Exiting the agent...")
            break
        # Read the page URL to operate on
        url = input("Page URL -> ")

        # Routing the user's request to the right agent
        output = await Runner.run(routing_agent, input=f"{request} {url}")
        # Conver the agent's output to a JSON string
        json_output = json.dumps(output.final_output.model_dump(), indent=4)
        print(f"Output -> \n{json_output}\n\n")


if __name__ == "__main__":
    asyncio.run(run())
```

### Step #11: Test the AI Agent

AI 에이전트를 실행하려면 다음을 실행하십시오:

```sh
python agent.py
```

예를 들어 [Bright Data의 AI services hub](https://brightdata.co.kr/ai) 콘텐츠를 요약하고 싶다면, 다음과 같은 요청을 입력하면 됩니다:

![The input to get a summary of Bright Data's AI services](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/The-input-to-get-a-summary-of-Bright-Datas-AI-services.png)

그러면 다음과 같은 JSON 형식의 결과를 받게 됩니다:

![The summary returned by your AI agent](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/The-summary-returned-by-your-AI-agent.png)

이제 [PS5 listing](https://www.amazon.com/PlayStation%C2%AE5-console-slim-PlayStation-5/dp/B0CL61F39H/)과 같은 Amazon 상품 페이지에서 상품 데이터를 추출하고 싶다고 가정해 보겠습니다:

![The Amazon PS5 page](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/The-Amazon-PS5-page.png)

일반적으로 [Amazon의 CAPTCHA 및 앤チボット 시스템](https://brightdata.co.kr/blog/web-data/bypass-amazon-captcha)이 요청을 차단합니다. Web Unlocker API를 사용하면, AI 에이전트는 차단되지 않고 페이지에 접근하여 분석할 수 있습니다:

![Getting Amazon product data](https://media.brightdata.com/2025/04/Getting-Amazon-product-data.gif)

출력은 다음과 같습니다:

```json
{
    "name": "PlayStation\u00ae5 console (slim)",
    "price": 499.0,
    "currency": "USD",
    "ratings": 6321,
    "rating_score": 4.7
}
```

마지막으로 [Yahoo News 기사](https://www.yahoo.com/news/pope-francis-dies-88-080859417.html)에서 구조화된 뉴스 정보를 얻고 싶다고 가정해 보겠습니다:

![The target Yahoo News article](https://github.com/luminati-io/openai-sdk-with-web-unlocker/blob/main/images/The-target-Yahoo-News-article.png)

다음 입력으로 수행할 수 있습니다:

```
Your request -> Give me news info
Page URL -> https://www.yahoo.com/news/pope-francis-dies-88-080859417.html
```

결과는 다음과 같습니다:

```json
{
    "title": "Pope Francis Dies at 88",
    "subtitle": null,
    "authors": [
        "Nick Vivarelli",
        "Wilson Chapman"
    ],
    "text": "Pope Francis, the 266th Catholic Church leader who tried to position the church to be more inclusive, died on Easter Monday, Vatican officials confirmed. He was 88. (omitted for brevity...)",
    "publication_date": "Mon, April 21, 2025 at 8:08 AM UTC"
}
```

## Conclusion

OpenAI SDK와 Bright Data의 [Web Unlocker API](https://brightdata.co.kr/products/web-unlocker)를 결합하면 사실상 거의 모든 웹 페이지에서 안정적으로 동작하는 AI 에이전트를 개발할 수 있습니다. 이는 Bright Data의 제품과 서비스가 고급 AI 통합을 지원하는 방식의 한 가지 예시에 불과합니다.

[AI products](https://brightdata.co.kr/ai/products-for-ai) 전체 라인업을 살펴보십시오: autonomous AI agents, vertical AI apps, foundation models, multimodal AI, data providers, data packages 등 다양한 선택지를 제공합니다.

지금 Bright Data 계정을 생성하고 AI 에이전트 개발을 위한 모든 제품과 서비스를 사용해 보십시오!