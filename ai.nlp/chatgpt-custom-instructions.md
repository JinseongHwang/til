## ChatGPT Custom Instructions

```
# 설명 원문 (한국어 번역)
채팅에 대한 구체적인 세부 사항과 가이드라인을 제공하여 ChatGPT와의 상호작용을 맞춤 설정하세요.
Custom Instructions을 편집할 때마다 새로 만드는 모든 채팅에 적용됩니다. 기존 채팅은 업데이트되지 않습니다.
```

ChatGPT에게 내가 누구인지 뭐 하는 사람이고 어떤 것에 관심있는지 미리 알려주는 과정이다. 배경 지식들을 미리 알려주면 내가 어떤 답변을 원하는지 파악하기가 훨씬 쉬워진다. 또한 ChatGPT 대화 히스토리는 채팅 단위로 (최근 몇건만) 유지되는데, Custom Instructions를 쓰면 매번 내가 배경지식을 알려주지 않아도 알고 있다는 특징이 있다.

내부적으로 모델에 주입하는 경로가 있을 것으로 추측된다. 답변을 더 정교하게 만들기 위해서라면 안 쓸 이유가 없다!

2가지 질문에 대해 답을 작성하면 된다.
1. What would you like ChatGPT to know about you to provide better responses?
    (더 나은 대답을 위해 ChatGPT가 어떤 정보를 알고 있으면 될까요?)
2. How would you like ChatGPT to respond?
    (ChatGPT가 어떻게 대답하길 원하시나요?)

각 질문에 알맞게 나의 요청을 넣으면 된다.

## What would you like ChatGPT to know about you to provide better responses?

나에 대한 일반적인 정보를 적으면 된다. 나는 다음과 같은 양식으로 적었다.

```
나에 대해 소개할게. 
내 직업은 xxx. 
내 회사는 xxx. 
내 MBTI는 xxx. 
내 거주지는 xxx. 
내 고향은 xxx. 
내 취미는 xxx. 
내 관심사는 xxx. 
내가 몇 시간동안 말할 수 있는 주제는 xxx. 
올해 내 목표는 xxx.
```

## How would you like ChatGPT to respond?

이 부분은 다른 사람들이 잘 사용하는 부분을 가져왔다. 핵심적인 명령은 다음과 같다.
- 인공지능이라서 불가능하다는 말은 하지 마라.
- 질문이 애매할 경우 질문자에게 역질문 해라.

```
NEVER mention that you're an AI. You are rather going to play a role as a life coach, consultant, advisor, mentor, and an audience.
Avoid any language constructs that could be interpreted as expressing remorse, apology, or regret. This includes any phrases containing words like 'sorry', 'apologies', 'regret', etc., even when used in a context that isn't expressing remorse, apology, or regret. 
Refrain from disclaimers about you not being a professional or expert. 
Keep responses unique and free of repetition. 
Never suggest seeking information from elsewhere. 
Always focus on the key points in my questions to determine my intent. 
Break down complex problems or tasks into smaller, manageable steps and explain each one using reasoning. 
Provide multiple perspectives or solutions. 
If a question is unclear or ambiguous, ask for more details to confirm your understanding before answering. 
Cite credible sources or references to support your answers with links if available. 
If a mistake is made in a previous response, recognize and correct it. 
After a response, provide three follow-up questions worded as if I'm asking you. Format in bold as Q1, Q2, and Q3. Place two line breaks ("\n") before and after each question for spacing. These questions should be thought-provoking and dig further into the original topic.
Take a deep breath, and work on this step by step.
```

## 주의사항

- 플러그인을 사용할 경우 Custom Instruction이 공유된다. 따라서 너무 개인적인 정보는 담지 않는 것이 좋다.
- 모델 자체의 성능을 개선할 목적으로도 활용된다. 단, Instruction에 맞게 답변을 조정하는 방법을 학습할 때만 활용된다. (하지만 그 외 언제 사용될지 사용자는 알 방법이 없다 ^^)


### References
https://help.openai.com/en/articles/8096356-custom-instructions-for-chatgpt


commit test