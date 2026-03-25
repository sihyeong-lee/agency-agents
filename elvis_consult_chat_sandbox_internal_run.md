# Elvis AI노무사 채팅 샌드박스 코드 문서 - 내부 실행 버전

이 문서는 `AI노무사` 메뉴의 최종 채팅 화면을 **API Deploy가 아니라 AI Canvas 내부 실행 방식**으로 붙이는 버전입니다.

핵심:
- 질문은 샌드박스에서 입력
- 샌드박스가 `sendDataToOutput()`으로 질문 데이터 전달
- 연결된 `3. 에이전트 프롬프트_질의정규화`가 그 dataset을 받도록 함
- 샌드박스가 `getNodeStatus()`로 최종 결과 노드를 polling
- 샌드박스가 `getNodeOutputDataset()`으로 답변을 읽어 렌더링

이 방식은 **품질이 무거운 본선 워크플로우**를 유지하면서도,
`Workflow API Deploy` timeout 문제를 피하는 데 목적이 있습니다.

## 1. 필요한 연결

샌드박스 노드명:
- `샌드박스_노무상담챗_Elvis`

입력 포트:
- 없음

출력 포트:
- `output`

필수 연결:

```text
샌드박스_노무상담챗_Elvis(output) -> 3. 에이전트 프롬프트_질의정규화(dataset)
```

중요:
- `19A`, `19B`, `18M3`를 샌드박스 입력 포트로 연결하지 않습니다.
- `Workflow API Deploy`도 이 버전에서는 쓰지 않습니다.

## 1.1 19C 내부 연결

샌드박스에는 `19C`를 직접 연결하지 않지만,
워크플로우 내부에서는 `19C`가 아래 3개를 받아야 합니다.

```text
19B -> 19C(dataset)
19A -> 19C(memo_package)
18M3 -> 19C(merged_package)
```

이유:
- `19B` : 최종 답변 본문
- `19A` : 메모 JSON
- `18M3` : 근거 패키지

그리고 샌드박스는 **선 연결 없이**
`RESULT_NODE_ID = 19C`를 `getNodeStatus()` / `getNodeOutputDataset()`으로 직접 읽습니다.

## 2. 필요한 node id

JavaScript 안에 아래 2개를 넣어야 합니다.

1. `RESULT_NODE_ID`
- 가능하면 `19C. 파이썬_ElvisAPI응답패키징`의 node id
- 아직 `19C`가 없거나 못 찾으면 임시로 `19B` node id를 넣어도 됩니다

현재 값:
```javascript
const RESULT_NODE_ID = "node-pythonScript-3je3pg8ru8qp1x8826bbe";
```

설명:
- `RESULT_NODE_ID`는 `19C. 파이썬_ElvisAPI응답패키징`

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
        placeholder="질문을 입력해 주세요. 예) 사내 음주가 징계사유가 되는지, 내부 유사사례까지 같이 설명해줘"
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
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html, body {
  height: 100%;
  background: #fff;
}

body {
  font-family: 'Segoe UI', 'Noto Sans KR', sans-serif;
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
  color: #1b1b1b;
}

.answer-text p {
  margin: 0 0 10px;
}

.answer-text p:last-child {
  margin-bottom: 0;
}

.answer-text .section-heading {
  margin: 16px 0 8px;
  padding-top: 14px;
  border-top: 1px solid #efefef;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.04em;
  color: #6b7280;
}

.answer-text .section-heading:first-child {
  margin-top: 0;
  padding-top: 0;
  border-top: none;
}

.answer-text ul,
.answer-text ol {
  margin: 4px 0 10px 20px;
  padding: 0;
}

.answer-text li {
  margin: 4px 0;
}

.answer-text strong {
  font-weight: 700;
  color: #111;
}

.answer-text code {
  background: #f4f4f4;
  border-radius: 6px;
  padding: 1px 6px;
  font-family: Consolas, 'Courier New', monospace;
  font-size: 13px;
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
  margin: 0 auto;
  padding: 0 20px;
}

.starter-bubble {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.starter-title {
  font-size: 16px;
  font-weight: 600;
  color: #111;
}

.starter-copy {
  color: #5f5f5f;
  font-size: 14px;
  line-height: 1.7;
}

.suggestions {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 8px;
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
const RESULT_NODE_ID = "node-pythonScript-3je3pg8ru8qp1x8826bbe";
const RESULT_OUTPUT_TARGET = 1;
const POLL_INTERVAL_MS = 2500;
const MAX_POLL_MS = 15 * 60 * 1000;
const PENDING_GRACE_MS = 15000;

function resolveSharedHost() {
  const targets = [];
  if (typeof window !== "undefined" && window.parent && window.parent !== window) targets.push(window.parent);
  if (typeof window !== "undefined" && window.top && window.top !== window && window.top !== window.parent) targets.push(window.top);
  if (typeof window !== "undefined") targets.push(window);

  for (const target of targets) {
    try {
      if (target && typeof target === "object") {
        return target;
      }
    } catch (error) {}
  }

  return typeof window !== "undefined" ? window : {};
}

const sharedHost = resolveSharedHost();

const store = (() => {
  if (typeof window === "undefined") {
    return { messages: [], meta: {} };
  }
  if (!sharedHost[STORE_KEY] || typeof sharedHost[STORE_KEY] !== "object") {
    sharedHost[STORE_KEY] = { messages: [], meta: {} };
  }
  return sharedHost[STORE_KEY];
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

function clearStoredConversation() {
  setMessages([]);
  setMeta({
    pendingQuestion: "",
    preserveOnce: 0,
    lastResolvedAt: ""
  });
}

function isPendingSubmissionStillFresh() {
  const meta = getMeta();
  const pendingQuestion = safeText(meta.pendingQuestion || "");
  const submittedAt = safeText(meta.submittedAt || "");

  if (!pendingQuestion || !submittedAt) {
    return false;
  }

  const submittedTs = Date.parse(submittedAt);
  if (Number.isNaN(submittedTs)) {
    return false;
  }

  return (Date.now() - submittedTs) < PENDING_GRACE_MS;
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

function renderInlineMarkdown(text) {
  return String(text || "")
    .replace(/\*\*(.+?)\*\*/g, "<strong>$1</strong>")
    .replace(/__(.+?)__/g, "<strong>$1</strong>")
    .replace(/`([^`]+)`/g, "<code>$1</code>");
}

function renderMarkdown(text) {
  const normalized = escapeHtml(text).replace(/\r\n/g, "\n");
  const blocks = normalized.split(/\n{2,}/).map(block => block.trim()).filter(Boolean);

  if (blocks.length === 0) {
    return "";
  }

  return blocks.map(block => {
    const lines = block.split("\n").map(line => line.trimEnd()).filter(Boolean);
    if (lines.length === 0) {
      return "";
    }

    if (lines.length === 1 && /^\[[^\]]+\]$/.test(lines[0].trim())) {
      return `<div class="section-heading">${renderInlineMarkdown(lines[0].trim())}</div>`;
    }

    if (/^\[[^\]]+\]$/.test(lines[0].trim()) && lines.length > 1) {
      const heading = `<div class="section-heading">${renderInlineMarkdown(lines[0].trim())}</div>`;
      const rest = lines.slice(1);

      if (rest.every(line => /^[-*]\s+/.test(line))) {
        return `${heading}<ul>${rest.map(line => `<li>${renderInlineMarkdown(line.replace(/^[-*]\s+/, ""))}</li>`).join("")}</ul>`;
      }

      if (rest.every(line => /^\d+\.\s+/.test(line))) {
        return `${heading}<ol>${rest.map(line => `<li>${renderInlineMarkdown(line.replace(/^\d+\.\s+/, ""))}</li>`).join("")}</ol>`;
      }

      return `${heading}<p>${rest.map(line => renderInlineMarkdown(line)).join("<br>")}</p>`;
    }

    if (lines.every(line => /^[-*]\s+/.test(line))) {
      return `<ul>${lines.map(line => `<li>${renderInlineMarkdown(line.replace(/^[-*]\s+/, ""))}</li>`).join("")}</ul>`;
    }

    if (lines.every(line => /^\d+\.\s+/.test(line))) {
      return `<ol>${lines.map(line => `<li>${renderInlineMarkdown(line.replace(/^\d+\.\s+/, ""))}</li>`).join("")}</ol>`;
    }

    return `<p>${lines.map(line => renderInlineMarkdown(line)).join("<br>")}</p>`;
  }).join("");
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

function finishPendingWithRow(row) {
  const answerText = safeText(
    row.output_response ||
    row.output_response_1 ||
    row.answer ||
    row.result_text ||
    row.text ||
    ""
  );

  if (!answerText) {
    throw new Error("?? ?? ?? ??? answer ???? ????.");
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

  const meta = getMeta();
  setMeta({
    ...meta,
    pendingQuestion: "",
    watcherId: "",
    lastResolvedAt: new Date().toISOString(),
    preserveOnce: 1
  });

  renderThread();
  hideStatus();
  toggleSubmitDisabled(false);
}

function failPendingWithMessage(message) {
  replacePendingAssistant({
    status: "failed",
    answer: `?? ?? ? ??? ??????.
${safeText(message, "unknown error")}`
  });

  const meta = getMeta();
  setMeta({
    ...meta,
    pendingQuestion: "",
    watcherId: "",
    lastResolvedAt: new Date().toISOString(),
    preserveOnce: 1
  });

  renderThread();
  showStatus("?? ??? ????? ??????. 19C ??? ??? ???.");
  toggleSubmitDisabled(false);
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
      <div class="msg-row ai">
        <div class="ai-avatar">E</div>
        <div class="bubble">
          <div class="starter-bubble">
            <div class="starter-title">AI노무사입니다. 무엇을 도와드릴까요?</div>
            <div class="starter-copy">질문을 입력하면 내부 사례, 법령, 판례, 행정해석을 함께 검토해서 답변합니다.</div>
            <div class="suggestions">
              <button type="button" class="suggestion-chip" onclick="window.__useSuggestion && window.__useSuggestion('직장 내 괴롭힘 신고자를 다른 부서로 전보한 게 불리한 처우가 되는지 설명해줘')">불리한 처우 판단</button>
              <button type="button" class="suggestion-chip" onclick="window.__useSuggestion && window.__useSuggestion('사내 음주가 징계사유가 되는지, 내부 유사 사례까지 같이 설명해줘')">사내 음주 징계</button>
              <button type="button" class="suggestion-chip" onclick="window.__useSuggestion && window.__useSuggestion('성희롱 조사 전후에 회사가 꼭 해야 하는 보호조치 의무를 정리해줘')">성희롱 보호조치</button>
            </div>
          </div>
        </div>
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
            <div class="answer-text">${renderMarkdown(message.content)}</div>
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

function firstRowFromNodeDataset(data) {
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
    const hasUsefulKeys = Object.keys(normalized).length > 0;
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

function buildRestoredMessages(question, answer) {
  const messages = [];
  if (safeText(question)) {
    messages.push(createUserMessage(question));
  }
  if (safeText(answer)) {
    messages.push({
      id: `assistant_restore_${Date.now()}`,
      role: "assistant",
      status: "done",
      content: answer,
      createdAt: new Date().toISOString()
    });
  }
  return messages;
}

function safeNodeApi(name) {
  return typeof window !== "undefined" && typeof window[name] === "function";
}

async function restoreConversationFromNodes() {
  if (!safeNodeApi("getNodeStatus") || !safeNodeApi("getNodeOutputDataset")) {
    return;
  }

  try {
    const resultStatus = await getNodeStatus({ nodeId: RESULT_NODE_ID });

    if (resultStatus === "complete") {
      let resultRows = [];
      try {
        resultRows = await getNodeOutputDataset({
          nodeId: RESULT_NODE_ID,
          target: RESULT_OUTPUT_TARGET,
          limit: 1,
          page: 1
        });
      } catch (error) {
        console.warn("result node output read skipped", error);
        return;
      }
      const row = firstRowFromNodeDataset(resultRows);
      const answer = safeText(row.output_response || row.output_response_1 || row.answer || row.result_text || row.text || "");
      const question = safeText(row.latest_user_question || getMeta().pendingQuestion || "");

      if (answer) {
        const messages = buildRestoredMessages(question, answer);
        setMessages(messages);
        setMeta({
          pendingQuestion: "",
          lastResolvedAt: new Date().toISOString()
        });
        renderThread();
        hideStatus();
        toggleSubmitDisabled(false);
        return;
      }
    }

    if (resultStatus === "loading") {
      const question = safeText(getMeta().pendingQuestion || "");
      if (question) {
        setMessages([
          createUserMessage(question),
          createPendingAssistant(question)
        ]);
        setMeta({
          pendingQuestion: question,
          submittedAt: getMeta().submittedAt || new Date().toISOString()
        });
        renderThread();
        showStatus("이전 질문의 답변을 계속 생성하고 있습니다.");
        toggleSubmitDisabled(true);
      }
      return;
    }

    if (resultStatus === "not start") {
      const question = safeText(getMeta().pendingQuestion || "");

      if (question && isPendingSubmissionStillFresh()) {
        setMessages([
          createUserMessage(question),
          createPendingAssistant(question)
        ]);
        renderThread();
        showStatus("질문을 전달했고, 답변 생성을 준비 중입니다.");
        toggleSubmitDisabled(true);
        return;
      }

      clearStoredConversation();
      renderThread();
      hideStatus();
      toggleSubmitDisabled(false);
    }
  } catch (error) {
    console.warn("restoreConversationFromNodes failed", error);
  }
}

function bootstrapChatState() {
  const messages = getMessages();
  const meta = getMeta();

  if (!Array.isArray(messages) || messages.length === 0) {
    return;
  }

  if (safeText(meta.pendingQuestion)) {
    return;
  }

  if (Number(meta.preserveOnce || 0) > 0) {
    setMeta({
      ...meta,
      preserveOnce: 0
    });
    return;
  }

  clearStoredConversation();
}

async function pollResultNode() {
  const startedAt = Date.now();

  while (Date.now() - startedAt < MAX_POLL_MS) {
    const status = await getNodeStatus({ nodeId: RESULT_NODE_ID });

    if (status === "complete") {
      const rows = await getNodeOutputDataset({
        nodeId: RESULT_NODE_ID,
        target: RESULT_OUTPUT_TARGET,
        limit: 1,
        page: 1
      });

      const row = firstRowFromNodeDataset(rows);
      if (!row || Object.keys(row).length === 0) {
        throw new Error("최종 결과 노드가 complete 상태지만 출력 데이터가 비어 있습니다.");
      }

      return row;
    }

    if (status === "failed") {
      throw new Error("최종 결과 노드 실행이 실패했습니다.");
    }

    await new Promise(resolve => setTimeout(resolve, POLL_INTERVAL_MS));
  }

  throw new Error("답변 생성이 너무 오래 걸려 제한 시간 안에 끝나지 않았습니다.");
}

async function watchPendingResult() {
  const question = safeText(getMeta().pendingQuestion || "");
  if (!question) {
    return;
  }

  const watcherId = `watch_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
  const meta = getMeta();
  setMeta({
    ...meta,
    watcherId,
    pendingQuestion: question,
    submittedAt: meta.submittedAt || new Date().toISOString()
  });

  showStatus("??? ????, ?? ??? ?? ????.");
  toggleSubmitDisabled(true);

  try {
    const row = await pollResultNode();
    const liveMeta = getMeta();
    if (safeText(liveMeta.watcherId || "") !== watcherId) {
      return;
    }
    finishPendingWithRow(row);
  } catch (error) {
    const liveMeta = getMeta();
    if (safeText(liveMeta.watcherId || "") !== watcherId) {
      return;
    }
    console.error(error);
    failPendingWithMessage(error && error.message);
  }
}


async function submitQuestion() {
  if (isPending()) {
    showStatus("?? ??? ??? ?? ?? ????. ?? ??? ?? ? ?? ??? ???.");
    return;
  }

  if (!safeNodeApi("sendDataToOutput") || !safeNodeApi("getNodeStatus") || !safeNodeApi("getNodeOutputDataset")) {
    showStatus("?????? ??? AI Canvas API? ?? ?????.");
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
    submittedAt: new Date().toISOString(),
    watcherId: ""
  });

  renderThread();
  toggleSubmitDisabled(true);
  showStatus("??? ????, ?? ??? ??????.");

  if (input) {
    input.value = "";
    input.style.height = "auto";
  }

  try {
    if (typeof resetSandboxOutput === "function") {
      try {
        resetSandboxOutput();
      } catch (error) {}
    }

    sendDataToOutput([{
      latest_user_question: question,
      conversation_context: conversationContext,
      submitted_at: new Date().toISOString()
    }]);

    watchPendingResult();
  } catch (error) {
    console.error(error);
    failPendingWithMessage(error && error.message);
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

  if (menuNo !== 2) {
    clearStoredConversation();
  }

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

if (isPending()) {
  watchPendingResult();
}
})();
```

## 6. 적용 순서

1. `샌드박스_노무상담챗_Elvis(output) -> 3(dataset)` 선만 남깁니다
2. `elvis_consult_chat_sandbox_internal_run.md`의 HTML/CSS/JS를 샌드박스에 붙입니다
3. JS 상단의 `RESULT_NODE_ID`가 아래 값으로 들어가 있는지 확인합니다
   - `node-pythonScript-3je3pg8ru8qp1x8826bbe`
4. 앱 다시 배포
5. 질문 1개로 테스트

## 7. 왜 이 방식이 맞는가

- 본선 워크플로우는 길고 무겁습니다
- `Workflow API Deploy`는 긴 플로우를 HTTP 한 번에 받으려 해서 timeout이 나기 쉽습니다
- 반면 이 구조는 실행을 **캔버스 내부 런타임**에 맡기고,
  샌드박스는 **상태 조회와 렌더링만** 합니다

즉:
- 답변 품질은 유지
- HTTP timeout 문제는 회피
- 채팅 UI는 자연스럽게 유지
