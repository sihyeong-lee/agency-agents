# Elvis AI노무사 단일 채팅 샌드박스 코드 문서

이 문서는 `AI노무사` 메뉴의 최종 화면인 `샌드박스_노무상담챗_Elvis` 코드 문서다.

중요:
- 이 샌드박스는 **질문 입력 + 대화 누적 + 답변 표시**를 한 화면에서 처리한다.
- 화면 레이아웃은 [chat.html](d:\AI canvas\새 폴더\chat.html) 을 최대한 그대로 따른다.
- 실제 추론은 뒤의 `3`, `19A`, `19B`가 수행한다.
- 이 샌드박스는 화면과 입력 흐름을 맡는다.

## 1. 노드 구조

노드명:
- `샌드박스_노무상담챗_Elvis`

입력 포트 생성 순서:
1. `answer_package`
2. `memo_package`
3. `merged_package`

출력 포트:
- 기본 출력 1개

연결:

```text
19B -> 샌드박스_노무상담챗_Elvis(answer_package)
19A -> 샌드박스_노무상담챗_Elvis(memo_package)
18M3 -> 샌드박스_노무상담챗_Elvis(merged_package)
샌드박스_노무상담챗_Elvis(output) -> 3(dataset)
```

샌드박스 편집 설정:
- `<head>`: 기본 meta 두 줄 유지
- 외부 CDN: 비움

## 2. HTML

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

## 3. CSS

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
  padding: 12px 16px;
  max-width: 72%;
  font-size: 14px;
  line-height: 1.6;
  white-space: pre-wrap;
  word-break: break-word;
}

.msg-row.ai {
  justify-content: flex-start;
  align-items: flex-start;
}

.ai-avatar {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #111;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  font-weight: 700;
  flex-shrink: 0;
  margin-top: 2px;
  letter-spacing: 0.03em;
}

.msg-row.ai .bubble {
  background: transparent;
  color: #111;
  max-width: 80%;
  font-size: 14px;
  line-height: 1.75;
  padding: 4px 0;
  white-space: pre-wrap;
  word-break: break-word;
}

.answer-card {
  display: grid;
  gap: 12px;
}

.answer-text {
  white-space: pre-wrap;
  word-break: break-word;
}

.inline-meta {
  display: grid;
  gap: 8px;
}

.badges {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.badge {
  font-size: 12px;
  color: #444;
  border: 1px solid #e5e5e5;
  border-radius: 999px;
  padding: 5px 10px;
}

.meta-card {
  border: 1px solid #ececec;
  border-radius: 12px;
  padding: 12px 14px;
  background: #fafafa;
}

.meta-card strong {
  display: block;
  font-size: 12px;
  color: #666;
  margin-bottom: 6px;
}

.meta-card ul {
  padding-left: 18px;
  display: grid;
  gap: 6px;
}

.meta-card li {
  line-height: 1.6;
}

.typing {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 10px 0 4px;
}

.typing span {
  width: 7px;
  height: 7px;
  background: #ccc;
  border-radius: 50%;
  animation: bounce 1.2s infinite;
}

.typing span:nth-child(2) { animation-delay: .2s; }
.typing span:nth-child(3) { animation-delay: .4s; }

@keyframes bounce {
  0%, 60%, 100% { transform: translateY(0); background: #ccc; }
  30% { transform: translateY(-5px); background: #999; }
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  gap: 12px;
  color: #aaa;
  user-select: none;
  padding: 0 20px;
  text-align: center;
}

.empty-state .big-logo {
  font-size: 38px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: #111;
}

.empty-state p {
  font-size: 14px;
  color: #666;
  max-width: 540px;
  line-height: 1.7;
}

.suggestions {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  margin-top: 8px;
  max-width: 560px;
}

.suggestion-chip {
  font-size: 13px;
  padding: 8px 16px;
  border: 1px solid #e5e5e5;
  border-radius: 100px;
  cursor: pointer;
  color: #333;
  background: #fff;
  transition: background .15s, border-color .15s;
}

.suggestion-chip:hover {
  background: #f5f5f5;
  border-color: #ccc;
}

.status {
  max-width: 780px;
  width: 100%;
  margin: 0 auto;
  padding: 0 16px 8px;
  font-size: 12px;
  color: #7d7d79;
}

.hidden { display: none !important; }

.input-wrap {
  flex-shrink: 0;
  padding: 12px 16px 20px;
  max-width: 780px;
  margin: 0 auto;
  width: 100%;
}

.input-box {
  display: flex;
  align-items: flex-end;
  gap: 10px;
  border: 1.5px solid #e0e0e0;
  border-radius: 16px;
  padding: 10px 10px 10px 16px;
  background: #fff;
  transition: border-color .2s, box-shadow .2s;
}

.input-box:focus-within {
  border-color: #aaa;
  box-shadow: 0 0 0 3px rgba(0,0,0,.04);
}

#questionInput {
  flex: 1;
  border: none;
  outline: none;
  font-family: inherit;
  font-size: 14px;
  line-height: 1.5;
  resize: none;
  max-height: 160px;
  min-height: 24px;
  background: transparent;
  color: #111;
}

#questionInput::placeholder { color: #bbb; }

.send-btn {
  width: 34px;
  height: 34px;
  border-radius: 10px;
  border: none;
  background: #111;
  color: #fff;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: background .15s, transform .1s;
}

.send-btn:hover { background: #333; }
.send-btn:active { transform: scale(.95); }
.send-btn:disabled { background: #e0e0e0; cursor: not-allowed; }

.send-btn svg {
  width: 16px;
  height: 16px;
  fill: none;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.input-footer {
  text-align: center;
  font-size: 11px;
  color: #bbb;
  margin-top: 10px;
}

@media (max-width: 900px) {
  .header {
    padding: 0 14px;
    gap: 10px;
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

## 4. JavaScript

```javascript
(() => {
const STORE_KEY = "__elvisChatState";
const PERSIST_KEY = "elvis_chat_v4";
const PARENT_KEY = "__elvisChatPersist_v4";
const POLL_MS = 1800;
let refreshLock = false;

function writeToParent(data) {
  const wins = [];
  try { if (window.parent && window.parent !== window) wins.push(window.parent); } catch(e) {}
  try { if (window.top && window.top !== window && window.top !== window.parent) wins.push(window.top); } catch(e) {}
  for (const w of wins) {
    try { w[PARENT_KEY] = data; return true; } catch(e) {}
  }
  return false;
}

function readFromParent() {
  const wins = [];
  try { if (window.parent && window.parent !== window) wins.push(window.parent); } catch(e) {}
  try { if (window.top && window.top !== window && window.top !== window.parent) wins.push(window.top); } catch(e) {}
  for (const w of wins) {
    try { const d = w[PARENT_KEY]; if (d && typeof d === "object") return d; } catch(e) {}
  }
  return null;
}

// ── 영속 스토리지: 부모 창 → sessionStorage → localStorage 순으로 시도 ──
function readPersist() {
  // 1순위: 부모 창 (iframe 재생성 후에도 유지됨)
  const fromParent = readFromParent();
  if (fromParent) return fromParent;
  // 2순위: sessionStorage → localStorage
  const storages = [];
  try { storages.push(sessionStorage); } catch (e) {}
  try { storages.push(localStorage); } catch (e) {}
  for (const s of storages) {
    try {
      const raw = s.getItem(PERSIST_KEY);
      if (!raw) continue;
      const data = JSON.parse(raw);
      if (data && typeof data === "object") return data;
    } catch (e) {}
  }
  return null;
}

function writePersist(messages, meta) {
  const data = { messages: messages || [], meta: meta || {} };
  // 1순위: 부모 창
  writeToParent(data);
  // 2순위: sessionStorage → localStorage
  const str = JSON.stringify(data);
  const storages = [];
  try { storages.push(sessionStorage); } catch (e) {}
  try { storages.push(localStorage); } catch (e) {}
  for (const s of storages) {
    try { s.setItem(PERSIST_KEY, str); } catch (e) {}
  }
}

// ── 인메모리 스토어: iframe 유지 시 보존, 재생성 시 영속 스토리지에서 복원 ──
const store = (() => {
  if (typeof window === "undefined") {
    return { messages: [], meta: {}, poller: null };
  }
  if (!window[STORE_KEY] || typeof window[STORE_KEY] !== "object") {
    const restored = readPersist();
    window[STORE_KEY] = {
      messages: (restored && Array.isArray(restored.messages)) ? restored.messages : [],
      meta: (restored && typeof restored.meta === "object") ? restored.meta : {},
      poller: null
    };
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
  writePersist(store.messages, store.meta);
}

function getMeta() {
  return store.meta && typeof store.meta === "object" ? store.meta : {};
}

function setMeta(meta) {
  store.meta = meta && typeof meta === "object" ? meta : {};
  writePersist(store.messages, store.meta);
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

function firstRow(rows) {
  return Array.isArray(rows) && rows.length > 0 ? rows[0] : {};
}

function replacePendingAssistant(payload) {
  const messages = getMessages();
  const pendingIndex = messages.findIndex(item => item.role === "assistant" && item.status === "pending");
  if (pendingIndex === -1) return;

  messages[pendingIndex] = {
    ...messages[pendingIndex],
    role: "assistant",
    status: "done",
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

function markPendingAsFailedIfStale() {
  const meta = getMeta();
  const pendingQuestion = safeText(meta.pendingQuestion);
  const submittedAt = safeText(meta.submittedAt);
  if (!pendingQuestion || !submittedAt) return false;

  const submittedTime = new Date(submittedAt).getTime();
  const ageMs = Date.now() - submittedTime;
  if (!Number.isFinite(submittedTime) || !Number.isFinite(ageMs) || ageMs < 120000) {
    return false;
  }

  const messages = getMessages().map(item => {
    if (item && item.role === "assistant" && item.status === "pending") {
      return {
        ...item,
        status: "failed",
        content: "이전 답변 생성이 중단되었습니다. 아래 입력창에서 질문을 다시 실행해 주세요."
      };
    }
    return item;
  });

  setMessages(messages);
  setMeta({
    pendingQuestion: "",
    lastResolvedSignature: safeText(meta.lastResolvedSignature),
    staleRecoveredAt: new Date().toISOString()
  });
  return true;
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

async function safeReadInput(target) {
  try {
    return await getDataset({ target, limit: 1, page: 1 });
  } catch (error) {
    console.warn(`input target ${target} read failed`, error);
    return [];
  }
}

function workflowQuestion(answerRow, memoRow, mergedRow) {
  return safeText(
    mergedRow.latest_user_question ||
    answerRow.latest_user_question ||
    memoRow.latest_user_question ||
    ""
  );
}

function resolveFromWorkflow(answerRow, memoRow, mergedRow) {
  const meta = getMeta();
  let pendingQuestion = safeText(meta.pendingQuestion);

  // pendingQuestion이 store에서 사라진 경우(iframe 재생성 등): pending 메시지에서 복원
  if (!pendingQuestion) {
    const messages = getMessages();
    const pendingMsg = messages.find(m => m.role === "assistant" && m.status === "pending");
    if (!pendingMsg) return;
    pendingQuestion = safeText(pendingMsg.question || "");
    if (!pendingQuestion) return;
  }

  const answerText = safeText(
    answerRow.output_response ||
    answerRow.output_response_1 ||
    answerRow.answer ||
    answerRow.result_text ||
    ""
  );
  if (!answerText) return;

  const qFromWorkflow = workflowQuestion(answerRow, memoRow, mergedRow);
  if (qFromWorkflow && qFromWorkflow !== pendingQuestion) return;

  const memo = parseMaybeJson(memoRow.legal_memo_json);
  const signature = JSON.stringify({
    question: qFromWorkflow || pendingQuestion,
    answer: answerText,
    sanctionRange: safeText(memo.recommended_sanction_range),
    needsMoreFacts: !!memo.needs_more_facts
  });

  if (safeText(meta.lastResolvedSignature) === signature) return;

  replacePendingAssistant({
    answer: answerText,
    coreConclusion: safeText(memo.core_conclusion || memo.coreConclusion || ""),
    queryClass: safeText(mergedRow.query_class || memo.query_class || answerRow.query_class || ""),
    needsMoreFacts: !!memo.needs_more_facts,
    factQuestions: toArray(memo.fact_questions),
    sanctionRange: safeText(memo.recommended_sanction_range || "")
  });

  setMeta({
    pendingQuestion: "",
    lastResolvedSignature: signature,
    lastResolvedAt: new Date().toISOString()
  });

  hideStatus();
  toggleSubmitDisabled(false);
  renderThread();
}

async function refreshFromWorkflow() {
  if (refreshLock) return;
  refreshLock = true;
  try {
    const answerRows = await safeReadInput(1);
    const memoRows = await safeReadInput(2);
    const mergedRows = await safeReadInput(3);
    resolveFromWorkflow(firstRow(answerRows), firstRow(memoRows), firstRow(mergedRows));
  } finally {
    refreshLock = false;
  }
}

function scrollBottom() {
  const messagesEl = document.getElementById("messages");
  if (!messagesEl) return;
  messagesEl.scrollTop = messagesEl.scrollHeight;
}

async function submitQuestion() {
  if (isPending()) {
    showStatus("이전 질문의 답변이 아직 생성 중입니다. 현재 답변이 끝난 뒤 다시 질문해 주세요.");
    return;
  }

  const input = document.getElementById("questionInput");
  const question = safeText(input && input.value);
  if (!question) return;

  const messages = getMessages();
  const nextMessages = [...messages, createUserMessage(question), createPendingAssistant(question)];
  setMessages(nextMessages);
  setMeta({
    pendingQuestion: question,
    lastResolvedSignature: safeText(getMeta().lastResolvedSignature),
    submittedAt: new Date().toISOString()
  });

  renderThread();
  toggleSubmitDisabled(true);
  showStatus("질문을 전달했습니다. 답변을 생성하는 중입니다.");

  const payload = [{
    latest_user_question: question,
    conversation_context: buildConversationContext(nextMessages),
    submitted_at: new Date().toISOString()
  }];

  try {
    sendDataToOutput(payload);
    if (input) {
      input.value = "";
      input.style.height = "auto";
    }
    showStatus("질문을 전달했습니다. 답변 생성을 기다리는 중입니다.");
    toggleSubmitDisabled(true);
  } catch (error) {
    console.error(error);
    showStatus("질문 전달 중 오류가 발생했습니다. `샌드박스_노무상담챗_Elvis(output) -> 3(dataset)` 연결을 확인해 주세요.");
    toggleSubmitDisabled(false);
  }
}

function resetChat() {
  setMessages([]);
  setMeta({ pendingQuestion: "", lastResolvedSignature: "" });
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
  if (!menuNo || menuNo < 1) {
    console.warn("invalid menu number:", menuNumber);
    return false;
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

  for (const target of targets) {
    try {
      if (target && typeof target.postMessage === "function") {
        target.postMessage({ type: "changeApplicationMenu", menuNumber: menuNo }, "*");
      }
    } catch (error) {
      console.warn("postMessage fallback failed", error);
    }
  }

  console.warn("menu navigation handler not found:", menuNo);
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
if (markPendingAsFailedIfStale()) {
  renderThread();
  showStatus("이전 질문 응답이 오래 지연되어 입력을 다시 열었습니다. 질문을 다시 실행해 주세요.");
  toggleSubmitDisabled(false);
} else if (isPending()) {
  showStatus("이전 질문의 답변 상태를 확인하는 중입니다.");
}
refreshFromWorkflow();
if (store.poller) {
  try {
    clearInterval(store.poller);
  } catch (error) {
    console.warn("clear previous poller failed", error);
  }
}
store.poller = setInterval(refreshFromWorkflow, POLL_MS);
})();
```

## 5. 사용 메모

- 이 샌드박스는 [chat.html](d:\AI canvas\새 폴더\chat.html) 와 거의 같은 화면 톤으로 맞춘 버전이다.
- 사용자가 질문하면 오른쪽에 질문이 붙고, Elvis 답변은 왼쪽에 누적된다.
- 입력과 출력이 한 화면에서 누적되므로, 별도 질문/답변 샌드박스는 공개 경로에서 쓰지 않는다.
- 아래는 유지한다.
  - 입력 포트 이름
  - output -> `3(dataset)` 연결
  - `sendDataToOutput(payload)` 구조
  - polling 기반 `getDataset({ target: 1/2/3 })`
- 상태 저장 우선순위: `window.parent[PARENT_KEY]` → `sessionStorage` → `localStorage` → `window.__elvisChatState`. iframe 재생성 후에도 부모 창을 통해 복원된다.
