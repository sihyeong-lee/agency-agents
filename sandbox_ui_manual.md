# 샌드박스 UI 코드 문서

이 문서는 `manual.md`의 **16. 애플리케이션 페이지 / 샌드박스 배포**를 실제로 구현할 때 필요한 `HTML / CSS / JavaScript`만 따로 모아 둔 문서다.

중요:

- 애플리케이션 페이지는 샌드박스를 메뉴에 직접 연결하는 것이 아니라, 보통 **페이지 노드**를 메뉴에 연결하고 그 페이지 노드 안에 샌드박스와 에이전트를 배치한다고 이해하면 가장 안전하다.
- 공개용 Elvis에서는 질문 입력도 샌드박스로 처리하는 것을 권장한다.
- 따라서 아래 두 샌드박스는 각각 대응하는 페이지 노드 안에 들어간다.
  - `페이지_Elvis_노무상담` 안의 `샌드박스_답변뷰어`
  - `페이지_Elvis_근거` 안의 `샌드박스_근거뷰어`
- 질문 입력용 샌드박스는 별도 문서 [elvis_consult_input_sandbox.md](d:\AI canvas\새 폴더\elvis_consult_input_sandbox.md)를 기준으로 만든다.

메인 구조, 검색 파이프라인, 병합 노드 설명은 `manual.md`와 `elvis_platform_manual.md`를 기준으로 보고,
이 문서는 아래 2개 샌드박스 노드의 코드만 복붙하면 된다.

- `샌드박스_답변뷰어`
- `샌드박스_근거뷰어`

---

## 1. 먼저 만들어야 하는 노드

### 1.1 애플리케이션 페이지 메뉴

권장 메뉴 이름:

1. `노무상담`
2. `근거`

### 1.2 페이지 노드

1. `페이지_Elvis_노무상담`
- 노드 타입: `페이지`
- 연결 메뉴: `노무상담`
- 권장 크기 맞춤: `자동 맞춤`
- 플로우 노드 설정: `사용하지 않음`

2. `페이지_Elvis_근거`
- 노드 타입: `페이지`
- 연결 메뉴: `근거`
- 권장 크기 맞춤: `자동 맞춤`
- 플로우 노드 설정: `사용하지 않음`

### 1.3 페이지 노드 내부에 올릴 노드

1. `샌드박스_질문입력_Elvis`
- 노드 타입: `샌드박스`
- 배치 위치: `페이지_Elvis_노무상담` 내부
- 역할: 공개용 사용자 질문 입력 UI
- 연결: 이 노드의 출력은 `3. 에이전트 프롬프트_질의정규화`의 `dataset`으로 연결한다.
- 이 공개 경로에서는 `1`, `2A`, `2B`, `2C`를 거치지 않는다. 그 노드들은 디버그용 에이전트 입력 경로를 남길 때만 유지한다.

2. `샌드박스_답변뷰어`
- 노드 타입: `샌드박스`
- 배치 위치: `페이지_Elvis_노무상담` 내부
- 출력 포트: 사용하지 않음
- 입력 포트 생성 순서:
  1. `answer_package`
  2. `memo_package`
  3. `merged_package`

3. `샌드박스_근거뷰어`
- 노드 타입: `샌드박스`
- 배치 위치: `페이지_Elvis_근거` 내부
- 출력 포트: 사용하지 않음
- 입력 포트 생성 순서:
  1. `merged_package`
  2. `memo_package`

### 1.4 선 연결

`샌드박스_답변뷰어`

```text
19B -> 샌드박스_답변뷰어(answer_package)
19A -> 샌드박스_답변뷰어(memo_package)
18M2 -> 샌드박스_답변뷰어(merged_package)
```

`샌드박스_근거뷰어`

```text
18M2 -> 샌드박스_근거뷰어(merged_package)
19A  -> 샌드박스_근거뷰어(memo_package)
```

중요:
- 샌드박스의 `getDataset({ target: N })`는 **입력 포트 생성 순서**를 그대로 따른다.
- 즉 `샌드박스_답변뷰어`에서는
  - `target: 1` = `answer_package`
  - `target: 2` = `memo_package`
  - `target: 3` = `merged_package`
- `샌드박스_근거뷰어`에서는
  - `target: 1` = `merged_package`
  - `target: 2` = `memo_package`

---

## 2. 샌드박스_답변뷰어

### 2.1 역할

이 노드는 아래만 보여준다.

- 현재 질문
- 질의 분류
- 최종 답변
- 메모 기준 핵심 결론
- 추가 필요정보

즉 질문을 실행하는 노드가 아니라, 이미 계산된 결과를 보기 좋게 보여주는 노드다.

### 2.2 HTML

```html
<div class="page">
  <header class="hero">
    <div class="hero__label">노무 상담 결과</div>
    <h1 class="hero__title">답변</h1>
    <p class="hero__desc">질문과 최종 답변, 핵심 결론, 추가 필요정보를 한 화면에서 확인합니다.</p>
  </header>

  <main class="stack">
    <section class="card">
      <div class="section-title">현재 질문</div>
      <div id="questionText" class="question-box">질문을 기다리는 중입니다.</div>
      <div id="queryClass" class="pill-row"></div>
    </section>

    <section class="card">
      <div class="section-title">핵심 결론</div>
      <div id="coreConclusion" class="answer-box">법률 메모를 기다리는 중입니다.</div>
    </section>

    <section class="card">
      <div class="section-title">최종 답변</div>
      <div id="answerText" class="answer-box">답변을 기다리는 중입니다.</div>
    </section>

    <section id="factSection" class="card hidden">
      <div class="section-title">추가 필요정보</div>
      <ul id="factQuestions" class="list"></ul>
    </section>
  </main>
</div>
```

### 2.3 CSS

```css
:root {
  --bg: #f7f7f5;
  --card: #ffffff;
  --line: #dedbd2;
  --text: #1f2937;
  --sub: #6b7280;
  --accent: #154c79;
  --accent-soft: #e7f0f8;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Segoe UI", "Noto Sans KR", sans-serif;
  color: var(--text);
  background: linear-gradient(180deg, #faf9f6 0%, #f2efe8 100%);
}

.page {
  max-width: 960px;
  margin: 0 auto;
  padding: 24px 16px 40px;
}

.hero {
  margin-bottom: 20px;
}

.hero__label {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 999px;
  background: var(--accent-soft);
  color: var(--accent);
  font-size: 12px;
  font-weight: 700;
}

.hero__title {
  margin: 10px 0 8px;
  font-size: 28px;
  line-height: 1.2;
}

.hero__desc {
  margin: 0;
  color: var(--sub);
  font-size: 14px;
}

.stack {
  display: grid;
  gap: 14px;
}

.card {
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 18px;
  padding: 18px;
  box-shadow: 0 10px 30px rgba(20, 20, 20, 0.05);
}

.section-title {
  font-size: 13px;
  font-weight: 700;
  color: var(--sub);
  margin-bottom: 10px;
}

.question-box,
.answer-box {
  white-space: pre-wrap;
  line-height: 1.72;
  font-size: 15px;
}

.pill-row {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 12px;
}

.pill {
  display: inline-flex;
  align-items: center;
  padding: 6px 10px;
  border-radius: 999px;
  background: #f2f4f7;
  color: #344054;
  font-size: 12px;
  font-weight: 700;
}

.list {
  margin: 0;
  padding-left: 18px;
  line-height: 1.7;
}

.hidden {
  display: none;
}

@media (max-width: 640px) {
  .page {
    padding: 18px 12px 28px;
  }

  .hero__title {
    font-size: 24px;
  }
}
```

### 2.4 JavaScript

```javascript
function firstRow(rows) {
  return Array.isArray(rows) && rows.length > 0 ? rows[0] : {};
}

function parseMaybeJson(value) {
  if (!value) return {};
  if (typeof value === "object") return value;
  try {
    return JSON.parse(value);
  } catch (e) {
    return {};
  }
}

function setText(id, value, fallback = "") {
  const el = document.getElementById(id);
  if (!el) return;
  const text = String(value || "").trim();
  el.textContent = text || fallback;
}

function renderPillRow(containerId, labels) {
  const root = document.getElementById(containerId);
  if (!root) return;
  root.innerHTML = "";
  (labels || []).filter(Boolean).forEach((label) => {
    const span = document.createElement("span");
    span.className = "pill";
    span.textContent = label;
    root.appendChild(span);
  });
}

function renderFactQuestions(memo) {
  const section = document.getElementById("factSection");
  const list = document.getElementById("factQuestions");
  if (!section || !list) return;

  const needsMoreFacts = !!memo.needs_more_facts;
  const questions = Array.isArray(memo.fact_questions) ? memo.fact_questions.filter(Boolean) : [];

  if (!needsMoreFacts || questions.length === 0) {
    section.classList.add("hidden");
    list.innerHTML = "";
    return;
  }

  section.classList.remove("hidden");
  list.innerHTML = "";
  questions.forEach((q) => {
    const li = document.createElement("li");
    li.textContent = q;
    list.appendChild(li);
  });
}

async function loadAnswerView() {
  const answerRows = await getDataset({ target: 1, limit: 1, page: 1 });
  const memoRows = await getDataset({ target: 2, limit: 1, page: 1 });
  const mergedRows = await getDataset({ target: 3, limit: 1, page: 1 });

  const answerRow = firstRow(answerRows);
  const memoRow = firstRow(memoRows);
  const mergedRow = firstRow(mergedRows);
  const memo = parseMaybeJson(memoRow.legal_memo_json);

  setText("questionText", mergedRow.latest_user_question, "질문이 아직 실행되지 않았습니다.");
  setText("coreConclusion", memo.core_conclusion, "핵심 결론이 아직 생성되지 않았습니다.");
  setText("answerText", answerRow.output_response, "최종 답변이 아직 생성되지 않았습니다.");

  const pills = [];
  if (mergedRow.query_class) pills.push(`질의 분류: ${mergedRow.query_class}`);
  if (memo.conclusion_strength) pills.push(`결론 강도: ${memo.conclusion_strength}`);
  if (memo.answer_mode) pills.push(`답변 모드: ${memo.answer_mode}`);
  if (memo.recommended_sanction_range) pills.push(`권장 수위: ${memo.recommended_sanction_range}`);
  renderPillRow("queryClass", pills);

  renderFactQuestions(memo);
}

loadAnswerView().catch((err) => {
  console.error(err);
  setText("answerText", "답변 데이터를 읽는 중 오류가 발생했습니다.");
});
```

---

## 3. 샌드박스_근거뷰어

### 3.1 역할

이 노드는 아래를 분리해서 보여준다.

- 공식 근거
- 공식 citation 요약
- 내부 근거
- 내부 유사 사례
- 의견서 스타일 예시 존재 여부
- 웹 보완 링크

### 3.2 HTML

```html
<div class="page">
  <header class="hero">
    <div class="hero__label">노무 상담 결과</div>
    <h1 class="hero__title">근거와 메모</h1>
    <p class="hero__desc">공식 근거, 내부 근거, 내부 유사 사례, 웹 보완 링크를 분리해서 확인합니다.</p>
  </header>

  <main class="grid">
    <section class="card">
      <div class="section-title">공식 근거</div>
      <div id="officialEvidence" class="text-block">공식 근거를 기다리는 중입니다.</div>
    </section>

    <section class="card">
      <div class="section-title">공식 citation 요약</div>
      <div id="officialCitations" class="text-block">citation 정보를 기다리는 중입니다.</div>
    </section>

    <section class="card">
      <div class="section-title">내부 근거</div>
      <div id="internalEvidence" class="text-block">내부 근거를 기다리는 중입니다.</div>
    </section>

    <section class="card">
      <div class="section-title">내부 유사 사례</div>
      <div id="internalCases" class="text-block">내부 유사 사례를 기다리는 중입니다.</div>
    </section>

    <section class="card">
      <div class="section-title">웹 보완 링크</div>
      <ul id="webLinks" class="list"></ul>
    </section>

    <section class="card">
      <div class="section-title">공식 링크</div>
      <ul id="officialLinks" class="list"></ul>
    </section>
  </main>
</div>
```

### 3.3 CSS

```css
:root {
  --bg: #f7f7f5;
  --card: #ffffff;
  --line: #dedbd2;
  --text: #1f2937;
  --sub: #6b7280;
  --accent: #154c79;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Segoe UI", "Noto Sans KR", sans-serif;
  color: var(--text);
  background: linear-gradient(180deg, #f7f8fb 0%, #eef2f6 100%);
}

.page {
  max-width: 1180px;
  margin: 0 auto;
  padding: 24px 16px 40px;
}

.hero {
  margin-bottom: 20px;
}

.hero__label {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 999px;
  background: #e7f0f8;
  color: var(--accent);
  font-size: 12px;
  font-weight: 700;
}

.hero__title {
  margin: 10px 0 8px;
  font-size: 28px;
  line-height: 1.2;
}

.hero__desc {
  margin: 0;
  color: var(--sub);
  font-size: 14px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
}

.card {
  background: var(--card);
  border: 1px solid var(--line);
  border-radius: 18px;
  padding: 18px;
  box-shadow: 0 10px 30px rgba(20, 20, 20, 0.05);
}

.section-title {
  font-size: 13px;
  font-weight: 700;
  color: var(--sub);
  margin-bottom: 10px;
}

.text-block {
  white-space: pre-wrap;
  line-height: 1.68;
  font-size: 14px;
}

.list {
  margin: 0;
  padding-left: 18px;
  line-height: 1.7;
}

.list a {
  color: var(--accent);
  text-decoration: none;
  word-break: break-all;
}

@media (max-width: 860px) {
  .grid {
    grid-template-columns: 1fr;
  }
}
```

### 3.4 JavaScript

```javascript
function firstRow(rows) {
  return Array.isArray(rows) && rows.length > 0 ? rows[0] : {};
}

function parseMaybeJson(value) {
  if (!value) return {};
  if (typeof value === "object") return value;
  try {
    return JSON.parse(value);
  } catch (e) {
    return {};
  }
}

function toArray(value) {
  if (!value) return [];
  if (Array.isArray(value)) return value.filter(Boolean);
  if (typeof value === "string") {
    const trimmed = value.trim();
    if (!trimmed) return [];
    try {
      const parsed = JSON.parse(trimmed);
      if (Array.isArray(parsed)) return parsed.filter(Boolean);
    } catch (e) {}
    return trimmed
      .split(/\n|,/)
      .map((v) => v.trim())
      .filter(Boolean);
  }
  return [];
}

function setText(id, value, fallback = "") {
  const el = document.getElementById(id);
  if (!el) return;
  const text = String(value || "").trim();
  el.textContent = text || fallback;
}

function renderLinks(id, links) {
  const root = document.getElementById(id);
  if (!root) return;
  root.innerHTML = "";
  const items = toArray(links);
  if (items.length === 0) {
    const li = document.createElement("li");
    li.textContent = "표시할 링크가 없습니다.";
    root.appendChild(li);
    return;
  }
  items.forEach((url) => {
    const li = document.createElement("li");
    const a = document.createElement("a");
    a.href = url;
    a.target = "_blank";
    a.rel = "noopener noreferrer";
    a.textContent = url;
    li.appendChild(a);
    root.appendChild(li);
  });
}

async function loadEvidenceView() {
  const mergedRows = await getDataset({ target: 1, limit: 1, page: 1 });
  const memoRows = await getDataset({ target: 2, limit: 1, page: 1 });

  const merged = firstRow(mergedRows);
  const memoRow = firstRow(memoRows);
  const memo = parseMaybeJson(memoRow.legal_memo_json);

  setText("officialEvidence", merged.official_evidence_context, "공식 근거가 아직 생성되지 않았습니다.");
  setText("officialCitations", merged.official_citation_briefs, "공식 citation 요약이 아직 생성되지 않았습니다.");
  setText("internalEvidence", merged.internal_evidence_context, "내부 근거가 아직 생성되지 않았습니다.");

  const internalCases = [];
  if (merged.internal_case_briefs) internalCases.push(String(merged.internal_case_briefs).trim());
  if (Array.isArray(memo.case_analogies) && memo.case_analogies.length > 0) {
    internalCases.push(memo.case_analogies.join("\n- "));
  }
  setText("internalCases", internalCases.filter(Boolean).join("\n\n"), "내부 유사 사례가 아직 생성되지 않았습니다.");

  renderLinks("webLinks", merged.web_source_urls);
  renderLinks("officialLinks", merged.official_source_urls);
}

loadEvidenceView().catch((err) => {
  console.error(err);
  setText("officialEvidence", "근거 데이터를 읽는 중 오류가 발생했습니다.");
});
```

---

## 4. 테스트 순서

1. 애플리케이션 페이지를 만든다.
2. 메뉴 `노무상담`, `근거`를 만든다.
3. `페이지_Elvis_노무상담`, `페이지_Elvis_근거`를 만든다.
4. 두 페이지 노드의 `크기 맞춤`을 `자동 맞춤`으로 둔다.
5. 두 페이지 노드의 `플로우 노드 설정`은 비워 둔다.
6. `페이지_Elvis_노무상담` 안에 `샌드박스_질문입력_Elvis`, `샌드박스_답변뷰어`를 둔다.
7. `페이지_Elvis_근거` 안에 `샌드박스_근거뷰어`를 둔다.
8. `샌드박스_답변뷰어` 입력 포트를 아래 순서로 만든다.
   - `answer_package`
   - `memo_package`
   - `merged_package`
9. `샌드박스_근거뷰어` 입력 포트를 아래 순서로 만든다.
   - `merged_package`
   - `memo_package`
10. 질문 1개를 넣어 실행한다.
11. `노무상담` 메뉴에서 `output_response`가 보이는지 확인한다.
12. `근거` 메뉴에서 `official_source_urls`, `internal_case_briefs`, `web_source_urls`가 보이는지 확인한다.

---

## 5. 처음부터 하지 말아야 하는 것

처음 배포에서는 아래를 하지 않는다.

- 샌드박스가 직접 질문을 만들어 `runNode`로 전체 그래프를 제어하는 구조
- 샌드박스가 `D6R_AIHUB`, `D6R_RULES` 같은 대용량 저장소를 직접 읽는 구조
- 하나의 샌드박스에서 질문 입력, 결과 렌더링, 실행 상태 polling, 노드 실행까지 전부 맡는 구조
- 페이지 노드의 `플로우 노드 설정`으로 질문 실행을 억지로 제어하는 구조

처음에는 아래 기준만 지키면 된다.

- 질문 입력: `샌드박스_질문입력_Elvis`
- 답변 보기: `샌드박스_답변뷰어`
- 근거 보기: `샌드박스_근거뷰어`
- 페이지 노드는 메뉴 컨테이너
