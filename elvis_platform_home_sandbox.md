# Elvis 홈 화면 샌드박스 코드

이 문서는 Elvis 플랫폼의 `HOME` 메뉴에 배치할 `샌드박스_Elvis_홈` 노드용 **실사용 코드 문서**다.

기준:

- Claude가 만든 [claude index.html](d:\AI canvas\새 폴더\claude index.html) 레이아웃을 Elvis용 홈 랜딩으로 변환한 버전이다.
- HOME은 정적인 브랜드 랜딩 역할을 한다.
- 질문은 실행하지 않는다.
- 사용자는 여기서 `AI노무사`로 진입한다.
- `근거` 메뉴는 없으므로 CTA도 두지 않는다.

노드 설정:

- 노드명: `샌드박스_Elvis_홈`
- 노드 타입: `샌드박스`
- 메뉴: `HOME`
- 입력 포트: 없음
- 출력 포트: 사용하지 않음

---

## HTML

```html
<div class="elvis-home-shell">
  <header id="homeHeader" class="home-header">
    <nav class="home-nav home-nav--left">
      <button type="button" class="nav-pill is-active" onclick="window.__navigateMenu && window.__navigateMenu(1)">HOME</button>
      <button type="button" class="nav-pill" onclick="window.__navigateMenu && window.__navigateMenu(2)">AI노무사</button>
      <button type="button" class="nav-pill" onclick="window.__navigateMenu && window.__navigateMenu(3)">추후 개발</button>
    </nav>

    <button type="button" class="home-logo" onclick="window.__navigateMenu && window.__navigateMenu(1)" aria-label="Elvis 홈으로 이동">
      <span class="home-logo__el">EL</span><span class="home-logo__vis">vis</span>
    </button>

    <div class="home-header__right">
      <span class="availability">Internal beta open</span>
      <button type="button" class="btn-primary" onclick="window.__navigateMenu && window.__navigateMenu(2)">AI노무사 시작</button>
      <button type="button" class="btn-text" onclick="window.__navigateMenu && window.__navigateMenu(3)">추후 개발 보기</button>
    </div>
  </header>

  <main>
    <section class="hero">
      <p class="hero-tag">Knowledge Platform for Labor Practice</p>
      <h1>
        Your shortcut<br>
        to <em>usable</em> labor knowledge<br>
        in one place.
      </h1>
      <p class="hero-sub">
        Elvis는 회사의 규정, 내부 사례, 판례, 행정해석, 실무 문안을 한 곳에 모아
        필요한 순간 바로 찾고, 묻고, 활용할 수 있게 만든 플랫폼입니다.
      </p>
      <div class="hero-tags">
        <div class="tag-chip blue">규정</div>
        <div class="tag-chip orange">내부 사례</div>
        <div class="tag-chip purple">판례</div>
        <div class="tag-chip red">행정해석</div>
        <div class="tag-chip green">노무상담</div>
        <div class="tag-chip teal">문안 작성</div>
      </div>
    </section>

    <div class="gallery">
      <article class="card card-1" onclick="window.__navigateMenu && window.__navigateMenu(2)">
        <div class="card-placeholder">
          <div class="card-icon">쨌</div>
          <div class="card-label">Ask the AI labor counsel</div>
          <div class="card-sub">질문을 입력하고 바로 답변을 받습니다.</div>
        </div>
      </article>

      <article class="card card-2">
        <div class="card-placeholder">
          <div class="card-icon">쨰</div>
          <div class="card-label">Internal cases that actually matter</div>
          <div class="card-sub">유사 사례와 양정 판단을 실무 중심으로 연결합니다.</div>
        </div>
      </article>

      <article class="card card-3">
        <div class="card-placeholder">
          <div class="card-icon">쨜</div>
          <div class="card-label">Official rules, precedents, interpretations</div>
          <div class="card-sub">조문, 사건번호, 근거를 답변 안에서 바로 읽습니다.</div>
        </div>
      </article>

      <article class="card card-4">
        <div class="card-placeholder">
          <div class="card-icon">쨶</div>
          <div class="card-label">Write with structure, not guesswork</div>
          <div class="card-sub">앞으로 의견서와 문안 작성 기능도 같은 플랫폼에 붙습니다.</div>
        </div>
      </article>

      <article class="card card-5" onclick="window.__navigateMenu && window.__navigateMenu(3)">
        <div class="card-placeholder">
          <div class="card-icon">쨄</div>
          <div class="card-label">What comes next</div>
          <div class="card-sub">지식문서, 작성도구, 검색 허브가 순차적으로 추가됩니다.</div>
        </div>
      </article>
    </div>

    <section class="section">
      <div class="section-label">How Elvis works</div>
      <div class="process-list">
        <div class="process-item">
          <div class="process-num">01</div>
          <div class="process-title">Find</div>
          <div class="process-desc">규정, 내부 사례, 판례, 행정해석, 웹 보완 근거를 하나의 흐름에서 연결합니다.</div>
        </div>
        <div class="process-item">
          <div class="process-num">02</div>
          <div class="process-title">Judge</div>
          <div class="process-desc">질문에 맞는 판단 기준과 내부 유사 사례를 바탕으로 결론을 더 명확하게 정리합니다.</div>
        </div>
        <div class="process-item">
          <div class="process-num">03</div>
          <div class="process-title">Use</div>
          <div class="process-desc">답변을 읽고 끝내지 않고, 실제 상담·보고·문안 작성까지 이어지도록 플랫폼을 확장합니다.</div>
        </div>
      </div>
    </section>
  </main>

  <footer class="home-footer">
    <div class="logo-sm">ELVIS</div>
    <span>Whole Knowledge, Used Wisely.</span>
    <span>Internal labor knowledge platform</span>
  </footer>
</div>
```

---

## CSS

```css
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --black: #111111;
  --gray: #7d7d79;
  --light: #f5f5f3;
  --border: #e8e8e4;
}

body {
  font-family: 'Segoe UI', 'Noto Sans KR', sans-serif;
  background: #ffffff;
  color: var(--black);
  overflow-x: hidden;
}

html, body {
  width: 100%;
  min-height: 100%;
}

.elvis-home-shell {
  min-height: 100dvh;
  width: 100%;
  overflow-x: hidden;
  background: #fff;
}

main {
  width: 100%;
  overflow-x: hidden;
}

.home-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: center;
  padding: 24px 40px;
  background: rgba(255,255,255,0.92);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid transparent;
  transition: border-color 0.3s;
}

.home-header.scrolled {
  border-bottom-color: var(--border);
}

.home-nav {
  display: flex;
  gap: 8px;
}

.home-nav--left {
  justify-content: flex-start;
}

.nav-pill {
  font-size: 13px;
  font-weight: 400;
  color: var(--black);
  background: #ffffff;
  border: 1px solid var(--border);
  border-radius: 999px;
  padding: 7px 14px;
  letter-spacing: 0.01em;
  cursor: pointer;
  transition: background 0.2s, border-color 0.2s, opacity 0.2s;
}

.nav-pill:hover {
  background: var(--light);
}

.nav-pill.is-active {
  background: var(--light);
}

.home-logo {
  border: none;
  background: transparent;
  cursor: pointer;
  text-align: center;
  font-family: Georgia, 'Times New Roman', serif;
  font-size: 20px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.home-logo__vis {
  font-style: italic;
}

.home-header__right {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 16px;
}

.availability {
  font-size: 12px;
  color: #2d9c5a;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 6px;
}

.availability::before {
  content: '';
  width: 7px;
  height: 7px;
  background: #2d9c5a;
  border-radius: 50%;
  display: inline-block;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.btn-primary {
  font-size: 13px;
  font-weight: 500;
  color: #fff;
  background: var(--black);
  border: none;
  padding: 9px 20px;
  border-radius: 999px;
  cursor: pointer;
  transition: opacity 0.2s;
  letter-spacing: 0.01em;
}

.btn-primary:hover {
  opacity: 0.78;
}

.btn-text {
  font-size: 13px;
  font-weight: 400;
  color: var(--black);
  background: transparent;
  border: none;
  cursor: pointer;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.btn-text:hover {
  opacity: 1;
}

.hero {
  width: min(1200px, calc(100% - 80px));
  margin: 0 auto;
  padding: 160px 0 100px;
}

.hero-tag {
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--gray);
  margin-bottom: 28px;
}

.hero h1 {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(48px, 7vw, 96px);
  font-weight: 400;
  line-height: 1.08;
  letter-spacing: -0.02em;
  color: var(--black);
  max-width: 860px;
}

.hero h1 em {
  font-style: italic;
  font-weight: 400;
}

.hero-sub {
  margin-top: 36px;
  font-size: 14px;
  color: var(--gray);
  max-width: 520px;
  line-height: 1.7;
}

.hero-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 20px;
  align-items: center;
}

.tag-chip {
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 4px 12px;
  border: 1px solid var(--border);
  border-radius: 100px;
  color: var(--black);
}

.tag-chip::before {
  content: '';
  width: 6px;
  height: 6px;
  border-radius: 50%;
  flex-shrink: 0;
}

.tag-chip.blue::before   { background: #4a90d9; }
.tag-chip.orange::before { background: #e87c3e; }
.tag-chip.purple::before { background: #9b59b6; }
.tag-chip.red::before    { background: #e74c3c; }
.tag-chip.green::before  { background: #27ae60; }
.tag-chip.teal::before   { background: #1abc9c; }

.gallery {
  width: min(1200px, calc(100% - 80px));
  margin: 40px auto 0;
  display: grid;
  grid-template-columns: repeat(12, minmax(0, 1fr));
  gap: 16px;
  overflow: visible;
  padding: 0;
}

.card {
  min-width: 0;
  height: 440px;
  border-radius: 16px;
  overflow: hidden;
  position: relative;
  background: var(--light);
  transition: transform 0.3s ease;
  cursor: default;
}

.card:hover { transform: translateY(-6px); }
.card.is-clickable,
.card[onclick] { cursor: pointer; }

.card-1 { grid-column: span 4; }
.card-2 { grid-column: span 3; }
.card-3 { grid-column: span 5; }
.card-4 { grid-column: span 6; }
.card-5 { grid-column: span 6; }

.card-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 28px;
}

.card-1 { background: #f0ede6; }
.card-2 { background: #c94a1a; }
.card-3 { background: #d4e8f0; }
.card-4 { background: #1a1a1a; }
.card-5 { background: #f7c948; }

.card-label {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 22px;
  font-weight: 700;
  line-height: 1.2;
}

.card-sub {
  font-size: 12px;
  margin-top: 8px;
  opacity: 0.7;
  line-height: 1.6;
}

.card-1 .card-label { color: #1a1a1a; }
.card-2 .card-label, .card-2 .card-sub { color: #ffffff; }
.card-3 .card-label { color: #1a1a1a; }
.card-4 .card-label, .card-4 .card-sub { color: #ffffff; }
.card-5 .card-label { color: #1a1a1a; }

.card-icon {
  width: 42px;
  height: 42px;
  border-radius: 10px;
  background: rgba(255,255,255,0.25);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  margin-bottom: 16px;
}

.section {
  width: min(1200px, calc(100% - 80px));
  padding: 100px 0;
  margin: 0 auto;
}

.section-label {
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--gray);
  margin-bottom: 48px;
  border-top: 1px solid var(--border);
  padding-top: 20px;
}

.process-list {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 40px;
}

.process-item {
  padding: 32px;
  border: 1px solid var(--border);
  border-radius: 16px;
  transition: background 0.2s;
}

.process-item:hover {
  background: var(--light);
}

.process-num {
  font-size: 11px;
  color: var(--gray);
  margin-bottom: 20px;
  font-variant-numeric: tabular-nums;
}

.process-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 22px;
  font-weight: 400;
  margin-bottom: 12px;
}

.process-desc {
  font-size: 13px;
  color: var(--gray);
  line-height: 1.7;
}

.home-footer {
  border-top: 1px solid var(--border);
  padding: 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  color: var(--gray);
}

.logo-sm {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: 16px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: var(--black);
}

@media (max-width: 960px) {
  .home-header {
    grid-template-columns: 1fr;
    gap: 14px;
    justify-items: start;
    padding: 20px 18px;
  }

  .home-header__right {
    justify-content: flex-start;
    flex-wrap: wrap;
  }

  .hero {
    width: calc(100% - 36px);
    padding: 168px 0 72px;
  }

  .gallery {
    width: calc(100% - 36px);
    grid-template-columns: 1fr;
    margin-top: 32px;
  }

  .process-list {
    grid-template-columns: 1fr;
    gap: 18px;
  }

  .section {
    width: calc(100% - 36px);
    padding: 72px 0;
  }

  .home-footer {
    padding: 28px 18px;
    flex-direction: column;
    gap: 10px;
    align-items: flex-start;
  }
}
```

---

## JavaScript

```javascript
function navigateMenu(menuNumber) {
  const menuNo = Number(menuNumber);
  if (!menuNo || menuNo < 1) {
    console.warn('invalid menu number:', menuNumber);
    return false;
  }

  const targets = [];
  if (typeof window !== 'undefined') targets.push(window);
  if (typeof window !== 'undefined' && window.parent && window.parent !== window) targets.push(window.parent);
  if (typeof window !== 'undefined' && window.top && window.top !== window && window.top !== window.parent) targets.push(window.top);

  for (const target of targets) {
    try {
      if (target && typeof target.changeApplicationMenu === 'function') {
        target.changeApplicationMenu(menuNo);
        return true;
      }
    } catch (error) {
      console.warn('changeApplicationMenu call failed', error);
    }
  }

  for (const target of targets) {
    try {
      if (target && typeof target.postMessage === 'function') {
        target.postMessage({ type: 'changeApplicationMenu', menuNumber: menuNo }, '*');
      }
    } catch (error) {
      console.warn('postMessage fallback failed', error);
    }
  }

  console.warn('menu navigation handler not found:', menuNo);
  return false;
}

function goHome() {
  navigateMenu(1);
}

function goLabor() {
  navigateMenu(2);
}

function goFuture() {
  navigateMenu(3);
}

window.__navigateMenu = navigateMenu;
window.__goHome = goHome;
window.__goLabor = goLabor;
window.__goFuture = goFuture;

const header = document.getElementById('homeHeader');
window.addEventListener('scroll', () => {
  if (!header) return;
  header.classList.toggle('scrolled', window.scrollY > 10);
});
```

---

## 사용 메모

1. 이 문서는 `claude index.html`의 레이아웃을 Elvis용으로 번역한 기준안이다.
2. 지금 상태로도 AI Canvas 샌드박스에 바로 붙여 사용할 수 있다.
3. 디자인을 더 다듬고 싶으면 이 문서를 Claude에게 다시 보내면 된다.
