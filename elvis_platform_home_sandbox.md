# Elvis 홈 화면 샌드박스 코드

이 문서는 Elvis 플랫폼의 `홈` 메뉴에 배치할 `샌드박스_Elvis_홈` 노드용 코드 문서다.

노드 설정:

- 노드명: `샌드박스_Elvis_홈`
- 노드 타입: `샌드박스`
- 메뉴: `홈`
- 입력 포트: 없음
- 출력 포트: 사용하지 않음

이 홈 화면은 정적인 브랜드 허브 역할만 한다.
즉 질문을 실행하지 않고, Elvis가 어떤 플랫폼인지 소개하고 사용자를 `노무상담`, `근거` 메뉴로 안내한다.

---

## HTML

```html
<div class="elvis-home">
  <section class="hero">
    <div class="hero__copy">
      <div class="badge">Elvis Platform</div>
      <h1 class="title">Whole Knowledge, Used Wisely</h1>
      <p class="subtitle">
        Elvis는 규정, 사례, 판례, 실무 문안을 한 곳에 모아
        필요한 순간 바로 꺼내 쓰게 돕는 지식 활용 플랫폼입니다.
      </p>
      <div class="cta-row">
        <button class="cta cta--primary" id="goConsult">노무상담 시작하기</button>
        <button class="cta cta--secondary" id="goEvidence">근거 보기</button>
      </div>
    </div>

    <div class="hero__art" aria-label="Elvis hero illustration">
      <svg viewBox="0 0 760 520" class="hero-graphic" fill="none" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <linearGradient id="heroBg" x1="90" y1="60" x2="650" y2="470" gradientUnits="userSpaceOnUse">
            <stop offset="0" stop-color="#173B63"/>
            <stop offset="1" stop-color="#2E648F"/>
          </linearGradient>
          <linearGradient id="heroGold" x1="290" y1="170" x2="470" y2="350" gradientUnits="userSpaceOnUse">
            <stop offset="0" stop-color="#E8D6AF"/>
            <stop offset="1" stop-color="#B98B34"/>
          </linearGradient>
        </defs>

        <rect x="48" y="52" width="664" height="416" rx="34" fill="url(#heroBg)"/>

        <g opacity="0.18">
          <circle cx="138" cy="120" r="4" fill="white"/>
          <circle cx="620" cy="112" r="5" fill="white"/>
          <circle cx="640" cy="392" r="4" fill="white"/>
          <circle cx="110" cy="402" r="6" fill="white"/>
        </g>

        <rect x="114" y="122" width="232" height="276" rx="22" fill="white" fill-opacity="0.1" stroke="white" stroke-opacity="0.18"/>
        <text x="144" y="174" fill="white" font-family="Segoe UI, Noto Sans KR, sans-serif" font-size="18" font-weight="700">Whole Knowledge</text>
        <text x="144" y="208" fill="white" font-family="Segoe UI, Noto Sans KR, sans-serif" font-size="18" font-weight="700">Used Wisely</text>
        <text x="144" y="264" fill="#F5E7BF" font-family="Segoe UI, Noto Sans KR, sans-serif" font-size="42" font-weight="800">Elvis</text>
        <text x="144" y="318" fill="#DCE8F4" font-family="Segoe UI, Noto Sans KR, sans-serif" font-size="16">Knowledge Platform</text>
        <text x="144" y="346" fill="#DCE8F4" font-family="Segoe UI, Noto Sans KR, sans-serif" font-size="16">Policy · Cases · Practice</text>

        <rect x="404" y="120" width="232" height="278" rx="26" fill="white" fill-opacity="0.82"/>
        <path d="M454 168C454 158.059 462.059 150 472 150H520V352H492C470.461 352 454 335.539 454 314V168Z" fill="#F8FBFD"/>
        <path d="M586 168C586 158.059 577.941 150 568 150H520V352H548C569.539 352 586 335.539 586 314V168Z" fill="#EDF4FA"/>
        <path d="M472 150H520V178H472C467.582 178 464 181.582 464 186V314C464 330.569 477.431 344 494 344H520V372H494C462.52 372 436 347.52 436 316V186C436 166.118 452.118 150 472 150Z" fill="url(#heroGold)"/>
        <path d="M568 150H520V178H568C572.418 178 576 181.582 576 186V314C576 330.569 562.569 344 546 344H520V372H546C577.48 372 604 347.52 604 316V186C604 166.118 587.882 150 568 150Z" fill="url(#heroGold)"/>

        <circle cx="520" cy="126" r="16" fill="#E8D6AF"/>
        <circle cx="422" cy="252" r="10" fill="#D7E6F5"/>
        <circle cx="618" cy="252" r="10" fill="#D7E6F5"/>
        <circle cx="520" cy="404" r="10" fill="#D7E6F5"/>

        <path d="M520 142V150" stroke="#E8D6AF" stroke-width="4" stroke-linecap="round"/>
        <path d="M432 252H436" stroke="#D7E6F5" stroke-width="4" stroke-linecap="round"/>
        <path d="M604 252H608" stroke="#D7E6F5" stroke-width="4" stroke-linecap="round"/>
        <path d="M520 372V394" stroke="#D7E6F5" stroke-width="4" stroke-linecap="round"/>
      </svg>
    </div>
  </section>

  <section class="panel">
    <div class="section-head">
      <div class="section-label">Platform Menu</div>
      <h2>Elvis 안에서 할 수 있는 일</h2>
    </div>

    <div class="menu-grid">
      <article class="menu-card menu-card--active">
        <div class="menu-card__title">노무상담</div>
        <p class="menu-card__desc">질문을 입력하면 내부 사례, 규정, 공식 근거, 웹 보완을 종합해 답변합니다.</p>
      </article>

      <article class="menu-card menu-card--active">
        <div class="menu-card__title">근거</div>
        <p class="menu-card__desc">공식 citation, 내부 사례, 웹 보완 링크를 나눠서 확인합니다.</p>
      </article>

      <article class="menu-card">
        <div class="menu-card__title">지식문서</div>
        <p class="menu-card__desc">규정집, 가이드, 판례집, 사내 문서를 나중에 한 곳에서 탐색하도록 확장할 메뉴입니다.</p>
      </article>

      <article class="menu-card">
        <div class="menu-card__title">작성도구</div>
        <p class="menu-card__desc">검토의견, 답변서, 의견서 초안을 만드는 기능을 나중에 연결할 메뉴입니다.</p>
      </article>
    </div>
  </section>

  <section class="two-col">
    <article class="panel">
      <div class="section-head">
        <div class="section-label">Why Elvis</div>
        <h2>플랫폼 철학</h2>
      </div>
      <ul class="feature-list">
        <li>지식을 쌓는 것에서 끝나지 않고, 바로 활용할 수 있게 만든다.</li>
        <li>답변과 근거를 분리해 보여줘 신뢰성과 실무성을 함께 확보한다.</li>
        <li>지금은 노무상담부터 시작하지만, 앞으로 더 많은 업무 도구를 같은 플랫폼 안에 붙일 수 있다.</li>
      </ul>
    </article>

    <article class="panel">
      <div class="section-head">
        <div class="section-label">Current Scope</div>
        <h2>지금 제공하는 범위</h2>
      </div>
      <ul class="feature-list">
        <li>내부 사례 기반 징계/양정 판단 보조</li>
        <li>규정, 판례, 행정해석, 웹 보완 근거 제시</li>
        <li>작성형 요청 시 의견서 스타일 참고</li>
      </ul>
    </article>
  </section>
</div>
```

---

## CSS

```css
:root {
  --bg: #f5f3ee;
  --card: #ffffff;
  --line: #ded7ca;
  --text: #17212b;
  --sub: #5f6b76;
  --navy: #163b63;
  --navy-soft: #e8eff7;
  --sand: #efe6d6;
  --gold: #b78b2f;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: "Segoe UI", "Noto Sans KR", sans-serif;
  color: var(--text);
  background:
    radial-gradient(circle at top right, rgba(183, 139, 47, 0.18), transparent 28%),
    linear-gradient(180deg, #fbfaf7 0%, #f1ede4 100%);
}

.elvis-home {
  max-width: 1180px;
  margin: 0 auto;
  padding: 28px 18px 56px;
}

.hero {
  display: grid;
  grid-template-columns: 1.15fr 0.85fr;
  gap: 20px;
  align-items: center;
  margin-bottom: 24px;
}

.hero__copy,
.panel {
  background: rgba(255, 255, 255, 0.86);
  border: 1px solid var(--line);
  border-radius: 24px;
  padding: 24px;
  box-shadow: 0 12px 40px rgba(18, 24, 32, 0.07);
  backdrop-filter: blur(10px);
}

.hero__art {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 320px;
  background: linear-gradient(135deg, rgba(255,255,255,0.78), rgba(244,239,229,0.9));
  border: 1px solid var(--line);
  border-radius: 24px;
  box-shadow: 0 12px 40px rgba(18, 24, 32, 0.07);
}

.hero-graphic {
  width: min(100%, 500px);
  height: auto;
  display: block;
}

.badge,
.section-label {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 10px;
  border-radius: 999px;
  background: var(--navy-soft);
  color: var(--navy);
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.02em;
}

.title {
  margin: 14px 0 10px;
  font-size: 42px;
  line-height: 1.08;
}

.subtitle {
  margin: 0;
  max-width: 58ch;
  line-height: 1.8;
  color: var(--sub);
  font-size: 15px;
}

.cta-row {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-top: 22px;
}

.cta {
  appearance: none;
  border: none;
  border-radius: 999px;
  padding: 12px 18px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
}

.cta--primary {
  background: var(--navy);
  color: #fff;
}

.cta--secondary {
  background: var(--sand);
  color: #4a3d25;
}

.section-head h2 {
  margin: 10px 0 0;
  font-size: 24px;
}

.menu-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 14px;
  margin-top: 20px;
}

.menu-card {
  border: 1px solid var(--line);
  border-radius: 18px;
  padding: 18px;
  background: linear-gradient(180deg, #fff 0%, #faf7f1 100%);
}

.menu-card--active {
  border-color: rgba(22, 59, 99, 0.28);
  box-shadow: inset 0 0 0 1px rgba(22, 59, 99, 0.08);
}

.menu-card__title {
  font-size: 18px;
  font-weight: 800;
  margin-bottom: 8px;
}

.menu-card__desc {
  margin: 0;
  line-height: 1.7;
  color: var(--sub);
  font-size: 14px;
}

.two-col {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 14px;
  margin-top: 14px;
}

.feature-list {
  margin: 18px 0 0;
  padding-left: 18px;
  line-height: 1.75;
}

@media (max-width: 980px) {
  .hero {
    grid-template-columns: 1fr;
  }

  .menu-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .two-col {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .elvis-home {
    padding: 18px 12px 36px;
  }

  .title {
    font-size: 32px;
  }

  .menu-grid {
    grid-template-columns: 1fr;
  }
}
```

---

## JavaScript

```javascript
function safeChangeMenu(menuName) {
  try {
    changeApplicationMenu({ menuName });
  } catch (error) {
    console.warn("메뉴 이동 실패", error);
  }
}

document.getElementById("goConsult")?.addEventListener("click", () => {
  safeChangeMenu("노무상담");
});

document.getElementById("goEvidence")?.addEventListener("click", () => {
  safeChangeMenu("근거");
});
```

---

## 구현 팁

1. 처음에는 이 홈 화면을 정적으로 둔다.
2. 질문 실행은 여기서 하지 않는다.
3. `노무상담` 메뉴 안의 `에이전트_법률상담입력`이 실제 실행 시작점이다.
4. Claude에게 스타일을 맡길 때는 이 문서의 구조를 기준으로 확장하면 된다.
