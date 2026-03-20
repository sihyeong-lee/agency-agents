# Elvis 브랜드 / UI 핸드오프

이 문서는 Claude 또는 디자이너가 `Elvis` 플랫폼의 시각 스타일을 입힐 때 참고하는 짧은 핸드오프 문서다.

구조와 노드 연결은 아래 문서를 우선으로 본다.

- [elvis_platform_manual.md](d:\AI canvas\새 폴더\elvis_platform_manual.md)
- [sandbox_ui_manual.md](d:\AI canvas\새 폴더\sandbox_ui_manual.md)
- [elvis_platform_home_sandbox.md](d:\AI canvas\새 폴더\elvis_platform_home_sandbox.md)

---

## 1. 브랜드 한 줄

Elvis는 **회사의 지식을 모아두고, 필요한 순간 현명하게 꺼내 쓰는 플랫폼**이다.

`el = entire`, `vis = wise`

즉 감정적인 법률 챗봇 느낌보다,
**차분하고 믿을 수 있고, 정보가 정돈된 지식 플랫폼** 느낌이 중요하다.

---

## 2. 시각 방향

원하는 방향:

1. `플랫폼`처럼 보여야 한다.
   - 단일 챗봇 사이트처럼 보이면 안 된다.
   - 홈 화면이 있고, 메뉴 카드가 있고, 지식 허브 같은 분위기가 나야 한다.

2. `전문적이지만 차갑지 않은 톤`
   - 법률/규정/사례를 다루므로 신뢰감이 있어야 한다.
   - 너무 병원처럼 차갑거나, 너무 핀테크처럼 공격적이면 안 된다.

3. `정돈된 밀도`
   - 답변과 근거가 길더라도 읽기 쉽게 카드 / 섹션 / 간격이 정리되어야 한다.
   - 정보량이 많은 서비스라는 점이 장점으로 보여야 한다.

4. `브랜드 자산 활용`
   - 로고: `image/elvis-mark.svg`
   - 히어로 이미지: `image/elvis-hero.svg`

---

## 3. 피해야 하는 방향

1. 보라색 SaaS 템플릿 느낌
2. AI 채팅 서비스 복제 느낌
3. 지나치게 장난스럽거나 캐릭터 중심인 톤
4. 법률 서비스인데 카지노/게임 UI처럼 보이는 강한 네온 계열
5. 회색 표만 가득한 무미건조한 내부 시스템 느낌

---

## 4. 추천 키워드

- knowledge platform
- legal intelligence
- calm authority
- collected wisdom
- structured depth
- evidence first
- warm navy
- parchment neutral
- archive meets modern

---

## 5. 컬러 방향

권장 무드:

- deep navy
- ivory / paper beige
- muted gold accent
- light stone gray

예시 톤:

- 네이비: `#173B63`
- 라이트 네이비: `#2F6A9C`
- 아이보리: `#F7F4EC`
- 샌드: `#EDE3D2`
- 골드 포인트: `#B98B34`
- 텍스트: `#17212B`

---

## 6. 메뉴 인식 방식

메뉴는 최소 아래 5개가 보이면 좋다.

1. 홈
2. 노무상담
3. 근거
4. 지식문서
5. 작성도구

초기 MVP는 `노무상담`, `근거`만 실제로 동작해도 되지만,
화면상으로는 `Elvis`가 **앞으로 확장 가능한 플랫폼**처럼 보여야 한다.

---

## 7. 홈 화면이 주어야 하는 인상

홈 화면은 사용자에게 아래 메시지를 줘야 한다.

1. 여기는 챗봇 하나만 있는 곳이 아니다.
2. 회사의 규정, 사례, 판례, 문안을 활용하는 플랫폼이다.
3. 가장 먼저 쓸 수 있는 기능은 `노무상담`이다.
4. 답변은 근거와 함께 확인할 수 있다.

---

## 8. 답변 화면이 주어야 하는 인상

`노무상담` 메뉴는 아래처럼 보여야 한다.

1. 질문 입력 영역은 명확해야 한다.
2. 답변은 읽기 쉬운 문단 구조여야 한다.
3. 핵심 결론은 시각적으로 먼저 보여야 한다.
4. 추가 필요정보가 있으면 별도 카드처럼 보여야 한다.
5. `근거` 메뉴와 연결된 서비스라는 느낌이 나야 한다.

---

## 9. 근거 화면이 주어야 하는 인상

`근거` 메뉴는 아래처럼 보여야 한다.

1. 공식 근거
2. 내부 근거
3. 내부 유사 사례
4. 웹 보완 링크

를 구분해서 보여줘야 한다.

즉 “자료를 잔뜩 붙여 놓은 화면”이 아니라,
**답변을 믿을 수 있게 만드는 증거 정리 화면**이어야 한다.

---

## 10. 원하는 레퍼런스 톤

원하는 기본 인상은 **OpenAI Platform / OpenAI Docs / ChatGPT workspace 계열의 정돈되고 세련된 UI**다.

가져오고 싶은 요소:

1. 과장되지 않은 고급스러움
   - 과한 그라데이션, 네온, 입체 효과보다는 절제된 완성도
2. 넓은 여백과 읽기 쉬운 밀도
   - 카드와 섹션 간 간격이 충분하고 답답하지 않아야 함
3. 정돈된 계층 구조
   - 헤더, 섹션, 보조설명, 메타정보의 위계가 분명해야 함
4. 정보 중심의 전문 툴 인상
   - “예쁜 마케팅 페이지”가 아니라 실제 업무용 플랫폼처럼 보여야 함
5. 긴 답변도 버티는 화면
   - 답변, 근거, 링크, 사례가 길어져도 레이아웃이 무너지면 안 됨

주의:

- OpenAI 화면을 그대로 복제하는 건 원하지 않는다.
- 참고하는 것은 “분위기와 완성도”이지, 색상/배치/컴포넌트의 직접 복사는 아니다.
- Elvis는 법률/지식 플랫폼이므로 OpenAI보다 약간 더 차분하고 문서 친화적이어야 한다.

---

## 11. Claude에게 반드시 전달할 파일

Claude에는 아래 파일만 주면 충분하다.

### 꼭 줘야 하는 파일

1. [elvis_claude_handoff.md](d:\AI canvas\새 폴더\elvis_claude_handoff.md)
- 브랜드 방향
- 시각 톤
- 피해야 할 스타일

2. [elvis_platform_manual.md](d:\AI canvas\새 폴더\elvis_platform_manual.md)
- 전체 메뉴 구조
- 페이지 노드 구조
- 공개용/디버그용 역할 구분

3. [elvis_platform_home_sandbox.md](d:\AI canvas\새 폴더\elvis_platform_home_sandbox.md)
- 홈 화면 샌드박스 코드

4. [elvis_consult_input_sandbox.md](d:\AI canvas\새 폴더\elvis_consult_input_sandbox.md)
- 노무상담 질문 입력 샌드박스 코드

5. [sandbox_ui_manual.md](d:\AI canvas\새 폴더\sandbox_ui_manual.md)
- `샌드박스_답변뷰어`
- `샌드박스_근거뷰어`

### 같이 주면 좋은 파일

6. [elvis-mark.svg](d:\AI canvas\새 폴더\image\elvis-mark.svg)
7. [elvis-hero.svg](d:\AI canvas\새 폴더\image\elvis-hero.svg)

### 굳이 안 줘도 되는 파일

- [manual.md](d:\AI canvas\새 폴더\manual.md)
  - 너무 길어서 Claude가 스타일 작업할 때 오히려 산만해질 수 있음
  - 구조 확인이 정말 더 필요할 때만 추가

---

## 12. Claude에게 꼭 같이 설명해야 하는 것

아래 설명을 같이 주는 게 좋다.

1. Elvis는 단일 챗봇이 아니라 **플랫폼 사이트**다.
2. `노무상담`은 여러 메뉴 중 하나다.
3. 지금 원하는 건 **스타일 개선**이지, 노드 구조 재설계가 아니다.
4. AI Canvas에 붙일 것이므로 **HTML/CSS/JS 안의 구조와 함수 이름을 함부로 바꾸면 안 된다.**
5. 질문 입력 샌드박스는 이미 작동 중이므로 **이벤트/실행 로직을 깨지 말아야 한다.**
6. 답변/근거 뷰어도 입력 포트 이름과 컬럼 계약을 유지해야 한다.

즉 Claude는:

- 화면 디자인
- 레이아웃
- 카드/탭/상태 배지
- 색상/타이포/간격
- 모바일 대응

까지만 맡고,

- 노드 연결
- input/output 계약
- JS 실행 로직

은 최대한 보존해야 한다.

---

## 13. Claude에게 주면 좋은 요청 문구

아래 문구를 거의 그대로 줘도 된다.

```text
Elvis는 회사의 규정, 사례, 판례, 의견서를 연결해 실무 지식을 찾아 쓰는 플랫폼입니다.
노무상담은 그중 하나의 메뉴입니다.

원하는 방향은 OpenAI Platform처럼 세련되고 정돈된 업무용 UI입니다.
다만 그대로 복제하는 것은 원하지 않고, 차분하고 문서 친화적인 Elvis만의 톤으로 재해석해 주세요.

아래 파일을 기준으로 스타일만 고쳐 주세요.

1. elvis_claude_handoff.md
2. elvis_platform_manual.md
3. elvis_platform_home_sandbox.md
4. elvis_consult_input_sandbox.md
5. sandbox_ui_manual.md

중요:
- 구조와 노드 연결은 바꾸지 말아 주세요.
- 입력 포트 이름, 출력 컬럼 계약, JS 실행 함수는 유지해 주세요.
- 질문 입력 샌드박스는 이미 작동하고 있으니 실행 로직을 깨지 말고, 화면만 더 고급스럽게 만들어 주세요.

특히 아래 3개 화면을 다듬고 싶습니다.
- 홈
- 노무상담 입력/답변 화면
- 근거 화면

원하는 느낌:
- calm authority
- knowledge platform
- evidence first
- spacious but dense
- navy / ivory / muted gold
- long-form answer friendly

피하고 싶은 느낌:
- 보라색 SaaS 템플릿
- 과한 AI 챗봇 복제 UI
- 장난스럽거나 네온 계열
- 내부 시스템처럼 투박한 회색 표 화면

HTML/CSS/JS를 AI Canvas 샌드박스에 바로 붙일 수 있게 결과를 주세요.
가능하면 각 파일별로 교체된 HTML / CSS / JS를 분리해서 주세요.
```

---

## 14. Claude 작업 범위 가이드

Claude에게 기대하는 산출물:

1. `샌드박스_Elvis_홈` 리디자인
2. `샌드박스_질문입력_Elvis` 리디자인
3. `샌드박스_답변뷰어` 리디자인
4. `샌드박스_근거뷰어` 리디자인
5. 모바일 화면까지 버티는 CSS 정리

Claude에게 기대하지 않는 것:

1. 검색 파이프라인 수정
2. AI Canvas 노드 구조 재배치
3. `runNode`, `sendDataToOutput`, `getDataset` 계약 변경
4. 입력 포트 이름 변경
5. 노드명 변경
