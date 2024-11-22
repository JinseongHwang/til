# Midjourney

## 목표

베이스 이미지를 내가 원하는 이미지로 변화시키기
참고: https://docs.midjourney.com/docs/discord-quick-start

## 디스코드로 사용하기

### 1. /setting 명령으로 설정 변경하기

![image](https://github.com/user-attachments/assets/02691ed0-3ad8-40e4-8e96-e98707b14009)

- 기본 설정에서 remix mode와 turbo mode를 활성화했다.

### 2. /describe 명령으로 베이스 이미지를 프롬프트화 하기

![image](https://github.com/user-attachments/assets/1950bc9f-5365-43d7-8d46-0253e11208d0)

4개의 프롬프트 중 괜찮아 보이는 것을 선택한다.

### 3. 프롬프트를 내가 원하는 모양으로 변경하기

![image](https://github.com/user-attachments/assets/753d1961-a39e-4c62-82d8-f4cf386bf7aa)

우선 ChatGPT의 GPTs에서 미드져니 전용 익스텐션을 사용한다.

```plain text
A young Japanese man in his mid-20s sits at an Apple computer and types on the keyboard, with boxes of goods behind him. 
The entire scene is illuminated by soft lighting. 
A white bar appears above his head as if it were data being displayed, adding to its visual appeal. 
This cinematic composition creates a cozy atmosphere while highlighting each character's unique personality through facial expressions and body language.

------------

위 프롬프트의 일반인 남성을 한국 여성으로 변경해줘.
관련된 프롬프트의 모든 항목을 함께 변경해줘.
총 세개 세트의 프롬프트를 제공해줘.
각 프롬프트가 모두 실사느낌이 났으면 좋겠어.
각 세트의 프롬프트는 각각 다른 카메라 기종을 사용해줘.
세트 구성은: 한글 프롬프트 + 영문 프롬프트 + 왜 그런 프롬프트를 썼는지 설명해줘.
```

내가 선택한 프롬프트와 변경하고자 하는 요구사항을 함께 작성해서 요청하자.
수정된 프롬프트 3개를 얻을 수 있다.

### 4. /imagine 명령과 변경된 프롬프트로 이미지 생성하기

![image](https://github.com/user-attachments/assets/dd615d14-ca74-4f9a-9144-a6d455a8c6f5)

계속해서 프롬프트를 바꿔가며 내가 원하는 것이 나올 때까지 반복한다.
원하는 느낌이 나왔다면 U1~4를 선택해서 upscailing 가능하고, 원하는 느낌이지만 조금 더 보고싶다면 V1~4를 선택해서 variation을 볼 수 있다 

![image](https://github.com/user-attachments/assets/96edbdde-29ca-49ed-ad1d-4ba95b7f4c29)

최종적으로 원하는 결과물을 선택할 수 있다.

### 5. 일부만 재생성 하는 방법 : Vary(Region)

![image](https://github.com/user-attachments/assets/2c81b02e-efdb-4742-8760-7d90e2c1d521)

박스 혹은 올가미로 내가 원하는 모습으로 변경할 수 있다.
- --cref 명령으로 원하는 사람의 모습으로 변경 가능하다.
- --ar 로 종횡비율 조절이 가능하며, 없애면 1:1 이다. 

예시)

```Plain Text
A Korean woman in her mid-20s working at an Apple computer. 
Her face is angled about 15 degrees downward, focusing on her work with a calm and confident expression. 
Behind her, a panoramic view of Seoul’s Han River from a high-rise apartment window is complemented 
by elegant decorative lighting within the room, creating a warm and sophisticated ambiance. 
Soft natural light from the window interacts with the decorative lights, enhancing the cozy and vibrant mood. 
A white bar floats above her head, adding a visual element. 
Ultra-high resolution, vivid details, Sony’s signature realistic color rendering, 16-bit RAW quality 
--cref https://img.khan.co.kr/news/2023/05/12/news-p.v1.20230512.e5fffd99806f4dcabd8426d52788f51a_P1.png
```

### 6. 비율 늘려서 배너 형태로 만들기

![image](https://github.com/user-attachments/assets/187efa48-de59-4a48-ad6a-38d4c38a125d)

웹 에디터로 접근하면 테두리를 드래그해서 비율 늘리기가 가능하다.

![image](https://github.com/user-attachments/assets/e605b2a7-1810-4507-94ba-46828a8edb50)

나는 우측 상단에 Edit prompt에 다음과 같이 작성했다.

```Plain Text
A grand, high-ceiling living room with classical design elements, soft velvet furniture, and a vast window overlooking a dreamy winter landscape with snow-covered mountains and tall pine trees. 
Cool, soft light permeates the room, creating a cozy and elegant ambiance. 
Created Using: high-detail rendering engines, HDR cold lighting setup, baroque interior design influence, winter color palette, luxurious material texturing, hd quality, natural look 
--v 6.1
```

위 조명이 어색하지만 프롬프트 수정 및 생성을 반복하면 충분히 개선할 수 있다.