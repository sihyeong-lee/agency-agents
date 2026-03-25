# ELVIS 추후개발 샌드박스 코드 문서

이 문서는 ELVIS 플랫폼의 `추후 개발` 메뉴 화면인 `샌드박스_추후개발_Elvis` 노드용 실사용 코드 문서다.

- 표시 전용(display-only) 화면. 입력/출력 포트 없음.
- 좌측 사이드바 + 중앙 준비 중 메시지.
- 챗봇 샌드박스는 건드리지 않는다.

---

## 1. 노드 구조

노드명: `샌드박스_추후개발_Elvis`

입출력 포트: 없음

메뉴 번호 매핑 (AI Canvas — 절대 변경 금지):
| 번호 | 메뉴명 |
|---|---|
| `1` | HOME |
| `2` | AI노무사 |
| `3` | 추후 개발 |

---

## 2. HTML

```html
<div class="elvis-future">

  <!-- 좌측 사이드바 -->
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

      <nav class="sidebar-nav">
        <button type="button" class="nav-item" onclick="window.__navigateMenu && window.__navigateMenu(1)">
          <span class="nav-icon">
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/>
              <polyline points="9 22 9 12 15 12 15 22"/>
            </svg>
          </span>
          HOME
        </button>
        <button type="button" class="nav-item" onclick="window.__navigateMenu && window.__navigateMenu(2)">
          <span class="nav-icon">
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
              <path d="M21 15a2 2 0 01-2 2H7l-4 4V5a2 2 0 012-2h14a2 2 0 012 2z"/>
            </svg>
          </span>
          AI노무사
        </button>
        <button type="button" class="nav-item is-active" onclick="window.__navigateMenu && window.__navigateMenu(3)">
          <span class="nav-icon">
            <svg width="15" height="15" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" viewBox="0 0 24 24">
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

  <!-- 중앙 콘텐츠 -->
  <main class="main-content">
    <div class="center-block">
      <p class="coming-label">Coming Soon</p>
      <h1 class="coming-title">추후 개발</h1>
      <p class="coming-desc">새로운 기능이 곧 추가될 예정입니다.</p>
      <button
        type="button"
        class="btn-home"
        onclick="window.__navigateMenu && window.__navigateMenu(1)"
      >
        ← HOME으로 돌아가기
      </button>
    </div>
  </main>

</div>
```

---

## 3. CSS

```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html, body { height: 100%; background: #fff; }

body {
  font-family: 'Inter', 'Segoe UI', 'Noto Sans KR', sans-serif;
  color: #111;
  -webkit-font-smoothing: antialiased;
}

.elvis-future {
  display: flex;
  height: 100dvh;
  overflow: hidden;
}

/* 사이드바 — HOME과 동일 */
.sidebar {
  width: 220px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border-right: 1px solid #ebebeb;
  background: #fafafa;
  padding: 24px 14px;
}
.sidebar-top { display: flex; flex-direction: column; gap: 28px; }

.sidebar-logo {
  display: flex; align-items: center; gap: 8px;
  background: none; border: none; cursor: pointer;
  padding: 2px 4px; border-radius: 6px; transition: background .15s;
}
.sidebar-logo:hover { background: #f0f0f0; }

.logo-dot {
  width: 9px; height: 9px; border-radius: 50%;
  background: #22c55e; flex-shrink: 0;
  animation: blink 2.2s ease-in-out infinite;
}
@keyframes blink { 0%,100%{opacity:1} 50%{opacity:.25} }

.logo-text { font-size: 15px; font-weight: 700; letter-spacing: 0.1em; color: #111; }

.sidebar-nav { display: flex; flex-direction: column; gap: 2px; }

.nav-item {
  display: flex; align-items: center; gap: 9px;
  width: 100%; padding: 9px 10px;
  border: none; border-radius: 8px; background: transparent;
  color: #666; font-family: inherit; font-size: 13.5px; font-weight: 400;
  cursor: pointer; text-align: left; transition: background .13s, color .13s;
}
.nav-item:hover { background: #efefef; color: #111; }
.nav-item.is-active { background: #111; color: #fff; font-weight: 500; }

.nav-icon {
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0; opacity: .55;
}
.nav-item:hover .nav-icon,
.nav-item.is-active .nav-icon { opacity: 1; }

.sidebar-bottom { padding: 0 4px; }
.sidebar-brand-label { font-size: 11px; color: #bbb; letter-spacing: 0.04em; }

/* 중앙 */
.main-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.center-block {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
  text-align: center;
  padding: 40px 24px;
}

.coming-label {
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #bbb;
}

.coming-title {
  font-size: clamp(44px, 8vw, 88px);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: #111;
  line-height: 1;
}

.coming-desc {
  font-size: 15px;
  color: #999;
  margin-top: 4px;
}

.btn-home {
  margin-top: 20px;
  padding: 10px 20px;
  background: none;
  border: 1px solid #e0e0e0;
  border-radius: 999px;
  font-family: inherit;
  font-size: 13px;
  color: #555;
  cursor: pointer;
  transition: background .15s, color .15s, border-color .15s;
}
.btn-home:hover { background: #f5f5f5; border-color: #ccc; color: #111; }

/* 반응형 */
@media (max-width: 680px) {
  .elvis-future { flex-direction: column; }
  .sidebar {
    width: 100%; flex-direction: row; align-items: center;
    justify-content: space-between; border-right: none;
    border-bottom: 1px solid #ebebeb; padding: 0 16px;
    height: 52px; flex-shrink: 0;
  }
  .sidebar-top { flex-direction: row; align-items: center; gap: 16px; }
  .sidebar-nav { flex-direction: row; gap: 4px; }
  .nav-item { padding: 6px 10px; font-size: 13px; }
  .nav-icon { display: none; }
  .sidebar-bottom { display: none; }
}
```

---

## 4. JavaScript

```javascript
(() => {
  function navigateMenu(menuNumber) {
    const menuNo = Number(menuNumber);
    if (!menuNo) return;

    const targets = [];
    try { targets.push(window); } catch(e) {}
    try { if (window.parent && window.parent !== window) targets.push(window.parent); } catch(e) {}
    try { if (window.top && window.top !== window && window.top !== window.parent) targets.push(window.top); } catch(e) {}

    for (const target of targets) {
      try {
        if (target && typeof target.changeApplicationMenu === 'function') {
          target.changeApplicationMenu(menuNo);
          return;
        }
      } catch(e) {}
    }

    for (const target of targets) {
      try {
        if (target && typeof target.postMessage === 'function') {
          target.postMessage({ type: 'changeApplicationMenu', menuNumber: menuNo }, '*');
        }
      } catch(e) {}
    }
  }

  window.__navigateMenu = navigateMenu;
})();
```

---

## 5. 붙여넣기 가이드

AI Canvas 샌드박스 노드에서:

1. **HTML** 탭 → `## 2. HTML` 코드블록 전체 붙여넣기
2. **CSS** 탭 → `## 3. CSS` 코드블록 전체 붙여넣기
3. **JavaScript** 탭 → `## 4. JavaScript` 코드블록 전체 붙여넣기
