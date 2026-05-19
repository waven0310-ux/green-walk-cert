# QR 스탬프 완주 인증 시스템 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** index.html 단일 파일에 QR 스탬프 3개 수집 흐름을 추가해 완주 인증서 발급과 연결한다.

**Architecture:** 기존 screen-input/screen-cert 화면 앞에 screen-stamp 화면을 추가한다. URL 파라미터(?cp=N&token=T)로 스탬프를 지급하고 localStorage에 저장한다. 3개 수집 완료 시 기존 이름 입력 → 인증서 흐름으로 진입한다.

**Tech Stack:** Vanilla JS, localStorage, URL SearchParams, CSS animation

---

## 파일 수정 대상

- Modify: `index.html` — CSS 추가(stamp UI), HTML 추가(screen-stamp), JS 추가(stamp 로직), 초기화 수정, goBack/downloadCert 수정

---

### Task 1: 도장 이미지 파일 준비

**Files:**
- Create: `assets/stamp.png`

- [ ] **Step 1: assets 폴더 확인**

```bash
ls "c:/Users/UserM.WH/Desktop/새 폴더/assets"
```

- [ ] **Step 2: 캐릭터 도장 이미지를 `assets/stamp.png`로 저장**

제공받은 도장 이미지(PNG, 투명배경 권장)를 `assets/stamp.png` 경로에 저장한다.
임시 테스트용으로는 아무 PNG를 넣어도 무방하다.

- [ ] **Step 3: 이미지 표시 확인**

브라우저에서 `file:///...assets/stamp.png` 직접 열어 이미지가 표시되는지 확인.

---

### Task 2: stamp 상태 관리 JS 추가

**Files:**
- Modify: `index.html` — `<script>` 블록 상단에 STAMPS 객체 추가

- [ ] **Step 1: 기존 스크립트 블록 시작 위치 확인**

`index.html`에서 `<script>` 태그를 찾는다. `currentName` 변수 선언 바로 위에 삽입.

```bash
grep -n "currentName\|<script>" "c:/Users/UserM.WH/Desktop/새 폴더/index.html" | head -20
```

- [ ] **Step 2: STAMPS 객체 삽입**

`currentName` 선언 바로 위에 다음 코드를 추가한다:

```js
const STAMPS = {
  tokens: { 1: 'Gw26cP1xK9m', 2: 'Gw26cP2nR4j', 3: 'Gw26cP3bT7p' },
  get()      { return JSON.parse(localStorage.getItem('greenWalkStamps') || '[]'); },
  save(arr)  { localStorage.setItem('greenWalkStamps', JSON.stringify(arr)); },
  clear()    { localStorage.removeItem('greenWalkStamps'); },
  has(n)     { return this.get().includes(n); },
  add(n)     { const s = this.get(); if (!s.includes(n)) { s.push(n); this.save(s); } },
  count()    { return this.get().length; }
};
```

- [ ] **Step 3: 브라우저 콘솔 검증**

`index.html`을 열고 콘솔에서:

```js
STAMPS.add(1); STAMPS.get(); // [1]
STAMPS.add(1); STAMPS.get(); // [1] (중복 없음)
STAMPS.add(2); STAMPS.count(); // 2
STAMPS.clear(); STAMPS.get(); // []
```

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: STAMPS 상태관리 객체 추가"
```

---

### Task 3: screen-stamp HTML 추가

**Files:**
- Modify: `index.html` — `<!-- 입력 화면 -->` 주석 바로 위에 삽입

- [ ] **Step 1: 삽입 위치 확인**

```bash
grep -n "입력 화면\|screen-input" "c:/Users/UserM.WH/Desktop/새 폴더/index.html" | head -10
```

- [ ] **Step 2: screen-stamp HTML 삽입**

`<!-- 입력 화면 -->` 주석 바로 앞에 다음을 삽입한다:

```html
<!-- 스탬프 화면 -->
<div id="screen-stamp" style="display:none; width:100%; max-width:500px; text-align:center;">
  <div class="stamp-card">
    <div class="stamp-event-tag">2026 · 환경의 날 기념 패밀리 그린워크</div>
    <div class="stamp-title">초록 발자국 남기기</div>
    <div class="stamp-progress" id="stampProgress">0 / 3 스탬프 수집</div>
    <div class="stamp-circles" id="stampCircles">
      <div class="stamp-circle" id="stamp-1"><span class="stamp-num">1</span></div>
      <div class="stamp-circle" id="stamp-2"><span class="stamp-num">2</span></div>
      <div class="stamp-circle" id="stamp-3"><span class="stamp-num">3</span></div>
    </div>
    <div class="stamp-hint" id="stampHint">체크포인트 1 QR을 찍어주세요</div>
    <div class="stamp-message" id="stampMessage"></div>
    <button class="btn-get-cert" id="btnGetCert" onclick="goToNameInput()">완주 인증서 받기</button>
  </div>
</div>
```

- [ ] **Step 3: 브라우저에서 HTML 구조 확인**

`index.html`을 열고 Elements 탭에서 `#screen-stamp` 존재 확인.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: screen-stamp HTML 추가"
```

---

### Task 4: stamp 화면 CSS 추가

**Files:**
- Modify: `index.html` — `</style>` 닫는 태그 바로 앞에 삽입

- [ ] **Step 1: 삽입 위치 확인**

```bash
grep -n "</style>" "c:/Users/UserM.WH/Desktop/새 폴더/index.html"
```

- [ ] **Step 2: CSS 삽입**

`</style>` 바로 앞에 다음을 추가한다:

```css
/* ── 스탬프 화면 ── */
.stamp-card {
  background: white;
  padding: 40px 36px;
  box-shadow: 0 16px 48px rgba(0,0,0,0.12);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}
.stamp-event-tag {
  display: inline-block;
  background: var(--green);
  color: white;
  font-size: 11px;
  font-weight: 700;
  padding: 5px 16px;
  letter-spacing: 0.5px;
}
.stamp-title {
  font-size: 24px;
  font-weight: 900;
  color: var(--green-dark);
  letter-spacing: -0.5px;
}
.stamp-progress {
  font-size: 14px;
  font-weight: 700;
  color: var(--green);
}
.stamp-circles {
  display: flex;
  gap: 20px;
  align-items: center;
  margin: 8px 0;
}
.stamp-circle {
  width: 90px;
  height: 90px;
  border-radius: 50%;
  background: var(--green-light);
  border: 2px dashed #7ab890;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s ease;
}
.stamp-circle.stamped {
  background: var(--green);
  border: none;
  box-shadow: 0 4px 12px rgba(0,136,58,0.4);
}
.stamp-circle.stamped img {
  width: 72px;
  height: 72px;
  object-fit: contain;
}
.stamp-circle.pop {
  animation: stampPop 0.45s cubic-bezier(0.34,1.56,0.64,1) both;
}
@keyframes stampPop {
  from { transform: scale(0.4); opacity: 0.5; }
  to   { transform: scale(1);   opacity: 1; }
}
.stamp-num {
  font-size: 26px;
  font-weight: 700;
  color: #7ab890;
}
.stamp-hint {
  font-size: 13px;
  color: var(--muted);
  min-height: 18px;
}
.stamp-message {
  font-size: 14px;
  font-weight: 700;
  color: var(--green);
  min-height: 20px;
}
.btn-get-cert {
  background: var(--green-dark);
  color: white;
  border: none;
  padding: 14px 40px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  border-radius: 4px;
  margin-top: 8px;
  opacity: 0.3;
  pointer-events: none;
  font-family: 'Pretendard', 'Noto Sans KR', sans-serif;
  transition: opacity 0.3s, background 0.2s;
}
.btn-get-cert.active {
  opacity: 1;
  pointer-events: auto;
}
.btn-get-cert.active:hover {
  background: var(--green);
}
@media (max-width: 660px) {
  .stamp-card { padding: 32px 20px; }
  .stamp-circle { width: 76px; height: 76px; }
  .stamp-circle.stamped img { width: 60px; height: 60px; }
}
```

- [ ] **Step 3: screen-stamp를 임시로 표시해 CSS 확인**

브라우저 콘솔:
```js
document.getElementById('screen-stamp').style.display = 'block';
document.getElementById('screen-input').style.display = 'none';
```
스탬프 화면이 올바르게 렌더링되는지 확인.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: stamp 화면 CSS 추가"
```

---

### Task 5: stamp 핵심 JS 함수 추가

**Files:**
- Modify: `index.html` — `downloadCert` 함수 아래에 추가

- [ ] **Step 1: 함수 추가**

`downloadCert` 함수 끝 (`}`) 바로 다음에 다음을 삽입한다:

```js
function renderStampScreen(newStamp) {
  const collected = STAMPS.get();
  const count = collected.length;

  document.getElementById('stampProgress').textContent = count + ' / 3 스탬프 수집';

  for (let i = 1; i <= 3; i++) {
    const el = document.getElementById('stamp-' + i);
    if (collected.includes(i) && !el.classList.contains('stamped')) {
      el.classList.add('stamped');
      el.innerHTML = '<img src="assets/stamp.png" alt="스탬프">';
      if (newStamp === i) {
        el.classList.remove('pop');
        void el.offsetWidth;
        el.classList.add('pop');
      }
    }
  }

  const hint = document.getElementById('stampHint');
  const btn  = document.getElementById('btnGetCert');

  if (count >= 3) {
    hint.textContent = '모든 체크포인트 완료! 🎉';
    btn.classList.add('active');
  } else {
    hint.textContent = '체크포인트 ' + (count + 1) + ' QR을 찍어주세요';
    btn.classList.remove('active');
  }
}

function processQRParam() {
  const params = new URLSearchParams(window.location.search);
  const cp     = parseInt(params.get('cp'));
  const token  = params.get('token');

  if (!cp || !token) return null;
  if (STAMPS.tokens[cp] !== token) return null;

  const collected = STAMPS.get();
  if (collected.includes(cp)) return cp; // 이미 수집 — 중복 무시

  if (cp > 1 && !collected.includes(cp - 1)) return 'wrong-order';

  STAMPS.add(cp);
  return cp;
}

function goToNameInput() {
  document.getElementById('screen-stamp').style.display = 'none';
  document.getElementById('screen-input').style.display = 'block';
  window.scrollTo(0, 0);
}

function initStampScreen() {
  const result = processQRParam();

  const msgEl = document.getElementById('stampMessage');
  if (result === 'wrong-order') {
    msgEl.textContent = '이전 체크포인트를 먼저 방문해주세요!';
  } else if (typeof result === 'number') {
    msgEl.textContent = '스탬프 ' + result + '번 완료! ✓';
    setTimeout(() => { msgEl.textContent = ''; }, 3000);
  }

  renderStampScreen(typeof result === 'number' ? result : null);

  // URL 파라미터 제거 (재방문 시 중복 방지)
  if (window.history.replaceState) {
    window.history.replaceState({}, document.title, window.location.pathname);
  }

  document.getElementById('screen-stamp').style.display = 'block';
}
```

- [ ] **Step 2: 브라우저 콘솔로 함수 동작 확인**

콘솔에서:
```js
STAMPS.clear();
STAMPS.add(1);
STAMPS.add(2);
renderStampScreen(2); // 원 1,2 초록, 3 점선
```
원 1, 2가 초록으로 채워지고 3은 점선으로 표시되는지 확인.

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: renderStampScreen / processQRParam / initStampScreen 함수 추가"
```

---

### Task 6: 페이지 초기화 수정

**Files:**
- Modify: `index.html` — 페이지 로드 시 실행 코드 수정

- [ ] **Step 1: 기존 초기화 코드 위치 확인**

현재 페이지 로드 시 실행되는 코드(EVENT 객체, scaleCert 호출 등)를 확인한다:

```bash
grep -n "scaleCert\|window.onload\|DOMContentLoaded\|certBottomDate\|addEventListener" "c:/Users/UserM.WH/Desktop/새 폴더/index.html" | head -20
```

- [ ] **Step 2: 초기화 블록 수정**

`scaleCert()` 호출 코드와 이벤트리스너가 있는 영역을 찾아 아래 초기화 코드를 추가한다. 기존 `scaleCert()` 단독 호출 앞에:

```js
// 페이지 로드 시: screen-input 숨기고 stamp 화면 초기화
document.getElementById('screen-input').style.display = 'none';
initStampScreen();
```

- [ ] **Step 3: 동작 확인**

1. `index.html` 새로고침 → 스탬프 화면이 기본으로 표시되는지 확인
2. `index.html?cp=1&token=Gw26cP1xK9m` 접속 → 스탬프 1번 찍히는지 확인
3. `index.html?cp=2&token=Gw26cP2nR4j` 접속 → "이전 체크포인트 먼저" 메시지 확인 (스탬프 1 없이)
4. 스탬프 1 있는 상태에서 `?cp=2&token=Gw26cP2nR4j` → 스탬프 2 찍히는지 확인
5. `?cp=1&token=wrongtoken` → 무시되는지 확인

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: 페이지 초기화 시 stamp 화면 표시"
```

---

### Task 7: goBack 함수 수정

**Files:**
- Modify: `index.html` — `goBack` 함수

- [ ] **Step 1: 기존 goBack 찾기**

```bash
grep -n "function goBack" "c:/Users/UserM.WH/Desktop/새 폴더/index.html"
```

- [ ] **Step 2: goBack 수정**

인증서 화면에서 뒤로가면 stamp 화면으로 돌아간다:

기존:
```js
function goBack() {
  document.getElementById('screen-cert').style.display = 'none';
  document.getElementById('screen-input').style.display = 'block';
}
```

변경 후:
```js
function goBack() {
  document.getElementById('screen-cert').style.display = 'none';
  document.getElementById('screen-input').style.display = 'none';
  renderStampScreen(null);
  document.getElementById('screen-stamp').style.display = 'block';
  window.scrollTo(0, 0);
}
```

- [ ] **Step 3: 동작 확인**

스탬프 3개 수집 → 이름 입력 → 인증서 화면 → "← 다시 입력" 클릭 → 스탬프 화면으로 돌아오는지 확인.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "fix: goBack이 stamp 화면으로 복귀하도록 수정"
```

---

### Task 8: downloadCert 완료 후 스탬프 초기화

**Files:**
- Modify: `index.html` — `downloadCert` 함수 내 성공 콜백

- [ ] **Step 1: 성공 콜백 위치 확인**

```bash
grep -n "저장 완료\|setTimeout(restoreAll" "c:/Users/UserM.WH/Desktop/새 폴더/index.html"
```

- [ ] **Step 2: 다운로드 성공 시 stamps 초기화 추가**

다운로드 성공 `.then(canvas => { ... })` 블록에서 `STAMPS.clear();`를 삽입한다:

기존:
```js
  }).then(canvas => {
    const a = document.createElement('a');
    a.download = `완주인증서_${currentName || '완주자'}.png`;
    a.href = canvas.toDataURL('image/png');
    a.click();
    btn.textContent = '✓ 저장 완료!';
    btn.style.background = '#00883A';
    setTimeout(restoreAll, 2000);
```

변경 후:
```js
  }).then(canvas => {
    const a = document.createElement('a');
    a.download = `완주인증서_${currentName || '완주자'}.png`;
    a.href = canvas.toDataURL('image/png');
    a.click();
    STAMPS.clear();
    btn.textContent = '✓ 저장 완료!';
    btn.style.background = '#00883A';
    setTimeout(restoreAll, 2000);
```

- [ ] **Step 3: 동작 확인**

다운로드 후 localStorage 확인:
```js
localStorage.getItem('greenWalkStamps'); // null 이어야 함
```

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: 인증서 다운로드 후 stamp localStorage 초기화"
```

---

### Task 9: 전체 흐름 통합 테스트

- [ ] **Step 1: 빈 상태에서 사이트 접속**

localStorage 비운 후 `index.html` 열기:
- 스탬프 화면 표시 확인
- 3개 원 모두 점선(미수집) 확인
- "완주 인증서 받기" 버튼 비활성 확인

- [ ] **Step 2: 순서대로 QR 시뮬레이션**

브라우저 주소창에 순서대로 입력:
1. `index.html?cp=1&token=Gw26cP1xK9m` → 스탬프 1 찍힘 확인
2. `index.html?cp=2&token=Gw26cP2nR4j` → 스탬프 2 찍힘 확인
3. `index.html?cp=3&token=Gw26cP3bT7p` → 스탬프 3 찍힘, 버튼 활성화 확인

- [ ] **Step 3: 완주 흐름 확인**

"완주 인증서 받기" → 이름 입력 → 인증서 발급 → 다운로드 → localStorage 비워지는지 확인

- [ ] **Step 4: 잘못된 순서 테스트**

localStorage 비운 후 `?cp=2&token=Gw26cP2nR4j` 접속 → "이전 체크포인트를 먼저 방문해주세요!" 메시지 확인

- [ ] **Step 5: 잘못된 토큰 테스트**

`?cp=1&token=wrongtoken` → 스탬프 안 찍히는지 확인

- [ ] **Step 6: 커밋 및 push**

```bash
git add index.html
git commit -m "feat: QR 스탬프 완주 인증 시스템 완성"
git push origin main
```

---

### Task 10: QR 코드 생성

- [ ] **Step 1: 실제 GitHub Pages URL 확인**

```bash
git remote get-url origin
```

URL 패턴: `https://waven0310-ux.github.io/green-walk-cert/`

- [ ] **Step 2: QR 코드 3개 생성**

아래 URL로 QR 코드를 생성한다 (QR 생성 사이트: qr-code-generator.com 등):

```
CP 1: https://waven0310-ux.github.io/green-walk-cert/?cp=1&token=Gw26cP1xK9m
CP 2: https://waven0310-ux.github.io/green-walk-cert/?cp=2&token=Gw26cP2nR4j
CP 3: https://waven0310-ux.github.io/green-walk-cert/?cp=3&token=Gw26cP3bT7p
```

- [ ] **Step 3: QR 코드 인쇄 및 현장 배치**

각 체크포인트에 해당 QR 코드를 인쇄해 부착한다.
