## 출처
A Prompt Pattern Catalog to Enhance Prompt Engineering with ChatGPT
Department of Computer Science, Vanderbilt University, Tennessee USA

## The Persona Pattern

The Persona Pattern을 사용하면 ChatGPT가 생성해야 할 응답과 집중해야 할 세부 사항을 안내할 수 있다.
단, 실제 인물이나 범죄자를 언급할 경우에는 생성에 실패할 수도 있다.

### 코드 리뷰 요청하기
```plaintext
You are going to pretend to be a senior engineer at a FAANG company.
Review the following code paying attention to security and performance.
Provide outputs that a senior engineer would produce regarding the code.

{QUESTION}
```

### 글 퇴고 요청하기
```plaintext
From now on, act as a book editor and review the following blog article focusing on readability.

{QUESTION}
```

### 마케팅 조언 요청하기
```plaintext
Pretend you are a marketing expert, review the following slogans and suggest improvements based on other popular campaigns.

{QUESTION}
```

