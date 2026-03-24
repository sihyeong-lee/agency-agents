# Elvis AI노무사 채팅 샌드박스 디버그 핸드오프

## 1. 목적

AI Canvas 애플리케이션 안에서 `AI노무사` 메뉴를 **ChatGPT 스타일의 단일 채팅 화면**으로 동작시키는 것이 목표입니다.

원하는 UX:
- 하단 입력창에서 질문 입력
- 내 질문은 오른쪽 버블
- 답변은 왼쪽 버블
- 질문/답변이 한 화면에서 계속 누적

현재 문제:
- 질문 제출 시 **흰 화면으로 넘어가거나**
- 제출 이후 실제 답변이 표시되지 않음

---

## 2. 지금 기준으로 바뀐 구조

초기에는 샌드박스가 직접 워크플로우에 dataset을 보내는 구조를 썼습니다.

기존 시도:
```text
샌드박스(output) -> 3(dataset)
19B -> 샌드박스(answer_package)
19A -> 샌드박스(memo_package)
18M3 -> 샌드박스(merged_package)
```

이 구조는 배포된 페이지 샌드박스 환경에서 매우 불안정했고, 제출 시 흰 화면/중단 문제가 반복되었습니다.

그래서 현재는 아래 구조로 바뀌었습니다.

현재 구조:
```text
샌드박스_노무상담챗_Elvis
-> fetch(WORKFLOW_API_URL)
-> 워크플로우API배포_ElvisAI노무사
-> API데이터_ElvisAI노무사
-> 3. 에이전트 프롬프트_질의정규화
-> 4
-> 5
-> ...
-> 18M3
-> 19A
-> 19B
-> 19C. 파이썬_ElvisAPI응답패키징
-> API 응답
-> 샌드박스가 응답을 렌더링
```

즉, **현재 샌드박스는 input/output 포트를 쓰지 않고**, `fetch()`로 Workflow API Deploy URL을 직접 호출합니다.

---

## 3. 관련 핵심 파일

다른 AI에게는 아래 2개 파일을 같이 주면 됩니다.

1. `elvis_chat_debug_handoff.md`
2. `elvis_consult_chat_sandbox.md`

상황을 더 넓게 봐야 하면 추가로:
- `elvis_platform_manual.md`
- `manual.md`

하지만 지금 문제는 거의 샌드박스/응답처리 계층이므로, 우선 위 2개면 충분합니다.

---

## 4. 샌드박스 쪽 최근 변경사항

`elvis_consult_chat_sandbox.md` 기준으로 최근에 바뀐 중요한 내용:

### 4.1 직접 workflow dataset 전달 제거

삭제된 방향:
- `sendDataToOutput(payload)`
- `output -> 3(dataset)`
- `getDataset()` 기반 polling

추가된 방향:
- `fetch(WORKFLOW_API_URL, { method: "POST", body: ... })`

### 4.2 localStorage 의존 제거

이전에는 localStorage 기반으로 채팅 상태를 저장했습니다.

하지만 AI Canvas 샌드박스는 `null origin`/`srcdoc`/`blob:` 계열일 가능성이 있어 localStorage 접근이 불안정할 수 있다고 판단했습니다.

현재는 메모리 기반 상태를 사용합니다.

예:
```javascript
const STORE_KEY = "__elvisChatState";
window[STORE_KEY] = { messages: [], meta: {} };
```

### 4.3 pending 복구 로직 변경

이전에는 재실행 시 pending assistant 메시지를 너무 빨리 실패로 바꾸는 로직이 있었습니다.

현재는 그 공격적인 복구 로직을 제거/완화한 상태입니다.

### 4.4 fetch 타임아웃 제거

이전에는 샌드박스 fetch에 AbortController 기반 180초 타임아웃이 있었고,
`signal is aborted without reason` 에러가 났습니다.

현재는 그 타임아웃을 제거했습니다.

### 4.5 API 응답 파서 수정

AI Canvas Workflow API Deploy 응답은 단순 object가 아니라 중첩 구조로 옵니다.

실제 예:
```json
{
  "task_status": "SUCCESS",
  "result": [
    [
      {
        "node3_probe_ok": 1,
        "normalized_json": "...",
        "question_prompt": "..."
      }
    ]
  ]
}
```

그래서 `firstRowFromApiResponse()`를 수정해
- `result[0][0]`
- `data[0]`
- `rows[0]`
- column-array object
등을 재귀적으로 unwrap 하도록 바꿨습니다.

### 4.6 `callWorkflowApi()` 문법 오류 수정

기존 JS에는 `try { ... }` 블록이 비정상적으로 남아 있어서
샌드박스 제출 직후 깨질 가능성이 있었습니다.

현재는 `callWorkflowApi()`를 단순화해서 아래처럼 정리했습니다.
- `fetch()`
- `response.text()`
- `JSON.parse()`
- `response.ok` 검사
- 실패 시 `Error` throw

### 4.7 메뉴 이동 방식

AI Canvas 메뉴 이동은 문자열이 아니라 숫자를 쓰는 것으로 정리했습니다.

예:
```javascript
changeApplicationMenu(1) // HOME
changeApplicationMenu(2) // AI노무사
changeApplicationMenu(3) // 추후 개발
```

---

## 5. 현재 샌드박스가 기대하는 응답 형식

샌드박스는 최종적으로 아래 필드를 기대합니다.

주 응답:
- `output_response`

부가 메모:
- `legal_memo_json`

보조:
- `latest_user_question`
- `conversation_context`
- `query_class`

그래서 현재 워크플로우 끝에 `19C. 파이썬_ElvisAPI응답패키징` 노드를 추가해서,
이 필드들만 API 응답 패키지로 정리해 반환하도록 구성했습니다.

---

## 6. 현재 확인된 사실

### 확인 1: Workflow API Deploy 자체는 동작함

테스트용 API deploy URL을 호출했을 때 정상 응답이 왔습니다.

### 확인 2: API Data -> 3번 노드까지는 실제로 감

테스트 경로:
```text
API데이터_ElvisAI노무사_3검증
-> 3
-> 3B. 파이썬_3출력확인
-> 워크플로우API배포_ElvisAI노무사_3검증
```

이 경로에서 응답으로 아래가 확인됐습니다.

```json
{
  "task_status": "SUCCESS",
  "result": [[{
    "node3_probe_ok": 1,
    "normalized_json": "...",
    "question_prompt": "..."
  }]]
}
```

즉:
- API 경계는 정상
- API Data 노드 정상
- 3번 노드 실행 정상

### 확인 3: 따라서 지금 문제는 3번 이전보다 이후/또는 샌드박스 렌더 계층일 가능성이 큼

현재는 아래 둘 중 하나가 핵심 의심 지점입니다.

1. **메인 본선 워크플로우**의 `4 -> ... -> 19C` 어딘가가 느리거나 멈춤
2. 샌드박스가 응답을 받아도 렌더링 과정에서 다시 깨짐

---

## 7. 현재 증상

사용자 체감 증상:
- AI노무사 페이지에서 질문 입력
- 제출 직후 흰 화면처럼 보임
- 아무 답변이 안 나타남
- 재접속 시 pending/중단 안내 메시지가 뜨는 경우가 있었음

최근엔 샌드박스 JS를 많이 수정해서,
이제는 최소한 **문법 오류와 응답 파싱 구조 문제는 한번 정리된 상태**입니다.

---

## 8. 다른 AI에게 물어보고 싶은 핵심 질문

다른 AI에게는 아래 질문 위주로 검토 요청하면 좋습니다.

### 질문 1
현재 `elvis_consult_chat_sandbox.md`의 JS 구조에서,
질문 제출 직후 페이지가 하얗게 보일 수 있는 런타임 예외 포인트가 더 있는지 검토해 달라.

### 질문 2
AI Canvas Workflow API Deploy 응답 구조가 중첩 배열(`result[0][0]`)인 환경에서,
현재 `firstRowFromApiResponse()`보다 더 안전한 파서가 있는지 검토해 달라.

### 질문 3
`fetch(WORKFLOW_API_URL)` 기반 구조에서,
답변이 매우 느릴 때도 페이지가 깨지지 않도록 하는 가장 안정적인 UI 상태관리 방식이 무엇인지 제안해 달라.

### 질문 4
현재 구조에서 흰 화면이 계속 난다면,
그 원인이 샌드박스 JS인지 아니면 본선 워크플로우(`4 ~ 19C`)인지 가장 빨리 분리할 수 있는 방법을 제안해 달라.

---

## 9. 다른 AI에게 함께 전달하면 좋은 코멘트

아래 문장을 그대로 같이 전달해도 됩니다.

```text
현재 AI Canvas 앱에서 `AI노무사` 채팅 페이지를 만들고 있습니다.
초기에는 sandbox output -> workflow dataset 직결 구조를 썼지만, 배포 환경에서 흰 화면과 중단 문제가 반복되어 현재는 Workflow API Deploy + fetch 구조로 바꿨습니다.

`elvis_consult_chat_sandbox.md`는 현재 샌드박스에 넣는 실제 HTML/CSS/JS 문서입니다.
`elvis_chat_debug_handoff.md`는 최근 변경사항과 현재 상태를 요약한 문서입니다.

이미 테스트용 API 경로에서 `API데이터 -> 3`까지는 정상 실행됨을 확인했습니다.
그래서 지금은 샌드박스 JS가 여전히 깨지는지, 아니면 본선 워크플로우(`4 ~ 19C`)가 느리거나 멈추는지를 좁히고 싶습니다.

이 전제에서 샌드박스 JS 구조와 응답 처리 방식을 검토해 주세요.
```

---

## 10. 지금 시점의 한 줄 정리

`3`까지는 실제로 API로 검증됐고, 지금은 **샌드박스 JS 안정화 + 본선 응답 처리**가 핵심입니다.
