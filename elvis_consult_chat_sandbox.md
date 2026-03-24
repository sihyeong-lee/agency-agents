# Elvis AI노무사 단일 채팅 샌드박스 코드 문서

이 문서는 `AI노무사` 메뉴의 최종 화면인 `샌드박스_노무상담챗_Elvis` 코드 문서다.

중요:
- 이 샌드박스는 **질문 입력 + 대화 누적 + 답변 표시**를 한 화면에서 처리한다.
- 화면 레이아웃은 [chat.html](d:\AI canvas\새 폴더\chat.html) 을 최대한 그대로 따른다.
- **이 샌드박스는 더 이상 `sendDataToOutput()`으로 3번 노드를 직접 흔들지 않는다.**
- 대신 AI Canvas의 공식 경계인 `API 데이터 -> 3 -> ... -> 워크플로우 API 배포`를 사용하고, 샌드박스에서는 `fetch()`로 배포 URL을 호출한다.
- 즉 `AI노무사` 흰 화면 문제를 피하려면, 이 문서의 구조대로 **워크플로우 API 배포 URL**을 연결해야 한다.

## 1. 노드 구조

노드명:
- `샌드박스_노무상담챗_Elvis`

입력 포트:
- 없음

출력 포트:
- 사용하지 않음

샌드박스 편집 설정:
- `<head>`: 기본 meta 두 줄 유지
- 외부 CDN: 비움

## 2. 백엔드 워크플로우 구조

이 샌드박스가 직접 런타임 노드를 호출하는 대신, 아래 전용 API 워크플로우를 만든다.

```text
API데이터_ElvisAI노무사
-> 3. 에이전트 프롬프트_질의정규화
-> 4. 파이썬_정규화파싱
-> 5. 파이썬_URL생성
-> 공식 / 내부구조화 / 내부문서 / 웹 branch
-> 18M3
-> 19A
-> 19B
-> 19C. 파이썬_ElvisAPI응답패키징
-> 워크플로우API배포_ElvisAI노무사
```

### 2.1 API 데이터 노드

노드명:
- `API데이터_ElvisAI노무사`

입력 구조 JSON:

```json
{
  "latest_user_question": [""],
  "conversation_context": [""],
  "submitted_at": [""]
}
```

연결:

```text
API데이터_ElvisAI노무사 -> 3(dataset)
```

### 2.2 응답 패키징 노드

노드명:
- `19C. 파이썬_ElvisAPI응답패키징`

입력 연결:

```text
19B -> 19C(dataset)
19A -> 19C(memo_package)
18M3 -> 19C(merged_package)
```

코드:

```python
frames = []
if isinstance(dataset, pd.DataFrame):
    frames.append(dataset.copy())
if isinstance(x, list):
    for part in x:
        if isinstance(part, pd.DataFrame):
            frames.append(part.copy())

merged = {}
for frame in frames:
    if frame is None or len(frame) < 1:
        continue
    row = frame.iloc[0].to_dict()
    for k, v in row.items():
        text = "" if v is None else str(v).strip()
        if k not in merged or not str(merged.get(k, "") or "").strip():
            merged[k] = v

memo_text = str(merged.get("legal_memo_json", "") or "").strip()
answer_text = ""
for col in ["output_response", "output_response_1", "draft_answer", "result_text", "answer", "text"]:
    if col in merged:
        value = str(merged.get(col, "") or "").strip()
        if value:
            answer_text = value
            break

result = pd.DataFrame([{
    "latest_user_question": str(merged.get("latest_user_question", "") or "").strip(),
    "conversation_context": str(merged.get("conversation_context", "") or "").strip(),
    "query_class": str(merged.get("query_class", "") or "").strip(),
    "output_response": answer_text,
    "legal_memo_json": memo_text,
    "answer_generated_at": pd.Timestamp.now().isoformat()
}])
```

### 2.3 워크플로우 API 배포 노드

노드명:
- `워크플로우API배포_ElvisAI노무사`

연결:

```text
19C -> 워크플로우API배포_ElvisAI노무사(dataset)
```

배포 후:
- `배포하기` 버튼 클릭
- 생성된 API URL 복사
- 아래 JavaScript의 `WORKFLOW_API_URL`에 붙여넣기

## 3. HTML

```html
<div class="elvis-chat-page">
  <div class="header">
    <button type="button" class="header-title-btn" onclick="window.__navigateMenu && window.__navigateMenu(1)">
      <span class="dot"></span>
      <span class="header-title">ELVIS</span>
    </button>

    <div class="header-actions">
      <button type="button" class="menu-btn" onclick="window.__navigateMenu && window.__navigateMenu(1)">HOME</button>
      <button type="button" class="menu-btn is-active">AI노무사</button>
      <button type="button" class="menu-btn" onclick="window.__navigateMenu && window.__navigateMenu(3)">추후 개발</button>
      <button type="button" class="icon-btn" title="대화 초기화" onclick="window.__resetChat && window.__resetChat()">
        <svg width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24" aria-hidden="true">
          <path d="M12 5v14M5 12h14"/>
        </svg>
      </button>
    </div>
  </div>

  <div class="messages" id="messages"></div>

  <div id="statusBar" class="status hidden">
    <div id="statusText"></div>
  </div>

  <div class="input-wrap">
    <div class="input-box">
      <textarea
        id="questionInput"
        rows="1"
        placeholder="질문을 입력하세요. 예: 사내 음주가 징계사유가 되는지, 내부 사례까지 같이 설명해줘"
        onkeydown="window.__handleComposerKeydown && window.__handleComposerKeydown(event)"
        oninput="window.__handleComposerInput && window.__handleComposerInput(this)"
      ></textarea>
      <button class="send-btn" id="submitQuestion" type="button" onclick="window.__submitQuestion && window.__submitQuestion()" disabled>
        <svg viewBox="0 0 24 24" aria-hidden="true">
          <line x1="12" y1="19" x2="12" y2="5"/>
          <polyline points="5 12 12 5 19 12"/>
        </svg>
      </button>
    </div>
    <div class="input-footer">Elvis는 내부 사례, 법령, 판례, 행정해석을 함께 검토합니다.</div>
  </div>
</div>
```

## 4. CSS

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html, body {
  height: 100%;
  background: #fff;
}

body {
  font-family: 'Inter', 'Segoe UI', 'Noto Sans KR', sans-serif;
  color: #111;
}

.elvis-chat-page {
  height: 100dvh;
  display: flex;
  flex-direction: column;
  background: #fff;
}

.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 0 20px;
  height: 56px;
  border-bottom: 1px solid #ececec;
  flex-shrink: 0;
  background: #fff;
}

.header-title-btn {
  border: none;
  background: none;
  color: inherit;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 6px;
}

.header-title {
  font-size: 15px;
  font-weight: 500;
}

.dot {
  width: 8px;
  height: 8px;
  background: #2ecc71;
  border-radius: 50%;
  animation: blink 2s infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: .3; }
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.menu-btn {
  font-size: 13px;
  color: #333;
  background: #fff;
  border: 1px solid #e5e5e5;
  border-radius: 999px;
  padding: 7px 14px;
  cursor: pointer;
  transition: background .15s, border-color .15s;
}

.menu-btn:hover {
  background: #f5f5f5;
  border-color: #d7d7d7;
}

.menu-btn.is-active {
  background: #111;
  color: #fff;
  border-color: #111;
}

.icon-btn {
  width: 34px;
  height: 34px;
  border: none;
  background: none;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #555;
  transition: background .15s;
}

.icon-btn:hover { background: #f5f5f5; }

.messages {
  flex: 1;
  overflow-y: auto;
  padding: 32px 0 16px;
  scroll-behavior: smooth;
}

.messages::-webkit-scrollbar { width: 4px; }
.messages::-webkit-scrollbar-thumb { background: #e0e0e0; border-radius: 4px; }

.msg-row {
  display: flex;
  padding: 4px 20px;
  gap: 10px;
  max-width: 780px;
  margin: 0 auto 8px;
  width: 100%;
}

.msg-row.user {
  justify-content: flex-end;
}

.msg-row.user .bubble {
  background: #1a1a1a;
  color: #fff;
  border-radius: 18px 18px 4px 18px;
}

.msg-row.ai {
  justify-content: flex-start;
  align-items: flex-start;
}

.ai-avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: #111;
  color: #fff;
  font-size: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  margin-top: 4px;
}

.bubble {
  max-width: min(700px, calc(100vw - 88px));
  padding: 14px 16px;
  line-height: 1.65;
  font-size: 14px;
  white-space: pre-wrap;
  word-break: break-word;
}

.msg-row.ai .bubble {
  background: #fff;
  color: #111;
  border-radius: 4px 18px 18px 18px;
  border: 1px solid #ececec;
  box-shadow: 0 10px 25px rgba(17, 17, 17, 0.03);
}

.answer-card {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.answer-text {
  white-space: pre-wrap;
  color: #1b1b1b;
}

.inline-meta {
  display: grid;
  gap: 10px;
}

.meta-card {
  border: 1px solid #ececec;
  border-radius: 14px;
  background: #fafafa;
  padding: 12px 14px;
  font-size: 13px;
  color: #444;
}

.meta-card strong {
  display: block;
  color: #111;
  margin-bottom: 6px;
  font-size: 12px;
}

.meta-card ul {
  margin-left: 18px;
}

.badges {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.badge {
  display: inline-flex;
  align-items: center;
  padding: 4px 10px;
  border-radius: 999px;
  background: #f2f2f2;
  color: #444;
  font-size: 12px;
}

.typing {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 5px 2px;
}

.typing span {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #999;
  animation: typingBlink 1.2s infinite ease-in-out;
}

.typing span:nth-child(2) { animation-delay: .2s; }
.typing span:nth-child(3) { animation-delay: .4s; }

@keyframes typingBlink {
  0%, 80%, 100% { transform: scale(.7); opacity: .35; }
  40% { transform: scale(1); opacity: 1; }
}

.empty-state {
  max-width: 780px;
  margin: 60px auto 0;
  padding: 0 24px;
}

.big-logo {
  font-size: 64px;
  font-weight: 500;
  letter-spacing: -0.06em;
  line-height: .95;
  margin-bottom: 18px;
}

.empty-state p {
  color: #5f5f5f;
  font-size: 16px;
  line-height: 1.7;
  max-width: 640px;
}

.suggestions {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 24px;
}

.suggestion-chip {
  border: 1px solid #e5e5e5;
  background: #fff;
  color: #222;
  border-radius: 999px;
  padding: 10px 14px;
  font-size: 13px;
  cursor: pointer;
  transition: background .15s, border-color .15s;
}

.suggestion-chip:hover {
  background: #f7f7f7;
  border-color: #d8d8d8;
}

.status {
  max-width: 780px;
  width: calc(100% - 40px);
  margin: 0 auto 12px;
  padding: 11px 14px;
  border: 1px solid #ececec;
  background: #fafafa;
  color: #555;
  border-radius: 14px;
  font-size: 13px;
}

.status.hidden {
  display: none;
}

.input-wrap {
  border-top: 1px solid #ececec;
  padding: 16px 20px 18px;
  background: rgba(255,255,255,.96);
  backdrop-filter: blur(10px);
  flex-shrink: 0;
}

.input-box {
  max-width: 780px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 10px;
  border: 1px solid #e7e7e7;
  border-radius: 22px;
  padding: 10px 10px 10px 16px;
  background: #fff;
  box-shadow: 0 10px 30px rgba(17,17,17,.03);
}

.input-box textarea {
  width: 100%;
  min-height: 24px;
  max-height: 160px;
  resize: none;
  border: none;
  outline: none;
  background: transparent;
  font: inherit;
  line-height: 1.55;
  color: #111;
}

.send-btn {
  width: 42px;
  height: 42px;
  border: none;
  border-radius: 50%;
  background: #111;
  color: #fff;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: opacity .15s, transform .15s;
}

.send-btn svg {
  width: 18px;
  height: 18px;
  stroke: currentColor;
  fill: none;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.send-btn:hover:not(:disabled) {
  transform: translateY(-1px);
}

.send-btn:disabled {
  opacity: .35;
  cursor: not-allowed;
}

.input-footer {
  max-width: 780px;
  margin: 10px auto 0;
  font-size: 12px;
  color: #777;
  padding: 0 4px;
}

@media (max-width: 860px) {
  .header {
    padding: 0 14px;
  }

  .header-actions {
    gap: 6px;
  }

  .menu-btn {
    padding: 7px 12px;
  }

  .msg-row.user .bubble,
  .msg-row.ai .bubble {
    max-width: 100%;
  }
}
```

## 5. JavaScript

```javascript
(() => {
const STORE_KEY = "__elvisChatState";
const WORKFLOW_API_URL = "https://api.ai-canvas.io/api-deploy/NjliNzVjNTdjMjAxM2YzMTg3MzBhYTdkL25vZGUtd29ya2Zsb3dBUElEZXBsb3ktZGJoamlxN2VlY2xjOXk3MmJ2c2R6dg==";

const store = (() => {
  if (typeof window === "undefined") {
    return { messages: [], meta: {} };
  }
  if (!window[STORE_KEY] || typeof window[STORE_KEY] !== "object") {
    window[STORE_KEY] = { messages: [], meta: {} };
  }
  return window[STORE_KEY];
})();

function safeText(value, fallback = "") {
  const text = String(value ?? "").trim();
  return text || fallback;
}

function getMessages() {
  return Array.isArray(store.messages) ? store.messages : [];
}

function setMessages(items) {
  store.messages = Array.isArray(items) ? items : [];
}

function getMeta() {
  return store.meta && typeof store.meta === "object" ? store.meta : {};
}

function setMeta(meta) {
  store.meta = meta && typeof meta === "object" ? meta : {};
}

function parseMaybeJson(value) {
  if (!value) return {};
  if (typeof value === "object") return value;
  try {
    return JSON.parse(value);
  } catch (error) {
    return {};
  }
}

function toArray(value) {
  if (!value) return [];
  if (Array.isArray(value)) return value.filter(Boolean);
  if (typeof value === "string") {
    const text = value.trim();
    if (!text) return [];
    try {
      const parsed = JSON.parse(text);
      if (Array.isArray(parsed)) return parsed.filter(Boolean);
    } catch (error) {}
    return text.split(/\n|,/).map(v => v.trim()).filter(Boolean);
  }
  return [];
}

function escapeHtml(text) {
  return String(text || "")
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/\"/g, "&quot;")
    .replace(/'/g, "&#39;");
}

function createUserMessage(text) {
  return {
    id: `user_${Date.now()}`,
    role: "user",
    content: text,
    createdAt: new Date().toISOString()
  };
}

function createPendingAssistant(question) {
  return {
    id: `assistant_${Date.now()}`,
    role: "assistant",
    status: "pending",
    question,
    content: "답변을 생성하는 중입니다.",
    createdAt: new Date().toISOString()
  };
}

function buildConversationContext(messages) {
  return messages
    .filter(item => item.role === "user" || (item.role === "assistant" && item.status === "done"))
    .slice(-12)
    .map(item => `${item.role === "user" ? "user" : "assistant"}: ${item.content || ""}`)
    .join("\n");
}

function replacePendingAssistant(payload) {
  const messages = getMessages();
  const pendingIndex = messages.findIndex(item => item.role === "assistant" && item.status === "pending");
  if (pendingIndex === -1) return;

  messages[pendingIndex] = {
    ...messages[pendingIndex],
    role: "assistant",
    status: payload.status || "done",
    content: payload.answer,
    coreConclusion: payload.coreConclusion || "",
    queryClass: payload.queryClass || "",
    needsMoreFacts: !!payload.needsMoreFacts,
    factQuestions: payload.factQuestions || [],
    sanctionRange: payload.sanctionRange || "",
    createdAt: new Date().toISOString()
  };

  setMessages(messages);
}

function renderBadges(values) {
  const items = values.filter(Boolean);
  if (items.length === 0) return "";
  return `<div class="badges">${items.map(item => `<span class="badge">${escapeHtml(item)}</span>`).join("")}</div>`;
}

function renderFactList(items) {
  if (!items || items.length === 0) return "";
  return `<div class="meta-card"><strong>추가 필요정보</strong><ul>${items.map(item => `<li>${escapeHtml(item)}</li>`).join("")}</ul></div>`;
}

function renderAssistantMeta(message) {
  const blocks = [];

  const badges = renderBadges([
    message.queryClass,
    message.needsMoreFacts ? "추가 사실 필요" : ""
  ]);
  if (badges) blocks.push(badges);

  if (message.coreConclusion) {
    blocks.push(`<div class="meta-card"><strong>핵심 결론</strong><div>${escapeHtml(message.coreConclusion)}</div></div>`);
  }

  if (message.sanctionRange) {
    blocks.push(`<div class="meta-card"><strong>권장 징계 범위</strong><div>${escapeHtml(message.sanctionRange)}</div></div>`);
  }

  if (message.needsMoreFacts && Array.isArray(message.factQuestions) && message.factQuestions.length > 0) {
    blocks.push(renderFactList(message.factQuestions));
  }

  if (blocks.length === 0) return "";
  return `<div class="inline-meta">${blocks.join("")}</div>`;
}

function renderEmptyState() {
  return `
    <div class="empty-state">
      <div class="big-logo">ELVIS</div>
      <p>실무형 노동 자문을 대화처럼 이어가세요. 내부 사례, 법령, 판례, 행정해석을 함께 읽고 답합니다.</p>
      <div class="suggestions">
        <button type="button" class="suggestion-chip" onclick="window.__useSuggestion && window.__useSuggestion('직장 내 괴롭힘 신고자를 다른 부서로 전보한 게 불리한 처우가 될 수 있어?')">불리한 처우 판단</button>
        <button type="button" class="suggestion-chip" onclick="window.__useSuggestion && window.__useSuggestion('사내 음주가 징계사유가 되는지, 내부 유사 사례까지 같이 설명해줘')">사내 음주 징계</button>
        <button type="button" class="suggestion-chip" onclick="window.__useSuggestion && window.__useSuggestion('성희롱 조사 후 피해자 의견청취가 꼭 필요한지 조문까지 설명해줘')">성희롱 조치 의무</button>
      </div>
    </div>
  `;
}

function renderThread() {
  const root = document.getElementById("messages");
  if (!root) return;

  const messages = getMessages();
  if (messages.length === 0) {
    root.innerHTML = renderEmptyState();
    return;
  }

  root.innerHTML = messages.map(message => {
    if (message.role === "user") {
      return `
        <div class="msg-row user">
          <div class="bubble">${escapeHtml(message.content)}</div>
        </div>
      `;
    }

    if (message.status === "pending") {
      return `
        <div class="msg-row ai">
          <div class="ai-avatar">E</div>
          <div class="bubble">
            <div class="typing"><span></span><span></span><span></span></div>
          </div>
        </div>
      `;
    }

    return `
      <div class="msg-row ai">
        <div class="ai-avatar">E</div>
        <div class="bubble">
          <div class="answer-card">
            <div class="answer-text">${escapeHtml(message.content)}</div>
            ${renderAssistantMeta(message)}
          </div>
        </div>
      </div>
    `;
  }).join("");

  requestAnimationFrame(scrollBottom);
}

function showStatus(text) {
  const bar = document.getElementById("statusBar");
  const line = document.getElementById("statusText");
  if (bar) bar.classList.remove("hidden");
  if (line) line.textContent = text;
}

function hideStatus() {
  const bar = document.getElementById("statusBar");
  if (bar) bar.classList.add("hidden");
}

function isPending() {
  return !!safeText(getMeta().pendingQuestion);
}

function toggleSubmitDisabled(disabled) {
  const button = document.getElementById("submitQuestion");
  const textarea = document.getElementById("questionInput");
  if (button) button.disabled = !!disabled || !safeText(textarea && textarea.value);
}

function autoResize(el) {
  if (!el) return;
  el.style.height = "auto";
  el.style.height = Math.min(el.scrollHeight, 160) + "px";
}

function handleComposerInput(el) {
  autoResize(el);
  toggleSubmitDisabled(false);
}

function scrollBottom() {
  const messagesEl = document.getElementById("messages");
  if (!messagesEl) return;
  messagesEl.scrollTop = messagesEl.scrollHeight;
}

function isPlaceholderApiUrl(url) {
  return !url || url.indexOf("REPLACE_WITH_YOUR_WORKFLOW_URL") >= 0;
}

function normalizeColumnArrayObject(obj) {
  const out = {};
  if (!obj || typeof obj !== "object" || Array.isArray(obj)) return out;
  Object.keys(obj).forEach(key => {
    const value = obj[key];
    if (Array.isArray(value)) {
      out[key] = value.length > 0 ? value[0] : "";
    } else {
      out[key] = value;
    }
  });
  return out;
}

function firstRowFromApiResponse(data) {
  function unwrap(value, depth) {
    if (depth > 6 || value == null) return {};

    if (Array.isArray(value)) {
      for (const item of value) {
        const row = unwrap(item, depth + 1);
        if (row && Object.keys(row).length > 0) {
          return row;
        }
      }
      return {};
    }

    if (typeof value !== "object") {
      return {};
    }

    const normalized = normalizeColumnArrayObject(value);
    const hasUsefulKeys = Object.keys(normalized).some(key => key !== "data" && key !== "rows" && key !== "result");
    if (hasUsefulKeys) {
      return normalized;
    }

    if ("data" in value) {
      const row = unwrap(value.data, depth + 1);
      if (Object.keys(row).length > 0) return row;
    }

    if ("rows" in value) {
      const row = unwrap(value.rows, depth + 1);
      if (Object.keys(row).length > 0) return row;
    }

    if ("result" in value) {
      const row = unwrap(value.result, depth + 1);
      if (Object.keys(row).length > 0) return row;
    }

    return normalized;
  }

  return unwrap(data, 0);
}

async function callWorkflowApi(requestBody) {
  const response = await fetch(WORKFLOW_API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(requestBody)
  });

  const text = await response.text();
  let data = {};
  try {
    data = text ? JSON.parse(text) : {};
  } catch (error) {
    throw new Error(`응답 JSON 파싱 실패: ${text.slice(0, 240)}`);
  }

  if (!response.ok) {
    const msg = safeText(data.detail || data.message || text || response.statusText, `HTTP ${response.status}`);
    throw new Error(msg);
  }

  return data;
}

async function submitQuestion() {
  if (isPending()) {
    showStatus("이전 질문의 답변이 아직 생성 중입니다. 현재 답변이 끝난 뒤 다시 질문해 주세요.");
    return;
  }

  if (isPlaceholderApiUrl(WORKFLOW_API_URL)) {
    showStatus("WORKFLOW_API_URL이 아직 비어 있습니다. `워크플로우API배포_ElvisAI노무사` URL을 JavaScript 상단 상수에 넣어 주세요.");
    return;
  }

  const input = document.getElementById("questionInput");
  const question = safeText(input && input.value);
  if (!question) return;

  const messages = getMessages();
  const nextMessages = [...messages, createUserMessage(question), createPendingAssistant(question)];
  const conversationContext = buildConversationContext(nextMessages);

  setMessages(nextMessages);
  setMeta({
    pendingQuestion: question,
    submittedAt: new Date().toISOString()
  });

  renderThread();
  toggleSubmitDisabled(true);
  showStatus("질문을 전달했습니다. 답변을 생성하는 중입니다.");

  if (input) {
    input.value = "";
    input.style.height = "auto";
  }

  try {
    const responseData = await callWorkflowApi({
      latest_user_question: [question],
      conversation_context: [conversationContext],
      submitted_at: [new Date().toISOString()]
    });

    const row = firstRowFromApiResponse(responseData);
    const answerText = safeText(
      row.output_response ||
      row.output_response_1 ||
      row.answer ||
      row.result_text ||
      row.text ||
      ""
    );

    if (!answerText) {
      throw new Error("워크플로우 응답에 output_response가 없습니다.");
    }

    const memo = parseMaybeJson(row.legal_memo_json);
    replacePendingAssistant({
      answer: answerText,
      coreConclusion: safeText(memo.core_conclusion || memo.coreConclusion || ""),
      queryClass: safeText(row.query_class || memo.query_class || ""),
      needsMoreFacts: !!memo.needs_more_facts,
      factQuestions: toArray(memo.fact_questions),
      sanctionRange: safeText(memo.recommended_sanction_range || "")
    });

    setMeta({
      pendingQuestion: "",
      lastResolvedAt: new Date().toISOString()
    });

    renderThread();
    hideStatus();
    toggleSubmitDisabled(false);
  } catch (error) {
    console.error(error);
    replacePendingAssistant({
      status: "failed",
      answer: `답변 호출 중 오류가 발생했습니다.\n${safeText(error && error.message, "unknown error")}`
    });
    setMeta({
      pendingQuestion: "",
      lastResolvedAt: new Date().toISOString()
    });
    renderThread();
    showStatus("답변 호출에 실패했습니다. 워크플로우 API 배포 URL과 배포 상태를 확인해 주세요.");
    toggleSubmitDisabled(false);
  }
}

function resetChat() {
  setMessages([]);
  setMeta({ pendingQuestion: "" });
  hideStatus();
  const input = document.getElementById("questionInput");
  if (input) {
    input.value = "";
    input.style.height = "auto";
  }
  renderThread();
  toggleSubmitDisabled(true);
}

function handleComposerKeydown(event) {
  if ((event.ctrlKey || event.metaKey) && event.key === "Enter") {
    event.preventDefault();
    submitQuestion();
  }
}

function useSuggestion(text) {
  const input = document.getElementById("questionInput");
  if (!input) return;
  input.value = text;
  input.focus();
  handleComposerInput(input);
}

function navigateMenu(menuNumber) {
  const menuNo = Number(menuNumber);
  if (!menuNo || menuNo < 1) return false;

  const targets = [];
  if (typeof window !== "undefined") targets.push(window);
  if (typeof window !== "undefined" && window.parent && window.parent !== window) targets.push(window.parent);
  if (typeof window !== "undefined" && window.top && window.top !== window && window.top !== window.parent) targets.push(window.top);

  for (const target of targets) {
    try {
      if (target && typeof target.changeApplicationMenu === "function") {
        target.changeApplicationMenu(menuNo);
        return true;
      }
    } catch (error) {
      console.warn("changeApplicationMenu call failed", error);
    }
  }

  return false;
}

window.__submitQuestion = submitQuestion;
window.__resetChat = resetChat;
window.__handleComposerKeydown = handleComposerKeydown;
window.__handleComposerInput = handleComposerInput;
window.__useSuggestion = useSuggestion;
window.__navigateMenu = navigateMenu;
window.onerror = function(message) {
  showStatus("화면 스크립트 오류가 발생했습니다: " + String(message || "unknown error"));
};

renderThread();
const initialInput = document.getElementById("questionInput");
autoResize(initialInput);
toggleSubmitDisabled(true);
})();
```

## 6. 사용 메모

- 이 샌드박스는 [chat.html](d:\AI canvas\새 폴더\chat.html) 와 거의 같은 화면 톤으로 맞춘 버전이다.
- 사용자가 질문하면 오른쪽에 질문이 붙고, Elvis 답변은 왼쪽에 누적된다.
- **더 이상 아래 구조를 쓰지 않는다.**
  - `sendDataToOutput(payload)`
  - `output -> 3(dataset)`
  - `19B/19A/18M3 입력 포트 polling`
- 즉 기존 흰 화면 원인이던 `페이지 샌드박스 -> 워크플로우 dataset 직접 전달` 경계를 버리고, `fetch -> Workflow API Deploy` 구조로 교체한 것이다.
- 워크플로우 API가 아직 배포되지 않았거나 URL을 안 넣은 상태면, 질문 제출 시 바로 안내 문구가 뜬다.
