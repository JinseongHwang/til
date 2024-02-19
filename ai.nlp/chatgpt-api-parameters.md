## ChatGPT API 파라미터

- [ChatGPT API 파라미터](#chatgpt-api-파라미터)
  - [Create chat completion](#create-chat-completion)
  - [1. `messages` / \[array\] / Required](#1-messages--array--required)
  - [2. `model` / \[string\] / Required](#2-model--string--required)
  - [3. `frequency_penalty` / \[number or null\] / Optional](#3-frequency_penalty--number-or-null--optional)
  - [4. `logit_bias` / \[map\] / Optional](#4-logit_bias--map--optional)
  - [5. `logprobs` / \[boolean or null\] / Optional](#5-logprobs--boolean-or-null--optional)
  - [6. `top_logprobs` / \[integer or null\] / Optional](#6-top_logprobs--integer-or-null--optional)
  - [7. `max_tokens` / \[integer or null\] / Optional](#7-max_tokens--integer-or-null--optional)
  - [8. `n` / \[integer or null\] / Optional](#8-n--integer-or-null--optional)
  - [9. `presence_penalty` / \[number or null\] / Optional](#9-presence_penalty--number-or-null--optional)
  - [10. `response_format` / \[object\] / Optional](#10-response_format--object--optional)
  - [11. `seed` / \[integer or null\] / Optional](#11-seed--integer-or-null--optional)
  - [12. `stop` / \[string / array / null\] / Optional](#12-stop--string--array--null--optional)
  - [13. `stream` / \[boolean or null\] / Optional](#13-stream--boolean-or-null--optional)
  - [14. `temperature` / \[number or null\] / Optional](#14-temperature--number-or-null--optional)
  - [15. `top_p` / \[number or null\] / Optional](#15-top_p--number-or-null--optional)
  - [16. `tools` / \[array\] / Optional](#16-tools--array--optional)
  - [17. `tool_choice` / \[string or object\] / Optional](#17-tool_choice--string-or-object--optional)
  - [18. `user` / string / Optional](#18-user--string--optional)
- [top\_p vs temperature](#top_p-vs-temperature)
- [References](#references)

### Create chat completion
```sh
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ]
  }'
```
Request Body에 더 다양한 파라미터를 넣어서 내 입맛에 맞게 튜닝이 가능하다. 파라미터 종류에 대해 알아보자.

### 1. `messages` / [array] / Required
```json
"messages": [
    {
        "role": "system",
        "content": "You are a helpful assistant."
    },
    {
        "role": "user",
        "content": "Hello!"
    }
]
```
system에는 GPT의 페르소나 설정을 진행하고, user에는 명령을 입력하면 된다.

### 2. `model` / [string] / Required

```json
"model": "gpt-3.5-turbo"
```
사용할 모델의 ID를 입력한다. GPT4를 사용하고 싶다면 작성일 기준 "gpt-4-turbo-preview"를 입력하면 된다.

### 3. `frequency_penalty` / [number or null] / Optional
```json
"frequency_penalty": 0.8
```
- 기본값 = 0
- -2.0 <= frequency_penalty <= 2.0

이 숫자가 커질수록 기존 토큰 대비 새로운 토큰에 낮은 점수를 줘서 모델이 같은 토큰을 반복할 가능성을 낮춰준다. 발화를 반복하는 이슈가 있을 때 이 값을 올리거나, 새로운 발화를 보고 싶을 때 이 값을 내리면 된다.

### 4. `logit_bias` / [map] / Optional
```
{
    "93538": 20
}
```
- 기본값 = null

특정 토큰를 선호하거나 피하도록 하는 방법이다. bias 값을 조정하여 모델에게 "이 단어를 사용하는 것이 좋다", "이 단어를 사용하지 말라"라고 말할 수 있다.

입력 값은 JSON 형식이고, Key는 token_id, Value는 -100과 100 사이의 정수이다. Value가 커질수록(양수) 선호의 의미이고, 낮아질수록(음수) 기피의 의미이다.

![스크린샷](https://github.com/JinseongHwang/til/assets/132965185/1113e84c-f78e-438e-84b5-1c2de0882683)  
token_id는 [OpenAI Tokenizer](https://platform.openai.com/tokenizer)에서 알아낼 수 있다. 예를 들어 pizza는 93538이다. 따라서 아래와 같이 작성하면 pizza의 선호 혹은 기피를 제안할 수 있다.

![스크린샷](https://github.com/JinseongHwang/til/assets/132965185/0e211173-76a5-4536-af2f-ce3af5f47e4c)  
단, 한국어는 제약 사항이 많아서 제대로 동작하지 않는다. 예시의 "호박고구마" 라는 하나의 단어가 7개의 토큰으로 분리된다. 각 토큰 별로 선호 및 기피 제어는 가능하지만, 예시처럼 하나의 단어가 여러 토큰으로 쪼개지는 경우 이 파라미터를 조절하면 나와야 할 글자가 나오지 않거나 이상한 글자가 계속 나올 수도 있다. 따라서 꼭 Tokenizer에서 잘 나오는지 확인해 보고 사용하자.

### 5. `logprobs` / [boolean or null] / Optional
```json
"logprobs": true
```
- 기본값 = false
- 작성일 기준 "gpt-4-turbo-preview" 모델에서는 사용할 수 없다!

이 파라미터가 true이면 생성된 토큰의 로그 확률을 함께 반환한다. 모델이 선택한 단어(토큰)에 대해 얼마나 확신하는지를 알려준다. "현재까지 구문을 봤을 때 이 자리에 이 토큰이 들어가는데 70% 확신합니다"라고 말하는 것과 같다.

### 6. `top_logprobs` / [integer or null] / Optional
```json
"top_logprobs": 1
```
각 토큰 위치에서 반환할 가능성이 가장 높은 토큰의 수를 지정할 수 있다. 이 파라미터를 사용하려면 반드시 logprobs=true 가 되어야 한다.

예를 들어, 기사 헤드라인에 대한 카테고리를 뽑는 것을 ChatGPT에게 부탁한다고 가정해보자. 이 파라미터를 활성화하면 다음과 같이 출력된다.

```plaintext
Headline: Tech Giant Unveils Latest Smartphone Model with Advanced Photo-Editing Features.
Output token 1: Technology, logprobs: -2.4584822e-06, linear probability: 100.0%
Output token 2: Techn, logprobs: -13.781253, linear probability: 0.0%

Headline: Local Mayor Launches Initiative to Enhance Urban Public Transport.
Output token 1: Politics, logprobs: -2.4584822e-06, linear probability: 100.0%
Output token 2: Technology, logprobs: -13.937503, linear probability: 0.0%

Headline: Tennis Champion Showcases Hidden Talents in Symphony Orchestra Debut
Output token 1: Art, logprobs: -0.009169078, linear probability: 99.09%
Output token 2: Sports, logprobs: -4.696669, linear probability: 0.91%
```

### 7. `max_tokens` / [integer or null] / Optional
```json
"max_tokens": 4096
```
생성할 수 있는 최대 토큰 수이다. "gpt-4-turbo-preview" 모델의 최대 토큰 수가 4096이다.

### 8. `n` / [integer or null] / Optional
```json
"n": 1
```
- 기본값 = 1

생성될 메시지의 선택지를 몇 개 줄지를 결정하는 파라미터이다. 단, n배 만큼 응답이 생성되니 비용도 n배만큼 부과된다. 따라서 비용 최소화를 위해 1로 설정해야 합니다.

### 9. `presence_penalty` / [number or null] / Optional
```json
"presence_penalty": 1.2
```
- 기본값 = 0
- -2.0 <= presence_penalty <= 2.0

이 파라미터를 양수로 설정하면 새로운 토큰이 지금까지 텍스트에 등장했는지 여부에 따라 새로운 토큰에 불이익을 주므로 모델이 새로운 주제에 대해 이야기할 가능성이 높아진다.

### 10. `response_format` / [object] / Optional
```json
"response_format": { 
    "type": "json_object"
}
```
모델의 출력 형식을 지정하는 파라미터이다. GPT-3.5 Turbo, GPT-4 모델들과 호환된다. 위와 같이 설정하면 json 형식으로 출력한다. json 형식 출력으로 설정하면 출력 값이 유효한 json 형식이라는 것을 보장한다. (type=text도 가능하다.)

json 모드를 사용할 때는 system 또는 user 메시지를 통해 모델에 직접 json을 생성하도록 지시해야 한다. 그렇지 않으면 모델이 max_tokens에 도달할 때까지 공백을 생성하여 요청이 오래 실행되고 멈춘것처럼 보일 수 있다. 또한 generation이 max_tokens를 초과했거나 대화가 최대 컨텍스트 길이를 초과했음을 나타내는 finish_reason="length"인 경우 메시지 콘텐츠가 부분적으로 잘릴 수 있다.

### 11. `seed` / [integer or null] / Optional
```json
"seed": null
```
- 작성일 기준 베타 버전이다.

이 파라미터를 사용하면 모델 생성 결과를 샘플링해서 동일한 seed와 매개변수를 사용한 반복 요청에 대해 최대한 동일한 결과가 나오도록 한다. 하지만 이는 꼭 보장되는 것이 아니다. system_fingerprint 파라미터로 변경 사항에 대해 모니터링 할 수 있다.

### 12. `stop` / [string / array / null] / Optional
```json
"stop": "$STOP"
```
- 기본값 = null

더 이상 토큰을 생성하지 않는 시퀀스를 결정한다. 최대 4개까지 넣을 수 있다. 모델에게 "이 단어를 만나면 생성을 멈추세요!"라고 요청하는 것과 같다.

### 13. `stream` / [boolean or null] / Optional
```json
"stream": false
```
- 기본값 = false

스트림 모드를 켜면 ChatGPT 웹사이트에서 사용하는 것처럼 생성된 토큰 별로 즉시 받아볼 수 있다. 기본은 false이기 때문에 생성이 끝나면 한방에 응답을 보내준다.

### 14. `temperature` / [number or null] / Optional
```json
"temperature": 0.2
```
- 기본값 = 1
- 0 <= temperature <= 2

이 파라미터 값이 0.8과 같이 높으면 출력이 더 무작위적이고, 0.2와 같이 낮으면 더 집중적이고 결정론적이다. 일반적으로 이 파라미터 또는 top_p를 변경하는 것이 좋지만 둘 다 변경하는 것은 권장하지 않는다.

### 15. `top_p` / [number or null] / Optional
```json
"top_p": 1
```
- 기본값 = 1

이 파라미터는 temperature를 대신할 수 있고, 역시 모델의 창의성을 제어하는 방법 중 하나이다. 모델에게 가장 확신하는 것을 고수하라고 명령하는 것과 같다. 일반적으로 이 파라미터 또는 temperature를 변경하는 것이 좋지만 둘 다 변경하는 것은 권장하지 않는다.

### 16. `tools` / [array] / Optional
```json
"tools": {
    "type": "function",
    "function": {
        "name": "my_function",
        "description": "...",
        "parameters": {...}
    }
}
```
모델이 호출할 수 있는 도구 목록이다. 현재는 function만 지원된다. 모델이 JSON 입력을 생성할 수 있는 것도 함수이다.

### 17. `tool_choice` / [string or object] / Optional
```json
"tool_choice": {
    "type": "function", 
    "function": {
        "name": "my_function"
    }
}
```
- 함수가 없는 경우에는 none이 기본값이며, 함수가 있는 경우에는 auto가 기본값이다.

모델에서 호출할 함수를 제어한다. none으로 설정하면 모델이 함수를 호출하지 않고 메시지를 생성하는 것을 의미한다. auto는 모델이 메시지 생성 또는 함수 호출 중 하나를 선택할 수 있음을 의미한다. 위와 같이 작성하면 모델이 my_function 함수를 강제로 호출한다.

### 18. `user` / string / Optional
```json
"user": "jinseong-only"
```
최종 사용자를 나타내는 고유 식별자이다. OpenAI 측에서 남용하는지 모니터링 할 때 사용한다.

## top_p vs temperature

- https://community.openai.com/t/cheat-sheet-mastering-temperature-and-top-p-in-chatgpt-api/172683
- https://medium.com/@1511425435311/understanding-openais-temperature-and-top-p-parameters-in-language-models-d2066504684f

## References
- https://platform.openai.com/docs/api-reference/chat/create
- https://cookbook.openai.com/examples/using_logprobs
- https://velog.io/@linea77/ChatGPT-API-logit-bias-쓰는-법