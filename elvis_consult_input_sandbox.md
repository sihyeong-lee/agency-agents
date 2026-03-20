# Elvis 노무상담 입력 샌드박스 코드

이 문서는 `Elvis` 플랫폼의 **공개용 노무상담 메뉴**에서 사용할
`샌드박스_질문입력_Elvis` 노드용 코드 문서다.

핵심 방향:

- 최종 공개용 Elvis에서는 `에이전트 UI`를 직접 노출하지 않는 것을 기본으로 한다.
- 대신 `샌드박스_질문입력_Elvis`가 질문 입력 UI를 맡는다.
- 즉 이 샌드박스는 **에이전트가 아니라 입력 프런트**이고, 실제 에이전트 역할은 뒤의 `에이전트 프롬프트` 노드들이 수행한다.
- 이 샌드박스는 사용자의 질문을 `latest_user_question`, `conversation_context` 형태로 출력한다.
- 출력 데이터는 기존 런타임 그래프의 `3. 에이전트 프롬프트_질의정규화`로 넣는다.
- 디버그와 검증을 위해 `에이전트_법률상담입력`은 내부 테스트용으로만 남겨둘 수 있다.

---

## 1. 노드명과 연결 구조

노드명:

- `샌드박스_질문입력_Elvis`

노드 타입:

- `샌드박스`

배치 위치:

- `페이지_Elvis_노무상담` 내부

입력 포트:

- 없음

출력 포트:

- 기본 출력 1개 사용

출력 데이터 스키마:

```text
latest_user_question
conversation_context
ui_request_id
submitted_at
```

권장 연결:

```text
샌드박스_질문입력_Elvis(output)
-> 3. 에이전트 프롬프트_질의정규화(dataset)
```

즉 이 샌드박스는 `2C`와 비슷한 역할을 한다고 이해하면 된다.

---

## 2. 같이 배치할 노드

`페이지_Elvis_노무상담` 안에는 아래 2개를 둔다.

1. `샌드박스_질문입력_Elvis`
2. `샌드박스_답변뷰어`

추가로 내부 테스트용으로만 아래를 남겨도 된다.

3. `에이전트_법률상담입력_디버그`

운영 권장:

- 공개 메뉴에서는 `샌드박스_질문입력_Elvis` 사용
- `에이전트_법률상담입력_디버그`는 숨기거나 별도 내부 메뉴에만 둠

---

## 3. HTML

```html
<div class="consult-shell">
  <section class="consult-hero">
    <div class="consult-hero__label">Elvis · Labor Counsel</div>
    <h1 class="consult-hero__title">노무상담</h1>
    <p class="consult-hero__desc">
      상황을 입력하면 내부 사례, 규정, 공식 근거, 웹 보완을 종합해 답변합니다.
    </p>
  </section>

  <section class="composer-card">
    <label class="composer-label" for="questionInput">질문 입력</label>
    <textarea
      id="questionInput"
      class="composer-textarea"
      placeholder="예: 직장 내 괴롭힘 신고자를 다른 부서로 전보한 게 불리한 처우가 될 수 있어?"
      onkeydown="window.__elvisHandleQuestionKeydown && window.__elvisHandleQuestionKeydown(event)"
    ></textarea>

    <div class="composer-actions">
      <button type="button" id="submitQuestion" class="btn btn--primary" onclick="window.__elvisSubmitQuestion && window.__elvisSubmitQuestion()">질문 실행</button>
      <button type="button" id="clearConversation" class="btn btn--secondary" onclick="window.__elvisClearConversation && window.__elvisClearConversation()">대화 초기화</button>
    </div>
  </section>

  <section class="history-card">
    <div class="history-head">
      <div class="history-title">최근 대화 맥락</div>
      <div id="historyMeta" class="history-meta">기록 없음</div>
    </div>
    <div id="historyList" class="history-list"></div>
  </section>

  <section id="statusCard" class="status-card hidden">
    <div class="status-title">실행 상태</div>
    <div id="statusText" class="status-text">질문을 실행 중입니다.</div>
  </section>
</div>
```

---

## 4. CSS

```css
:root {
  --bg: #f7f4ed;
  --card: #ffffff;
  --line: #ddd5c8;
  --text: #17212b;
  --sub: #677381;
  --navy: #163b63;
  --navy-soft: #e8eff7;
  --sand: #efe5d5;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Segoe UI", "Noto Sans KR", sans-serif;
  color: var(--text);
  background: linear-gradient(180deg, #faf8f3 0%, #f0ebdf 100%);
}

.consult-shell {
  max-width: 980px;
  margin: 0 auto;
  padding: 24px 16px 36px;
}

.consult-hero {
  margin-bottom: 18px;
}

.consult-hero__label {
  display: inline-flex;
  align-items: center;
  padding: 6px 10px;
  border-radius: 999px;
  background: var(--navy-soft);
  color: var(--navy);
  font-size: 12px;
  font-weight: 700;
}

.consult-hero__title {
  margin: 12px 0 8px;
  font-size: 30px;
  line-height: 1.15;
}

.consult-hero__desc {
  margin: 0;
  color: var(--sub);
  line-height: 1.7;
  font-size: 14px;
}

.composer-card,
.history-card,
.status-card {
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 20px;
  padding: 18px;
  box-shadow: 0 10px 30px rgba(20, 20, 20, 0.05);
}

.history-card,
.status-card {
  margin-top: 14px;
}

.composer-label,
.history-title,
.status-title {
  display: block;
  margin-bottom: 10px;
  font-size: 13px;
  font-weight: 700;
  color: var(--sub);
}

.composer-textarea {
  width: 100%;
  min-height: 180px;
  resize: vertical;
  border: 1px solid #d2d9e1;
  border-radius: 16px;
  padding: 16px;
  font: inherit;
  line-height: 1.7;
  background: #fbfcfe;
}

.composer-actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-top: 14px;
}

.btn {
  appearance: none;
  border: none;
  border-radius: 999px;
  padding: 12px 18px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  pointer-events: auto;
  touch-action: manipulation;
}

.btn--primary {
  background: var(--navy);
  color: #fff;
}

.btn--secondary {
  background: var(--sand);
  color: #4d4027;
}

.history-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 10px;
}

.history-meta {
  color: var(--sub);
  font-size: 12px;
}

.history-list {
  display: grid;
  gap: 10px;
}

.history-item {
  border: 1px solid #e7e2d8;
  border-radius: 14px;
  padding: 12px 14px;
  background: #fcfbf8;
}

.history-item__role {
  display: inline-block;
  margin-bottom: 6px;
  font-size: 11px;
  font-weight: 800;
  color: var(--navy);
  letter-spacing: 0.04em;
}

.history-item__text {
  white-space: pre-wrap;
  line-height: 1.65;
  font-size: 14px;
}

.status-text {
  color: var(--sub);
  line-height: 1.6;
}

.hidden {
  display: none;
}

@media (max-width: 640px) {
  .consult-shell {
    padding: 18px 12px 28px;
  }

  .consult-hero__title {
    font-size: 26px;
  }

  .composer-textarea {
    min-height: 160px;
  }
}
```

---

## 5. JavaScript

```javascript
const STORAGE_KEY = "elvis_labor_conversation_v1";
const RUN_START_NODE_ID = "node-agenticPrompt-ddd4ac2bd03bbc3fb7df5e";
const ANSWER_NODE_ID = "node-agenticPrompt-1795e50163097569438964";
let isSubmitting = false;

function isConfiguredNodeId(value) {
  const text = String(value || "").trim();
  if (!text) return false;
  if (text.startsWith("__SET_")) return false;
  return true;
}

function getConversation() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    const parsed = raw ? JSON.parse(raw) : [];
    return Array.isArray(parsed) ? parsed : [];
  } catch (e) {
    return [];
  }
}

function setConversation(items) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(items || []));
}

function firstRowsToText(items) {
  return (items || [])
    .map((item) => `${item.role}: ${item.content}`)
    .join("\\n");
}

function renderHistory() {
  const items = getConversation();
  const root = document.getElementById("historyList");
  const meta = document.getElementById("historyMeta");
  if (!root || !meta) return;

  root.innerHTML = "";
  if (items.length === 0) {
    meta.textContent = "기록 없음";
    const empty = document.createElement("div");
    empty.className = "history-item";
    empty.innerHTML = '<div class="history-item__text">아직 대화 기록이 없습니다.</div>';
    root.appendChild(empty);
    return;
  }

  meta.textContent = `${items.length}건`;
  items.slice(-6).forEach((item) => {
    const box = document.createElement("div");
    box.className = "history-item";
    box.innerHTML = `
      <div class="history-item__role">${item.role}</div>
      <div class="history-item__text"></div>
    `;
    box.querySelector(".history-item__text").textContent = item.content;
    root.appendChild(box);
  });
}

function showStatus(text) {
  const card = document.getElementById("statusCard");
  const line = document.getElementById("statusText");
  if (!card || !line) return;
  card.classList.remove("hidden");
  line.textContent = text;
}

function hideStatus() {
  const card = document.getElementById("statusCard");
  if (card) card.classList.add("hidden");
}

async function submitQuestion() {
  if (isSubmitting) return;

  const input = document.getElementById("questionInput");
  const submitButton = document.getElementById("submitQuestion");
  const text = String(input?.value || "").trim();
  if (!text) {
    alert("질문을 입력해주세요.");
    return;
  }

  if (!isConfiguredNodeId(RUN_START_NODE_ID)) {
    showStatus("질문 실행 노드 ID가 아직 설정되지 않았습니다. `3. 에이전트 프롬프트_질의정규화`의 nodeId를 넣어주세요.");
    alert("RUN_START_NODE_ID에 `3. 에이전트 프롬프트_질의정규화`의 실제 nodeId를 넣어주세요.");
    return;
  }

  isSubmitting = true;
  if (submitButton) submitButton.disabled = true;

  const conversation = getConversation();
  const nextConversation = [...conversation, { role: "user", content: text }];
  setConversation(nextConversation);
  renderHistory();

  const payload = [{
    latest_user_question: text,
    conversation_context: firstRowsToText(nextConversation),
    ui_request_id: `req_${Date.now()}`,
    submitted_at: new Date().toISOString()
  }];

  try {
    showStatus("질문 데이터를 그래프에 전달합니다.");
    sendDataToOutput(payload);
    await new Promise((resolve) => setTimeout(resolve, 150));

    showStatus("답변 생성 그래프를 실행합니다.");
    await runNode({ nodeId: RUN_START_NODE_ID });

    if (isConfiguredNodeId(ANSWER_NODE_ID)) {
      const startedAt = Date.now();
      const timeoutMs = 180000;
      const intervalMs = 1500;

      while (Date.now() - startedAt < timeoutMs) {
        try {
          const status = await getNodeStatus({ nodeId: ANSWER_NODE_ID });
          const statusText = String(status?.status || status?.state || "").toLowerCase();
          if (statusText.includes("success") || statusText.includes("done") || statusText.includes("completed")) {
            hideStatus();
            if (input) input.value = "";
            return;
          }
          if (statusText.includes("fail") || statusText.includes("error")) {
            showStatus("답변 생성 중 오류가 발생했습니다. `19B` 노드 상태를 확인해주세요.");
            return;
          }
        } catch (e) {
          console.error(e);
        }
        await new Promise((resolve) => setTimeout(resolve, intervalMs));
      }

      showStatus("답변 생성이 오래 걸리고 있습니다. 잠시 후 답변 뷰어에서 결과를 확인해주세요.");
    } else {
      showStatus("질문은 실행됐습니다. 다만 `ANSWER_NODE_ID`가 비어 있어 완료 상태 polling은 생략합니다.");
    }
  } catch (e) {
    console.error(e);
    showStatus("질문 실행에 실패했습니다. `샌드박스 출력 -> 3(dataset)` 연결과 `RUN_START_NODE_ID`를 확인해주세요.");
  } finally {
    isSubmitting = false;
    if (submitButton) submitButton.disabled = false;
    if (input) input.value = "";
  }
}

function clearConversation() {
  setConversation([]);
  renderHistory();
  hideStatus();
}

function handleQuestionKeydown(event) {
  if (!event) return;
  if ((event.ctrlKey || event.metaKey) && event.key === "Enter") {
    event.preventDefault();
    submitQuestion();
  }
}

window.__elvisSubmitQuestion = submitQuestion;
window.__elvisClearConversation = clearConversation;
window.__elvisHandleQuestionKeydown = handleQuestionKeydown;

setTimeout(renderHistory, 0);
```

---

## 6. 꼭 바꿔야 하는 값

아래 두 값은 샌드박스에 붙여넣은 뒤 반드시 네 캔버스 실제 노드 ID로 바꿔야 한다.

1. `RUN_START_NODE_ID`
   - 권장: `3. 에이전트 프롬프트_질의정규화`의 nodeId

2. `ANSWER_NODE_ID`
   - 권장: `19B. 에이전트 프롬프트_최종답변작성`의 nodeId

예시:

```javascript
const RUN_START_NODE_ID = "node-agenticPrompt-ddd4ac2bd03bbc3fb7df5e";
const ANSWER_NODE_ID = "node-agenticPrompt-1795e50163097569438964";
```

이 두 값을 바꾸지 않으면 `질문 실행` 버튼은 동작하지 않는다.

---

## 7. 테스트 순서

1. `페이지_Elvis_노무상담` 안에 `샌드박스_질문입력_Elvis`, `샌드박스_답변뷰어`를 둔다.
2. `샌드박스_질문입력_Elvis`의 HTML/CSS/JS를 이 문서대로 붙인다.
3. `샌드박스_질문입력_Elvis`의 `RUN_START_NODE_ID`, `ANSWER_NODE_ID`를 실제 nodeId로 바꾼다.
4. `샌드박스_질문입력_Elvis(output)`를 `3. 에이전트 프롬프트_질의정규화(dataset)`에 연결한다.
5. 질문 1개를 넣고 실행한다.
6. `샌드박스_답변뷰어`에 답변이 보이는지 확인한다.

질문 실행이 안 되면 아래 3가지를 먼저 본다.

1. `RUN_START_NODE_ID`가 실제 `3` 노드 ID로 바뀌었는지
2. `ANSWER_NODE_ID`가 실제 `19B` 노드 ID로 바뀌었는지
3. `샌드박스_질문입력_Elvis(output) -> 3(dataset)` 선이 실제로 연결되어 있는지
4. 버튼이 아예 안 눌리면 HTML의 `onclick`, `onkeydown`와 JS의 `window.__elvisSubmitQuestion` 할당까지 같이 다시 붙였는지

---

## 8. 운영 권장

최종 공개용 Elvis:

- `샌드박스_질문입력_Elvis`
- `샌드박스_답변뷰어`
- `샌드박스_근거뷰어`

내부 디버그용:

- `에이전트_법률상담입력_디버그`

즉 결론은 이거다.

- **공개용 UI는 샌드박스 질문입력**
- **에이전트 UI는 내부 테스트/장애 대응용**
