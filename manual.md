# AAI Canvas 법률 챗봇 마스터 런북

최종 업데이트: 2026-03-13
적용 범위: `내부 구조화 DB + 내부 문서 DB + 공식 API + 웹 보완` 4축 검토형 법률 챗봇
이 문서만 사용한다. 이전 순차형 구조, `16A-1`, `16A-2`, 구형 `18C/18D`, AIHub 전수 스캔형 설명은 모두 폐기한다.

## 1. 목표

만들려는 챗봇은 아래 4단 검토 구조를 가진다.

1. 내부 구조화 DB 검토징계DB 엑셀의 사건/기준/분류 시트를 읽는다.
2. 내부 문서 DB 검토AIHub prechunked 문서, 사규 PDF, 그리고 의견서 문안 저장소를 문서 DB로 적재하고, 질문 시 top-k 문서만 다시 읽는다.
3. 공식 API 검토법령, 판례, 행정해석, 노동위원회 결정문(`nlrc`)을 조회한다.
4. 웹 보완 검토
   앞의 3축이 비거나 약할 때 보완 검색을 한다.

최종 답변은 항상 다음 순서로 합성한다.

- 내부 구조화 DB
- 내부 문서 DB
- 공식 API
- 웹 보완

단, 내부 판단과 명시적 법령/판례/행정해석이 충돌하면 그 충돌을 숨기지 말고 공식근거 기준의 법적 리스크를 함께 설명한다.

## 2. 전체 구조

문서는 2개 그래프로 나뉜다.

1. 1회성 문서 DB 적재 그래프
   AIHub, 규정/가이드, 의견서 스타일 예시를 각 저장소에 넣는다.
2. 런타임 질의응답 그래프
   사용자의 질문 1건을 4축으로 검토하고 답변한다.

### 2.1 1회성 문서 DB 적재 그래프

```text
데이터셋_문서근거_AIHUB580_판결_적재용
  -> D4_판결. 파이썬_문서메타데이터생성
  -> D5_판결. 텍스트 임베딩_문서벡터화
  -> D6W_AIHUB_판결. DB저장_AIHUB저장소_쓰기

데이터셋_문서근거_AIHUB580_약관_적재용
  -> D4_약관. 파이썬_문서메타데이터생성
  -> D5_약관. 텍스트 임베딩_문서벡터화
  -> D6W_AIHUB_약관. DB저장_AIHUB저장소_쓰기

데이터셋_문서근거_RULES_적재용
  -> D4_RULES. 파이썬_문서메타데이터생성
  -> D5_RULES. 텍스트 임베딩_문서벡터화
  -> D6W_RULES. DB저장_규정가이드저장소_쓰기

데이터셋_문서근거_의견서_적재용
  -> D4_의견서. 파이썬_문서메타데이터생성
  -> D5_의견서. 텍스트 임베딩_문서벡터화
  -> D6W_의견서. DB저장_의견서문안저장소_쓰기
```

### 2.2 런타임 질의응답 그래프

```text
에이전트
  -> 에이전트 메시지 가로채기
  -> 2A. 파이썬_최신사용자질문추출
  -> 2B. 파이썬_최근대화맥락추출
  -> 2C. 데이터 연결_질문맥락병합
  -> 3. 에이전트 프롬프트_질의정규화
  -> 4. 파이썬_정규화파싱
  -> 5. 파이썬_URL생성
  -> 17A. 파이썬_질문기준행생성

공식 branch
5 -> 6/7/8/26A
6/7/8/26A -> 9/10/11/26B
9/10/11/26B -> 12/13/14/26C
12/13/14 -> 15 -> 16
16 + 26C -> 26D
26D -> 17_공식
17_공식 + 17A -> 17B_공식
17B_공식 -> 18_공식패키징

내부 구조화 branch
데이터셋_내부근거_엑셀 -> 16A
16A -> 17_내부구조화
17_내부구조화 + 17A -> 17B_내부구조화
17B_내부구조화 -> 18_내부패키징_구조화

내부 문서 공통 준비
17A -> 17D1. 프롬프트_문서검색질문생성
17D1 -> 17D2. 텍스트 임베딩_질문벡터화

AIHub 문서 branch
D6R_AIHUB. 데이터 저장소_AIHUB저장소_읽기 + 17D2 -> 17D3_AIHUB. 파이썬_벡터유사문서TOPK
17D3_AIHUB -> 17D4_AIHUB. 열 선택_문서근거
17D4_AIHUB + 17A -> 18_내부패키징_AIHUB

규정/가이드 문서 branch
D6R_RULES. 데이터 저장소_규정가이드저장소_읽기 + 17D2 -> 17D3_RULES. 파이썬_벡터유사문서TOPK
17D3_RULES -> 17D4_RULES. 열 선택_문서근거
17D4_RULES + 17A -> 18_내부패키징_RULES

의견서 문안 branch
D6R_의견서. 데이터 저장소_의견서문안저장소_읽기 + 17D2 + 17A -> 17D3_의견서. 파이썬_의견서유사문서TOPK
17D3_의견서 -> 17D4_의견서. 열 선택_의견서근거
17D4_의견서 + 17A -> 18_내부패키징_의견서

내부 병합
18_내부패키징_AIHUB + 18_내부패키징_RULES -> 18I_DOC
18_내부패키징_구조화 + 18I_DOC + 18_내부패키징_의견서 -> 18I1

웹 branch
17A -> 18W_A -> 18W_B

ELABOR 보조검색 branch
17A -> 18E_A -> 18E_B -> 18E_C

최종 병합
18_공식패키징 + 18I1 -> 18M1
18M1 + 18W_B -> 18M2
18M2 + 18E_C -> 18M3
18M3 -> 19A -> 19B -> 20 -> 21 -> 22/23
```

## 3. 공통 규칙

### 3.1 Python 노드 제약

- `return`, `yield`, `compile`, `bytes`, `chr`, `format` 같은 컨텍스트 의존 문법은 쓰지 않는다.
- 함수 정의(`def`)는 가능한 한 쓰지 않는다.
- 입력 포트가 여러 개면 기본 `dataset` 외 입력은 `x` 리스트로 읽는다.
- 출력은 항상 `result = pd.DataFrame(...)` 로 끝낸다.
- 두 데이터를 단순히 합치는 목적이면 Python보다 `데이터 연결` 노드를 먼저 쓴다.
- 문서에서 `A + B -> C`라고 적혀 있어도, `C`가 `데이터 연결_...` 노드라면 Python 입력 포트 추가가 필요 없다.
- 입력 포트를 직접 추가해야 하는 노드는 문서에 `입력 포트:` 항목으로 따로 적는다.

권장 기준:

- `17B_공식`, `17B_내부구조화` 같은 **단순 결합**은 `데이터 연결`
- `17D3_AIHUB`, `17D3_RULES`, `17D3_의견서`, `18_내부패키징_AIHUB`, `18_내부패키징_RULES`, `18_내부패키징_의견서`, `18I_DOC`, `18I1`, `18M1`, `18M2`, `18M3` 같은 **벡터검색/병합 후 집계/가공**은 `2입력 이상 Python`

### 3.2 프롬프트 노드 규칙

- 프롬프트 노드는 열 이름만 적으면 실제 값을 모른다.
- 실제 값을 넣으려면 반드시 `{{column_name}}` 를 쓴다.
- 특히 아래 노드는 `{{}}` 주입을 반드시 쓴다.
  - `3`
  - `17D1`
  - `18W_A`
  - `19`
  - `22`

### 3.3 Data Connect 규칙

- Data Connect는 2입력만 지원한다.
- 3개 이상 병합은 체인으로 만든다.
- `18I1`, `18M1`, `18M2`, `18M3`는 Data Connect가 아니라 Python 병합 노드를 쓴다.
- `노드5`는 출력 포트를 늘리지 않는다. 기본 `output` 하나를 여러 노드에 동시에 연결한다.

### 3.4 Worker 규칙

- 기본 Worker URL: `https://law-retry-proxy.tud1211.workers.dev`
- 현재 운영 기준 Worker는 `(구)retry_proxy_cloudflare_worker.js` 기준이다.
- `노드5`, `노드9`, `노드10`, `노드11`, `노드26B`는 Worker 경로를 기본으로 쓴다.

## 4. 에이전트 / 챗 UI 층

### 4.1 에이전트 / 챗 UI 시스템 프롬프트

시스템 프롬프트 입력란이 있으면 아래 블록 전체를 넣는다.

```text
역할:
- 너는 법률정보 챗봇의 대화 인터페이스다.
- 내부 workflow가 이미 만든 답변을 사용자에게 자연스럽고 읽기 쉽게 전달한다.
- 법률 판단, 근거 우선순위, 차단 여부는 workflow가 담당한다.

핵심 원칙:
1) workflow가 만든 결론과 사실 범위를 벗어나 새 사실을 추가하지 않는다.
2) 법령명, 판례명, 사건번호, 날짜, 링크를 새로 지어내지 않는다.
3) 메타 안내문, 템플릿 재요청, 입력값 요구 문구를 출력하지 않는다.
4) 딱딱한 나열보다 자연스러운 한국어 문장으로 정리한다.
5) 내부DB가 우선 검토 대상이라는 점을 유지하되, 공식근거 충돌이 있으면 그 차이를 숨기지 않는다.

문체:
- 과장하지 않는다.
- 상담원식 군더더기 표현을 줄인다.
- 사용자가 바로 이해하고 다음 행동을 정할 수 있게 쓴다.
```

### 4.2 2A. 파이썬_최신사용자질문추출

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
result = dataset.tail(1).copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

if "content" in result.columns:
    result["latest_user_question"] = result["content"]
elif "query" in result.columns:
    result["latest_user_question"] = result["query"]
else:
    result["latest_user_question"] = ""

result = result[["latest_user_question"]]
```

### 4.3 2B. 파이썬_최근대화맥락추출

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
work = df.copy()

if "content" in work.columns:
    keep = []
    for v in work["content"].tolist():
        s = "" if v is None else str(v).strip()
        keep.append(bool(s and s.lower() != "nan"))
    try:
        work = work.loc[keep].copy()
    except Exception:
        pass

if len(work) > 6:
    work = work.tail(6).copy()

if len(work) > 1:
    work = work.iloc[:-1].copy()
else:
    work = work.iloc[0:0].copy()

lines = []
for _, r in work.iterrows():
    role = ""
    if "role" in work.columns:
        role = str(r.get("role", "") or "").strip()
    text = str(r.get("content", "") or "").strip()
    if not text:
        continue
    if role:
        lines.append(role + ": " + text)
    else:
        lines.append(text)

ctx = "\n".join(lines).strip()
result = pd.DataFrame([{"conversation_context": ctx}], columns=["conversation_context"])
```

### 4.4 2C. 데이터 연결_질문맥락병합

- 입력1: `2A`
- 입력2: `2B`
- 축(axis): `수평`
- 병합 모드(merge mode): `모든 열 사용`
- 대상 레이블(target label): `None`

## 5. 공통 정규화 층

### 5.1 3. 에이전트 프롬프트_질의정규화

- 입력 컬럼: `latest_user_question`, `conversation_context`
- 모델: `gpt-5.2` 또는 `gpt-5-mini`
- Output column name: `normalized_json`
- num response: `1`
- Include Question Prompt in output: `ON`

```text
최신 사용자 질문: {{latest_user_question}}
최근 대화 맥락: {{conversation_context}}

역할: 최신 사용자 질문을 공식 검색용 구조로 정규화
반드시 JSON만 출력한다.

스키마:
{
  "case_type": "",
  "issue_keywords": [""],
  "law_query": "",
  "precedent_query": "",
  "interpretation_query": "",
  "incident_date": "YYYY-MM-DD 또는 빈 문자열",
  "date_from": "YYYYMMDD 또는 빈 문자열",
  "date_to": "YYYYMMDD 또는 빈 문자열",
  "must_have": ["법령명", "조문", "시행일", "법원", "선고일", "사건번호"],
  "query_class": "legal_analysis | internal_db_only | mixed"
}

규칙:
1) latest_user_question를 우선 기준으로 사용한다.
2) conversation_context는 최신 질문이 `그럼`, `이 경우`, `저 상황`, `위 사례`처럼 맥락 의존형일 때만 보조적으로 사용한다.
3) issue_keywords는 최소 1개 이상 채운다.
4) law_query, precedent_query, interpretation_query 중 최소 1개는 채운다.
5) 내부 통계/내부DB 없이는 답할 수 없으면 internal_db_only, 내부DB와 법률 검토가 동시에 필요하면 mixed, 그 외는 legal_analysis로 둔다.
6) generic 출력 금지: `법률 질의`, `법률 상담`, `사용자 질의 누락`, `적용범위`, `사례`, `공식 해석`
7) 질문에 구체 행위가 있으면 그대로 살린다. 예: `사내 음주`, `직장 내 괴롭힘`, `무단결근`
8) `징계처분 불복 절차`, `불복 절차`, `재심 절차`처럼 절차성 질문이면 issue_keywords에 절차어를 남기되, 검색어는 `징계`, `부당해고`, `노동위원회`, `취업규칙`처럼 검색 가능한 핵심어로 축약한다.
9) 질문이 특별법 이슈면 `law_query`를 근로기준법 하나로 축약하지 않는다. 예:
   - `직장 내 성희롱` -> `남녀고용평등법 직장 내 성희롱 조치 피해자 의견 시행령 시행규칙`
   - `육아휴직`, `육아기 근로시간 단축` -> `남녀고용평등법 육아휴직 육아기 근로시간 단축 시행령 시행규칙`
   - `산업재해`, `안전보건` -> 관련 특별법 중심으로 쓴다.
10) 질문에 `시행령`, `시행규칙`, `고시`, `예규`, `훈령`이 직접 들어 있거나, 해당 하위 규정이 중요해 보이는 사안이면 `law_query`에 그 단어를 함께 넣는다.
```

### 5.2 4. 파이썬_정규화파싱

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

if len(df) > 1:
    df = df.tail(1).copy()

if "normalized_json" in df.columns:
    source_col = "normalized_json"
elif "output_response" in df.columns:
    source_col = "output_response"
else:
    source_col = None

rows = []
for row in df.to_dict(orient="records"):
    raw = row.get(source_col, "") if source_col else ""
    obj = {}

    latest_question = str(row.get("latest_user_question", "") or "").strip()
    conversation_context = str(row.get("conversation_context", "") or "").strip()

    if isinstance(raw, dict):
        obj = raw
    else:
        text = "" if raw is None else str(raw).strip()
        if text and text.lower() != "nan":
            try:
                obj = json.loads(text)
            except Exception:
                m = re.search(r"\{.*\}", text, re.S)
                if m:
                    try:
                        obj = json.loads(m.group(0))
                    except Exception:
                        obj = {}

    kws = obj.get("issue_keywords", [])
    if isinstance(kws, str):
        kws = [x.strip() for x in re.split(r"[,\s/|]+", kws) if x.strip()]
    elif isinstance(kws, list):
        cleaned = []
        for x in kws:
            if x is None:
                continue
            sx = str(x).strip()
            if sx and sx.lower() != "nan":
                cleaned.append(sx)
        kws = cleaned
    else:
        kws = []

    stop_meta = [
        "관련", "법령", "조문", "검색", "확인", "적용", "가능한", "법률",
        "시행일", "파악", "및", "각", "대한", "검토", "여부", "기준",
        "법령해석", "행정해석", "판례", "해설", "조문의", "법령의", "조항",
        "대법원", "고등법원", "지방법원", "법원", "판결", "결정", "선고",
        "사건", "사건번호", "문서", "고시", "예규", "훈령", "회시", "질의",
        "적용범위", "요건", "가능성", "판단", "근거", "입증", "절차", "단계", "방법"
    ]

    q_clean = re.sub(r"[^0-9A-Za-z가-힣\s]", " ", latest_question)
    q_clean = re.sub(r"\s+", " ", q_clean).strip()
    q_tokens = []
    for t in q_clean.split(" "):
        s = str(t).strip()
        s = re.sub(r"(으로|에서|에게|까지|부터|처럼|보다|마다|의|에|이|가|은|는|을|를|와|과|도|만|로)$", "", s)
        if s and s not in stop_meta and s not in ["있어", "될까", "되나", "되나요", "알려줘", "어떻게", "뭐야", "수", "할", "있는가"]:
            q_tokens.append(s)

    if not kws and q_tokens:
        if len(q_tokens) >= 2:
            kws = [q_tokens[0], q_tokens[1]]
        else:
            kws = [q_tokens[0]]

    if not kws:
        kws = ["징계"]

    scenario_term = ""
    if "직장 내 괴롭힘" in latest_question or (("직장" in latest_question) and ("괴롭힘" in latest_question)):
        scenario_term = "직장 내 괴롭힘"
    elif "음주" in latest_question:
        scenario_term = "사내 음주"
    elif "무단결근" in latest_question:
        scenario_term = "무단결근"
    elif "성희롱" in latest_question:
        scenario_term = "성희롱"
    elif "폭행" in latest_question:
        scenario_term = "폭행"
    elif "횡령" in latest_question:
        scenario_term = "횡령"

    if scenario_term:
        kws = [scenario_term, "징계"]

    law_query = str(obj.get("law_query", "") or "").strip()
    precedent_query = str(obj.get("precedent_query", "") or "").strip()
    interpretation_query = str(obj.get("interpretation_query", "") or "").strip()

    if not law_query:
        law_query = "근로기준법"
    if not precedent_query:
        precedent_query = scenario_term if scenario_term else "징계"
    if not interpretation_query:
        interpretation_query = "징계"

    if "불복" in latest_question or "구제" in latest_question or "재심" in latest_question or "절차" in latest_question:
        if "해고" in latest_question or "부당해고" in latest_question:
            law_query = "근로기준법"
            precedent_query = "부당해고"
            interpretation_query = "부당해고"
        else:
            law_query = "근로기준법"
            precedent_query = "징계"
            interpretation_query = "징계"

    if scenario_term == "사내 음주":
        law_query = "근로기준법"
        precedent_query = "음주"
        interpretation_query = "징계"

    if ("직장 내 괴롭힘" in latest_question or (("직장" in latest_question) and ("괴롭힘" in latest_question))) and ("전보" in latest_question):
        law_query = "근로기준법 직장 내 괴롭힘 불리한 처우"
        precedent_query = "직장 내 괴롭힘 불리한 처우 전보"
        interpretation_query = "직장 내 괴롭힘 불리한 처우"

    if "불리한 처우" in latest_question:
        law_query = "근로기준법 직장 내 괴롭힘 불리한 처우"
        if "전보" in latest_question:
            precedent_query = "직장 내 괴롭힘 불리한 처우 전보"
        else:
            precedent_query = "직장 내 괴롭힘 불리한 처우"
        interpretation_query = "직장 내 괴롭힘 불리한 처우"

    if "성희롱" in latest_question:
        law_query = "남녀고용평등법 직장 내 성희롱 조치 피해자 의견 시행령 시행규칙"
        if not precedent_query or precedent_query in ["징계", "음주"]:
            precedent_query = "직장 내 성희롱 불리한 처우 징계"
        interpretation_query = "직장 내 성희롱 피해자 의견 조치"

    if ("남녀고용평등" in latest_question) or ("직장 내 성희롱" in latest_question):
        if "시행령" in latest_question or "시행규칙" in latest_question:
            law_query = "남녀고용평등법 직장 내 성희롱 시행령 시행규칙"

    if "육아휴직" in latest_question or "육아기 근로시간 단축" in latest_question:
        law_query = "남녀고용평등법 육아휴직 육아기 근로시간 단축 시행령 시행규칙"
        if not interpretation_query or interpretation_query == "징계":
            interpretation_query = "육아휴직 육아기 근로시간 단축"

    if "시행령" in latest_question and "시행령" not in law_query:
        law_query = (law_query + " 시행령").strip()
    if "시행규칙" in latest_question and "시행규칙" not in law_query:
        law_query = (law_query + " 시행규칙").strip()

    query_class = str(obj.get("query_class", "legal_analysis") or "legal_analysis").strip().lower()
    if query_class not in ["legal_analysis", "internal_db_only", "mixed"]:
        query_class = "legal_analysis"

    must_have = obj.get("must_have", [])
    if isinstance(must_have, str):
        must_have = [x.strip() for x in must_have.split(",") if x.strip()]
    elif not isinstance(must_have, list):
        must_have = []
    if not must_have:
        must_have = ["법령명", "조문", "시행일", "법원", "선고일", "사건번호"]

    rows.append({
        "latest_user_question": latest_question,
        "conversation_context": conversation_context,
        "issue_keywords_csv": ", ".join(kws),
        "law_query": law_query,
        "precedent_query": precedent_query,
        "interpretation_query": interpretation_query,
        "date_from": str(obj.get("date_from", "") or "").strip(),
        "date_to": str(obj.get("date_to", "") or "").strip(),
        "must_have": ",".join(must_have),
        "query_class": query_class
    })

result = pd.DataFrame(rows, columns=[
    "latest_user_question", "conversation_context",
    "issue_keywords_csv", "law_query", "precedent_query", "interpretation_query",
    "date_from", "date_to", "must_have", "query_class"
])
```

### 5.3 5. 파이썬_URL생성

- 입력 포트: `dataset`
- 출력 포트: `output`
- `output` 하나를 `6`, `7`, `8`, `26A`, `17A`에 동시에 연결한다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

worker_base = "https://law-retry-proxy.tud1211.workers.dev"

rows = []
for _, r in df.iterrows():
    latest_user_question = str(r.get("latest_user_question", "") or "").strip()
    conversation_context = str(r.get("conversation_context", "") or "").strip()
    issue_keywords_csv = str(r.get("issue_keywords_csv", "") or "").strip()

    law_raw = str(r.get("law_query", "") or "").strip() or "근로기준법"
    prec_raw = str(r.get("precedent_query", "") or "").strip() or "징계"
    expc_raw = str(r.get("interpretation_query", "") or "").strip() or "징계"
    nlrc_raw = prec_raw if prec_raw else "징계"

    if "불복" in latest_user_question or "구제" in latest_user_question or "재심" in latest_user_question or "절차" in latest_user_question:
        law_raw = "근로기준법"
        if "해고" in latest_user_question or "부당해고" in latest_user_question:
            prec_raw = "부당해고"
            expc_raw = "부당해고"
            nlrc_raw = "부당해고"
        else:
            prec_raw = "징계"
            expc_raw = "징계"
            nlrc_raw = "징계"

    if "음주" in latest_user_question:
        law_raw = "근로기준법"
        prec_raw = "음주"
        expc_raw = "징계"
        nlrc_raw = "징계"

    if (("직장 내 괴롭힘" in latest_user_question) or (("직장" in latest_user_question) and ("괴롭힘" in latest_user_question))) and ("전보" in latest_user_question):
        law_raw = "근로기준법 직장 내 괴롭힘 불리한 처우"
        prec_raw = "직장 내 괴롭힘 불리한 처우 전보"
        expc_raw = "직장 내 괴롭힘 불리한 처우"
        nlrc_raw = "직장 내 괴롭힘 전보"

    if "불리한 처우" in latest_user_question and "전보" not in latest_user_question:
        law_raw = "근로기준법 직장 내 괴롭힘 불리한 처우"
        prec_raw = "직장 내 괴롭힘 불리한 처우"
        expc_raw = "직장 내 괴롭힘 불리한 처우"
        nlrc_raw = "직장 내 괴롭힘 불리한 처우"

    if "성희롱" in latest_user_question:
        law_raw = "남녀고용평등법 직장 내 성희롱 조치 피해자 의견 시행령 시행규칙"
        if not prec_raw or prec_raw in ["징계", "음주"]:
            prec_raw = "직장 내 성희롱 불리한 처우 징계"
        expc_raw = "직장 내 성희롱 피해자 의견 조치"
        nlrc_raw = "직장 내 성희롱 징계"

    if "육아휴직" in latest_user_question or "육아기 근로시간 단축" in latest_user_question:
        law_raw = "남녀고용평등법 육아휴직 육아기 근로시간 단축 시행령 시행규칙"
        if not expc_raw or expc_raw == "징계":
            expc_raw = "육아휴직 육아기 근로시간 단축"

    if "시행령" in latest_user_question and "시행령" not in law_raw:
        law_raw = (law_raw + " 시행령").strip()
    if "시행규칙" in latest_user_question and "시행규칙" not in law_raw:
        law_raw = (law_raw + " 시행규칙").strip()

    law_q = law_raw.replace(" ", "%20")
    prec_q = prec_raw.replace(" ", "%20")
    expc_q = expc_raw.replace(" ", "%20")
    nlrc_q = nlrc_raw.replace(" ", "%20")

    rows.append({
        "query_class": str(r.get("query_class", "legal_analysis") or "legal_analysis").strip(),
        "issue_keywords_csv": issue_keywords_csv,
        "law_query": law_raw,
        "precedent_query": prec_raw,
        "interpretation_query": expc_raw,
        "latest_user_question": latest_user_question,
        "conversation_context": conversation_context,
        "law_query_used": law_raw,
        "prec_query_used": prec_raw,
        "expc_query_used": expc_raw,
        "nlrc_query_used": nlrc_raw,
        "law_list_url": f"{worker_base}/drf/search?target=eflaw&query={law_q}&display=15&page=1&sort=ddes&oc=tud1211",
        "prec_list_url": f"{worker_base}/drf/search?target=prec&query={prec_q}&display=10&page=1&sort=ddes&oc=tud1211",
        "expc_list_url": f"{worker_base}/drf/search?target=expc&query={expc_q}&display=10&page=1&sort=ddes&oc=tud1211",
        "nlrc_list_url": f"{worker_base}/drf/search?target=nlrc&query={nlrc_q}&display=10&page=1&sort=ddes&oc=tud1211",
        "api_method": "GET",
        "api_headers_json": "{\"Accept\":\"application/json\",\"User-Agent\":\"Mozilla/5.0\"}",
        "api_body": ""
    })

result = pd.DataFrame(rows, columns=[
    "query_class", "issue_keywords_csv", "law_query", "precedent_query", "interpretation_query",
    "latest_user_question", "conversation_context",
    "law_query_used", "prec_query_used", "expc_query_used", "nlrc_query_used",
    "law_list_url", "prec_list_url", "expc_list_url", "nlrc_list_url",
    "api_method", "api_headers_json", "api_body"
])
```

### 5.4 17A. 파이썬_질문기준행생성

- 입력 포트: `dataset`
- 출력 포트: `output`
- 공통 질문 앵커 노드다. 웹 branch만이 아니라 공식 branch와 내부 branch도 이 행을 함께 본다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
if len(df) > 1:
    df = df.tail(1).copy()

rows = []
for _, r in df.iterrows():
    qclass = str(r.get("query_class", "") or "").strip().lower()
    if qclass not in ["legal_analysis", "internal_db_only", "mixed"]:
        qclass = "legal_analysis"

    latest_question = str(r.get("latest_user_question", "") or "").strip()
    conversation_context = str(r.get("conversation_context", "") or "").strip()
    issue_keywords_csv = str(r.get("issue_keywords_csv", "") or "").strip()

    anchor_lines = []
    if latest_question:
        anchor_lines.append("질문: " + latest_question)
    if conversation_context:
        anchor_lines.append("맥락: " + conversation_context)
    if issue_keywords_csv:
        anchor_lines.append("키워드: " + issue_keywords_csv)

    rows.append({
        "query_class": qclass,
        "issue_keywords_csv": issue_keywords_csv,
        "law_query": str(r.get("law_query", "") or "").strip(),
        "precedent_query": str(r.get("precedent_query", "") or "").strip(),
        "interpretation_query": str(r.get("interpretation_query", "") or "").strip(),
        "latest_user_question": latest_question,
        "conversation_context": conversation_context,
        "source_type": "query_fallback",
        "source_url": "",
        "chunk_text": "\n".join(anchor_lines).strip()
    })

result = pd.DataFrame(rows, columns=[
    "query_class", "issue_keywords_csv", "law_query", "precedent_query", "interpretation_query",
    "latest_user_question", "conversation_context",
    "source_type", "source_url", "chunk_text"
])
```

## 6. 공식 branch

### 6.1 Custom API 고정 설정

다음 노드는 모두 같은 방식으로 설정한다.

- `6. API_법령목록`URL column: `law_list_url`
- `7. API_판례목록`URL column: `prec_list_url`
- `8. API_해석목록`URL column: `expc_list_url`
- `26A. API_노동위목록`URL column: `nlrc_list_url`
- `12. API_법령본문`URL column: `law_detail_url`
- `13. API_판례본문`URL column: `prec_detail_url`
- `14. API_해석본문`URL column: `expc_detail_url`
- `26C. API_노동위본문`
  URL column: `nlrc_detail_url`

공통값:

- 요청 모드: `데이터셋 요청`
- Method column: `api_method`
- Headers column: `api_headers_json`
- Body column: `api_body`
- 자동 변환(JSON <-> CSV): `ON`

### 6.2 9. 파이썬_법령상세URL_TOPK

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
base = "https://law-retry-proxy.tud1211.workers.dev"
safe_health_url = base + "/health"

rows = []
for _, r in df.iterrows():
    rec = r.to_dict()
    mst = ""

    for k in ["MST", "법령MST", "법령일련번호", "mst"]:
        if k in rec:
            v = rec.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                mst = s
                break

    if not mst:
        blob = " ".join([str(v) for v in rec.values() if v is not None])
        m = re.search(r"(?:MST|법령MST|법령일련번호)[^0-9]{0,10}(\d+)", blob)
        if m:
            mst = m.group(1)

    detail_url = safe_health_url
    source_type = "official_law_empty"
    if mst:
        detail_url = f"{base}/drf/detail?target=eflaw&MST={mst}&type=JSON&oc=tud1211"
        source_type = "official_law"

    rows.append({
        "query_class": str(r.get("query_class", "") or "").strip(),
        "issue_keywords_csv": str(r.get("issue_keywords_csv", "") or "").strip(),
        "law_query": str(r.get("law_query", "") or "").strip(),
        "precedent_query": str(r.get("precedent_query", "") or "").strip(),
        "interpretation_query": str(r.get("interpretation_query", "") or "").strip(),
        "latest_user_question": str(r.get("latest_user_question", "") or "").strip(),
        "conversation_context": str(r.get("conversation_context", "") or "").strip(),
        "source_type": source_type,
        "law_detail_url": detail_url,
        "api_method": "GET",
        "api_headers_json": "{\"Accept\":\"application/json\",\"User-Agent\":\"Mozilla/5.0\"}",
        "api_body": ""
    })

result = pd.DataFrame(rows)
```

### 6.3 10. 파이썬_판례상세URL_TOPK

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
base = "https://law-retry-proxy.tud1211.workers.dev"
safe_health_url = base + "/health"

rows = []
for _, r in df.iterrows():
    rec = r.to_dict()
    item_id = ""

    for k in ["판례정보일련번호", "ID", "id", "prec_id"]:
        if k in rec:
            v = rec.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                item_id = s
                break

    if not item_id:
        blob = " ".join([str(v) for v in rec.values() if v is not None])
        m = re.search(r"(?:판례정보일련번호|ID|id|prec_id)[^0-9]{0,10}(\d+)", blob)
        if m:
            item_id = m.group(1)

    detail_url = safe_health_url
    source_type = "official_prec_empty"
    if item_id:
        detail_url = f"{base}/drf/detail?target=prec&ID={item_id}&type=JSON&oc=tud1211"
        source_type = "official_prec"

    rows.append({
        "query_class": str(r.get("query_class", "") or "").strip(),
        "issue_keywords_csv": str(r.get("issue_keywords_csv", "") or "").strip(),
        "law_query": str(r.get("law_query", "") or "").strip(),
        "precedent_query": str(r.get("precedent_query", "") or "").strip(),
        "interpretation_query": str(r.get("interpretation_query", "") or "").strip(),
        "latest_user_question": str(r.get("latest_user_question", "") or "").strip(),
        "conversation_context": str(r.get("conversation_context", "") or "").strip(),
        "source_type": source_type,
        "prec_detail_url": detail_url,
        "api_method": "GET",
        "api_headers_json": "{\"Accept\":\"application/json\",\"User-Agent\":\"Mozilla/5.0\"}",
        "api_body": ""
    })

result = pd.DataFrame(rows)
```

### 6.4 11. 파이썬_해석상세URL_TOPK

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
base = "https://law-retry-proxy.tud1211.workers.dev"
safe_health_url = base + "/health"

rows = []
for _, r in df.iterrows():
    rec = r.to_dict()
    item_id = ""

    for k in ["행정해석ID", "ID", "id", "expc_id"]:
        if k in rec:
            v = rec.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                item_id = s
                break

    if not item_id:
        blob = " ".join([str(v) for v in rec.values() if v is not None])
        m = re.search(r"(?:행정해석ID|ID|id|expc_id)[^0-9]{0,10}(\d+)", blob)
        if m:
            item_id = m.group(1)

    detail_url = safe_health_url
    source_type = "official_expc_empty"
    if item_id:
        detail_url = f"{base}/drf/detail?target=expc&ID={item_id}&type=JSON&oc=tud1211"
        source_type = "official_expc"

    rows.append({
        "query_class": str(r.get("query_class", "") or "").strip(),
        "issue_keywords_csv": str(r.get("issue_keywords_csv", "") or "").strip(),
        "law_query": str(r.get("law_query", "") or "").strip(),
        "precedent_query": str(r.get("precedent_query", "") or "").strip(),
        "interpretation_query": str(r.get("interpretation_query", "") or "").strip(),
        "latest_user_question": str(r.get("latest_user_question", "") or "").strip(),
        "conversation_context": str(r.get("conversation_context", "") or "").strip(),
        "source_type": source_type,
        "expc_detail_url": detail_url,
        "api_method": "GET",
        "api_headers_json": "{\"Accept\":\"application/json\",\"User-Agent\":\"Mozilla/5.0\"}",
        "api_body": ""
    })

result = pd.DataFrame(rows)
```

### 6.5 26B. 파이썬_노동위상세URL_TOPK

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
base = "https://law-retry-proxy.tud1211.workers.dev"
safe_health_url = base + "/health"

rows = []
for _, r in df.iterrows():
    rec = r.to_dict()
    item_id = ""
    for k in ["결정문 일련번호", "ID", "id", "nlrc_id"]:
        if k in rec:
            v = rec.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                item_id = s
                break
    if not item_id:
        blob = " ".join([str(v) for v in rec.values() if v is not None])
        m = re.search(r"(?:결정문\s*일련번호|ID|id|nlrc_id)[^0-9]{0,10}(\d+)", blob)
        if m:
            item_id = m.group(1)

    detail_url = safe_health_url
    source_type = "official_nlrc_empty"
    if item_id:
        detail_url = f"{base}/drf/detail?target=nlrc&ID={item_id}&type=JSON&oc=tud1211"
        source_type = "official_nlrc"

    rows.append({
        "query_class": str(r.get("query_class", "") or "").strip(),
        "issue_keywords_csv": str(r.get("issue_keywords_csv", "") or "").strip(),
        "law_query": str(r.get("law_query", "") or "").strip(),
        "precedent_query": str(r.get("precedent_query", "") or "").strip(),
        "interpretation_query": str(r.get("interpretation_query", "") or "").strip(),
        "latest_user_question": str(r.get("latest_user_question", "") or "").strip(),
        "conversation_context": str(r.get("conversation_context", "") or "").strip(),
        "source_type": source_type,
        "nlrc_detail_url": detail_url,
        "api_method": "GET",
        "api_headers_json": "{\"Accept\":\"application/json\",\"User-Agent\":\"Mozilla/5.0\"}",
        "api_body": ""
    })

result = pd.DataFrame(rows)
```

### 6.6 공식 Data Connect 체인

`15. 데이터 연결_근거통합1`

- 입력1: `12. API_법령본문`
- 입력2: `13. API_판례본문`
- 축(axis): `수직`
- 병합 모드: `모든 열 사용`
- 대상 레이블: `None`

`16. 데이터 연결_근거통합2`

- 입력1: `15`
- 입력2: `14. API_해석본문`
- 축(axis): `수직`
- 병합 모드: `모든 열 사용`
- 대상 레이블: `None`

`26D. 데이터 연결_공식근거통합3_노동위`

- 입력1: `16`
- 입력2: `26C. API_노동위본문`
- 축(axis): `수직`
- 병합 모드: `모든 열 사용`
- 대상 레이블: `None`

### 6.7 17_공식 / 17B_공식 / 18_공식패키징

#### 17_공식. 파이썬_근거청킹_공식

- 입력 포트: `dataset`
- 출력 포트: `output`
- `17_내부구조화`도 이 코드와 동일하게 쓴다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

chunk_size = 700
stride = 560
text_min_len = 10

rows = []
for _, r in df.iterrows():
    row = r.to_dict()

    qclass = str(row.get("query_class", "") or "").strip().lower()
    if qclass not in ["legal_analysis", "internal_db_only", "mixed"]:
        qclass = "legal_analysis"

    safe_question = str(row.get("latest_user_question", "") or "").strip()
    context_text = str(row.get("conversation_context", "") or "").strip()

    src = str(row.get("source_type", "") or "").strip()
    url = str(row.get("source_url", "") or "").strip()

    title = ""
    for k in ["법령명", "제목", "표제", "판례명", "사건명", "문서명", "판정사항", "건명"]:
        if k in row:
            s = "" if row.get(k) is None else str(row.get(k)).strip()
            if s and s.lower() != "nan":
                title = s
                break

    article_ref = ""
    for k in ["조문", "조항", "조문번호", "관련조문", "법조문"]:
        if k in row:
            s = "" if row.get(k) is None else str(row.get(k)).strip()
            if s and s.lower() != "nan":
                article_ref = s
                break

    court_name = ""
    for k in ["법원명", "법원", "기관명", "기관"]:
        if k in row:
            s = "" if row.get(k) is None else str(row.get(k)).strip()
            if s and s.lower() != "nan":
                court_name = s
                break

    decision_date = ""
    for k in ["선고일자", "선고일", "결정일자", "결정일", "시행일", "등록일", "date"]:
        if k in row:
            s = "" if row.get(k) is None else str(row.get(k)).strip()
            if s and s.lower() != "nan":
                decision_date = s
                break

    case_number = ""
    for k in ["사건번호", "판례번호", "번호", "안건번호"]:
        if k in row:
            s = "" if row.get(k) is None else str(row.get(k)).strip()
            if s and s.lower() != "nan":
                case_number = s
                break

    summary_hint = ""
    for k in ["판결요지", "판시사항", "판정요지", "판정결과", "요지", "내용요약", "주요내용"]:
        if k in row:
            s = "" if row.get(k) is None else str(row.get(k)).strip()
            if s and s.lower() != "nan":
                summary_hint = s
                break

    blob_urls = []
    for vv in row.values():
        sv = "" if vv is None else str(vv).strip()
        if not sv:
            continue
        blob_urls.extend(re.findall(r"https?://[^\s\]\)\>\"]+", sv))
    user_source_url = url
    for u in blob_urls:
        lu = u.lower()
        if ("law.go.kr" in lu) or ("scourt.go.kr" in lu) or ("moel.go.kr" in lu) or ("kli.re.kr" in lu):
            user_source_url = u
            break

    direct_text = ""
    for k in ["evidence_text", "내용", "판정요지", "판정결과", "본문", "content", "text", "chunk_text"]:
        if k in row:
            v = row.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                direct_text = s
                break

    if not direct_text:
        text_candidates = []
        for kk, vv in row.items():
            sv = "" if vv is None else str(vv).strip()
            if not sv or sv.lower() == "nan":
                continue
            if len(sv) < text_min_len:
                continue
            if kk in ["api_method", "api_headers_json", "api_body", "issue_keywords_csv", "law_query", "precedent_query", "interpretation_query", "latest_user_question", "conversation_context"]:
                continue
            text_candidates.append(sv)
        direct_text = "\n\n".join(text_candidates)

    if not direct_text:
        continue

    i = 0
    order = 0
    n = len(direct_text)
    while i < n:
        piece = direct_text[i:i+chunk_size].strip()
        if piece:
            rows.append({
                "query_class": qclass,
                "issue_keywords_csv": row.get("issue_keywords_csv", ""),
                "law_query": row.get("law_query", ""),
                "precedent_query": row.get("precedent_query", ""),
                "interpretation_query": row.get("interpretation_query", ""),
                "latest_user_question": safe_question,
                "conversation_context": context_text,
                "source_type": src,
                "source_url": url,
                "user_source_url": user_source_url,
                "source_title": title,
                "article_ref": article_ref,
                "court_name": court_name,
                "decision_date": decision_date,
                "case_number": case_number,
                "summary_hint": summary_hint,
                "chunk_order": int(order),
                "chunk_text": piece
            })
            order += 1
        if i + chunk_size >= n:
            break
        i += stride

result = pd.DataFrame(rows)
```

#### 17B_공식. 데이터 연결_근거청킹입력통합_공식

- 이 노드는 Python 노드가 아니라 `데이터 연결` 노드다.
- 따라서 입력 포트 추가를 하지 않는다.
- 입력1: `17_공식`
- 입력2: `17A`
- 축(axis): `수직`
- 병합 모드: `모든 열 사용`
- 대상 레이블: `None`

#### 18_공식패키징. 파이썬_관련도필터_공식

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

keys = []
for c in ["issue_keywords_csv", "law_query", "precedent_query", "interpretation_query", "latest_user_question"]:
    if c in df.columns:
        for v in df[c].tolist():
            s = "" if v is None else str(v).strip()
            if s.lower() == "nan":
                s = ""
            if s:
                keys.extend([x.strip() for x in s.replace("/", " ").replace(",", " ").split(" ") if x.strip()])

uniq = []
for k in keys:
    if len(k) < 2:
        continue
    if k not in uniq:
        uniq.append(k)
if not uniq:
    uniq = ["징계"]

rows = []
fallback_qclass = "legal_analysis"
fallback_issue = ""
fallback_q = ""
fallback_ctx = ""

for _, r in df.iterrows():
    qv = "" if r.get("query_class", "") is None else str(r.get("query_class", "")).strip()
    if qv.lower() == "nan":
        qv = ""
    qv = qv.lower()
    if qv in ["legal_analysis", "internal_db_only", "mixed"]:
        fallback_qclass = qv
    iv = "" if r.get("issue_keywords_csv", "") is None else str(r.get("issue_keywords_csv", "")).strip()
    if iv.lower() == "nan":
        iv = ""
    if iv and not fallback_issue:
        fallback_issue = iv
    qtxt = "" if r.get("latest_user_question", "") is None else str(r.get("latest_user_question", "")).strip()
    if qtxt.lower() == "nan":
        qtxt = ""
    if qtxt and not fallback_q:
        fallback_q = qtxt
    ctxt = "" if r.get("conversation_context", "") is None else str(r.get("conversation_context", "")).strip()
    if ctxt.lower() == "nan":
        ctxt = ""
    if ctxt and not fallback_ctx:
        fallback_ctx = ctxt

for _, r in df.iterrows():
    row = r.to_dict()
    txt = "" if row.get("chunk_text", "") is None else str(row.get("chunk_text", "")).strip()
    if txt.lower() == "nan":
        txt = ""
    if len(txt) > 900:
        txt = txt[:900].strip()
    if not txt:
        continue
    st = "" if row.get("source_type", "") is None else str(row.get("source_type", "")).strip()
    if st.lower() == "nan":
        st = ""
    st = st.lower()
    title = "" if row.get("source_title", "") is None else str(row.get("source_title", "")).strip()
    if title.lower() == "nan":
        title = ""
    article_ref = "" if row.get("article_ref", "") is None else str(row.get("article_ref", "")).strip()
    if article_ref.lower() == "nan":
        article_ref = ""
    court_name = "" if row.get("court_name", "") is None else str(row.get("court_name", "")).strip()
    if court_name.lower() == "nan":
        court_name = ""
    decision_date = "" if row.get("decision_date", "") is None else str(row.get("decision_date", "")).strip()
    if decision_date.lower() == "nan":
        decision_date = ""
    case_number = "" if row.get("case_number", "") is None else str(row.get("case_number", "")).strip()
    if case_number.lower() == "nan":
        case_number = ""
    summary_hint = "" if row.get("summary_hint", "") is None else str(row.get("summary_hint", "")).strip()
    if summary_hint.lower() == "nan":
        summary_hint = ""
    if len(summary_hint) > 260:
        summary_hint = summary_hint[:260].strip()
    combined = " ".join([txt, title, article_ref, court_name, decision_date, case_number, summary_hint]).strip()
    score = 0
    for k in uniq:
        if k in combined:
            score += 2
    cite_bonus = 0
    if title:
        cite_bonus += 3
    if article_ref:
        cite_bonus += 2
    if court_name:
        cite_bonus += 2
    if decision_date:
        cite_bonus += 1
    if case_number:
        cite_bonus += 3
    if "official_law" in st:
        score += 3
    elif "official_prec" in st:
        score += 5
    elif "official_expc" in st or "official_nlrc" in st:
        score += 3
    score += cite_bonus
    rows.append({
        "query_class": row.get("query_class", fallback_qclass),
        "issue_keywords_csv": row.get("issue_keywords_csv", fallback_issue),
        "latest_user_question": row.get("latest_user_question", fallback_q),
        "conversation_context": row.get("conversation_context", fallback_ctx),
        "source_type": row.get("source_type", ""),
        "source_url": row.get("source_url", ""),
        "user_source_url": row.get("user_source_url", ""),
        "source_title": title,
        "article_ref": article_ref,
        "court_name": court_name,
        "decision_date": decision_date,
        "case_number": case_number,
        "summary_hint": summary_hint,
        "chunk_text": txt,
        "score": int(score)
    })

ranked = pd.DataFrame(rows)
if ranked.empty:
    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue,
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "official_evidence_context": "",
        "official_source_urls": "",
        "official_evidence_count": 0,
        "official_citation_briefs": ""
    }])
else:
    ranked = ranked.sort_values(by=["score"], ascending=False).reset_index(drop=True)
    top_k = ranked.head(12).copy()
    evidence_lines = []
    urls = []
    citation_briefs = []
    for _, rr in top_k.iterrows():
        st = "" if rr.get("source_type", "") is None else str(rr.get("source_type", "")).strip()
        if st.lower() == "nan":
            st = ""
        st = st.lower()
        if "query_fallback" in st:
            continue
        su = rr.get("user_source_url", "") or rr.get("source_url", "")
        su = "" if su is None else str(su).strip()
        if su.lower() == "nan":
            su = ""
        tx = "" if rr.get("chunk_text", "") is None else str(rr.get("chunk_text", "")).strip()
        if tx.lower() == "nan":
            tx = ""
        if len(tx) > 360:
            tx = tx[:360].strip()
        sc = rr.get("score", 0)
        title = "" if rr.get("source_title", "") is None else str(rr.get("source_title", "")).strip()
        if title.lower() == "nan":
            title = ""
        article_ref = "" if rr.get("article_ref", "") is None else str(rr.get("article_ref", "")).strip()
        if article_ref.lower() == "nan":
            article_ref = ""
        court_name = "" if rr.get("court_name", "") is None else str(rr.get("court_name", "")).strip()
        if court_name.lower() == "nan":
            court_name = ""
        decision_date = "" if rr.get("decision_date", "") is None else str(rr.get("decision_date", "")).strip()
        if decision_date.lower() == "nan":
            decision_date = ""
        case_number = "" if rr.get("case_number", "") is None else str(rr.get("case_number", "")).strip()
        if case_number.lower() == "nan":
            case_number = ""
        summary_hint = "" if rr.get("summary_hint", "") is None else str(rr.get("summary_hint", "")).strip()
        if summary_hint.lower() == "nan":
            summary_hint = ""
        if len(summary_hint) > 220:
            summary_hint = summary_hint[:220].strip()
        kind = "공식"
        cite_inline = ""
        label_bits = []
        if "official_law" in st:
            kind = "법령"
            if title:
                label_bits.append(title)
            if article_ref:
                label_bits.append(article_ref)
            if decision_date:
                label_bits.append("시행 " + decision_date)
            cite_inline = " ".join([x for x in [title, article_ref] if x]).strip()
        elif "official_prec" in st:
            kind = "판례"
            if court_name:
                label_bits.append(court_name)
            elif title:
                label_bits.append(title)
            if case_number:
                label_bits.append(case_number)
            if decision_date:
                label_bits.append(decision_date)
            cite_inline = " ".join([x for x in [court_name if court_name else title, case_number] if x]).strip()
        elif "official_expc" in st:
            kind = "행정해석"
            if title:
                label_bits.append(title)
            if decision_date:
                label_bits.append(decision_date)
            cite_inline = title if title else "행정해석"
        elif "official_nlrc" in st:
            kind = "노동위"
            if title:
                label_bits.append(title)
            if case_number:
                label_bits.append(case_number)
            if decision_date:
                label_bits.append(decision_date)
            cite_inline = " ".join([x for x in [title if title else "노동위", case_number] if x]).strip()

        label = " | ".join([b for b in label_bits if b]).strip()
        body = summary_hint if summary_hint else tx
        if body:
            evidence_lines.append(f"[{kind}] {label}: {body}")

        if cite_inline:
            brief = cite_inline
            if body:
                brief = brief + " -> " + body[:140]
            if brief and brief not in citation_briefs:
                citation_briefs.append(brief)
        elif label:
            if label and label not in citation_briefs:
                citation_briefs.append(label)

        if su:
            if su not in urls:
                urls.append(su)

    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue if fallback_issue else ", ".join(uniq[:5]),
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "official_evidence_context": "\n\n".join(evidence_lines),
        "official_source_urls": " | ".join(urls),
        "official_evidence_count": len(citation_briefs),
        "official_citation_briefs": "\n".join(citation_briefs[:8])
    }])
```

## 7. 내부 구조화 branch

### 7.1 입력 대상

업로드 대상:

- [★징계DB_AI활용 DB최종 작성중.xlsx](d:\AI canvas\새 폴더\★징계DB_AI활용%20DB최종%20작성중.xlsx)

초기 사용 시트:

- `Cases`
- `Rules`
- `Sanction_Guideline`
- `Lists`

### 7.2 16A. 파이썬_내부DB정규화

- 입력 포트: `dataset`
- 출력 포트: `output`
- 이 코드는 엑셀 내부DB를 구조화 검색용 행으로 표준화한다.
- AIHub prechunked는 이 노드로 보내지 않는다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

rows = []
for row in df.to_dict(orient="records"):
    st = "internal_structured"

    title = ""
    for k in [
        "case_title", "사건명", "제목", "title", "case_name",
        "caseNm", "name", "file_name", "filename", "파일명",
        "document_title", "문서명", "sheet_title"
    ]:
        if k in row:
            v = row.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                title = s
                break
    if not title:
        sid = str(row.get("사건ID*", "") or row.get("사건ID", "") or row.get("case_id", "") or "").strip()
        major = str(row.get("대분류", "") or "").strip()
        minor = str(row.get("소분류", "") or "").strip()
        bits = []
        if sid:
            bits.append("[" + sid + "]")
        if major:
            bits.append(major)
        if minor:
            bits.append(minor)
        title = " ".join(bits).strip()

    text = ""
    for k in [
        "evidence_text", "본문", "내용", "text", "summary_text", "요약", "원문", "주요 내용"
    ]:
        if k in row:
            v = row.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                text = s
                break

    if not text:
        parts = []
        for k in [
            "신고요지(주장)*", "신고요지", "쟁점(핵심 판단 포인트)", "쟁점",
            "조사결론요약(사실인정)*", "조사결론요약", "성립요건판단(요건별)",
            "규정위반 성립여부 판단*", "규정위반 성립여부 판단",
            "징계양정(검토)", "양정사유요약(가중/감경 포함)", "징계상세내용",
            "최종징계명(표준)", "대분류", "소분류"
        ]:
            if k in row:
                v = row.get(k, "")
                s = "" if v is None else str(v).strip()
                if s and s.lower() != "nan":
                    parts.append(k + ": " + s)
        text = "\n\n".join(parts).strip()

    if not text:
        continue

    doc_id = str(row.get("사건ID*", "") or row.get("사건ID", "") or row.get("case_id", "") or row.get("id", "") or "").strip()
    case_no = str(row.get("case_number", "") or row.get("사건번호", "") or doc_id).strip()
    dt = str(row.get("date", "") or row.get("심의 확정일", "") or row.get("발생일", "") or row.get("접수일", "") or "").strip()

    kwords = ""
    for k in [
        "keywords", "키워드", "issue_keywords_csv",
        "대분류", "소분류", "최종징계명(표준)", "징계부문", "사원유형",
        "취업규칙 근거", "추가혐의_소분류들", "추가혐의_근거표기"
    ]:
        if k in row:
            v = row.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                kwords = s
                break

    src_url = ""
    for k in ["source_url", "참조URL", "링크", "url", "file_path", "파일경로", "excel_path"]:
        if k in row:
            v = row.get(k, "")
            s = "" if v is None else str(v).strip()
            if s and s.lower() != "nan":
                src_url = s
                break
    if not src_url and title:
        src_url = "internal://" + title

    major_category = str(row.get("대분류", "") or "").strip()
    minor_category = str(row.get("소분류", "") or "").strip()
    discipline_name = str(row.get("최종징계명(표준)", "") or "").strip()
    discipline_level = str(row.get("최종징계Level(0~6)", "") or "").strip()
    fact_summary = str(row.get("신고요지(주장)*", "") or row.get("신고요지", "") or "").strip()
    issue_point = str(row.get("쟁점(핵심 판단 포인트)", "") or row.get("쟁점", "") or "").strip()
    investigation_summary = str(row.get("조사결론요약(사실인정)*", "") or row.get("조사결론요약", "") or "").strip()
    violation_judgment = str(row.get("규정위반 성립여부 판단*", "") or row.get("규정위반 성립여부 판단", "") or "").strip()
    sanction_review = str(row.get("징계양정(검토)", "") or "").strip()
    sanction_reason = str(row.get("양정사유요약(가중/감경 포함)", "") or "").strip()
    rule_basis = str(row.get("취업규칙 근거", "") or "").strip()

    rows.append({
        "source_type": st,
        "doc_id": doc_id,
        "case_title": title,
        "case_number": case_no,
        "date": dt,
        "source_url": src_url,
        "keywords": kwords,
        "major_category": major_category,
        "minor_category": minor_category,
        "discipline_name": discipline_name,
        "discipline_level": discipline_level,
        "fact_summary": fact_summary,
        "issue_point": issue_point,
        "investigation_summary": investigation_summary,
        "violation_judgment": violation_judgment,
        "sanction_review": sanction_review,
        "sanction_reason": sanction_reason,
        "rule_basis": rule_basis,
        "section_type": "structured_case",
        "orig_chunk_order": "",
        "evidence_text": text
    })

result = pd.DataFrame(rows, columns=[
    "source_type", "doc_id", "case_title", "case_number", "date",
    "source_url", "keywords", "major_category", "minor_category",
    "discipline_name", "discipline_level", "fact_summary", "issue_point",
    "investigation_summary", "violation_judgment", "sanction_review",
    "sanction_reason", "rule_basis", "section_type", "orig_chunk_order", "evidence_text"
])
```

### 7.3 17_내부구조화 / 17B_내부구조화 / 18_내부패키징_구조화

- `17_내부구조화`는 `17_공식`과 다르게 쓴다.
- 이유:
  - 구조화 사례는 `사건 1건 = 판단 단위 1건`으로 유지하는 것이 좋다.
  - 공식 branch처럼 잘게 청킹하면 사례 맥락이 깨지고, 양정 답변에서 두루뭉술해지기 쉽다.

`17_내부구조화`

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

rows = []
for _, r in df.iterrows():
    row = r.to_dict()

    qclass = str(row.get("query_class", "") or "").strip().lower()
    if qclass not in ["legal_analysis", "internal_db_only", "mixed"]:
        qclass = "legal_analysis"

    safe_question = str(row.get("latest_user_question", "") or "").strip()
    context_text = str(row.get("conversation_context", "") or "").strip()

    txt = str(row.get("evidence_text", "") or row.get("chunk_text", "") or "").strip()
    if not txt:
        continue
    if len(txt) > 2200:
        txt = txt[:2200].strip()

    rows.append({
        "query_class": qclass,
        "issue_keywords_csv": row.get("issue_keywords_csv", ""),
        "latest_user_question": safe_question,
        "conversation_context": context_text,
        "source_type": str(row.get("source_type", "") or "").strip(),
        "source_url": str(row.get("source_url", "") or "").strip(),
        "source_title": str(row.get("case_title", "") or "").strip(),
        "case_number": str(row.get("case_number", "") or "").strip(),
        "decision_date": str(row.get("date", "") or "").strip(),
        "article_ref": str(row.get("rule_basis", "") or "").strip(),
        "chunk_order": 0,
        "chunk_text": txt,
        "major_category": str(row.get("major_category", "") or "").strip(),
        "minor_category": str(row.get("minor_category", "") or "").strip(),
        "discipline_name": str(row.get("discipline_name", "") or "").strip(),
        "discipline_level": str(row.get("discipline_level", "") or "").strip(),
        "fact_summary": str(row.get("fact_summary", "") or "").strip(),
        "issue_point": str(row.get("issue_point", "") or "").strip(),
        "investigation_summary": str(row.get("investigation_summary", "") or "").strip(),
        "violation_judgment": str(row.get("violation_judgment", "") or "").strip(),
        "sanction_review": str(row.get("sanction_review", "") or "").strip(),
        "sanction_reason": str(row.get("sanction_reason", "") or "").strip()
    })

result = pd.DataFrame(rows)
```

`17B_내부구조화`

- 이 노드는 Python 노드가 아니라 `데이터 연결` 노드다.
- 따라서 입력 포트 추가를 하지 않는다.
- 입력1: `17_내부구조화`
- 입력2: `17A`
- 축(axis): `수직`
- 병합 모드: `모든 열 사용`
- 대상 레이블: `None`

`18_내부패키징_구조화`

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

keys = []
for c in ["issue_keywords_csv", "law_query", "precedent_query", "interpretation_query", "latest_user_question"]:
    if c in df.columns:
        for v in df[c].tolist():
            s = "" if v is None else str(v).strip()
            if s.lower() == "nan":
                s = ""
            if s:
                keys.extend([x.strip() for x in s.replace("/", " ").replace(",", " ").split(" ") if x.strip()])

uniq = []
for k in keys:
    if len(k) < 2:
        continue
    if k not in uniq:
        uniq.append(k)
if not uniq:
    uniq = ["징계"]

trigger_text = " ".join([
    ("" if len(df) == 0 or df.iloc[0].get("latest_user_question", "") is None else str(df.iloc[0].get("latest_user_question", "")).strip()),
    ("" if len(df) == 0 or df.iloc[0].get("issue_keywords_csv", "") is None else str(df.iloc[0].get("issue_keywords_csv", "")).strip())
]).lower()

sanction_query = False
for tok in ["징계", "양정", "징계수위", "견책", "근신", "감봉", "정직", "강등", "해고", "전보", "대기발령"]:
    if tok in trigger_text:
        sanction_query = True
        break

rows = []
fallback_qclass = "legal_analysis"
fallback_issue = ""
fallback_q = ""
fallback_ctx = ""

for _, r in df.iterrows():
    qv = "" if r.get("query_class", "") is None else str(r.get("query_class", "")).strip().lower()
    if qv == "nan":
        qv = ""
    if qv in ["legal_analysis", "internal_db_only", "mixed"]:
        fallback_qclass = qv
    iv = "" if r.get("issue_keywords_csv", "") is None else str(r.get("issue_keywords_csv", "")).strip()
    if iv.lower() == "nan":
        iv = ""
    if iv and not fallback_issue:
        fallback_issue = iv
    qtxt = "" if r.get("latest_user_question", "") is None else str(r.get("latest_user_question", "")).strip()
    if qtxt.lower() == "nan":
        qtxt = ""
    if qtxt and not fallback_q:
        fallback_q = qtxt
    ctxt = "" if r.get("conversation_context", "") is None else str(r.get("conversation_context", "")).strip()
    if ctxt.lower() == "nan":
        ctxt = ""
    if ctxt and not fallback_ctx:
        fallback_ctx = ctxt

for _, r in df.iterrows():
    row = r.to_dict()
    txt = "" if row.get("chunk_text", "") is None else str(row.get("chunk_text", "")).strip()
    if txt.lower() == "nan":
        txt = ""
    if len(txt) > 1200:
        txt = txt[:1200].strip()
    if not txt:
        continue
    title = "" if (row.get("source_title", "") or row.get("case_title", "")) is None else str(row.get("source_title", "") or row.get("case_title", "")).strip()
    if title.lower() == "nan":
        title = ""
    major = "" if row.get("major_category", "") is None else str(row.get("major_category", "")).strip()
    if major.lower() == "nan":
        major = ""
    minor = "" if row.get("minor_category", "") is None else str(row.get("minor_category", "")).strip()
    if minor.lower() == "nan":
        minor = ""
    discipline_name = "" if row.get("discipline_name", "") is None else str(row.get("discipline_name", "")).strip()
    if discipline_name.lower() == "nan":
        discipline_name = ""
    discipline_level = "" if row.get("discipline_level", "") is None else str(row.get("discipline_level", "")).strip()
    if discipline_level.lower() == "nan":
        discipline_level = ""
    fact_summary = "" if row.get("fact_summary", "") is None else str(row.get("fact_summary", "")).strip()
    if fact_summary.lower() == "nan":
        fact_summary = ""
    if len(fact_summary) > 260:
        fact_summary = fact_summary[:260].strip()
    issue_point = "" if row.get("issue_point", "") is None else str(row.get("issue_point", "")).strip()
    if issue_point.lower() == "nan":
        issue_point = ""
    if len(issue_point) > 180:
        issue_point = issue_point[:180].strip()
    investigation_summary = "" if row.get("investigation_summary", "") is None else str(row.get("investigation_summary", "")).strip()
    if investigation_summary.lower() == "nan":
        investigation_summary = ""
    if len(investigation_summary) > 220:
        investigation_summary = investigation_summary[:220].strip()
    violation_judgment = "" if row.get("violation_judgment", "") is None else str(row.get("violation_judgment", "")).strip()
    if violation_judgment.lower() == "nan":
        violation_judgment = ""
    if len(violation_judgment) > 220:
        violation_judgment = violation_judgment[:220].strip()
    sanction_review = "" if row.get("sanction_review", "") is None else str(row.get("sanction_review", "")).strip()
    if sanction_review.lower() == "nan":
        sanction_review = ""
    if len(sanction_review) > 220:
        sanction_review = sanction_review[:220].strip()
    sanction_reason = "" if row.get("sanction_reason", "") is None else str(row.get("sanction_reason", "")).strip()
    if sanction_reason.lower() == "nan":
        sanction_reason = ""
    if len(sanction_reason) > 220:
        sanction_reason = sanction_reason[:220].strip()
    rule_basis = "" if (row.get("article_ref", "") or row.get("rule_basis", "")) is None else str(row.get("article_ref", "") or row.get("rule_basis", "")).strip()
    if rule_basis.lower() == "nan":
        rule_basis = ""
    case_number = "" if row.get("case_number", "") is None else str(row.get("case_number", "")).strip()
    if case_number.lower() == "nan":
        case_number = ""
    decision_date = "" if row.get("decision_date", "") is None else str(row.get("decision_date", "")).strip()
    if decision_date.lower() == "nan":
        decision_date = ""
    combined = " ".join([
        txt, title, case_number, decision_date, major, minor, discipline_name, discipline_level,
        fact_summary, issue_point, investigation_summary, violation_judgment,
        sanction_review, sanction_reason, rule_basis
    ]).strip()
    score = 0
    for k in uniq:
        if k in combined:
            score += 2
    if major:
        score += 1
    if minor:
        score += 1
    if discipline_name:
        score += 2
    if sanction_query and sanction_reason:
        score += 2
    if sanction_query and violation_judgment:
        score += 1
    if sanction_query and fact_summary:
        score += 1
    rows.append({
        "case_title": title,
        "case_number": case_number,
        "decision_date": decision_date,
        "source_type": row.get("source_type", ""),
        "source_url": row.get("source_url", ""),
        "chunk_text": txt,
        "major_category": major,
        "minor_category": minor,
        "discipline_name": discipline_name,
        "discipline_level": discipline_level,
        "fact_summary": fact_summary,
        "issue_point": issue_point,
        "investigation_summary": investigation_summary,
        "violation_judgment": violation_judgment,
        "sanction_review": sanction_review,
        "sanction_reason": sanction_reason,
        "rule_basis": rule_basis,
        "score": int(score)
    })

ranked = pd.DataFrame(rows)
if ranked.empty:
    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue,
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "internal_evidence_context": "",
        "internal_source_urls": "",
        "internal_evidence_count": 0,
        "internal_case_briefs": "",
        "internal_case_count": 0,
        "sanction_query_flag": 1 if sanction_query else 0
    }])
else:
    ranked = ranked.sort_values(by=["score"], ascending=False).reset_index(drop=True)
    top_k = ranked.head(10).copy()
    evidence_lines = []
    case_briefs = []
    urls = []
    for _, rr in top_k.iterrows():
        st = "" if rr.get("source_type", "") is None else str(rr.get("source_type", "")).strip().lower()
        if st == "nan":
            st = ""
        if "query_fallback" in st:
            continue
        su = "" if rr.get("source_url", "") is None else str(rr.get("source_url", "")).strip()
        if su.lower() == "nan":
            su = ""
        tx = "" if rr.get("chunk_text", "") is None else str(rr.get("chunk_text", "")).strip()
        if tx.lower() == "nan":
            tx = ""
        if len(tx) > 260:
            tx = tx[:260].strip()
        sc = rr.get("score", 0)
        title = "" if rr.get("case_title", "") is None else str(rr.get("case_title", "")).strip()
        if title.lower() == "nan":
            title = ""
        case_number = "" if rr.get("case_number", "") is None else str(rr.get("case_number", "")).strip()
        if case_number.lower() == "nan":
            case_number = ""
        decision_date = "" if rr.get("decision_date", "") is None else str(rr.get("decision_date", "")).strip()
        if decision_date.lower() == "nan":
            decision_date = ""
        case_year = ""
        for i in range(0, max(0, len(decision_date) - 3)):
            chunk = decision_date[i:i+4]
            if chunk.isdigit() and (chunk.startswith("19") or chunk.startswith("20")):
                case_year = chunk
                break
        discipline_name = "" if rr.get("discipline_name", "") is None else str(rr.get("discipline_name", "")).strip()
        if discipline_name.lower() == "nan":
            discipline_name = ""
        discipline_level = "" if rr.get("discipline_level", "") is None else str(rr.get("discipline_level", "")).strip()
        if discipline_level.lower() == "nan":
            discipline_level = ""
        fact_summary = "" if rr.get("fact_summary", "") is None else str(rr.get("fact_summary", "")).strip()
        if fact_summary.lower() == "nan":
            fact_summary = ""
        if len(fact_summary) > 160:
            fact_summary = fact_summary[:160].strip()
        issue_point = "" if rr.get("issue_point", "") is None else str(rr.get("issue_point", "")).strip()
        if issue_point.lower() == "nan":
            issue_point = ""
        if len(issue_point) > 120:
            issue_point = issue_point[:120].strip()
        violation_judgment = "" if rr.get("violation_judgment", "") is None else str(rr.get("violation_judgment", "")).strip()
        if violation_judgment.lower() == "nan":
            violation_judgment = ""
        if len(violation_judgment) > 140:
            violation_judgment = violation_judgment[:140].strip()
        sanction_reason = "" if (rr.get("sanction_reason", "") or rr.get("sanction_review", "")) is None else str(rr.get("sanction_reason", "") or rr.get("sanction_review", "")).strip()
        if sanction_reason.lower() == "nan":
            sanction_reason = ""
        if len(sanction_reason) > 160:
            sanction_reason = sanction_reason[:160].strip()
        behavior = fact_summary if fact_summary else (issue_point if issue_point else tx)
        reason = sanction_reason if sanction_reason else violation_judgment
        if tx:
            year_head = case_year + "년" if case_year else "연도미상"
            evidence_lines.append(f"[내부사례|{year_head}|최종징계:{discipline_name}|score={sc}] 행위:{behavior} / 판단:{violation_judgment} / 양정:{reason}")
        brief_parts = []
        if case_year:
            brief_parts.append(case_year + "년")
        if title:
            brief_parts.append(title)
        elif case_number:
            brief_parts.append(case_number)
        if behavior:
            brief_parts.append("행위 요지: " + behavior)
        if discipline_name:
            brief_parts.append("최종징계: " + discipline_name)
        if reason:
            brief_parts.append("양정사유: " + reason)
        brief = " | ".join([b for b in brief_parts if b])
        if brief and brief not in case_briefs:
            case_briefs.append(brief)
        if su:
            disp = su
            if "//pdf/" in disp:
                disp = disp.split("//pdf/", 1)[1].strip()
            elif disp.startswith("internal://"):
                disp = disp.replace("internal://", "", 1).strip()
            disp = disp.replace(".pdf", "").replace(".xlsx", "").replace(".csv", "")
            if disp.startswith("\ud68c\uc0ac_"):
                disp = disp[len("\ud68c\uc0ac_"): ]
            if disp and disp not in urls:
                urls.append(disp)

    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue if fallback_issue else ", ".join(uniq[:5]),
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "internal_evidence_context": "\n\n".join(evidence_lines),
        "internal_source_urls": " | ".join(urls),
        "internal_evidence_count": len(evidence_lines),
        "internal_case_briefs": "\n".join(case_briefs[:5]),
        "internal_case_count": len(case_briefs),
        "sanction_query_flag": 1 if sanction_query else 0
    }])
```

## 8. 내부 문서 DB 적재 그래프

운영 기준:

- **권장 기본 경로**: 로컬에서 문서를 표준화/청킹한 `적재용 CSV/XLSX`를 만든 뒤 AI Canvas에 업로드한다.
- **현재 표준 운영**: 문서 저장소를 3개로 분리한다.
  - `docdb_aihub_v1`: AIHub 판결/약관 전용
  - `docdb_rules_v1`: 내부 규정 + 외부 가이드/질의회시집 전용
  - `docdb_opinion_style_v1`: 의견서 작성 스타일 예시 전용
- **fallback 경로**: 로컬 전처리가 어렵거나 소량 테스트만 할 때만 `D1A/D1B/D2B`를 AI Canvas에서 직접 실행한다.
- 이유: 대용량 AIHub/PDF는 `D1A`, `D1B`에서 `timed out`, `Server disconnected without sending a response`가 날 수 있기 때문이다.

### 8.1 입력 파일

업로드 기준 파일은 4개다.

- `internal_doc_ingest_aihub_판결.xlsx`
- `internal_doc_ingest_aihub_약관.xlsx`
- `internal_doc_ingest_rules_통합.xlsx`
- `internal_doc_ingest_opinion_letters_내부.xlsx`

이 4개만 AI Canvas에 올리면 된다.

입력 위치는 기본 4곳이다.

1. `데이터셋_문서근거_AIHUB580_판결_적재용`

- 여기에 `internal_doc_ingest_aihub_판결.xlsx`만 업로드한다.

2. `데이터셋_문서근거_AIHUB580_약관_적재용`

- 여기에 `internal_doc_ingest_aihub_약관.xlsx`만 업로드한다.

3. `데이터셋_문서근거_RULES_적재용`

- 여기에 `internal_doc_ingest_rules_통합.xlsx`만 업로드한다.
- 이 파일 안에는 아래 문서군이 함께 들어간다.
  - 내부 문서 PDF: 취업규칙, 윤리규범, 징계규정, 조사사무 처리기준
  - 외부 공식 문서 PDF: 직장 내 괴롭힘/성희롱 대응지침, 고용노동부 질의회시집, 공식 가이드

4. `데이터셋_문서근거_의견서_적재용`

- 여기에 `internal_doc_ingest_opinion_letters_내부.xlsx`만 업로드한다.
- 이 저장소는 근거 검색용이 아니라 작성 스타일 예시용이다.

기본 원칙:

- AIHub는 `판결`, `약관` 2개 파일로 유지하되 같은 AIHub 저장소에 적재한다.
- PDF 규정/가이드류는 내부/외부로 나눠 적재하지 않고, 업로드 단계에서는 `rules 통합` 파일 1개로 관리한다.
- `Opinion_Letters`는 일반 문서 저장소와 반드시 분리한다.

### 8.1B 권장 기본 경로: 로컬 전처리 후 업로드

로컬에서 아래 스키마로 `적재용 CSV`를 먼저 만든다.

- `file_name`
- `chunk_id`
- `txt`
- `source_type`
- `doc_id`
- `case_title`
- `case_number`
- `date`
- `source_url`
- `keywords`
- `section_type`

권장 업로드 파일 예:

- `internal_doc_ingest_aihub_판결.xlsx`
- `internal_doc_ingest_aihub_약관.xlsx`
- `internal_doc_ingest_rules_통합.xlsx`
- `internal_doc_ingest_opinion_letters_내부.xlsx`

보조 산출물로 아래 파일이 추가로 있을 수 있지만, AI Canvas 업로드 기본 파일은 아니다.

- `internal_doc_ingest_pdf_내부.xlsx`
- `internal_doc_ingest_pdf_외부.xlsx`

이 파일들은 `rules 통합` 파일을 만들기 위한 중간 산출물로만 본다.

### 8.1A 무엇을 문서 데이터에 넣고, 무엇을 구조화 branch에 둘지

기준은 단순하다.

1. `설명형 문장`으로 검색하는 것이 더 유리한 데이터

- `문서 데이터` 또는 `데이터셋_문서근거_AIHUB580_prechunked`로 넣는다.
- 예:
  - AIHub prechunked CSV
  - 취업규칙 PDF
  - 윤리규범 PDF
  - 징계규정 PDF
  - 성희롱/괴롭힘 대응지침 PDF

2. `열 단위 값`, `코드값`, `분류값`, `양정표`, `사건 메타`처럼 구조를 보존해야 하는 데이터

- `데이터셋_내부근거_엑셀 -> 16A -> 17_내부구조화` 경로로 넣는다.
- 예:
  - `Cases`
  - `Rules`
  - `Sanction_Guideline`
  - `Lists`

3. 엑셀이라고 해서 모두 구조화 branch에만 두는 것은 아니다.

- 업체 설명처럼 `문서 검색형에서 CSV는 테이블 자체보다 설명형 문장 데이터가 더 잘 맞는다`.
- 따라서 엑셀 원본이더라도, 한 행이 이미 `설명형 문장`으로 정리돼 있으면 문서 branch로 보내도 된다.
- 반대로 열 의미가 중요한 표면, 그대로 문서 데이터에 넣지 않는다.

실무 기준:

- `징계 규정`, `취업규칙`, `윤리규범` 같은 규범 문서는 `문서 데이터`
- `Cases 엑셀`, `양정표`, `사건 분류표` 같은 구조화 데이터는 `내부 구조화 branch`

권장 운영:

- 같은 자료를 한쪽에만 고집하지 않는다.
- 필요하면 `구조화용 원본`은 엑셀 branch에 두고,
- 별도로 `설명형 문장 버전`을 만들어 문서 branch에도 넣는다.
- 다만 초보자 1차 구축에서는 중복 적재보다 아래 원칙을 우선한다.

초보자용 기본 원칙:

- PDF 규정류 -> 문서 데이터
- AIHub prechunked -> 문서 DB 적재
- 엑셀 내부DB -> 구조화 branch
- `Opinion_Letters` -> 일반 구조화 branch에 넣지 않고 별도 의견서 문안 저장소로 분리

### 8.1B Opinion_Letters 운영 원칙

- `Opinion_Letters`는 `Cases`처럼 구조화 검색에 섞지 않는다.
- 이 시트는 `노무사/변호사 검토의견 본문`, `회신문 초안`, `답변서 문안`처럼 문장형 텍스트를 저장하는 별도 문서 저장소로 운영한다.
- 이 저장소의 목적은 `법적 근거 검색`이 아니라 `작성 스타일 학습`이다.
- 따라서 과거 의견서를 그대로 복사해 쓰는 용도가 아니라, 제목 형식, 문단 구성, 논리 전개, 결론 정리 방식을 참고하는 용도로만 사용한다.
- 런타임에서는 아래 4개 트리거가 질문에 있을 때만 검색한다.
  - `의견서`
  - `검토의견`
  - `법률검토`
  - `답변서`
- 현재 시트에 본문 실데이터가 없거나 예시 행만 있으면, 적재용 파일이 `0행`으로 생성돼도 정상이다.
- 최소 입력 스키마(권장):
  - `제목*`
  - `질문/쟁점(선택)`
  - `태그(콤마)`
  - `본문*`
  - `사건ID(선택)`
  - `작성일(선택)`
  - `비고`
- `사건ID`나 `사건번호`는 필수가 아니다. 의견서 저장소는 `본문(txt)`와 `질문/쟁점`이 더 중요하다.
- `질문/쟁점(선택)`이 있으면 전처리 시 `txt` 앞부분에 함께 붙여 넣어 검색 품질을 높인다.
- 업로드용 산출물 파일:
  - `internal_doc_ingest_opinion_letters_내부.csv`
  - `internal_doc_ingest_opinion_letters_내부.xlsx`
- 입력 템플릿 파일:
  - `opinion_letters_minimal_template.csv`
  - `opinion_letters_minimal_template.xlsx`
- MSG 아카이브에서 템플릿 자동 생성:
  - `python extract_opinion_letters_from_msg.py`
- 로컬 재생성 명령:
  - `python build_doc_ingest_outputs.py --only opinion`
- 이 경로도 런타임에서는 `에이전트 메시지 가로채기 -> 2A/2B/2C -> 3 -> 4 -> 5 -> 17A`를 공유한 뒤, 의견서 전용 검색 branch로 갈라진다.

### 8.2 D1A_판결, D1A_약관. 파이썬_AIHUB문서DB표준화

- 입력 포트: `dataset`
- 출력 포트: `output`
- 목적: AIHub prechunked CSV를 문서 DB 적재용 공통 스키마로 맞춘다.
- 이 코드는 `D1A_판결`, `D1A_약관`에 동일하게 복사해서 사용한다.
- 사용 시점: **fallback 경로**. 로컬 전처리 없이 AI Canvas에서 직접 적재용 스키마를 만들 때만 사용한다.

```python
df = dataset if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

rows = []
for _, r in df.iterrows():
    doc_id = str(r.get("doc_id", "") or "").strip()
    case_title = str(r.get("case_title", "") or "").strip()
    case_number = str(r.get("case_number", "") or "").strip()
    date = str(r.get("date", "") or "").strip()
    source_url = str(r.get("source_url", "") or "").strip()
    keywords = str(r.get("keywords", "") or "").strip()
    section_type = str(r.get("section_type", "") or "").strip()
    chunk_order = str(r.get("chunk_order", "") or "").strip()
    txt = str(r.get("chunk_text", "") or "").strip()
    source_type = str(r.get("source_type", "") or "").strip()

    if not txt:
        continue

    file_name = doc_id if doc_id else case_title
    if not file_name:
        file_name = "aihub_doc"

    rows.append({
        "file_name": file_name,
        "chunk_id": chunk_order if chunk_order else "0",
        "txt": txt,
        "source_type": source_type if source_type else "internal_doc_aihub_chunked",
        "doc_id": doc_id,
        "case_title": case_title,
        "case_number": case_number,
        "date": date,
        "source_url": source_url if source_url else "internal://aihub/" + file_name,
        "keywords": keywords,
        "section_type": section_type if section_type else "aihub_chunk"
    })

result = pd.DataFrame(rows, columns=[
    "file_name", "chunk_id", "txt", "source_type", "doc_id",
    "case_title", "case_number", "date", "source_url", "keywords", "section_type"
])
```

- 실행 팁:
  - AIHub prechunked가 너무 크면 이 노드에서 `timed out` 또는 `Server disconnected without sending a response`가 날 수 있다.
  - 이 경우 `copy()`를 제거한 현재 버전을 사용한다.
  - 그래도 무거우면 AIHub 파일을 더 잘게 나눠 `판결-1`, `판결-2`, `약관-1`, `약관-2`처럼 별도 데이터셋 노드로 추가한다.
  - 기본 권장 분할은 `판결`, `약관` 2개다.

### 8.3 D1B_내부, D1B_외부. 파이썬_파일텍스트추출

- 입력 포트: `dataset`
- 출력 포트: `output`
- 목적: 문서 데이터 노드가 읽어온 PDF/PPTX/XLSX/TXT/CSV를 텍스트로 펼친다.
- 이 코드는 `D1B_내부`, `D1B_외부`에 동일하게 복사해서 사용한다.
- 사용 시점: **fallback 경로**. 로컬 전처리 없이 PDF 원문을 AI Canvas에서 직접 펼칠 때만 사용한다.

```python
rows = []

for file_name, obj in dataset.items():
    ext = "." + file_name.rsplit(".", 1)[-1].lower()

    if ext == ".pptx":
        for slide_num, slide in enumerate(obj.slides, start=1):
            slide_texts = []
            for shape in slide.shapes:
                if shape.has_text_frame:
                    for para in shape.text_frame.paragraphs:
                        text = para.text.strip()
                        if text:
                            slide_texts.append(text)
            slide_content = "\n".join(slide_texts).strip()
            if slide_content:
                rows.append({
                    "file_name": file_name + "#slide-" + str(slide_num),
                    "ext": ext,
                    "location": "document",
                    "content": slide_content
                })

    elif ext == ".pdf":
        for page_num, page in enumerate(obj.pages, start=1):
            text = page.extract_text()
            if text is not None:
                text = str(text).strip()
                if text:
                    rows.append({
                        "file_name": file_name + "#page-" + str(page_num),
                        "ext": ext,
                        "location": "document",
                        "content": text
                    })

    elif ext == ".xlsx":
        for ws in obj.worksheets:
            sheet_lines = []
            for row in ws.iter_rows(values_only=True):
                vals = []
                for cell in row:
                    if cell is None:
                        vals.append("")
                    else:
                        vals.append(str(cell).strip())
                if any(v != "" for v in vals):
                    sheet_lines.append(" | ".join(vals))
            sheet_text = "\n".join(sheet_lines).strip()
            if sheet_text:
                rows.append({
                    "file_name": file_name + "#sheet-" + str(ws.title),
                    "ext": ext,
                    "location": "document",
                    "content": sheet_text
                })

    elif ext == ".txt":
        text = str(obj).strip()
        if text:
            rows.append({
                "file_name": file_name,
                "ext": ext,
                "location": "document",
                "content": text
            })

    elif ext == ".csv":
        if isinstance(obj, pd.DataFrame) and len(obj) > 0:
            csv_lines = []
            csv_lines.append(" | ".join([str(c).strip() for c in obj.columns]))
            for row in obj.itertuples(index=False, name=None):
                vals = []
                for cell in row:
                    if pd.isna(cell):
                        vals.append("")
                    else:
                        vals.append(str(cell).strip())
                csv_lines.append(" | ".join(vals))

            csv_text = "\n".join(csv_lines).strip()
            if csv_text:
                rows.append({
                    "file_name": file_name,
                    "ext": ext,
                    "location": "document",
                    "content": csv_text
                })

result = pd.DataFrame(rows)
```

- 이 버전은 파일 전체를 한 문자열로 합치지 않고 `page/slide/sheet` 단위로 바로 행을 만든다.
- 따라서 큰 PDF나 XLSX에서 메모리 부담이 더 적고, `Server disconnected without sending a response`가 날 가능성을 줄인다.
- 그래도 `timed out`가 나면 `문서 데이터_문서근거_내부`, `문서 데이터_문서근거_외부`를 더 잘게 나눈다.
- 예:
  - `문서 데이터_문서근거_내부_1`
  - `문서 데이터_문서근거_내부_2`
- 이 경우도 기본 원칙은 `소스 노드 추가 + 병합 체인 확장`이다.

### 8.4 D2B. 파이썬_문서청킹

- 입력 포트: `dataset`
- 출력 포트: `output`
- 중요: 템플릿 원본은 `row["txt"]`를 읽는데, D1B는 `content`를 출력한다. 여기서는 이 불일치를 수정해서 사용한다.
- 사용 시점: **fallback 경로**. 로컬에서 이미 chunk 단위 CSV를 만들었다면 이 노드는 쓰지 않는다.

```python
chunk_size = 1000
overlap = 200

rows = []

for _, row in dataset.iterrows():
    file_name = str(row.get("file_name", "") or "").strip()
    text = str(row.get("content", "") or "").strip()
    if not text:
        continue

    start = 0
    chunk_id = 0

    while start < len(text):
        end = start + chunk_size
        chunk = text[start:end].strip()

        if chunk:
            rows.append({
                "file_name": file_name,
                "chunk_id": chunk_id,
                "txt": chunk,
                "source_type": "internal_doc_pdf_chunked",
                "doc_id": file_name,
                "case_title": file_name,
                "case_number": "",
                "date": "",
                "source_url": "internal://" + file_name,
                "keywords": "",
                "section_type": "pdf_chunk"
            })

        start += chunk_size - overlap
        chunk_id += 1

result = pd.DataFrame(rows)
```

### 8.5 D4. 파이썬_문서메타데이터생성

- 입력 포트: `dataset`
- 출력 포트: `output`
- 역할:
  - 각 chunk에 `doc_meta_json`을 결정적으로 생성한다.
  - 대용량 적재 단계에서 LLM 노드 대신 Python으로 메타데이터를 만든다.
- 주의:
  - 이 코드는 호출마다 같은 입력에 같은 출력을 만들어야 한다.
  - `doc_meta_json`은 여러 필드를 요약한 보조 메타데이터이며, 실제 검색 축은 `txt` 임베딩이다.
  - 편집 구간에는 `import json`을 추가하지 말고, 상단 고정 컨텍스트에 있는 `json`을 그대로 사용한다.

```python
df = dataset if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

rows = []
for _, r in df.iterrows():
    row = r.to_dict()

    txt = str(row.get("txt", "") or "").strip()
    source_type = str(row.get("source_type", "") or "").strip()
    section_type = str(row.get("section_type", "") or "").strip()
    case_title = str(row.get("case_title", "") or "").strip()
    keywords_raw = str(row.get("keywords", "") or "").strip()

    keyword_parts = []
    for p in keywords_raw.replace("|", ",").replace("/", ",").split(","):
        p = p.strip()
        if not p:
            continue
        compact = p.replace("-", "").replace("_", "").replace(".", "").replace(" ", "")
        if compact.isdigit() or len(p) < 2:
            continue
        if p not in keyword_parts:
            keyword_parts.append(p)
    keywords = " | ".join(keyword_parts[:12])
    row["keywords"] = keywords

    summary = " ".join(txt.split())
    if len(summary) > 180:
        summary = summary[:180].strip()

    tags = []
    for raw in [source_type, section_type, keywords, case_title]:
        s = "" if raw is None else str(raw).strip()
        if not s:
            continue
        if raw == keywords:
            parts = []
            for p in s.replace("|", ",").replace("/", ",").split(","):
                p = p.strip()
                if p:
                    parts.append(p)
        else:
            parts = [s]
        for p in parts:
            compact = p.replace("-", "").replace("_", "").replace(".", "").replace("(", "").replace(")", "").replace("[", "").replace("]", "").replace(" ", "")
            if compact.isdigit():
                continue
            if len(p) < 2:
                continue
            if p and p not in tags:
                tags.append(p)

    if len(tags) > 8:
        tags = tags[:8]

    source_hint = source_type
    if section_type:
        source_hint = source_hint + ":" + section_type if source_hint else section_type

    row["doc_meta_json"] = json.dumps({
        "summary": summary,
        "tags": tags,
        "source_hint": source_hint
    }, ensure_ascii=False)

    rows.append(row)

result = pd.DataFrame(rows)
```

### 8.6 D5. 텍스트 임베딩_문서벡터화

- 대상 레이블: `txt`
- 모델: `text-embedding-3-small`
- 배치 사이즈(권장 시작값): `128`
- 출력 차원: `512`
- `128`이 안정적으로 끝난 환경이면 `256`으로 올려도 된다.
- `선택한 대상 열 삭제`: `OFF`

### 8.7 D6W_AIHUB_판결, D6W_AIHUB_약관. DB저장_AIHUB저장소_쓰기

- 저장소 이름: `docdb_aihub_v1`
- 입력: `D5_판결`, `D5_약관`
- 목적: AIHub 판결/약관 chunk와 임베딩 벡터를 AIHub 전용 저장소에 적재한다.
- 저장소 유형: `데이터 추가`
- 중복 제거: `ON`
- 운영 원칙:
  - 새 저장소 이름으로 시작하면 `판결 -> 약관` 순서로 두 노드를 순서대로 실행한다.
  - AIHub는 무겁고 자주 변하지 않으므로, 정상 적재가 끝난 뒤에는 평소 재실행하지 않는다.
  - AIHub 전체를 다시 만들고 싶으면 기존 저장소를 덮어쓰는 대신 새 저장소 이름(`docdb_aihub_v2` 등)으로 다시 시작한다.

### 8.8 D6R_AIHUB. 데이터 저장소_AIHUB저장소_읽기

- 저장소 이름: `docdb_aihub_v1`
- 모드: `데이터 보기`
- 목적: 런타임 질의 시 AIHub 저장소의 문서 임베딩과 메타데이터를 읽어온다.
- 입력 포트: 없음
- 주의:
  - `txt`, `doc_meta_json`, 메타데이터 열, 임베딩 숫자 열이 모두 보여야 한다.
  - 이 읽기 노드는 `17D3_AIHUB`의 기본 `dataset` 포트에 연결한다.

### 8.8A D6W_RULES. DB저장_규정가이드저장소_쓰기

- 저장소 이름: `docdb_rules_v1`
- 입력: `D5_RULES`
- 목적: 내부 규정 PDF와 외부 공식 가이드 PDF를 하나의 규정/가이드 전용 저장소에 적재한다.
- 저장소 유형: `데이터 덮어쓰기`
- 중복 제거: `ON`
- 운영 원칙:
  - `internal_doc_ingest_rules_통합.xlsx`는 항상 현재 기준 전체 묶음이다.
  - 새 규정/가이드를 추가하면 파일을 다시 만들고, 이 저장소를 `덮어쓰기`로 갱신한다.
  - 이 저장소는 `AIHub`와 분리되어 있으므로, `덮어쓰기`를 써도 AIHub 문서는 영향받지 않는다.

### 8.8B D6R_RULES. 데이터 저장소_규정가이드저장소_읽기

- 저장소 이름: `docdb_rules_v1`
- 모드: `데이터 보기`
- 목적: 런타임 질의 시 규정/가이드 저장소의 문서 임베딩과 메타데이터를 읽어온다.
- 입력 포트: 없음
- 주의:
  - `txt`, `doc_meta_json`, 메타데이터 열, 임베딩 숫자 열이 모두 보여야 한다.
  - 이 읽기 노드는 `17D3_RULES`의 기본 `dataset` 포트에 연결한다.

### 8.8C D6W_의견서. DB저장_의견서문안저장소_쓰기

- 저장소 이름: `docdb_opinion_style_v1`
- 입력: `D5_의견서`
- 목적: `Opinion_Letters`를 일반 문서 저장소와 섞지 않고, 작성형 요청 전용 문안 저장소에 적재한다.
- 저장소 유형: `데이터 덮어쓰기`
- 중복 제거: `ON`
- 주의:
  - 일반 문서 저장소(`docdb_aihub_v1`, `docdb_rules_v1`)와 반드시 분리한다.
  - `internal_doc_ingest_opinion_letters_내부.xlsx`가 `0행`이면 이 적재 단계는 생략해도 된다.

### 8.8D D6R_의견서. 데이터 저장소_의견서문안저장소_읽기

- 저장소 이름: `docdb_opinion_style_v1`
- 모드: `데이터 보기`
- 목적: 작성형 질문일 때만 과거 검토의견/답변서 문안을 읽어온다.
- 입력 포트: 없음
- 주의:
  - 이 읽기 노드는 일반 질의에서도 항상 연결돼 있을 수 있지만, 실제 검색은 `17D3_의견서`의 트리거 게이트에서 막는다.
  - 따라서 저장소를 분리해 두면 일반 법률질문에서 의견서 문안이 내부 근거에 섞이는 문제를 막을 수 있다.

## 9. 내부 문서 검색 branch

### 9.1 17D1. 프롬프트_문서검색질문생성

- 입력 컬럼: `latest_user_question`, `conversation_context`, `query_class`, `issue_keywords_csv`
- Output column name: `txt`
- 모델: `gpt-5.2`
- Include Question Prompt in output: `ON`

```text
현재 질문: {{latest_user_question}}
최근 대화 맥락: {{conversation_context}}
질의 분류: {{query_class}}
쟁점 키워드: {{issue_keywords_csv}}

다음은 사용자와 AI의 대화 맥락을 바탕으로 만든 검색용 질문이다.
벡터 DB에서 관련 문서를 찾기 위한 검색 문장을 1문장만 작성하라.

규칙:
- 이전 대화 문맥을 반영한다.
- 질문을 완결된 문장으로 재작성한다.
- 불필요한 설명 없이 검색에 적합한 짧은 문장으로 작성한다.
- 출력은 검색 질문 한 문장만 작성한다.
```

### 9.2 17D2. 텍스트 임베딩_질문벡터화

- 대상 레이블: `txt`
- 모델: `text-3-small`
- 배치 사이즈: `256`
- 출력 차원: `512`

### 9.3 17D3_AIHUB, 17D3_RULES. 파이썬_벡터유사문서TOPK

- 이 노드는 `2입력 Python` 노드다.
- 아래 2개 노드에 같은 코드를 복사해서 쓴다.
  - `17D3_AIHUB`
  - `17D3_RULES`
- 입력 포트:
  - `dataset`: `D6R_AIHUB` 또는 `D6R_RULES`
  - 추가 입력1 이름 권장: `query`
  - `query`: `17D2`
- 출력 포트: `output`
- 목적:
  - 차원 축소 없이 저장된 문서 임베딩과 질문 임베딩을 직접 비교한다.
  - cosine similarity 기준으로 top-k 문서를 뽑는다.
- 성능 규칙:
  - 질문 텍스트 토큰으로 먼저 lexical prefilter를 건 뒤 cosine similarity를 계산한다.
  - AIHub와 RULES 저장소를 분리했기 때문에, 예전처럼 하나의 문서 저장소를 전부 훑는 구조보다 느려질 가능성이 크게 줄어든다.
  - 징계/양정 질문에서는 `17D3_AIHUB`를 가능한 한 빨리 건너뛰어, 구조화 사례와 RULES 쪽을 먼저 보게 한다.

```python
doc_df = dataset if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
query_df = pd.DataFrame()
if isinstance(x, list) and len(x) > 0 and isinstance(x[0], pd.DataFrame):
    query_df = x[0]

meta_cols = set([
    "file_name", "chunk_id", "txt", "source_type", "doc_id",
    "case_title", "case_number", "date", "source_url", "keywords",
    "section_type", "doc_meta_json", "summary", "tags", "source_hint",
    "latest_user_question", "conversation_context", "issue_keywords_csv",
    "query_class", "similarity", "distance", "lex_text", "lex_hits"
])

vector_cols = []
for c in doc_df.columns:
    if c in query_df.columns and c not in meta_cols:
        vector_cols.append(c)

preferred_cols = []
for c in vector_cols:
    lc = str(c).lower()
    if ("embed" in lc) or ("vector" in lc):
        preferred_cols.append(c)
if preferred_cols:
    vector_cols = preferred_cols

if len(query_df) < 1 or len(vector_cols) < 1:
    result = doc_df.iloc[0:0].copy()
else:
    q_text = ""
    if "txt" in query_df.columns and len(query_df) > 0:
        q_text = str(query_df.iloc[0].get("txt", "") or "").strip()

    source_blob = ""
    if len(doc_df) > 0 and "source_type" in doc_df.columns:
        try:
            source_blob = " ".join(doc_df["source_type"].fillna("").astype(str).head(20).tolist()).lower()
        except Exception:
            source_blob = ""
    is_aihub_store = ("aihub" in source_blob)

    sanction_fast_skip = False
    low_q = q_text.lower()
    if is_aihub_store:
        sanction_like = False
        for tok in ["징계", "양정", "견책", "근신", "감봉", "정직", "해고", "전보", "대기발령"]:
            if tok in low_q:
                sanction_like = True
                break
        litigation_like = False
        for tok in ["판례", "법원", "소송", "재판", "노동위", "행정소송", "형사", "민사"]:
            if tok in low_q:
                litigation_like = True
                break
        if sanction_like and not litigation_like:
            sanction_fast_skip = True

    if sanction_fast_skip:
        result = doc_df.iloc[0:0].copy()
        if "similarity" not in result.columns:
            result["similarity"] = pd.Series(dtype="float64")
    else:
        work = doc_df.copy()

        tokens = []
        q_norm_text = q_text
        for sep in [",", ".", "/", "|", "(", ")", "[", "]", "-", "_", "\n", "\t", ":", ";"]:
            q_norm_text = q_norm_text.replace(sep, " ")
        for tok in q_norm_text.split(" "):
            t = str(tok).strip().lower()
            if len(t) < 2:
                continue
            if t.isdigit():
                continue
            if t not in tokens:
                tokens.append(t)

        if tokens:
            lex_parts = []
            for c in ["txt", "keywords", "case_title", "section_type", "source_type", "file_name"]:
                if c in work.columns:
                    lex_parts.append(work[c].fillna("").astype(str))
            if lex_parts:
                lex = lex_parts[0]
                for s in lex_parts[1:]:
                    lex = lex + " " + s
                work["lex_text"] = lex.str.lower()
                work["lex_hits"] = 0
                for tok in tokens[:8]:
                    try:
                        work["lex_hits"] = work["lex_hits"] + work["lex_text"].str.contains(tok, regex=False, na=False).astype(int)
                    except Exception:
                        pass
                try:
                    hit_max = int(pd.to_numeric(work["lex_hits"], errors="coerce").fillna(0).max())
                except Exception:
                    hit_max = 0
                if hit_max > 0:
                    prefilter_n = 400 if is_aihub_store else 800
                    work = work.sort_values(by=["lex_hits"], ascending=False).head(prefilter_n).copy()

        for c in vector_cols:
            work[c] = pd.to_numeric(work[c], errors="coerce")
        work = work.dropna(subset=vector_cols, how="all")

        if len(work) < 1:
            result = work
        else:
            q_vec = pd.to_numeric(query_df[vector_cols].iloc[0], errors="coerce").fillna(0.0).astype(float).values
            q_norm = float(np.sqrt((q_vec ** 2).sum()))

            doc_mat = work[vector_cols].fillna(0.0).astype(float).values
            doc_norm = np.sqrt((doc_mat ** 2).sum(axis=1))
            dot = np.dot(doc_mat, q_vec)
            denom = doc_norm * q_norm
            denom = np.where(denom == 0, np.nan, denom)
            sim = dot / denom
            sim = np.nan_to_num(sim, nan=0.0, posinf=0.0, neginf=0.0)
            work["similarity"] = sim

            if "txt" in work.columns:
                keep = []
                for v in work["txt"].tolist():
                    s = "" if v is None else str(v).strip()
                    keep.append(bool(s and s.lower() != "nan"))
                try:
                    work = work.loc[keep].copy()
                except Exception:
                    pass

            final_top_k = 6 if is_aihub_store else 8
            result = work.sort_values(by=["similarity"], ascending=False).head(final_top_k).copy()
```

### 9.4 17D4_AIHUB, 17D4_RULES. 열 선택_문서근거

- `17D4_AIHUB`, `17D4_RULES`는 같은 열만 남긴다.

남길 열:

- `txt`
- `file_name`
- `chunk_id`
- `source_type`
- `doc_id`
- `case_title`
- `case_number`
- `date`
- `source_url`
- `keywords`
- `section_type`
- `doc_meta_json`
- `similarity`

`embedded_txt*` 같은 임베딩 숫자 열은 여기서 모두 제거한다.

### 9.5 18_내부패키징_AIHUB, 18_내부패키징_RULES

- 이 노드는 `2입력 Python` 노드다.
- Python 노드 기본 입력은 `dataset` 1개뿐이므로, 입력 포트를 직접 1개 추가해야 한다.
- 아래 2개 노드에 같은 코드를 복사해서 쓴다.
  - `18_내부패키징_AIHUB`
  - `18_내부패키징_RULES`
- 입력 포트:
  - `dataset`: `17D4_AIHUB` 또는 `17D4_RULES`
  - 추가 입력1 이름 권장: `query_anchor`
  - `query_anchor`: `17A`
- 즉 선 연결은 `17D4_AIHUB -> 18_내부패키징_AIHUB(dataset)`, `17A -> 18_내부패키징_AIHUB(query_anchor)`를 기본으로 두고, RULES도 같은 방식으로 연결한다.
- 다음 Python 코드 사용

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
anchor_df = pd.DataFrame()
if isinstance(x, list) and len(x) > 0 and isinstance(x[0], pd.DataFrame):
    anchor_df = x[0].copy()

keys = []
for src in [anchor_df, df]:
    for c in ["issue_keywords_csv", "latest_user_question"]:
        if c in src.columns:
            for v in src[c].tolist():
                s = "" if v is None else str(v).strip()
                if s and s.lower() != "nan":
                    keys.extend([p.strip() for p in s.replace("/", " ").replace(",", " ").split(" ") if p.strip()])

uniq = []
for k in keys:
    if len(k) < 2:
        continue
    if k not in uniq:
        uniq.append(k)
if not uniq:
    uniq = ["징계"]

fallback_qclass = "legal_analysis"
fallback_issue = ""
fallback_q = ""
fallback_ctx = ""
for src in [anchor_df, df]:
    for _, r in src.iterrows():
        qv = str(r.get("query_class", "") or "").strip().lower()
        if qv in ["legal_analysis", "internal_db_only", "mixed"]:
            fallback_qclass = qv
        iv = str(r.get("issue_keywords_csv", "") or "").strip()
        if iv and not fallback_issue:
            fallback_issue = iv
        qtxt = str(r.get("latest_user_question", "") or "").strip()
        if qtxt and not fallback_q:
            fallback_q = qtxt
        ctxt = str(r.get("conversation_context", "") or "").strip()
        if ctxt and not fallback_ctx:
            fallback_ctx = ctxt

rows = []
for _, r in df.iterrows():
    st = str(r.get("source_type", "") or "").strip().lower()
    if "query_fallback" in st:
        continue
    txt = str(r.get("txt", "") or r.get("chunk_text", "") or "").strip()
    if not txt:
        continue
    score = 0
    for k in uniq:
        if k in txt:
            score += 1
    if "similarity" in r and pd.notna(r["similarity"]):
        try:
            sim = float(r["similarity"])
            score = score + max(0, int(sim * 10))
        except Exception:
            pass
    rows.append({
        "source_type": st,
        "source_url": str(r.get("source_url", "") or "").strip(),
        "txt": txt,
        "score": int(score),
        "similarity": r.get("similarity", 0)
    })

ranked = pd.DataFrame(rows)
if ranked.empty:
    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue,
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "internal_evidence_context": "",
        "internal_source_urls": "",
        "internal_evidence_count": 0
    }])
else:
    try:
        ranked["similarity"] = pd.to_numeric(ranked["similarity"], errors="coerce").fillna(0.0)
    except Exception:
        ranked["similarity"] = 0.0
    ranked = ranked.sort_values(by=["similarity", "score"], ascending=[False, False]).reset_index(drop=True)
    top_k = ranked.head(8).copy()
    evidence_lines = []
    urls = []
    for _, rr in top_k.iterrows():
        tx = str(rr.get("txt", "") or "").strip()
        su = str(rr.get("source_url", "") or "").strip()
        sc = rr.get("score", 0)
        sim = rr.get("similarity", 0)
        st = str(rr.get("source_type", "") or "").strip()
        if tx:
            evidence_lines.append(f"[{st}|sim={sim:.4f}|score={sc}] {tx[:600]}")
        if su:
            disp = su
            if "//pdf/" in disp:
                disp = disp.split("//pdf/", 1)[1].strip()
            elif disp.startswith("internal://"):
                disp = disp.replace("internal://", "", 1).strip()
            disp = disp.replace(".pdf", "").replace(".xlsx", "").replace(".csv", "")
            if disp.startswith("\ud68c\uc0ac_"):
                disp = disp[len("\ud68c\uc0ac_"): ]
            if disp and disp not in urls:
                urls.append(disp)

    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue if fallback_issue else ", ".join(uniq[:5]),
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "internal_evidence_context": "\n\n".join(evidence_lines),
        "internal_source_urls": " | ".join(urls),
        "internal_evidence_count": len(evidence_lines)
    }])
```

### 9.6 18I_DOC. 파이썬_문서근거패키지병합

- 입력 포트:
  - `dataset`: `18_내부패키징_AIHUB`
  - 추가 입력1 이름 권장: `rules_package`
  - `rules_package`: `18_내부패키징_RULES`
- 즉 선 연결은 `18_내부패키징_AIHUB -> 18I_DOC(dataset)`, `18_내부패키징_RULES -> 18I_DOC(rules_package)`로 한다.
- 출력 포트: `output`

```python
frames = []
if isinstance(dataset, pd.DataFrame):
    frames.append(dataset.copy())
if isinstance(x, list):
    for part in x:
        if isinstance(part, pd.DataFrame):
            frames.append(part.copy())

query_class = ""
issue_keywords_csv = ""
latest_user_question = ""
conversation_context = ""
contexts = []
urls = []
count_total = 0

for frame in frames:
    if frame is None or len(frame) < 1:
        continue
    row = frame.iloc[0].to_dict()
    if not query_class:
        query_class = str(row.get("query_class", "") or "").strip()
    if not issue_keywords_csv:
        issue_keywords_csv = str(row.get("issue_keywords_csv", "") or "").strip()
    if not latest_user_question:
        latest_user_question = str(row.get("latest_user_question", "") or "").strip()
    if not conversation_context:
        conversation_context = str(row.get("conversation_context", "") or "").strip()

    ctx = str(row.get("internal_evidence_context", "") or "").strip()
    if ctx:
        contexts.append(ctx)

    srcs = str(row.get("internal_source_urls", "") or "").strip()
    if srcs:
        for s in srcs.split("|"):
            u = str(s).strip()
            if u and u not in urls:
                urls.append(u)

    try:
        count_total += int(float(str(row.get("internal_evidence_count", 0) or 0).strip()))
    except Exception:
        pass

result = pd.DataFrame([{
    "query_class": query_class if query_class else "legal_analysis",
    "issue_keywords_csv": issue_keywords_csv,
    "latest_user_question": latest_user_question,
    "conversation_context": conversation_context,
    "internal_evidence_context": "\n\n".join([c for c in contexts if c]),
    "internal_source_urls": " | ".join(urls),
    "internal_evidence_count": count_total
}])
```

### 9.7 17D3_의견서. 파이썬_의견서유사문서TOPK

- 이 노드는 `3입력 Python` 노드다.
- 입력 포트:
  - `dataset`: `D6R_의견서`
  - 추가 입력1 이름 권장: `query`
  - `query`: `17D2`
  - 추가 입력2 이름 권장: `query_anchor`
  - `query_anchor`: `17A`
- 출력 포트: `output`
- 목적:
  - `Opinion_Letters` 저장소는 일반 질의에서 검색하지 않는다.
  - 질문 원문에 아래 4개 트리거가 있을 때만 유사 의견서 문안을 top-k로 검색한다.
  - `의견서`
  - `검토의견`
  - `법률검토`
  - `답변서`
  - 검색 목적은 `같은 내용을 복사`하는 것이 아니라, `제목/문단/결론/문체` 예시를 몇 개 가져오는 것이다.

```python
doc_df = dataset if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
query_df = pd.DataFrame()
anchor_df = pd.DataFrame()
if isinstance(x, list):
    if len(x) > 0 and isinstance(x[0], pd.DataFrame):
        query_df = x[0]
    if len(x) > 1 and isinstance(x[1], pd.DataFrame):
        anchor_df = x[1]

trigger_text = ""
if len(anchor_df) > 0:
    trigger_text = str(anchor_df.iloc[0].get("latest_user_question", "") or "").strip().lower()

draft_mode = False
for token in ["의견서", "검토의견", "법률검토", "답변서"]:
    if token in trigger_text:
        draft_mode = True
        break

if not draft_mode:
    empty_df = doc_df.iloc[0:0].copy() if isinstance(doc_df, pd.DataFrame) else pd.DataFrame()
    if "similarity" not in empty_df.columns:
        empty_df["similarity"] = pd.Series(dtype="float64")
    result = empty_df
else:
    meta_cols = set([
        "file_name", "chunk_id", "txt", "source_type", "doc_id",
        "case_title", "case_number", "date", "source_url", "keywords",
        "section_type", "doc_meta_json", "summary", "tags", "source_hint",
        "latest_user_question", "conversation_context", "issue_keywords_csv",
        "query_class", "similarity", "distance", "lex_text", "lex_hits"
    ])

    vector_cols = []
    for c in doc_df.columns:
        if c in query_df.columns and c not in meta_cols:
            vector_cols.append(c)

    preferred_cols = []
    for c in vector_cols:
        lc = str(c).lower()
        if ("embed" in lc) or ("vector" in lc):
            preferred_cols.append(c)
    if preferred_cols:
        vector_cols = preferred_cols

    if len(query_df) < 1 or len(vector_cols) < 1:
        empty_df = doc_df.iloc[0:0].copy()
        if "similarity" not in empty_df.columns:
            empty_df["similarity"] = pd.Series(dtype="float64")
        result = empty_df
    else:
        work = doc_df.copy()

        q_text = ""
        if "txt" in query_df.columns and len(query_df) > 0:
            q_text = str(query_df.iloc[0].get("txt", "") or "").strip()

        tokens = []
        q_norm_text = q_text
        for sep in [",", ".", "/", "|", "(", ")", "[", "]", "-", "_", "\n", "\t", ":", ";"]:
            q_norm_text = q_norm_text.replace(sep, " ")
        for tok in q_norm_text.split(" "):
            t = str(tok).strip().lower()
            if len(t) < 2:
                continue
            if t.isdigit():
                continue
            if t not in tokens:
                tokens.append(t)

        if tokens:
            lex_parts = []
            for c in ["txt", "keywords", "case_title", "section_type", "source_type", "file_name", "case_number"]:
                if c in work.columns:
                    lex_parts.append(work[c].fillna("").astype(str))
            if lex_parts:
                lex = lex_parts[0]
                for s in lex_parts[1:]:
                    lex = lex + " " + s
                work["lex_text"] = lex.str.lower()
                work["lex_hits"] = 0
                for tok in tokens[:8]:
                    try:
                        work["lex_hits"] = work["lex_hits"] + work["lex_text"].str.contains(tok, regex=False, na=False).astype(int)
                    except Exception:
                        pass
                try:
                    hit_max = int(pd.to_numeric(work["lex_hits"], errors="coerce").fillna(0).max())
                except Exception:
                    hit_max = 0
                if hit_max > 0:
                    work = work.sort_values(by=["lex_hits"], ascending=False).head(300).copy()

        for c in vector_cols:
            work[c] = pd.to_numeric(work[c], errors="coerce")
        work = work.dropna(subset=vector_cols, how="all")

        if len(work) < 1:
            result = work
        else:
            q_vec = pd.to_numeric(query_df[vector_cols].iloc[0], errors="coerce").fillna(0.0).astype(float).values
            q_norm = float(np.sqrt((q_vec ** 2).sum()))

            doc_mat = work[vector_cols].fillna(0.0).astype(float).values
            doc_norm = np.sqrt((doc_mat ** 2).sum(axis=1))
            dot = np.dot(doc_mat, q_vec)
            denom = doc_norm * q_norm
            denom = np.where(denom == 0, np.nan, denom)
            sim = dot / denom
            sim = np.nan_to_num(sim, nan=0.0, posinf=0.0, neginf=0.0)
            work["similarity"] = sim

            result = work.sort_values(by=["similarity"], ascending=False).head(5).copy()
```

### 9.7 17D4_의견서. 열 선택_의견서근거

- `17D4_AIHUB`, `17D4_RULES`와 동일한 열만 남긴다.
- 즉 아래 열만 선택한다.
  - `txt`
  - `file_name`
  - `chunk_id`
  - `source_type`
  - `doc_id`
  - `case_title`
  - `case_number`
  - `date`
  - `source_url`
  - `keywords`
  - `section_type`
  - `doc_meta_json`
  - `similarity`

### 9.8 18_내부패키징_의견서

- 입력 포트:
  - `dataset`: `17D4_의견서`
  - 추가 입력1 이름 권장: `query_anchor`
  - `query_anchor`: `17A`
- 목적:
  - 작성형 요청일 때만 과거 검토의견/답변서 문안을 `스타일 예시` 패키지로 만든다.
  - 이 노드는 의견서 문안을 `법적 근거`로 합치지 않는다.
  - 트리거가 없으면 `17D3_의견서`가 빈 결과를 내므로, 이 노드도 자연스럽게 빈 스타일 패키지로 지나간다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
anchor_df = pd.DataFrame()
if isinstance(x, list) and len(x) > 0 and isinstance(x[0], pd.DataFrame):
    anchor_df = x[0].copy()

fallback_qclass = "legal_analysis"
fallback_issue = ""
fallback_q = ""
fallback_ctx = ""
trigger_text = ""

for src in [anchor_df, df]:
    if src is None or len(src) < 1:
        continue
    for _, r in src.iterrows():
        qv = str(r.get("query_class", "") or "").strip().lower()
        if qv in ["legal_analysis", "internal_db_only", "mixed"]:
            fallback_qclass = qv
        iv = str(r.get("issue_keywords_csv", "") or "").strip()
        if iv and not fallback_issue:
            fallback_issue = iv
        qtxt = str(r.get("latest_user_question", "") or "").strip()
        if qtxt and not fallback_q:
            fallback_q = qtxt
            trigger_text = qtxt.lower()
        ctxt = str(r.get("conversation_context", "") or "").strip()
        if ctxt and not fallback_ctx:
            fallback_ctx = ctxt

draft_mode = False
for token in ["의견서", "검토의견", "법률검토", "답변서"]:
    if token in trigger_text:
        draft_mode = True
        break

if not draft_mode or df.empty:
    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue,
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "draft_style_examples": "",
        "draft_style_source_urls": "",
        "draft_style_count": 0
    }])
else:
    if "similarity" in df.columns:
        try:
            df["similarity"] = pd.to_numeric(df["similarity"], errors="coerce").fillna(0.0)
            df = df.sort_values(by=["similarity"], ascending=False).reset_index(drop=True)
        except Exception:
            pass

    style_lines = []
    style_urls = []
    for _, r in df.head(4).iterrows():
        title = str(r.get("case_title", "") or r.get("file_name", "") or "").strip()
        txt = str(r.get("txt", "") or "").strip()
        sim = 0.0
        try:
            sim = float(r.get("similarity", 0) or 0)
        except Exception:
            pass
        if txt:
            prefix = f"[제목:{title}|sim={sim:.4f}] " if title else f"[sim={sim:.4f}] "
            style_lines.append(prefix + txt[:700])
        su = str(r.get("source_url", "") or "").strip()
        if su:
            disp = su.replace("internal://opinion_letter/", "").replace(".txt", "").strip()
            if disp and disp not in style_urls:
                style_urls.append(disp)

    result = pd.DataFrame([{
        "query_class": fallback_qclass,
        "issue_keywords_csv": fallback_issue,
        "latest_user_question": fallback_q,
        "conversation_context": fallback_ctx,
        "draft_style_examples": "\n\n".join(style_lines),
        "draft_style_source_urls": " | ".join(style_urls),
        "draft_style_count": len(style_lines)
    }])
```

### 9.10 18I1. 파이썬_내부근거패키지병합

- 입력 포트:
  - `dataset`: `18_내부패키징_구조화`
  - 추가 입력1 이름 권장: `doc_package`
  - `doc_package`: `18I_DOC`
  - 추가 입력2 이름 권장: `opinion_package`
  - `opinion_package`: `18_내부패키징_의견서`
- 즉 선 연결은 `18_내부패키징_구조화 -> 18I1(dataset)`, `18I_DOC -> 18I1(doc_package)`, `18_내부패키징_의견서 -> 18I1(opinion_package)`로 한다.
- 출력 포트: `output`

```python
frames = []
if isinstance(dataset, pd.DataFrame):
    frames.append(dataset.copy())
if isinstance(x, list):
    for part in x:
        if isinstance(part, pd.DataFrame):
            frames.append(part.copy())

query_class = ""
issue_keywords_csv = ""
latest_user_question = ""
conversation_context = ""
contexts = []
urls = []
count_total = 0
case_briefs = []
case_count = 0
sanction_query_flag = 0
style_examples = []
style_urls = []
style_count = 0

for frame in frames:
    if frame is None or len(frame) < 1:
        continue
    row = frame.iloc[0].to_dict()
    if not query_class:
        query_class = str(row.get("query_class", "") or "").strip()
    if not issue_keywords_csv:
        issue_keywords_csv = str(row.get("issue_keywords_csv", "") or "").strip()
    if not latest_user_question:
        latest_user_question = str(row.get("latest_user_question", "") or "").strip()
    if not conversation_context:
        conversation_context = str(row.get("conversation_context", "") or "").strip()

    ctx = str(row.get("internal_evidence_context", "") or "").strip()
    if ctx:
        contexts.append(ctx)

    srcs = str(row.get("internal_source_urls", "") or "").strip()
    if srcs:
        for s in srcs.split("|"):
            u = str(s).strip()
            if u and u not in urls:
                urls.append(u)

    try:
        count_total += int(float(str(row.get("internal_evidence_count", 0) or 0).strip()))
    except Exception:
        pass

    briefs = str(row.get("internal_case_briefs", "") or "").strip()
    if briefs:
        case_briefs.append(briefs)

    try:
        case_count += int(float(str(row.get("internal_case_count", 0) or 0).strip()))
    except Exception:
        pass

    try:
        sanction_query_flag = max(sanction_query_flag, int(float(str(row.get("sanction_query_flag", 0) or 0).strip())))
    except Exception:
        pass

    ex = str(row.get("draft_style_examples", "") or "").strip()
    if ex:
        style_examples.append(ex)

    surls = str(row.get("draft_style_source_urls", "") or "").strip()
    if surls:
        for s in surls.split("|"):
            u = str(s).strip()
            if u and u not in style_urls:
                style_urls.append(u)

    try:
        style_count += int(float(str(row.get("draft_style_count", 0) or 0).strip()))
    except Exception:
        pass

result = pd.DataFrame([{
    "query_class": query_class if query_class else "legal_analysis",
    "issue_keywords_csv": issue_keywords_csv,
    "latest_user_question": latest_user_question,
    "conversation_context": conversation_context,
    "internal_evidence_context": "\n\n".join([c for c in contexts if c]),
    "internal_source_urls": " | ".join(urls),
    "internal_evidence_count": count_total,
    "internal_case_briefs": "\n".join([c for c in case_briefs if c]),
    "internal_case_count": case_count,
    "sanction_query_flag": sanction_query_flag,
    "draft_style_examples": "\n\n".join([c for c in style_examples if c]),
    "draft_style_source_urls": " | ".join(style_urls),
    "draft_style_count": style_count
}])
```

### 10.1 18W_A. 에이전트 프롬프트_웹보완검색_병렬

- 입력 컬럼:
  - `latest_user_question`
  - `conversation_context`
  - `query_class`
  - `issue_keywords_csv`
- 모델: `gpt-5.2`
- Output column name: `output_response`
- Include Question Prompt in output: `ON`
- is bind tools: `ON`

```text
현재 질문: {{latest_user_question}}
최근 대화 맥락: {{conversation_context}}
질의 분류: {{query_class}}
쟁점 키워드: {{issue_keywords_csv}}

역할:
- 공식 API와 내부 DB에서 비거나 약한 부분을 보완하기 위해, 공식기관/법원/정부/공공기관/공식 발간자료를 먼저 검색한다.
- 특히 법령 조문, 판례 식별정보(법원/선고일/사건번호), 고용노동부 해석·매뉴얼, 노동위·법원 보도자료를 우선 확보한다.
- 한 번에 답하지 말고 내부적으로 아래 순서를 거쳐 심층 리서치를 수행한다.
  - 1차: 쟁점에 맞는 공식 도메인과 문서 유형 선정
  - 2차: 공식 자료 2건 이상 또는 공식 1건 + 준공식 1건으로 교차검증
  - 3차: 공식근거와 충돌하거나 과장된 설명 제거
- 답변이 아니라 JSON만 출력한다.
- A&B식 검색 프로토콜을 차용해, 내부적으로 `검색 청사진 -> 후보 검증 -> 교차검증 -> 과장 제거` 순서로 생각하되 그 계획은 출력하지 않는다.

출력 스키마:
{
  "web_evidence_context": "",
  "web_source_urls": []
}

규칙:
1) 현재 질문 1건만 기준으로 검색한다.
2) 검색 우선순위는 아래 순서를 따른다.
   - 1순위: `law.go.kr`, `scourt.go.kr`, `moel.go.kr`, `epeople.go.kr`, `kli.re.kr`, 중앙노동위원회/지방노동위, 기타 정부·공공기관
   - 2순위: 대형 로펌/공신력 있는 노무법인/학술·연구기관의 판례리뷰·실무자료
   - 3순위: 일반 블로그/마케팅성 글
3) 3순위 소스는 1·2순위가 부족할 때만 최소한으로 사용한다. 1·2순위가 있으면 블로그는 넣지 않는다.
4) 내부DB와 공식근거를 뒤집는 용도로 쓰지 말고, 절차 설명·실무 안내·최신 행정 안내를 보완한다.
5) 공식근거와 같은 말을 반복하지 말고, 공식근거에 없는 실무 포인트만 보강한다.
6) 전보/불리한 처우/부당해고/징계무효처럼 판례성이 강한 쟁점이면, 가능하면 공식 판례 또는 법원/KLI 요약자료를 1건 이상 확보한다.
7) 절차형 질문이면, 가능하면 법령 또는 고용노동부 매뉴얼/해석을 1건 이상 확보한다.
8) 공식 자료가 하나뿐이면 그 자료가 실제로 질문에 직접 답하는지 다시 검토하고, 부족하면 준공식 실무자료를 보완한다.
6) URL은 실제로 열어본 것만 넣는다.
9) 가능하면 `web_evidence_context` 안에 아래 식별정보를 먼저 드러낸다.
   - 문서명 또는 판례명
   - 기관명
   - 날짜
   - 핵심 포인트 1~2문장
10) `web_evidence_context`는 단순 링크 나열이 아니라, 검색 전략이 드러나는 요약 근거여야 한다.
11) 답변 문체 금지. 메타 안내문, 입력 부족 안내문, 컬럼명 언급 금지.
12) 예시 형식:
   - `[고용노동부|2023-05-10] 직장 내 괴롭힘 판단 및 예방·대응 매뉴얼: 피해자 의사에 반하는 조치는 신중해야 함`
   - `[대법원 보도자료|2017-12-22] 불리한 조치 판단은 시간적 근접성, 경위, 불이익 정도를 종합 평가`
13) 내부적으로 아래 순서를 지켜 검색한다.
   - `search blueprint`: 질문이 법령형/판례형/절차형/실무형 중 무엇인지 결정
   - `source targeting`: 그 질문에 맞는 공식 도메인과 문서유형 선택
   - `cross check`: 공식 2건 이상 또는 공식 1건 + 준공식 1건으로 교차검증
   - `claim pruning`: 교차검증 안 된 문장은 버림
```

### 10.2 18W_B. 파이썬_웹검색파싱_병렬

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
if len(df) > 1:
    df = df.tail(1).copy()

raw = ""
for c in ["output_response", "web_search_json", "result_text", "answer"]:
    if c in df.columns:
        raw = str(df.iloc[0].get(c, "") or "").strip()
        if raw:
            break

obj = {}
if raw:
    try:
        obj = json.loads(raw)
    except Exception:
        m = re.search(r"\{.*\}", raw, re.S)
        if m:
            try:
                obj = json.loads(m.group(0))
            except Exception:
                obj = {}

web_ctx = ""
if isinstance(obj, dict):
    web_ctx = str(obj.get("web_evidence_context", "") or "").strip()

urls = []
if isinstance(obj, dict):
    arr = obj.get("web_source_urls", [])
    if isinstance(arr, list):
        for x in arr:
            sx = "" if x is None else str(x).strip()
            if sx:
                urls.append(sx)

if (not urls) and raw:
    found = re.findall(r"https?://[^\s\]\)\>\"]+", raw)
    for u in found:
        if u not in urls:
            urls.append(u)

row = {}
if len(df) > 0:
    row = df.iloc[0].to_dict()

result = pd.DataFrame([{
    "query_class": str(row.get("query_class", "") or "legal_analysis").strip(),
    "issue_keywords_csv": str(row.get("issue_keywords_csv", "") or "").strip(),
    "latest_user_question": str(row.get("latest_user_question", "") or "").strip(),
    "conversation_context": str(row.get("conversation_context", "") or "").strip(),
    "web_evidence_context": web_ctx,
    "web_source_urls": " | ".join(urls),
    "web_evidence_count": len(urls)
}])
```

### 10.3 18E_A. 파이썬_ELABOR질의정리

- 입력 포트: `dataset`
- 출력 포트: `output`
- 역할:
  - `elabor` 보조검색을 켤 질문인지 판정한다.
  - 판례/행정해석/징계·양정형 질문에서만 검색어를 만들어 `ELABOR proxy`로 넘긴다.
  - 검색을 안 써야 하는 질문이면 `elabor_use_flag=0`으로 내려 속도를 아낀다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

row = df.iloc[0].to_dict() if len(df) > 0 else {}

latest_user_question = "" if row.get("latest_user_question", "") is None else str(row.get("latest_user_question", "")).strip()
if latest_user_question.lower() == "nan":
    latest_user_question = ""
conversation_context = "" if row.get("conversation_context", "") is None else str(row.get("conversation_context", "")).strip()
if conversation_context.lower() == "nan":
    conversation_context = ""
query_class = "" if row.get("query_class", "") is None else str(row.get("query_class", "")).strip()
if query_class.lower() == "nan":
    query_class = ""
if not query_class:
    query_class = "legal_analysis"
issue_keywords_csv = "" if row.get("issue_keywords_csv", "") is None else str(row.get("issue_keywords_csv", "")).strip()
if issue_keywords_csv.lower() == "nan":
    issue_keywords_csv = ""
precedent_query = "" if row.get("precedent_query", "") is None else str(row.get("precedent_query", "")).strip()
if precedent_query.lower() == "nan":
    precedent_query = ""
interpretation_query = "" if row.get("interpretation_query", "") is None else str(row.get("interpretation_query", "")).strip()
if interpretation_query.lower() == "nan":
    interpretation_query = ""

trigger_text = f"{latest_user_question} {issue_keywords_csv}".lower()

use_elabor = False
for tok in [
    "판례", "대법원", "고등법원", "법원", "행정해석",
    "전보", "불리한 처우", "부당해고", "징계", "양정",
    "음주", "직장 내 괴롭힘", "성희롱", "정직", "해고"
]:
    if tok in trigger_text:
        use_elabor = True
        break

elabor_mode = "precedent"
if "행정해석" in trigger_text:
    elabor_mode = "interpretation"
elif any(tok in trigger_text for tok in ["징계", "양정", "음주", "해고", "괴롭힘"]):
    elabor_mode = "mixed"

if elabor_mode == "precedent":
    elabor_query = precedent_query if precedent_query else latest_user_question
elif elabor_mode == "interpretation":
    elabor_query = interpretation_query if interpretation_query else latest_user_question
else:
    if precedent_query:
        elabor_query = precedent_query
    elif issue_keywords_csv:
        elabor_query = issue_keywords_csv.replace(",", " ").strip()
    else:
        elabor_query = latest_user_question

if "성희롱" in trigger_text and ("전보" in trigger_text or "보호조치" in trigger_text or "근무장소" in trigger_text):
    elabor_mode = "mixed"
    elabor_query = "직장 내 성희롱 보호조치 전보"

if "괴롭힘" in trigger_text and ("전보" in trigger_text or "보호조치" in trigger_text):
    elabor_mode = "mixed"
    elabor_query = "직장 내 괴롭힘 보호조치 전보"

result = pd.DataFrame([{
    "query_class": query_class,
    "issue_keywords_csv": issue_keywords_csv,
    "latest_user_question": latest_user_question,
    "conversation_context": conversation_context,
    "elabor_use_flag": 1 if use_elabor else 0,
    "elabor_mode": elabor_mode,
    "elabor_query": elabor_query
}])
```

### 10.4 18E_B. Custom API_ELABOR검색

- 입력: `18E_A`
- 출력 컬럼 예: `output_response`
- 목적:
  - AI Canvas 안에서 직접 로그인 스크래핑하지 않고, 외부 `elabor-proxy`에 검색만 위임한다.
  - 유료회원 계정, 쿠키, 세션 관리는 모두 `elabor-proxy`가 맡는다.

권장 설정:

- Method: `POST`
- URL: `https://<your-elabor-proxy>/search`
- Headers(JSON):

```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer <ELABOR_PROXY_TOKEN>"
}
```

- 인증 없이 먼저 로컬 테스트만 할 경우:

```json
{
  "Content-Type": "application/json"
}
```
- Timeout: `30~40초`

권장 요청 body:

```json
{
  "use_flag": "{{elabor_use_flag}}",
  "mode": "{{elabor_mode}}",
  "query": "{{elabor_query}}",
  "question": "{{latest_user_question}}"
}
```

권장 응답 형식:

```json
{
  "ok": true,
  "items": [
    {
      "type": "precedent",
      "title": "직장 내 괴롭힘 신고 후 원거리 전보 사건",
      "case_number": "대법원 2022도4925",
      "date": "2023-03-21",
      "summary": "신고 이후 원거리 전보와 실질적 불이익이 문제된 사안",
      "url": "https://www.elabor.co.kr/..."
    }
  ]
}
```

주의:

- `elabor_use_flag=0`이면 proxy는 바로 빈 결과를 반환하게 한다.
- 아이디/비밀번호는 AI Canvas에 넣지 않는다. proxy 서버 환경변수로만 관리한다.
- `elabor`는 공식 근거를 대체하지 않고, 판례/행정해석 후보를 빨리 찾는 보조 레이어다.

### 10.5 18E_C. 파이썬_ELABOR파싱

- 입력 포트: `dataset`
- 출력 포트: `output`
- 역할:
  - `ELABOR proxy` JSON 응답을 AI Canvas용 보조근거 컬럼으로 정리한다.
  - 제목, 사건번호, 날짜, 요지, URL만 짧게 살린다.

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()
base = df.iloc[0].to_dict() if len(df) > 0 else {}

items = []

# 1) AI Canvas가 JSON을 컬럼으로 펼쳐준 경우
if "items" in df.columns and len(df) > 0:
    raw_items = df.iloc[0].get("items", [])
    if isinstance(raw_items, list):
        items = raw_items
    else:
        text_items = "" if raw_items is None else str(raw_items).strip()
        if text_items and text_items.lower() != "nan":
            try:
                items = json.loads(text_items)
            except Exception:
                blocks = re.findall(r"\{[^{}]*\}", text_items, re.S)
                parsed_items = []
                for block in blocks:
                    one = {}
                    for key in ["type", "title", "case_number", "date", "summary", "url"]:
                        m = re.search(r"'" + key + r"'\s*:\s*'((?:\\'|[^'])*)'", block, re.S)
                        if m:
                            one[key] = m.group(1).replace("\\'", "'").strip()
                        else:
                            one[key] = ""
                    if one.get("title") or one.get("case_number") or one.get("url"):
                        parsed_items.append(one)
                items = parsed_items

# 2) 원문 JSON 문자열로 내려온 경우 fallback
if not items:
    raw = ""
    for c in ["output_response", "result_text", "answer", "response", "body", "data"]:
        if c in df.columns:
            raw = str(df.iloc[0].get(c, "") or "").strip()
            if raw:
                break

    obj = {}
    if raw:
        try:
            obj = json.loads(raw)
        except Exception:
            m = re.search(r"\{.*\}", raw, re.S)
            if m:
                try:
                    obj = json.loads(m.group(0))
                except Exception:
                    obj = {}

    arr = obj.get("items", []) if isinstance(obj, dict) else []
    if isinstance(arr, list):
        items = arr

lines = []
urls = []

for item in items[:5]:
    if not isinstance(item, dict):
        continue
    kind = str(item.get("type", "") or "precedent").strip()
    title = str(item.get("title", "") or "").strip()
    case_number = str(item.get("case_number", "") or "").strip()
    date = str(item.get("date", "") or "").strip()
    summary = str(item.get("summary", "") or "").strip()
    url = str(item.get("url", "") or "").strip()

    bits = ["ELABOR 보조DB"]
    if kind:
        bits.append(kind)
    if case_number:
        bits.append(case_number)
    elif title:
        bits.append(title)
    if date:
        bits.append(date)

    head = " | ".join(bits)
    body = summary if summary else title
    if body:
        lines.append(f"[{head}] {body[:220]}")
    if url and url not in urls:
        urls.append(url)

result = pd.DataFrame([{
    "query_class": str(base.get("query_class", "") or "legal_analysis").strip(),
    "issue_keywords_csv": str(base.get("issue_keywords_csv", "") or "").strip(),
    "latest_user_question": str(base.get("latest_user_question", "") or "").strip(),
    "conversation_context": str(base.get("conversation_context", "") or "").strip(),
    "elabor_evidence_context": "\n\n".join(lines),
    "elabor_source_urls": " | ".join(urls),
    "elabor_evidence_count": len(lines)
}])
```

## 11. 최종 병합 / 답변 / 차단

### 11.1 18M1. 파이썬_근거패키지병합1_공식내부

- 입력 포트:
  - `dataset`: `18_공식패키징`
  - 추가 입력1 이름 권장: `internal_package`
  - `internal_package`: `18I1`
- 즉 선 연결은 `18_공식패키징 -> 18M1(dataset)`, `18I1 -> 18M1(internal_package)`로 한다.
- 출력 포트: `output`

```python
frames = []
if isinstance(dataset, pd.DataFrame):
    frames.append(dataset.copy())
if isinstance(x, list):
    for part in x:
        if isinstance(part, pd.DataFrame):
            frames.append(part.copy())

merged = {
    "query_class": "",
    "issue_keywords_csv": "",
    "latest_user_question": "",
    "conversation_context": "",
    "official_evidence_context": "",
    "official_source_urls": "",
    "official_evidence_count": 0,
    "official_citation_briefs": "",
    "internal_evidence_context": "",
    "internal_source_urls": "",
    "internal_evidence_count": 0,
    "internal_case_briefs": "",
    "internal_case_count": 0,
    "sanction_query_flag": 0,
    "draft_style_examples": "",
    "draft_style_source_urls": "",
    "draft_style_count": 0
}

for frame in frames:
    if frame is None or len(frame) < 1:
        continue
    row = frame.iloc[0].to_dict()
    for k in ["query_class", "issue_keywords_csv", "latest_user_question", "conversation_context"]:
        if not str(merged.get(k, "") or "").strip():
            merged[k] = row.get(k, "")
    for k in [
        "official_evidence_context", "official_source_urls", "official_citation_briefs", "internal_evidence_context",
        "internal_source_urls", "internal_case_briefs", "draft_style_examples", "draft_style_source_urls"
    ]:
        if not str(merged.get(k, "") or "").strip():
            merged[k] = row.get(k, "")
    for k in ["official_evidence_count", "internal_evidence_count", "internal_case_count", "draft_style_count", "sanction_query_flag"]:
        try:
            merged[k] = int(merged.get(k, 0)) + int(float(str(row.get(k, 0) or 0).strip()))
        except Exception:
            pass

result = pd.DataFrame([merged])
```

### 11.2 18M2. 파이썬_근거패키지병합2_웹

- 입력 포트:
  - `dataset`: `18M1`
  - 추가 입력1 이름 권장: `web_package`
  - `web_package`: `18W_B`
- 즉 선 연결은 `18M1 -> 18M2(dataset)`, `18W_B -> 18M2(web_package)`로 한다.
- 출력 포트: `output`

```python
frames = []
if isinstance(dataset, pd.DataFrame):
    frames.append(dataset.copy())
if isinstance(x, list):
    for part in x:
        if isinstance(part, pd.DataFrame):
            frames.append(part.copy())

merged = {
    "query_class": "",
    "issue_keywords_csv": "",
    "latest_user_question": "",
    "conversation_context": "",
    "official_evidence_context": "",
    "official_source_urls": "",
    "official_evidence_count": 0,
    "official_citation_briefs": "",
    "internal_evidence_context": "",
    "internal_source_urls": "",
    "internal_evidence_count": 0,
    "internal_case_briefs": "",
    "internal_case_count": 0,
    "sanction_query_flag": 0,
    "web_evidence_context": "",
    "web_source_urls": "",
    "web_evidence_count": 0,
    "draft_style_examples": "",
    "draft_style_source_urls": "",
    "draft_style_count": 0
}

for frame in frames:
    if frame is None or len(frame) < 1:
        continue
    row = frame.iloc[0].to_dict()
    for k in ["query_class", "issue_keywords_csv", "latest_user_question", "conversation_context"]:
        if not str(merged.get(k, "") or "").strip():
            merged[k] = row.get(k, "")
    for k in [
        "official_evidence_context", "official_source_urls", "official_citation_briefs", "internal_evidence_context",
        "internal_source_urls", "internal_case_briefs", "web_evidence_context", "web_source_urls",
        "draft_style_examples", "draft_style_source_urls"
    ]:
        if not str(merged.get(k, "") or "").strip():
            merged[k] = row.get(k, "")
    for k in ["official_evidence_count", "internal_evidence_count", "internal_case_count", "web_evidence_count", "draft_style_count", "sanction_query_flag"]:
        try:
            merged[k] = int(merged.get(k, 0)) + int(float(str(row.get(k, 0) or 0).strip()))
        except Exception:
            pass

result = pd.DataFrame([merged])
```

### 11.3 18M3. 파이썬_근거패키지병합3_ELABOR

- 입력 포트:
  - `dataset`: `18M2`
  - 추가 입력1 이름 권장: `elabor_package`
  - `elabor_package`: `18E_C`
- 즉 선 연결은 `18M2 -> 18M3(dataset)`, `18E_C -> 18M3(elabor_package)`로 한다.
- 출력 포트: `output`
- 역할:
  - 기존 `18M2`에 `elabor` 보조검색 결과를 합친다.
  - `elabor`는 공식 근거를 대체하지 않고, 웹 보완 레이어에 추가하는 방식으로 합친다.

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
        if k not in merged or not str(merged.get(k, "") or "").strip():
            merged[k] = v

web_ctx_parts = []
web_urls = []
web_count = 0

base_ctx = str(merged.get("web_evidence_context", "") or "").strip()
if base_ctx:
    web_ctx_parts.append(base_ctx)

base_urls = str(merged.get("web_source_urls", "") or "").strip()
if base_urls:
    for u in base_urls.split("|"):
        su = str(u).strip()
        if su and su not in web_urls:
            web_urls.append(su)

try:
    web_count += int(float(str(merged.get("web_evidence_count", 0) or 0).strip()))
except Exception:
    pass

for frame in frames:
    if frame is None or len(frame) < 1:
        continue
    row = frame.iloc[0].to_dict()

    ectx = str(row.get("elabor_evidence_context", "") or "").strip()
    if ectx:
        web_ctx_parts.append("[ELABOR 보조DB]\n" + ectx)

    eurls = str(row.get("elabor_source_urls", "") or "").strip()
    if eurls:
        for u in eurls.split("|"):
            su = str(u).strip()
            if su and su not in web_urls:
                web_urls.append(su)

    try:
        web_count += int(float(str(row.get("elabor_evidence_count", 0) or 0).strip()))
    except Exception:
        pass

merged["web_evidence_context"] = "\n\n".join([x for x in web_ctx_parts if x])
merged["web_source_urls"] = " | ".join(web_urls)
merged["web_evidence_count"] = web_count

result = pd.DataFrame([merged])
```

호환 메모:

- 아직 `19` 단일 최종답변 노드를 쓰는 경우엔 `18M3 -> 19`로 연결한다.
- `19A -> 19B` 구조를 쓰는 경우엔 `18M3 -> 19A`로 연결한다.

### 11.4 19A. 에이전트 프롬프트_법률메모생성

- 모델: `gpt-5.2`
- Output column name: `legal_memo_json`
- num response: `1`
- Include Question Prompt in output: `ON`
- is bind tools: `OFF`

```text
현재 질문:
{{latest_user_question}}

최근 대화 맥락:
{{conversation_context}}

질의 분류:
{{query_class}}

쟁점 키워드:
{{issue_keywords_csv}}

내부 구조화/문서 근거:
{{internal_evidence_context}}

내부 참고 출처(사용자 노출용 정리값):
{{internal_source_urls}}

내부 근거 수:
{{internal_evidence_count}}

내부 유사 사례 요약:
{{internal_case_briefs}}

내부 유사 사례 수:
{{internal_case_count}}

징계/양정 질문 플래그:
{{sanction_query_flag}}

작성 스타일 예시(Opinion_Letters, 작성형 요청일 때만 사용):
{{draft_style_examples}}

작성 스타일 참고 문서:
{{draft_style_source_urls}}

작성 스타일 예시 수:
{{draft_style_count}}

공식 근거:
{{official_evidence_context}}

공식 근거 링크:
{{official_source_urls}}

공식 근거 수:
{{official_evidence_count}}

공식 citation 요약:
{{official_citation_briefs}}

웹 보완 근거:
{{web_evidence_context}}

웹 보완 링크:
{{web_source_urls}}

웹 보완 수:
{{web_evidence_count}}

역할:
- 내부 구조화 DB, 내부 문서 DB, 공식근거, 웹근거를 종합해 먼저 `법률 메모 JSON`을 만든다.
- 사고 과정은 출력하지 말고, 최종 산출물은 반드시 JSON만 출력한다.
- 내부적으로 아래 4단계를 반드시 거친다.
  - 1단계: 쟁점별 근거 우선순위 확정
  - 2단계: 반대논리/패소논리/사용자에게 불리한 포인트 검토
  - 3단계: 결론과 리스크 잠금
  - 4단계: 최종 메모 JSON 작성
- A&B식 블루프린트 방식을 차용해, 먼저 메모 구조를 내부적으로 설계한 뒤 그 구조를 따라 JSON을 채운다. 내부 설계도는 출력하지 않는다.

핵심 원칙:
1) 반드시 `내부 구조화 DB -> 내부 문서 DB -> 공식근거 -> 웹근거` 순서로 검토한다.
2) 내부DB가 있으면 그것을 판단의 출발점으로 삼는다.
3) 공식근거는 내부 판단을 법률 기준으로 검증하는 층으로 사용한다.
4) 웹근거는 절차, 실무 안내, 최신 설명을 보완하는 층으로 사용한다.
5) 내부 판단과 공식근거가 충돌하면 그 차이를 숨기지 않고 법적 리스크를 함께 설명한다.
6) 법령명, 판례명, 사건번호, 날짜, 링크를 새로 지어내지 않는다.
7) 메타 안내문, 템플릿 재요청, 입력값 요구 문구를 절대 출력하지 않는다.
8) 답변 본문에 입력 컬럼명을 그대로 쓰지 않는다.
9) 사용자가 예/아니오, 해야 하나/안 해도 되나, 가능/불가 형태로 물으면 첫 문장에서 바로 결론을 말한다.
10) 결론은 완곡하게 돌리지 말고 `원칙적으로 ~해야 한다 / 아니다 / 가능하다 / 곤란하다`처럼 직접 쓴다.
11) 내부 출처는 `internal://` 같은 raw 식별자를 그대로 노출하지 말고 문서명 수준으로만 정리한다.
12) 블로그·마케팅성 링크는 공식 자료가 부족할 때만 보조로 최소한만 적는다.
13) 공식근거에 조문, 사건번호, 선고일, 법원명 정보가 있으면 답변 본문에 1~3개를 직접 표기한다.
14) 공식근거가 있으면 `[판단 기준]`에서 일반론보다 공식 조문/판례 식별정보를 우선 적는다.
15) `[근거 링크]`는 공식 링크를 먼저, 내부 문서는 문서명으로만, 웹 보완 링크는 마지막에 최소한만 적는다.
16) 질문에 `의견서`, `검토의견`, `법률검토`, `답변서` 중 하나가 있으면 일반 상담형 답변보다 문안 초안 형식에 가깝게 작성한다.
17) 이 경우에도 내부 근거, 공식 조문, 사건번호, 판례명은 가능한 범위에서 본문에 직접 녹여 쓴다.
18) `draft_style_examples`는 문체·목차·정리 방식만 참고한다. 기존 의견서의 사건번호, 인명, 일정, 사실관계, 결론을 그대로 재사용하지 않는다.
19) `draft_style_examples`가 비어 있어도 작성형 요청이면 현재 질문과 근거에 맞춰 새 초안을 작성한다.
20) 결론을 만들기 전에 반드시 `이 결론을 뒤집을 수 있는 반대논리`를 1회 검토하고, 그 반대논리를 이긴 경우에만 결론을 잠근다.
21) 공식 citation 요약에 있는 조문/사건번호/선고일/기관명은 가능한 범위에서 메모 JSON에 식별정보로 남긴다.
22) 요약 편향을 막기 위해, 내부적으로 각 쟁점마다 `결론 -> 근거 -> 반대논리 -> 실무조치` 순서로 점검하고 빠진 축이 있으면 채운 뒤 JSON을 완성한다.
23) 검색 전략이 약하면 `search_strategy`에 부족한 지점을 명시하고, 결론 강도를 낮춘다.
24) 징계/양정 질문이면 `internal_case_briefs`를 최우선으로 보고, 내부 유사 사례의 최종징계와 양정사유를 먼저 정리한다.
25) 징계/양정 질문인데 사실관계가 거칠고 내부 사례 비교가 충분히 정밀하지 않으면, 좁은 수위(예: 정직 확정)를 단정하지 말고 `needs_more_facts=true`로 둔다.
26) 이 경우 `fact_questions`에 2~5개의 구체 질문을 넣는다. 질문은 사안유형에 맞게 좁혀 쓴다.
   - 음주: 횟수, 혈중알코올농도, 사고 여부, 과거 징계 여부
   - 폭행/괴롭힘: 횟수, 기간, 신체접촉/상해 여부, 욕설·모욕 수위, 증거 여부
   - 성희롱: 발언/행위 수위, 반복성, 피해 진술, 증거
   - 근태: 무단결근 일수, 반복성, 사전경고 여부
27) 반대로 사실관계가 충분하고 내부 사례가 2건 이상 유사하면 `recommended_sanction_range`를 명확히 적는다.
28) 판례나 행정해석이 있으면 `official_anchor`와 `must_cite`에 실제 인라인 인용형 식별정보를 남긴다. 예: `대법원 2013다2314`, `근로기준법 제76조의3 제6항`, `고용노동부 행정해석 2023-05-10`.
29) `official_anchor`는 일반론 금지다. `어떤 법/판례가 무엇을 말하는지`가 드러나게 한 줄로 적는다.
30) 징계/양정 질문이면 `case_analogies` 각 항목을 `연도 | 행위 요지 | 최종징계 | 양정사유` 형식으로 가능한 범위에서 많이 작성한다. 최소 1건, 충분하면 3~5건까지 적는다.
31) 내부 사례가 1건 이상이면 총 건수와 대표 사례를 함께 정리한다. 사례가 부족하면 부족하다고 적고, 존재하지 않는 연도나 징계를 만들지 않는다.
32) 결론을 뒷받침하는 판례/행정해석 문장이 있으면 `must_cite`에 그대로 넣어 `대법원 판례에 따르면 ... (대법원 2013다2314)`처럼 본문에 직접 쓸 수 있게 만든다.
33) 질문이 특별법 사안이면 `근로기준법` 하나로 결론을 축약하지 않는다. `남녀고용평등법`, 그 시행령, 시행규칙처럼 직접 적용되는 법 체계를 함께 정리한다.
34) 공식 근거에 특별법·시행령·시행규칙이 함께 보이면 `official_anchor`와 `must_cite`에 각각 분리해서 남긴다.
35) `의견을 들어야 한다`와 `동의를 받아야 한다`처럼 법적 의미가 다른 표현은 구분해서 적는다. 조문 문구가 `의견 청취` 수준이면 veto권처럼 과장하지 않는다.

반드시 아래 JSON 스키마로만 출력:
{
  "answer_mode": "normal or draft",
  "search_strategy": ["검색 방향 1", "검색 방향 2"],
  "core_conclusion": "한 문장 결론",
  "conclusion_strength": "high or medium or low",
  "internal_anchor": ["내부 규정/사례 핵심 1", "내부 규정/사례 핵심 2"],
  "official_anchor": ["조문/사건번호/판례 식별정보 포함 핵심 1", "핵심 2", "핵심 3"],
  "web_support": ["웹 보완 핵심 1", "핵심 2"],
  "counterargument_or_risk": ["반대논리 또는 패소 리스크 1", "리스크 2"],
  "practical_actions": ["실무 조치 1", "실무 조치 2", "실무 조치 3"],
  "uncertainties": ["결론을 실제로 바꿀 수 있는 추가확인 1", "추가확인 2"],
  "needs_more_facts": true,
  "fact_questions": ["추가로 물어볼 사실 1", "사실 2", "사실 3"],
  "recommended_sanction_range": "견책~근신 또는 감봉~정직 또는 빈값",
  "case_analogies": ["2013년 | 행위 요지 | 최종징계 | 양정사유", "2022년 | 행위 요지 | 최종징계 | 양정사유"],
  "must_cite": ["본문에 직접 써야 할 조문/사건번호/기관명 1", "2", "3"],
  "link_priority": ["공식 링크 1", "공식 링크 2", "내부 문서명 1", "웹 링크 1"]
}
```

### 11.5 19B. 에이전트 프롬프트_최종답변작성

- 모델: `gpt-5.2`
- Output column name: `output_response`
- num response: `1`
- Include Question Prompt in output: `ON`
- is bind tools: `OFF`

```text
현재 질문:
{{latest_user_question}}

최근 대화 맥락:
{{conversation_context}}

질의 분류:
{{query_class}}

쟁점 키워드:
{{issue_keywords_csv}}

법률 메모 JSON:
{{legal_memo_json}}

내부 구조화/문서 근거:
{{internal_evidence_context}}

내부 참고 출처(사용자 노출용 정리값):
{{internal_source_urls}}

공식 근거:
{{official_evidence_context}}

공식 citation 요약:
{{official_citation_briefs}}

공식 근거 링크:
{{official_source_urls}}

웹 보완 근거:
{{web_evidence_context}}

웹 보완 링크:
{{web_source_urls}}

작성 스타일 예시(Opinion_Letters, 작성형 요청일 때만 사용):
{{draft_style_examples}}

작성 스타일 참고 문서:
{{draft_style_source_urls}}

역할:
- `legal_memo_json`을 우선 기준으로 최종 답변을 작성한다.
- 내부 근거/공식 근거/웹 근거는 `legal_memo_json`을 보강하거나 검증하는 용도로만 다시 본다.
- 사고 과정은 출력하지 말고 최종 답변만 출력한다.
- A&B식 요약 편향 방지 원칙을 차용해, 각 섹션이 너무 짧으면 `결론 / 근거 / 실무 포인트`를 보강해 충분히 설명한다.
- 이 노드는 압축 요약용이 아니라 최종 사용자 답변용이다. 직접 관련된 근거가 여러 개 있으면 가능한 범위에서 폭넓게 반영한다.

작성 규칙:
1) 예/아니오형 질문이면 첫 문장에서 바로 직접 답한다.
2) `legal_memo_json.must_cite`에 있는 조문/사건번호/기관명은 가능한 범위에서 본문에 직접 쓴다.
3) 공식 citation 요약과 공식 링크가 있으면, 조문/사건번호/선고일/법원명을 일반론보다 우선해서 적는다.
4) 사용자가 작성형 요청(`의견서`, `검토의견`, `법률검토`, `답변서`)을 했으면 `제목`, `검토의견`, `결론 및 권고`, `근거 링크` 중심으로 쓴다.
5) 이 경우에도 `draft_style_examples`는 문체·목차만 참고하고 사실관계/결론/번호를 복사하지 않는다.
6) 내부 출처는 `internal://` 같은 raw 식별자를 그대로 노출하지 말고 문서명 수준으로만 정리한다.
7) 블로그·마케팅성 링크는 공식 자료가 부족할 때만 최소한으로 적는다.
8) [불확실/추가확인]은 `legal_memo_json.uncertainties`에 실질 쟁점이 있을 때만 0~2개 적고, 별 의미 없으면 생략한다.
9) 최종 답변은 `legal_memo_json.core_conclusion`과 `legal_memo_json.must_cite`를 최우선으로 반영한다.
10) `legal_memo_json.counterargument_or_risk`가 비어 있지 않으면, [판단 기준] 또는 [불확실/추가확인]에서 반드시 반영한다.
11) 작성형 요청이면 `draft_style_examples`의 제목 패턴, 문단 길이, 결론 톤은 참고하되 문체만 차용하고 내용은 새로 쓴다.
12) `legal_memo_json.needs_more_facts=true`이면, 징계수위를 단정하지 말고 필요한 사실을 구체적으로 질문한다.
13) `legal_memo_json.recommended_sanction_range`가 있으면 [핵심 결론]에서 바로 `현재 정보 기준으로 견책~근신 가능성이 높다`처럼 명확히 적는다.
14) `legal_memo_json.case_analogies`가 있으면 [판단 기준]에서 `내부 유사 사례`를 별도 소제목처럼 먼저 정리한다.
15) 판례나 행정해석을 근거로 문장을 쓸 때는 가능한 범위에서 문장 끝에 괄호 인용을 붙인다. 예: `대법원 판례에 따르면 ... (대법원 2013다2314)`, `행정해석상 ...입니다(고용노동부 행정해석 2023-05-10)`.
16) 공식 citation이 여러 개면 가장 직접적인 것 1~2개만 남기지 말고, 직접 관련된 판례/행정해석/조문은 가능한 범위에서 본문과 [판단 기준]에 넓게 반영한다. 중복 문장만 줄이고 근거는 줄이지 않는다.
17) 징계/양정 질문이고 내부 사례가 있으면 `[내부 유사 사례]`에서 `총 n건 중 대표 사례` 형식으로 가능한 범위에서 여러 건을 번호로 쓴다. 직접 관련성이 높으면 4~5건까지도 허용한다.
18) 각 사례는 `연도`, `행위 요지`, `최종징계`, `양정사유`를 빠뜨리지 않는다. 연도가 없으면 연도는 생략하되 사실을 꾸며내지 않는다.
19) 내부 사례가 충분하지 않으면 `현재 내부 유사 사례는 제한적이다`라고 적고, 부족한 사실은 [추가 필요정보]에 넘긴다.
20) 내부 사례와 공식근거가 함께 있으면 `내부 사례상 ~였고, 법원/행정해석상 ~이므로`처럼 연결해서 쓴다.
21) 법령, 판례, 행정해석, 내부 사례를 단순 나열하지 말고 `무슨 사실 때문에 어떤 결론에 이르렀는지`가 보이게 서술한다.
22) 공식 근거가 3건 이상 있으면 핵심 판단을 지지하는 판례, 행정해석, 조문을 가능한 범위에서 모두 반영한다. 단, 동일 취지 반복은 줄인다.
23) 사용자가 징계 가능 여부나 징계 수위를 묻는 경우에는 공식 법적 기준뿐 아니라 내부 사례의 사실관계 차이도 상세히 비교한다.
24) 본문 길이를 억지로 줄이지 않는다. 사용자가 이해에 필요한 근거라면 길어져도 유지한다.
25) `[근거 링크]`도 최소 1~2개만 남기지 말고, 직접 관련 공식 링크는 가능한 한 빠짐없이 정리한다. 내부 문서명과 웹 보완 링크는 그 뒤에 붙인다.
26) 질문이 특별법 사안이면 `근로기준법에 따르면`만 반복하지 말고, 직접 적용되는 `남녀고용평등법`, 시행령, 시행규칙까지 함께 적는다.
27) 조문 문구가 `의견을 들어야 한다` 수준이면 그대로 적고, `동의가 필요하다`, `허락을 받아야 한다`처럼 의미를 키워서 쓰지 않는다.
28) 법 조항 인용이 가능하면 추상화하지 말고 `남녀고용평등법 제14조 제5항은 ...라고 정한다`처럼 조문 주체와 핵심 문구를 직접 드러낸다.

출력 형식:
[핵심 결론]
질문이 예/아니오형이면 첫 문장에서 바로 직접 답한다. 그 다음 2~4문장 안에서 예외와 리스크를 짧게 덧붙인다. 가능하면 첫 단락 안에 조문 또는 판례 1개를 바로 녹여 쓴다.

[내부 유사 사례]
징계/양정 질문이고 `legal_memo_json.case_analogies`가 있을 때만 적는다. `총 n건 중 대표 사례`처럼 먼저 총 건수를 말하고, 그 아래 관련성 높은 사례를 가능한 범위에서 여러 건 번호로 쓴다. 각 항목은 `연도 | 행위 요지 | 최종징계 | 양정사유` 순서를 지킨다.

[판단 기준]
내부DB 기준, 공식 법적 기준, 웹 보완 설명을 구분해 설명한다. 두루뭉술한 일반론보다 이 질문의 판단 포인트를 우선 적는다. 공식근거에 조문/사건번호가 있으면 식별정보를 문장 안에 직접 써서 정리한다. 가능하면 `대법원 판례에 따르면 ... (대법원 2013다2314)`처럼 바로 붙여 쓴다. 직접 관련된 판례, 행정해석, 조문이 여러 개 있으면 가능한 범위에서 폭넓게 적는다.

[실무 체크리스트]
입증자료, 절차상 확인사항, 추가 확보자료를 적는다. 실제로 바로 할 행동 중심으로 쓴다.

[근거 링크]
공식 링크와 내부 참고 문서명, 필요한 경우의 웹 보완 링크를 정리한다. 공식 링크를 가장 먼저 적고, 직접 관련된 공식 링크는 가능한 한 빠짐없이 적는다. 내부 출처는 raw 식별자 대신 사람이 읽을 수 있는 문서명으로 쓴다. 의미 없는 링크 나열은 하지 않는다.

[불확실/추가확인]
`legal_memo_json.uncertainties`에 실질 쟁점이 있을 때만 0~2개 적고, 별 의미 없으면 이 섹션은 생략한다.

[추가 필요정보]
`legal_memo_json.needs_more_facts=true`일 때만 적는다. `legal_memo_json.fact_questions`를 2~5개로 정리해, 사용자가 답하면 수위를 더 구체화할 수 있게 쓴다.
```

### 11.6 20. 파이썬_검열게이트

- 입력 포트: `dataset`
- 출력 포트: `output`

```python
df = dataset.copy() if isinstance(dataset, pd.DataFrame) else pd.DataFrame()

for c in ["output_response", "output_response_1", "draft_answer", "result_text", "answer", "text"]:
    if c in df.columns:
        ans_col = c
        break
else:
    ans_col = None

for c in ["query_class", "query_type", "qtype"]:
    if c in df.columns:
        q_col = c
        break
else:
    q_col = None

for c in ["official_evidence_count", "official_count"]:
    if c in df.columns:
        off_col = c
        break
else:
    off_col = None

for c in ["internal_evidence_count", "internal_count"]:
    if c in df.columns:
        in_col = c
        break
else:
    in_col = None

for c in ["web_evidence_count", "web_count"]:
    if c in df.columns:
        web_col = c
        break
else:
    web_col = None

rows = []
for _, r in df.iterrows():
    row = r.to_dict()

    text = str(row.get(ans_col, "") if ans_col else "").strip()
    qclass = str(row.get(q_col, "legal_analysis") if q_col else "legal_analysis").strip().lower()
    if qclass not in ["legal_analysis", "internal_db_only", "mixed"]:
        qclass = "legal_analysis"

    try:
        official = int(float(str(row.get(off_col, 0) if off_col else 0).strip()))
    except Exception:
        official = 0
    try:
        internal = int(float(str(row.get(in_col, 0) if in_col else 0).strip()))
    except Exception:
        internal = 0
    try:
        web = int(float(str(row.get(web_col, 0) if web_col else 0).strip()))
    except Exception:
        web = 0

    has_text = bool(text)
    is_pass = False
    reason = ""

    if qclass == "legal_analysis":
        if has_text:
            is_pass = True
        else:
            reason = "답변 텍스트 없음"
    elif qclass == "internal_db_only":
        if has_text and internal >= 1:
            is_pass = True
        else:
            reason = "내부DB 근거 없음"
    else:
        if has_text and (official >= 1 or internal >= 1 or web >= 1):
            is_pass = True
        else:
            reason = "공식·내부·웹 근거 모두 부족"

    rows.append({
        "is_pass": bool(is_pass),
        "fail_reason": reason,
        "query_class": qclass,
        "official_evidence_count": int(official),
        "web_evidence_count": int(web),
        "internal_evidence_count": int(internal),
        "output_response": text
    })

result = pd.DataFrame(rows)
```

### 11.7 21. 데이터 조건 분기

- 기준 컬럼: `is_pass`
- 조건: `true`
- 참 경로: `23. 에이전트로 전달`
- 거짓 경로: `22. 프롬프트_차단응답 -> 23. 에이전트로 전달`

### 11.8 22. 프롬프트_차단응답

- 모델: `gpt-5-nano`
- Output column name: `output_response`

```text
현재 질문: {{latest_user_question}}
질의 분류: {{query_class}}
쟁점 키워드: {{issue_keywords_csv}}
실패 사유: {{fail_reason}}

규칙:
- legal_analysis와 mixed를 범용 차단하지 않는다.
- internal_db_only일 때만 내부 확인 부족 안내를 짧게 쓴다.
- 메타 안내문, 템플릿 재요청 금지.

출력:
[핵심 결론]
현재 내부DB에서 직접 대응되는 근거가 확인되지 않아 내부 기준형 답변을 단정할 수 없다.

[불확실/추가확인]
- 내부 유사 사례 또는 내부 기준 문서가 추가로 필요하다.
```

### 11.9 23. 에이전트로 전달

- Select target label: `output_response`

## 12. 구축 순서

아래 순서만 따르면 된다.

### 12.1 1회성 문서 DB 적재 그래프 구축

1. 로컬에서 아래 적재용 XLSX를 만든다.

   - `internal_doc_ingest_aihub_판결.xlsx`
   - `internal_doc_ingest_aihub_약관.xlsx`
   - `internal_doc_ingest_rules_통합.xlsx`
   - `internal_doc_ingest_opinion_letters_내부.xlsx`
2. 아래 데이터셋 노드를 만든다.

   - `데이터셋_문서근거_AIHUB580_판결_적재용`
   - `데이터셋_문서근거_AIHUB580_약관_적재용`
   - `데이터셋_문서근거_RULES_적재용`
   - `데이터셋_문서근거_의견서_적재용`
3. 적재 체인을 만든다.

   - `판결_적재용 -> D4_판결 -> D5_판결 -> D6W_AIHUB_판결`
   - `약관_적재용 -> D4_약관 -> D5_약관 -> D6W_AIHUB_약관`
   - `RULES_적재용 -> D4_RULES -> D5_RULES -> D6W_RULES`
   - `의견서_적재용 -> D4_의견서 -> D5_의견서 -> D6W_의견서`
4. 저장소 이름을 아래처럼 고정한다.

   - AIHub 저장소: `docdb_aihub_v1`
   - 규정/가이드 저장소: `docdb_rules_v1`
   - 의견서 스타일 저장소: `docdb_opinion_style_v1`
5. 저장소 설정을 아래처럼 둔다.

   - `D6W_AIHUB_판결`, `D6W_AIHUB_약관`: `데이터 추가`, `중복 제거 ON`
   - `D6W_RULES`: `데이터 덮어쓰기`, `중복 제거 ON`
   - `D6W_의견서`: `데이터 덮어쓰기`, `중복 제거 ON`
6. 적재 실행 순서

   - `AIHub 판결`
   - `AIHub 약관`
   - `RULES 통합`
   - `의견서 스타일`(실데이터가 있을 때만)
7. 문서 적재 테스트

   - `D6R_AIHUB`를 만들었을 때 `txt`, `source_type`, `doc_id`, `source_url`, `doc_meta_json`와 임베딩 숫자 열이 읽히는지 확인
   - `D6R_RULES`를 만들었을 때 같은 열이 읽히는지 확인
   - `D6R_의견서`를 만들었을 때 같은 열이 읽히는지 확인
8. fallback

   - 로컬 전처리를 하지 못하는 경우에만 `D1A`, `D1B`, `D2B`를 사용한다.
   - 기본 경로는 업로드용 XLSX 4개를 각 저장소에 적재하는 구조다.

### 12.1A 기존 저장소를 갱신할 때

1. `AIHub`는 무겁고 자주 바뀌지 않으므로, 평소에는 `docdb_aihub_v1`를 다시 돌리지 않는다.
2. 새 `징계규정`, `취업규칙`, `윤리규범`, `조사사무 처리기준`, 외부 가이드가 생기면 로컬에서 `internal_doc_ingest_rules_통합.xlsx`를 다시 만든다.
3. 그 다음 `데이터셋_문서근거_RULES_적재용 -> D4_RULES -> D5_RULES -> D6W_RULES`만 다시 실행한다.
4. 이때 `D6W_RULES`는 `데이터 덮어쓰기`이므로, 기존 규정/가이드 저장소 전체가 새 파일 기준으로 깔끔하게 교체된다.
5. `Opinion_Letters`를 갱신할 때도 같은 원칙을 쓴다.
   - 템플릿/아카이브를 갱신
   - `internal_doc_ingest_opinion_letters_내부.xlsx` 재생성
   - `D6W_의견서`를 `데이터 덮어쓰기`로 다시 실행
6. 즉 덮어쓰기를 안전하게 쓰려면 저장소가 역할별로 분리돼 있어야 한다.

### 12.2 런타임 그래프 구축

1. `에이전트`
2. `에이전트 메시지 가로채기`
3. `2A`, `2B`, `2C`
4. `3`, `4`, `5`, `17A`
5. 공식 branch `6/7/8/26A -> 9/10/11/26B -> 12/13/14/26C -> 15 -> 16 -> 26D -> 17_공식 -> 17B_공식 -> 18_공식패키징`
6. 내부 구조화 branch `데이터셋_내부근거_엑셀 -> 16A -> 17_내부구조화 -> 17B_내부구조화 -> 18_내부패키징_구조화`
7. 내부 문서 공통 준비 `17A -> 17D1 -> 17D2`
8. AIHub 문서 branch `D6R_AIHUB + 17D2 -> 17D3_AIHUB -> 17D4_AIHUB -> 18_내부패키징_AIHUB`
9. 규정/가이드 문서 branch `D6R_RULES + 17D2 -> 17D3_RULES -> 17D4_RULES -> 18_내부패키징_RULES`
10. 의견서 문안 branch `D6R_의견서 + 17D2 + 17A -> 17D3_의견서 -> 17D4_의견서 -> 18_내부패키징_의견서`
11. `18I_DOC`
12. `18I1`
13. `18W_A`, `18W_B`
14. `18E_A`, `18E_B`, `18E_C`
15. `18M1`, `18M2`, `18M3`
16. `19A`, `19B`, `20`, `21`, `22`, `23`

## 13. 빠른 점검 포인트

### 13.1 5번 노드

정상 예:

- `law_query_used = 근로기준법`
- `prec_query_used = 징계` 또는 `음주`
- `expc_query_used = 징계`
- `nlrc_query_used = 징계` 또는 `부당해고`

### 13.2 공식 branch

- `6/7/8/26A`에서 JSON 응답이 들어오는지
- `9/10/11/26B`에서 상세 URL이 비어 있지 않은지
- `26D` 이후 `17_공식`으로 데이터가 내려오는지

### 13.3 내부 구조화 branch

- `16A` 결과에 `source_type`, `case_title`, `evidence_text`가 있는지
- `18_내부패키징_구조화` 결과가 `internal_*` 컬럼으로 나오는지

### 13.4 내부 문서 branch

- `D6R_AIHUB`에서 AIHub 임베딩/메타데이터가 읽히는지
- `17D3_AIHUB`가 `similarity` 기준 top-k 문서를 뽑는지
- `18_내부패키징_AIHUB` 결과가 `internal_*` 컬럼으로 나오는지
- `D6R_RULES`에서 규정/가이드 임베딩/메타데이터가 읽히는지
- `17D3_RULES`가 `similarity` 기준 top-k 문서를 뽑는지
- `18_내부패키징_RULES` 결과가 `internal_*` 컬럼으로 나오는지
- `18I_DOC`가 AIHub와 RULES의 `internal_*`를 하나로 합치는지

### 13.5 의견서 문안 branch

- `D6R_의견서`는 일반 문서 저장소가 아니라 별도 의견서 저장소를 읽는지
- `17D3_의견서`는 질문에 `의견서`, `검토의견`, `법률검토`, `답변서`가 없으면 빈 결과를 내는지
- 트리거가 있을 때만 `17D3_의견서`가 `similarity` 기준 top-k 문안을 뽑는지
- `17D4_의견서`에는 `similarity` 열이 남아 있는지
- `18_내부패키징_의견서`가 일반 질문에서는 빈 스타일 패키지, 작성형 질문에서는 `draft_style_*` 컬럼 패키지로 나오는지

### 13.6 웹 branch

- `18W_A`는 반드시 `{{}}` 주입형 프롬프트를 써야 한다.
- `18W_B`에서 `web_evidence_context`, `web_source_urls`, `web_evidence_count`가 채워지는지 본다.

### 13.7 최종 답변

- `18M3`에 아래 컬럼들이 있는지
  - `query_class`
  - `issue_keywords_csv`
  - `latest_user_question`
  - `conversation_context`
  - `official_evidence_context`
  - `official_source_urls`
  - `official_evidence_count`
  - `official_citation_briefs`
  - `internal_evidence_context`
  - `internal_source_urls`
  - `internal_evidence_count`
  - `draft_style_examples`
  - `draft_style_source_urls`
  - `draft_style_count`
  - `web_evidence_context`
  - `web_source_urls`
  - `web_evidence_count`
- `elabor`를 붙인 경우 `web_evidence_context` 안에 `[ELABOR 보조DB]` 블록이 합쳐지는지
- `19A` 결과에 `legal_memo_json` 컬럼이 생기는지
- `19B`가 `legal_memo_json`을 바탕으로 최종 답변을 만드는지
- `19 output_response`가 메타 안내문 없이 바로 서술형 답변인지
- `internal_db_only`만 내부근거 없을 때 `22`로 가는지

## 14. 장애 복구

### 14.1 공식 API totalCnt = 0

- 호출 실패가 아니라 검색 결과 0건이다.
- 이 경우에도 `17A`, 내부 구조화 branch, 내부 문서 branch, 웹 branch는 계속 진행한다.

### 14.2 Worker 불안정

- `/health` 확인
- 목록 URL이 아니라 상세 URL 생성부(`9/10/11/26B`)를 먼저 점검

### 14.3 18W_A가 “입력 컬럼 값이 제공되지 않았다”고 답함

원인:

- 프롬프트에 열 이름만 적고 `{{latest_user_question}}` 같은 주입을 안 한 상태

조치:

- `18W_A`, `19`, `22`, `3`, `17D1` 프롬프트를 다시 점검

### 14.4 Data Connect가 “같은 데이터셋은 병합할 수 없습니다”라고 뜸

- `18I1`, `18M1`, `18M2`, `18M3`는 Data Connect를 쓰지 않는다.

### 14.5 A&B 프롬프트 차용 원칙

- [Ailey & Bailey X_260304.md](d:\AI canvas\새 폴더\Ailey%20%26%20Bailey%20X_260304.md)의 전체 프롬프트를 통째로 넣지 않는다.
- 대신 아래 5가지만 추출해 런타임 노드에 분산 적용한다.
  - `검색 전략을 먼저 잠그고 답변은 나중에 쓰는 방식`
  - `공식/공공기관 우선, 다중 소스 교차검증`
  - `반대논리/엣지케이스를 한 번 더 검토하는 단계`
  - `요약 편향을 줄이기 위한 구조화 메모 단계`
  - `최종 출력은 내부 계획을 숨기고 사용자용 결과만 쓰는 방식`
- 가져오지 않는 것:
  - Ailey/Bailey 페르소나 말투
  - UI 메뉴/네비게이션/타임스탬프 규칙
  - 강의/커리큘럼 생성 엔진
  - 과도한 분량 강제 규칙
- 적용 위치:
  - `18W_A`: 검색 전략/교차검증
  - `19A`: 내부 블루프린트/반대논리 검토/결론 잠금
  - `19B`: 메모 기반 최종 작성

## 15. 운영 수용 기준

아래를 모두 만족해야 완료로 본다.

1. 질문 1개가 공식 / 내부 구조화 / 내부 문서 / 웹 4축으로 모두 흘러간다.
2. AIHub prechunked는 더 이상 `16A -> 17` 전수 스캔 경로를 타지 않는다.
3. `nlrc`는 선택이 아니라 공식 branch 기본축으로 포함된다.
4. `노드5 output` 하나가 `6`, `7`, `8`, `26A`, `17A`로 fan-out 된다.
5. `19`는 내부DB를 먼저 읽고, 그 다음 공식근거와 웹근거를 검토하는 구조로 답한다.
6. `20`은 `internal_db_only`만 내부근거 부족 시 차단한다.
7. 이 문서만 보고 초보자가 처음부터 끝까지 재현할 수 있다.

## 16. 애플리케이션 페이지 / 샌드박스 배포

이제 Elvis의 애플리케이션 페이지는 **최종형 단일 셸 구조**로 설계한다.

중요:

- 공개 메뉴는 `HOME`, `AI노무사`, `추후 개발` 3개를 먼저 사용한다.
- `HOME`은 깔끔한 랜딩 페이지다.
- `AI노무사`는 실제 질문/답변이 누적되는 작업 화면이다.
- `근거` 전용 메뉴는 제거한다.
- `AI노무사` 화면 하나에서 답변과 근거를 같이 읽히게 설계한다.
- 애플리케이션 기본 헤더/메뉴보다, 샌드박스가 화면 전체를 꽉 채우는 **사이트형 셸**을 우선한다.
- 상단 좌측에 브랜드와 메뉴 pill이 놓이는 얇은 네비게이션 바를 쓴다.
- `HOME`에서는 상단 좌측 또는 본문 CTA에서 `AI노무사`로 진입할 수 있어야 한다.
- 메뉴 버튼은 nodeId가 아니라 `changeApplicationMenu("HOME")`, `changeApplicationMenu("AI노무사")`, `changeApplicationMenu("추후 개발")`로 이동한다.
- 실제 추론은 여전히 `3`, `19A`, `19B`가 수행한다.

샌드박스용 기준 문서:

- [elvis_platform_manual.md](d:\AI canvas\새 폴더\elvis_platform_manual.md)
- [elvis_platform_home_sandbox.md](d:\AI canvas\새 폴더\elvis_platform_home_sandbox.md)
- [elvis_consult_chat_sandbox.md](d:\AI canvas\새 폴더\elvis_consult_chat_sandbox.md)
- [elvis_future_placeholder_sandbox.md](d:\AI canvas\새 폴더\elvis_future_placeholder_sandbox.md)

### 16.1 최종 메뉴 구조

```text
애플리케이션 페이지
  ├─ 메뉴 1: HOME
  │   └─ 페이지_Elvis_HOME
  │       └─ 샌드박스_Elvis_홈
  ├─ 메뉴 2: AI노무사
  │   └─ 페이지_Elvis_AI노무사
  │       └─ 샌드박스_노무상담챗_Elvis
  └─ 메뉴 3: 추후 개발
      └─ 페이지_Elvis_추후개발
          └─ 샌드박스_Elvis_추후개발
```

### 16.2 AI노무사 최종형 연결

`샌드박스_노무상담챗_Elvis`의 입력 포트 생성 순서:

1. `answer_package`
2. `memo_package`
3. `merged_package`

선 연결:

```text
19B -> 샌드박스_노무상담챗_Elvis(answer_package)
19A -> 샌드박스_노무상담챗_Elvis(memo_package)
18M3 -> 샌드박스_노무상담챗_Elvis(merged_package)
샌드박스_노무상담챗_Elvis(output) -> 3(dataset)
```

### 16.3 메뉴 이동 원칙

샌드박스 상단 메뉴 버튼은 아래처럼 이동한다.

```javascript
changeApplicationMenu("HOME")
changeApplicationMenu("AI노무사")
changeApplicationMenu("추후 개발")
```

좌측 상단의 로고/사이트명 클릭은 아래처럼 처리한다.

```javascript
changeApplicationMenu("HOME")
```

즉 nodeId를 직접 호출하는 방식이 아니라 **메뉴명 기반 이동**이다.

### 16.4 샌드박스가 읽어야 하는 핵심 컬럼

`샌드박스_노무상담챗_Elvis`

- `output_response`
- `legal_memo_json`
- `latest_user_question`
- `query_class`

핵심 사용자 노출 값은 `output_response`다.

`legal_memo_json`은 아래 보조 정보 노출에 쓴다.

- 추가 필요정보
- 권장 징계 범위
- 핵심 결론

### 16.5 별도 근거 메뉴를 두지 않는 이유

- 별도 근거 탭을 두면 사용자가 화면을 오가야 한다.
- Elvis의 첫 공개형은 `AI노무사` 화면 하나에서 답변과 근거를 같이 읽히는 쪽이 더 낫다.
- 따라서 `19B`는 본문 안에 아래를 충분히 포함하도록 설계한다.
  - 핵심 결론
  - 판단 기준
  - 내부 유사 사례
  - 공식 조문/사건번호/선고일
  - 실무상 추가 필요정보
  - 필요한 경우 링크

### 16.6 시각/레이아웃 방향

- 사용자가 준 참고 이미지처럼 넓은 여백, 미니멀한 pill 버튼, 강한 타이포, 깨끗한 배경을 지향한다.
- 사진은 꼭 필요하지 않다.
- 타이포그래피와 레이아웃만으로도 충분히 고급스럽게 갈 수 있다.
- `HOME`은 랜딩 페이지로, `AI노무사`로 들어가는 CTA가 잘 보여야 한다.
- `AI노무사`는 첫 진입 시에도 정돈된 워크스페이스처럼 보여야 한다.

### 16.7 페이지 노드 설정

`페이지_Elvis_HOME`, `페이지_Elvis_AI노무사`, `페이지_Elvis_추후개발`은 공통으로 아래 값을 쓴다.

- 크기 맞춤: `자동 맞춤`
- 플로우 노드 설정: `사용하지 않음`

### 16.8 샌드박스 작성 원칙

1. 샌드박스는 화면 셸과 입력 UI를 맡는다.
2. 실제 추론은 뒤의 `에이전트 프롬프트` 노드가 맡는다.
3. 샌드박스는 앱처럼 보이되, 입력 계약과 연결선은 단순하게 유지한다.
4. `HOME`, `AI노무사`, `추후 개발`은 모두 상단 좌측 브랜드/메뉴 바가 포함된 같은 셸 구조를 유지한다.
5. 공통 상단 셸은 각 샌드박스에 반복 포함하는 쪽이 AI Canvas에서 가장 안전하다.

### 16.9 배포 순서

1. 런타임 그래프 `3 ~ 23`을 먼저 안정화
2. 애플리케이션 페이지 생성
3. 메뉴를 아래 3개 생성
   - `HOME`
   - `AI노무사`
   - `추후 개발`
4. 페이지 노드 생성
   - `페이지_Elvis_HOME`
   - `페이지_Elvis_AI노무사`
   - `페이지_Elvis_추후개발`
5. 샌드박스 배치
   - `샌드박스_Elvis_홈`
   - `샌드박스_노무상담챗_Elvis`
   - `샌드박스_Elvis_추후개발`
6. `샌드박스_노무상담챗_Elvis`에만 런타임 선 연결
7. 모든 화면을 풀스크린 상단 네비게이션 셸 레이아웃으로 정리
8. 애플리케이션 페이지 설정 후 배포

### 16.10 더 이상 기본이 아닌 구조

아래 구조는 Elvis의 최종 기본값이 아니다.

- `근거` 별도 메뉴
- `지식문서`, `작성도구` 공개 메뉴 구조
- 분리형 `질문입력 샌드박스 + 답변뷰어 샌드박스`
- 공개용 에이전트 UI 직접 노출
- 무거운 좌측 사이드바형 내비게이션
