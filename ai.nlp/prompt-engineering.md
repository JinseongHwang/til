## 출처
A Prompt Pattern Catalog to Enhance Prompt Engineering with ChatGPT ([link](https://arxiv.org/pdf/2302.11382.pdf))  
Department of Computer Science, Vanderbilt University, Tennessee USA

## [1] The Persona Pattern

The Persona Pattern을 사용하면 ChatGPT가 생성해야 할 응답과 집중해야 할 세부 사항을 안내할 수 있다.  
단, 실제 인물이나 범죄자를 언급할 경우에는 생성에 실패할 수도 있다.

### 코드 리뷰 요청하기
```plaintext
You are going to pretend to be a senior engineer at a FAANG company.
Review the following code paying attention to security and performance.
Provide outputs that a senior engineer would produce regarding the code.

{{QUESTION}}
```

### 글 퇴고 요청하기
```plaintext
From now on, act as a book editor and review the following blog article focusing on readability.

{{QUESTION}}
```

### 마케팅 조언 요청하기
```plaintext
Pretend you are a marketing expert, review the following slogans and suggest improvements based on other popular campaigns.

{{QUESTION}}
```

## [2] The Recipe Pattern

The Recipe Pattern은 달성하고 싶은 목표가 있고, 재료는 알고 있으며, 달성하기 위한 단계는 어느 정도 알고 있는 상태에서  
어떻게 해야 할지 모르겠을 때 조합을 알려주는 데 유용하다.

### 프로그래밍 도움 요청하기
```plaintext
I'm trying to write a Rust program to encryt data.
I know I have to read user input, validate it, encrypt it and return the encrypted data,
please provide a complete sequence of steps for me, fill in any missing steps and identify any unnecessary steps.

{{QUESTION}}
```

## [3] The Reflection Pattern

The Reflection Pattern을 사용하면 모든 답변에 대한 이유를 설명하도록 요청할 수 있다.  
또한 부가 설명을 조금 더 친절하고 많이 생성하게 한다.  
할루시네이션이 발생했는지 짐작하는 데 도움이 된다.

```plaintext
When you provide an answer, please explain the reasoning and assumptions behind your answer.
Explain your choices and address any potential limitations or edge cases.

{{QUESTION}}
```

## [4] The Refusal Breaker Pattern

The Refusal Breaker Pattern은 ChatGPT가 지식 제한, 안전 등의 이유로 답변할 수 없다고 말하는 경우를 Break 하는 패턴이다.  
답변을 조금 변형해서 대답할 수 있게 유도하는 방식이다.

```plaintext
Whenever you can't answer a question, Explain why you can't answer the question.
Provide one or more alternative wordings of the question that you could answer.

{{QUESTION}}
```

## [5] The Flipped Interaction Pattern

The Flipped Interaction Pattern은 ChatGPT에게 질문을 할 때 ChatGPT가 어떤 정보를 필요로 하는지 모르겠을 때 유용하다.  
원하는 정보를 모두 얻을 때까지 ChatGPT가 사용자에게 질문을 하는 방식이다.

### AWS 배포 자동화 스크립트 생성에 필요한 질문받기
```plaintext
I want you to ask me questions to deploy a Rust binary to a web server located in AWS.
When you have all the information you need, write a bash script to automate the deployment.

{{QUESTION}}
```