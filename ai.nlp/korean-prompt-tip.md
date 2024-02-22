## 글자수를 인식하지 못하는 상황

### 이유

대부분의 언어 모델의 input과 output의 가장 작은 단위는 토큰이다. 영어 기준으로 봤을 땐, 띄어쓰기 == 토큰이라고 봐도 무방하다.

하지만 한국어는 쉽게 나뉘어지지 않는다. 예를 들어, "pizza"는 1 토큰인데, "피자"는 4 토큰이다. "호박고구마"는 7토큰이다. 딱히 규칙은 없다. 토큰 단위로 과금을 하기도 하기 때문에 토큰을 카운트하는 것은 굉장히 중요하다고 볼 수 있다.

### 해결 팁1

재귀적으로 계속 호출해보는 것이다.

1000자의 게시글과 함께 "100-150자로 요약해줘"라는 명령을 했다고 가정하자. 원하는 결과를 얻을 때까지 계속 반복한다. 단, 본문 컨텍스트를 계속 가지고 다녀야 자연스러운 결과물을 얻을 수 있다.

대충 아래처럼 코드를 작성할 수 있다.

```python
def 요약하기(본문, 요약본="", 길이조절="SHORTER") -> str:
    요약본 = 생성하기(본문, 요약본, 길이조절)

    if (len(요약본) < 100):
        return 요약하기(본문, 요약본, "LONGER")

    elif len(요약본) > 150:
        return 요약하기(본문, 요약본, "SHORTER")

    return 요약본

def 생성하기(본문, 요약본, 길이조절):
    if 길이조절 == "LONGER":
        프롬프트 = "더 길게 요약해줘~"
    
    elif 길이조절 == "SHORTER":
        프롬프트 = "더 짧게 요약해줘~"

    프롬프트 += f"{본문}이랑 {요약본} 참고해~"

    return openai.gpt(프롬프트)
```

### 해결 팁1 - 장단점

장점:
- 언젠간 원하는 결과를 얻을 수 있다.

단점:
- 하나의 결과물을 내기 위해 훨씬 많은 토큰이 요구된다. 따라서 비싸다.
- 여러번 거치기 때문에 top_p, temperature가 높게 설정되어 있는 경우라면 결과물 편차가 커진다.

### 해결 팁2

```
[문서]
{여기에 요약하고자 하는 텍스트 입력}

[요청]
Summarize the above document in between 100 and 150 characters. Then count the characters in the summarized result and make sure it's between 100 and 150 characters. If it's not, summarize it again and print it out if it's between 100 and 150 characters.
```
