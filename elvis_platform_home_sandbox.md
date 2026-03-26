# ELVIS HOME 샌드박스 코드 문서 (v2 — 사이드바 레이아웃)

이 문서는 ELVIS AI 플랫폼의 `HOME` 메뉴 화면인 `샌드박스_Elvis_홈` 노드용 실사용 코드 문서다.

- 이 샌드박스는 **표시 전용(display-only)** 화면이다. 입력/출력 포트 없음.
- **좌측 고정 사이드바 + 우측 히어로 영역** 레이아웃이다.
- 챗봇 샌드박스(`elvis_consult_chat_sandbox_internal_run.md`)는 건드리지 않는다.

---

## 1. 노드 구조

노드명: `샌드박스_Elvis_홈`

입출력 포트: 없음

샌드박스 편집 설정:
- `<head>`: 기본 meta 두 줄 유지
- 외부 CDN: Google Fonts (Inter) 만 허용

메뉴 번호 매핑 (AI Canvas — 절대 변경 금지):
| 번호 | 메뉴명 |
|---|---|
| `1` | HOME |
| `2` | AI노무사 |
| `3` | 추후 개발 |

---

## 2. HTML

```html
<div class="elvis-home">

  <!-- ── 좌측 사이드바 ── -->
  <aside class="sidebar">
    <div class="sidebar-top">
      <button
        type="button"
        class="sidebar-logo"
        onclick="window.__navigateMenu && window.__navigateMenu(1)"
        aria-label="홈으로 이동"
      >
        <span class="logo-dot"></span>
        <span class="logo-text">ELVIS</span>
      </button>

      <nav class="sidebar-nav" aria-label="메인 내비게이션">
        <button
          type="button"
          class="nav-item is-active"
          onclick="window.__navigateMenu && window.__navigateMenu(1)"
        >
          <span class="nav-icon">
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="1.8"
                 stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/>
              <polyline points="9 22 9 12 15 12 15 22"/>
            </svg>
          </span>
          HOME
        </button>

        <button
          type="button"
          class="nav-item"
          onclick="window.__navigateMenu && window.__navigateMenu(2)"
        >
          <span class="nav-icon">
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="1.8"
                 stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z"/>
            </svg>
          </span>
          AI노무사
        </button>

        <button
          type="button"
          class="nav-item"
          onclick="window.__navigateMenu && window.__navigateMenu(3)"
        >
          <span class="nav-icon">
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="1.8"
                 stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <circle cx="12" cy="12" r="10"/>
              <line x1="12" y1="8" x2="12" y2="12"/>
              <line x1="12" y1="16" x2="12.01" y2="16"/>
            </svg>
          </span>
          추후 개발
        </button>
      </nav>
    </div>

    <div class="sidebar-bottom">
      <span class="sidebar-brand-label">ER AI Platform</span>
    </div>
  </aside>

  <!-- ── 우측 메인 콘텐츠 ── -->
  <main class="main-content">
    <div class="hero-wrap">

      <!-- 히어로 텍스트 블록 -->
      <div class="hero-text">
        <p class="hero-eyebrow">ER AI Platform</p>
        <h1 class="hero-headline">
          <span class="headline-main">ELVIS</span>
        </h1>
        <p class="hero-tagline">ALL + Wise — ER팀을 위한 AI 노동법 플랫폼</p>
        <p class="hero-desc">
          법령, 판례, 행정해석, 내부 사례를 연결해<br>
          실무형 노동 자문을 대화처럼 제공합니다.
        </p>

        <div class="chip-row">
          <span class="chip">AI노무사</span>
          <span class="chip">내부 사례 기반</span>
          <span class="chip">판례·행정해석</span>
          <span class="chip">실무형 자문</span>
        </div>

        <div class="cta-row">
          <button
            type="button"
            class="btn-cta"
            onclick="window.__navigateMenu && window.__navigateMenu(2)"
          >
            AI노무사 시작하기
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="2"
                 stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <line x1="5" y1="12" x2="19" y2="12"/>
              <polyline points="12 5 19 12 12 19"/>
            </svg>
          </button>
        </div>
      </div>

      <!-- 우측 패널 카드 -->
      <div class="panel-wrap">
        <div class="panel-card">
          <div class="panel-header">
            <span class="panel-live-dot"></span>
            <span class="panel-title">현재 사용 가능</span>
          </div>

          <ul class="panel-list">
            <li class="panel-item">
              <span class="pi-icon">
                <svg width="13" height="13" fill="none" stroke="currentColor" stroke-width="2.2"
                     stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
                  <polyline points="20 6 9 17 4 12"/>
                </svg>
              </span>
              <span>근로기준법·판례 실시간 자문</span>
            </li>
            <li class="panel-item">
              <span class="pi-icon">
                <svg width="13" height="13" fill="none" stroke="currentColor" stroke-width="2.2"
                     stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
                  <polyline points="20 6 9 17 4 12"/>
                </svg>
              </span>
              <span>내부 징계·조사 사례 비교</span>
            </li>
            <li class="panel-item">
              <span class="pi-icon">
                <svg width="13" height="13" fill="none" stroke="currentColor" stroke-width="2.2"
                     stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
                  <polyline points="20 6 9 17 4 12"/>
                </svg>
              </span>
              <span>행정해석·고용부 지침 자동 참조</span>
            </li>
            <li class="panel-item">
              <span class="pi-icon">
                <svg width="13" height="13" fill="none" stroke="currentColor" stroke-width="2.2"
                     stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
                  <polyline points="20 6 9 17 4 12"/>
                </svg>
              </span>
              <span>대화형 질의로 맥락 유지</span>
            </li>
          </ul>

          <div class="panel-divider"></div>

          <button
            type="button"
            class="panel-link"
            onclick="window.__navigateMenu && window.__navigateMenu(2)"
          >
            AI노무사로 이동
            <svg width="12" height="12" fill="none" stroke="currentColor" stroke-width="2"
                 stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <line x1="5" y1="12" x2="19" y2="12"/>
              <polyline points="12 5 19 12 12 19"/>
            </svg>
          </button>
        </div>
      </div>

    </div>
  </main>
</div>
```

---

## 3. CSS

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html, body {
  height: 100%;
  width: 100%;
  background: #fff;
}

body {
  font-family: 'Inter', 'Segoe UI', 'Noto Sans KR', sans-serif;
  font-size: 14px;
  color: #111;
  -webkit-font-smoothing: antialiased;
}

/* ── 전체 레이아웃 ── */
.elvis-home {
  display: flex;
  height: 100dvh;
  width: 100%;
  overflow: hidden;
}

/* ────────────────────────────
   사이드바
──────────────────────────── */
.sidebar {
  width: 220px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border-right: 1px solid #ebebeb;
  background: #fafafa;
  padding: 20px 14px;
}

.sidebar-top {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* 로고 */
.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 8px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 2px 4px;
  border-radius: 6px;
  text-decoration: none;
  transition: background .15s;
}
.sidebar-logo:hover { background: #f0f0f0; }

.logo-dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: #22c55e;
  flex-shrink: 0;
  animation: blink 2.2s ease-in-out infinite;
}
@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: .25; }
}

.logo-text {
  font-size: 15px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: #111;
}

/* 네비게이션 */
.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 9px;
  width: 100%;
  padding: 9px 10px;
  border: none;
  border-radius: 8px;
  background: transparent;
  color: #666;
  font-family: inherit;
  font-size: 13.5px;
  font-weight: 400;
  cursor: pointer;
  text-align: left;
  transition: background .13s, color .13s;
}
.nav-item:hover {
  background: #efefef;
  color: #111;
}
.nav-item.is-active {
  background: #111;
  color: #fff;
  font-weight: 500;
}
.nav-item.is-active .nav-icon { opacity: 1; }

.nav-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  opacity: .55;
}
.nav-item.is-active .nav-icon,
.nav-item:hover .nav-icon { opacity: 1; }

/* 하단 브랜드 */
.sidebar-bottom { padding: 0 4px; }
.sidebar-brand-label {
  font-size: 11px;
  color: #bbb;
  letter-spacing: 0.04em;
}

/* ────────────────────────────
   메인 콘텐츠
──────────────────────────── */
.main-content {
  flex: 1;
  overflow-y: auto;
  display: flex;
  align-items: center;
  padding: 24px 36px;
}

.hero-wrap {
  display: flex;
  align-items: flex-start;
  gap: 40px;
  max-width: 900px;
  width: 100%;
}

/* ── 히어로 텍스트 ── */
.hero-text {
  flex: 1;
  min-width: 0;
}

.hero-eyebrow {
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #999;
  margin-bottom: 10px;
}

.hero-headline {
  margin-bottom: 12px;
}
.headline-main {
  display: block;
  font-size: clamp(40px, 5.5vw, 64px);
  font-weight: 700;
  letter-spacing: -0.02em;
  line-height: 1;
  color: #111;
}

.hero-tagline {
  font-size: 14px;
  font-weight: 500;
  color: #333;
  margin-bottom: 10px;
  line-height: 1.5;
}

.hero-desc {
  font-size: 13px;
  color: #666;
  line-height: 1.75;
  margin-bottom: 16px;
}

/* 칩 */
.chip-row {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 20px;
}
.chip {
  font-size: 11.5px;
  color: #555;
  border: 1px solid #e0e0e0;
  border-radius: 999px;
  padding: 4px 10px;
  background: #fff;
  letter-spacing: 0.01em;
}

/* CTA */
.cta-row { display: flex; gap: 12px; flex-wrap: wrap; }

.btn-cta {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 10px 18px;
  background: #111;
  color: #fff;
  border: none;
  border-radius: 10px;
  font-family: inherit;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: background .15s, transform .1s;
  letter-spacing: 0.01em;
}
.btn-cta:hover { background: #333; }
.btn-cta:active { transform: scale(.97); }

/* ── 패널 카드 ── */
.panel-wrap { flex-shrink: 0; width: 240px; }

.panel-card {
  border: 1px solid #e8e8e8;
  border-radius: 14px;
  padding: 16px 16px;
  background: #fff;
  box-shadow: 0 2px 12px rgba(0,0,0,.04);
}

.panel-header {
  display: flex;
  align-items: center;
  gap: 7px;
  margin-bottom: 12px;
}
.panel-live-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #22c55e;
  animation: blink 2.2s ease-in-out infinite;
}
.panel-title {
  font-size: 12px;
  font-weight: 600;
  color: #111;
  letter-spacing: 0.03em;
}

.panel-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 14px;
}
.panel-item {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 12px;
  color: #444;
  line-height: 1.5;
}
.pi-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #f0fdf4;
  color: #22c55e;
  margin-top: 1px;
}

.panel-divider {
  height: 1px;
  background: #f0f0f0;
  margin-bottom: 16px;
}

.panel-link {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-family: inherit;
  font-size: 13px;
  font-weight: 500;
  color: #111;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  transition: gap .15s;
}
.panel-link:hover { gap: 8px; }

/* ────────────────────────────
   반응형
──────────────────────────── */
@media (max-width: 860px) {
  .sidebar {
    width: 180px;
    padding: 20px 10px;
  }
  .main-content { padding: 32px 28px; }
  .hero-wrap { gap: 32px; }
  .panel-wrap { width: 220px; }
}

@media (max-width: 680px) {
  .elvis-home { flex-direction: column; overflow: auto; }

  .sidebar {
    width: 100%;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    border-right: none;
    border-bottom: 1px solid #ebebeb;
    padding: 0 16px;
    height: 52px;
    flex-shrink: 0;
  }
  .sidebar-top {
    flex-direction: row;
    align-items: center;
    gap: 16px;
  }
  .sidebar-nav { flex-direction: row; gap: 4px; }
  .nav-item { padding: 6px 10px; font-size: 13px; }
  .nav-icon { display: none; }
  .sidebar-bottom { display: none; }

  .main-content {
    padding: 28px 20px;
    align-items: flex-start;
    overflow-y: auto;
  }
  .hero-wrap {
    flex-direction: column;
    gap: 28px;
  }
  .panel-wrap { width: 100%; }
}
```

---

## 4. JavaScript

```javascript
(() => {
  // ── 메뉴 이동 헬퍼 ──────────────────────────────────────────
  // AI Canvas changeApplicationMenu 는 숫자(1/2/3)로 호출한다.
  // 절대 이 숫자를 바꾸지 말 것.
  function navigateMenu(menuNumber) {
    const menuNo = Number(menuNumber);
    if (!menuNo) {
      console.warn('[ELVIS] navigateMenu: invalid menu number', menuNumber);
      return;
    }

    const targets = [];
    try { targets.push(window); } catch(e) {}
    try { if (window.parent && window.parent !== window) targets.push(window.parent); } catch(e) {}
    try { if (window.top && window.top !== window && window.top !== window.parent) targets.push(window.top); } catch(e) {}

    // 1순위: changeApplicationMenu(숫자) 직접 호출
    for (const target of targets) {
      try {
        if (target && typeof target.changeApplicationMenu === 'function') {
          target.changeApplicationMenu(menuNo);
          return;
        }
      } catch(e) {
        console.warn('[ELVIS] changeApplicationMenu call failed', e);
      }
    }

    // 2순위: postMessage fallback
    for (const target of targets) {
      try {
        if (target && typeof target.postMessage === 'function') {
          target.postMessage({ type: 'changeApplicationMenu', menuNumber: menuNo }, '*');
        }
      } catch(e) {
        console.warn('[ELVIS] postMessage fallback failed', e);
      }
    }

    console.warn('[ELVIS] navigation handler not found for menu:', menuNo);
  }

  window.__navigateMenu = navigateMenu;
})();
```

---

## 5. 연결 유지 포인트

| 항목 | 유지해야 할 값 | 비고 |
|---|---|---|
| HOME 이동 | `navigateMenu(1)` | 로고 클릭, HOME 버튼 |
| AI노무사 이동 | `navigateMenu(2)` | CTA 버튼, 패널 링크, 사이드바 |
| 추후 개발 이동 | `navigateMenu(3)` | 사이드바 세 번째 항목 |
| 전역 함수 | `window.__navigateMenu` | inline onclick 에서 참조 |

**챗봇 파일(`elvis_consult_chat_sandbox_internal_run.md`)은 이번 작업에서 수정하지 않았습니다.**

---

## 6. 붙여넣기 가이드

AI Canvas 샌드박스 노드에서:

1. **HTML** 탭 → `## 2. HTML` 코드블록 내용 전체 붙여넣기
2. **CSS** 탭 → `## 3. CSS` 코드블록 내용 전체 붙여넣기
3. **JavaScript** 탭 → `## 4. JavaScript` 코드블록 내용 전체 붙여넣기
4. 외부 CDN 설정 없이 바로 사용 가능 (Google Fonts는 CSS `@import`로 자동 로드)
