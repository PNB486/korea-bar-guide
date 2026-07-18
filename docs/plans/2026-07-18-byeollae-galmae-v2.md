# 별내 맛집 가이드 v2 (갈매 확장) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 별내 근처 맛집 가이드를 별내 100 + 갈매 50 = 150곳(회기·석계 삭제)으로 재편하고, 지역 네비를 경량 SVG 2구역맵으로 바꾸고, "오늘의 픽" 랜덤 버튼을 지도밴드에 추가한다.

**Architecture:** 단일 `index.html`(= `korea-bar-guide.html` 사본) 안의 `DATA` 배열과 바닐라 JS를 수정한다. 외부 의존성·빌드·테스트 프레임워크 없음. 검증은 grep 카운트 + 브라우저 육안 확인.

**Tech Stack:** 순수 HTML/CSS/JS, dep 0. 갈매 데이터는 `insane-search`로 네이버 플레이스 실측.

## Global Constraints

- 단일 파일, 외부 의존성 0. CDN/라이브러리 추가 금지.
- `index.html`과 `korea-bar-guide.html`은 **동일 사본** — 모든 변경 후 index→korea 복사로 동기화.
- DATA 삽입은 **라인 기반 single-quote**. `JSON.stringify` 금지, 파일 whole-rewrite 금지 (메모리 교훈).
- 사진 URL은 실제 로드되는 `pstatic.net` 직링크만. 깨진 URL 금지.
- 리뷰수·평점은 리서치 스냅샷 — 기준일 명기(갈매 2026-07-18).
- 타이틀/히어로 "위대한 AI님의 별내 근처 맛집 가이드" 문구 유지('근처' 유지).
- 커밋 자주. 각 Task 끝에 커밋.

## 파일 구조

- Modify: `index.html` — DATA 배열, CITY_ORDER, 지도밴드 마크업, initMapNav, 오늘의픽 JS/CSS, 범례/푸터 텍스트
- Sync: `korea-bar-guide.html` — index.html 사본(각 Task 후 갱신)

기준 위치(현 파일):
- 스키마 주석: `index.html:1492-1493`
- `const DATA = [` : `index.html:1494`
- `const CITY_ORDER` : `index.html:1637`
- 지도밴드 `<section class="map-band">` : `index.html:1422-1442`
- 범례 legend : `index.html:1474-1476`
- 푸터 : `index.html:1482-1487`
- `pickCity()` : `index.html:2215`
- `initMapNav()` : `index.html:2223-2240`

---

### Task 1: 회기·석계 데이터 삭제 + CITY_ORDER 교체

**Files:**
- Modify: `index.html` (DATA 배열, CITY_ORDER, 스키마 주석)

**Interfaces:**
- Produces: `CITY_ORDER = ['별내','갈매']`, DATA에 `city:'별내'` 100곳만 남음(갈매는 Task 3에서 추가).

- [ ] **Step 1: 회기·석계 DATA 라인 삭제**

`city:'회기'`(21줄) + `city:'석계'`(20줄) 항목을 DATA 배열에서 전부 제거. 각 항목은 정확히 한 줄. `grep -n "city:'회기'\|city:'석계'" index.html`로 줄번호 뽑아 삭제. 배열 콤마·대괄호 문법 유지(마지막 항목 뒤 trailing comma는 JS 허용이라 무방).

- [ ] **Step 2: CITY_ORDER 교체**

`index.html:1637`
```js
const CITY_ORDER = ['별내','갈매'];
```
(기존 `['별내','회기','석계']` 대체)

- [ ] **Step 3: 스키마 주석 갱신**

`index.html:1492`
```js
// 스키마: {city:'별내'|'갈매', name, bucket, genre, address, access, budget, closed?,
//          score?, visitorReviews, blogReviews, naverUrl, menu:[문자열3], photo, extraPhotos:[3], card?, reserve?}
```

- [ ] **Step 4: 검증 — 카운트**

Run: `grep -c "city:'별내'" index.html && grep -c "city:'회기'\|city:'석계'" index.html`
Expected: `100` 그리고 `0`

- [ ] **Step 5: korea-bar-guide.html 동기화 + 커밋**

```bash
cp index.html korea-bar-guide.html
git add index.html korea-bar-guide.html
git commit -m "refactor: 회기·석계 삭제, CITY_ORDER 별내·갈매로 교체"
```

---

### Task 2: 범례·푸터·텍스트 정리

**Files:**
- Modify: `index.html` (legend, 푸터 문단)

- [ ] **Step 1: 범례(legend) 교체**

`index.html:1475` 문구를 회기·석계 언급 제거 후 교체:
```html
<span>네이버 플레이스 방문자리뷰·블로그리뷰 수 기준 선별 · 별내·갈매 신도시 상권의 밥집+술집</span>
```

- [ ] **Step 2: 푸터 선별기준 문단 교체**

`index.html:1484` 문단을 교체:
```html
<p>선별 기준: 네이버 방문자리뷰 수 상위 + 블로그 언급 교차 검증. 별내·갈매 두 신도시 상권의 밥집과 술집을 함께 실었습니다.</p>
```

- [ ] **Step 3: 리서치 기준일 문단 교체**

`index.html:1485` 문단을 교체:
```html
<p>리서치 기준일 2026-07-18(별내), 2026-07-18(갈매). 영업시간·정기휴무·가격은 방문 전 재확인 권장.</p>
```

- [ ] **Step 4: 검증 — 잔여 문구 없음**

Run: `grep -c "회기\|석계" index.html`
Expected: `0`

- [ ] **Step 5: 동기화 + 커밋**

```bash
cp index.html korea-bar-guide.html
git add index.html korea-bar-guide.html
git commit -m "docs: 범례·푸터 별내·갈매 기준으로 정리"
```

---

### Task 3: 갈매 50곳 실데이터 리서치 + 삽입

이 Task가 최대 작업. 10곳 단위 5배치로 나눠 진행하고 배치마다 커밋.

**Files:**
- Modify: `index.html` (DATA 배열에 `city:'갈매'` 50줄 추가)

**Interfaces:**
- Consumes: Task 1의 DATA 구조(별내 100곳 뒤에 이어붙임).
- Produces: DATA 총 150곳(별내 100 + 갈매 50).

- [ ] **Step 1: 갈매 후보 매장 수집 (배치당 10곳)**

`insane-search`로 네이버 플레이스에서 "갈매동 맛집", "갈매역 맛집", "갈매지구 술집" 등 검색. 방문자리뷰 상위 + 블로그 교차. bucket 분포는 별내 감각 따라 밥집~35 / 술집~15 목표. 중복 매장·폐업 제외.

- [ ] **Step 2: 매장별 필드 실측**

각 매장 네이버 플레이스 페이지에서 추출:
- name, address, access(가까운 역/버스 기준 도보 분), budget
- visitorReviews(방문자리뷰수), blogReviews(블로그리뷰수), score(방문자평점, 있으면)
- naverUrl(`https://m.place.naver.com/restaurant/<id>/home`)
- menu 3개(가격 포함 문자열), closed(정기휴무), reserve, card
- photo(대표사진 pstatic URL) + extraPhotos 3장(리뷰/업체 사진 pstatic URL)
- bucket은 기존 카테고리 중 매칭(고기구이/면·국수/이자카야/파스타·양식/…), 없으면 신규(자동 반영됨), genre는 세부 표기.

- [ ] **Step 3: DATA 라인 작성·삽입 (배치 10곳)**

별내 마지막 항목 다음 줄에 삽입. 한 매장 = 한 줄, single-quote, 기존 항목과 동일 필드 순서:
```js
{city:'갈매', name:'...', bucket:'...', genre:'...', address:'...', access:'...', budget:'₩...', closed:'...', score:0.00, visitorReviews:000, blogReviews:00, naverUrl:'https://m.place.naver.com/restaurant/000/home', menu:['...','...','...'], photo:'https://...pstatic.net/...', extraPhotos:['https://...','https://...','https://...'], card:true, reserve:'가능'},
```
라인 기반 삽입만. 파일 전체 재작성 금지.

- [ ] **Step 4: 배치 사진 URL 로드 확인**

작성한 배치의 photo/extraPhotos URL이 실제 200 응답인지 확인(`insane-search` 또는 curl로 헤더 체크). 깨진 URL은 해당 매장 다른 사진으로 교체.

- [ ] **Step 5: 배치 커밋 후 Step 1~4 반복 (총 5배치 = 50곳)**

```bash
cp index.html korea-bar-guide.html
git add index.html korea-bar-guide.html
git commit -m "feat: 갈매 맛집 데이터 추가 (배치 N/5, 누적 NN곳)"
```

- [ ] **Step 6: 최종 검증 — 총 150곳**

Run: `grep -c "city:'별내'" index.html && grep -c "city:'갈매'" index.html`
Expected: `100` 그리고 `50`

---

### Task 4: 지도밴드 → 경량 SVG 2구역맵

**Files:**
- Modify: `index.html` (`<section class="map-band">` 마크업, `initMapNav()`, 관련 CSS)

**Interfaces:**
- Consumes: 기존 `pickCity(city)` 함수(그대로 재사용).
- Produces: SVG 존 클릭 → `pickCity` 호출로 도시 필터.

- [ ] **Step 1: map-city-row 3개 → 인라인 SVG 2존 교체**

`index.html:1427-1438`의 `map-city-row` 3개 블록을 SVG로 교체. 별내(좌/상)·갈매(우/하) 2존, 각 존에 `data-city`, 한글 라벨+로마자+곳수 `<text>`. 예:
```html
<svg class="area-map" viewBox="0 0 400 220" role="img" aria-label="별내·갈매 지역 지도">
  <g class="zone" data-city="별내" tabindex="0" role="button" aria-label="별내 선택">
    <path d="M8 8 H240 V150 H8 Z" class="zone-shape"/>
    <text x="124" y="72" class="zone-ko">별내</text>
    <text x="124" y="94" class="zone-jp">BYEOLLAE · 신도시</text>
    <text x="124" y="120" class="zone-cnt" data-city="별내">100곳</text>
  </g>
  <g class="zone" data-city="갈매" tabindex="0" role="button" aria-label="갈매 선택">
    <path d="M248 60 H392 V212 H160 V158 H248 Z" class="zone-shape"/>
    <text x="300" y="130" class="zone-ko">갈매</text>
    <text x="300" y="152" class="zone-jp">GALMAE · 신도시</text>
    <text x="300" y="178" class="zone-cnt" data-city="갈매">50곳</text>
  </g>
</svg>
```
`<h2>` 총계 라인(`index.html:1426`)은 유지(코드가 동적 갱신).

- [ ] **Step 2: SVG CSS 추가**

map-band 관련 CSS 근처에 존 스타일 추가(라이트/다크 대비, hover/포커스 강조, `.area-map{max-width:100%;height:auto}`, `.zone{cursor:pointer}`, `.zone-shape` 채움·테두리, `.zone-ko/.zone-jp/.zone-cnt` text-anchor:middle 폰트). accent 변수 재사용.

- [ ] **Step 3: initMapNav() 리스너 대상 교체**

`index.html:2224-2226`의 `.map-city-row` 셀렉터를 SVG 존으로 교체:
```js
document.querySelectorAll('.area-map .zone').forEach(el => {
  el.addEventListener('click', () => pickCity(el.dataset.city));
  el.addEventListener('keydown', e => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); pickCity(el.dataset.city); } });
});
```

- [ ] **Step 4: 곳수 동적 반영 대상 교체**

`index.html:2230-2234`의 곳수 갱신 로직을 SVG `.zone-cnt[data-city]` 대상으로:
```js
const counts = {};
DATA.forEach(d => { counts[d.city] = (counts[d.city] || 0) + 1; });
document.querySelectorAll('.zone-cnt[data-city]').forEach(el => {
  const c = counts[el.dataset.city];
  if (c != null) el.textContent = c + '곳';
});
```

- [ ] **Step 5: 검증 — 브라우저**

`index.html`을 브라우저로 열어: SVG 2존 렌더, 별내/갈매 곳수 100/50 자동표기, 각 존 클릭 → 해당 도시 필터+스크롤, 키보드 포커스+Enter 동작, 다크/라이트 대비.

- [ ] **Step 6: 동기화 + 커밋**

```bash
cp index.html korea-bar-guide.html
git add index.html korea-bar-guide.html
git commit -m "feat: 지역 네비를 경량 SVG 2구역맵으로 교체"
```

---

### Task 5: 오늘의 픽 랜덤 버튼 (지도밴드)

**Files:**
- Modify: `index.html` (map-band 내 버튼+패널 마크업, JS 함수, CSS)

**Interfaces:**
- Consumes: 전역 `DATA`(150곳), 카드 렌더에 쓰는 기존 헬퍼(`naverPlaceUrl(item)` 등).
- Produces: `rollTodaysPick()` — DATA에서 3곳 무중복 랜덤 추출 후 패널 렌더.

- [ ] **Step 1: 버튼 + 패널 컨테이너 마크업**

map-band `.map-inner` 안(SVG 아래)에 추가:
```html
<button class="pick-btn" id="pickBtn" type="button">🎲 오늘의 픽 3곳</button>
<div class="pick-panel" id="pickPanel" hidden>
  <div class="pick-cards" id="pickCards"></div>
  <button class="pick-reroll" id="pickReroll" type="button">다시 뽑기 ↻</button>
</div>
```

- [ ] **Step 2: CSS 추가**

`.pick-btn`(accent 버튼), `.pick-panel`(등장 애니 subtle), `.pick-cards`(그리드 3열, 모바일 1열), 미니카드(사진 썸네일·이름·지역·업종·평점) 스타일. 기존 카드 톤 재사용.

- [ ] **Step 3: rollTodaysPick() 구현**

```js
function rollTodaysPick() {
  const pool = DATA.slice();
  for (let i = pool.length - 1; i > 0; i--) { const j = Math.floor(Math.random() * (i + 1)); [pool[i], pool[j]] = [pool[j], pool[i]]; }
  const picks = pool.slice(0, 3);
  const wrap = document.getElementById('pickCards');
  wrap.innerHTML = picks.map(item => `
    <a class="pick-card" href="${naverPlaceUrl(item)}" target="_blank" rel="noopener">
      <span class="pc-photo" style="background-image:url('${item.photo}')"></span>
      <span class="pc-body">
        <span class="pc-name">${item.name}</span>
        <span class="pc-meta">${item.city} · ${item.bucket}${item.score ? ` · ★ ${item.score.toFixed(2)}` : ''}</span>
      </span>
    </a>`).join('');
  document.getElementById('pickPanel').hidden = false;
}
```
(`naverPlaceUrl`가 없으면 `item.naverUrl` 직접 사용)

- [ ] **Step 4: 리스너 연결**

`initMapNav()` 끝 또는 초기화부에 추가:
```js
document.getElementById('pickBtn')?.addEventListener('click', rollTodaysPick);
document.getElementById('pickReroll')?.addEventListener('click', rollTodaysPick);
```

- [ ] **Step 5: 검증 — 브라우저**

버튼 클릭 → 3곳 카드 표시(별내·갈매 섞여 나옴), 다시뽑기 → 다른 3곳, 카드 클릭 → 네이버 새탭, 사진 로드, 모바일 1열, 다크/라이트.

- [ ] **Step 6: 동기화 + 커밋**

```bash
cp index.html korea-bar-guide.html
git add index.html korea-bar-guide.html
git commit -m "feat: 오늘의 픽 랜덤 3곳 버튼 추가 (지도밴드)"
```

---

### Task 6: 최종 통합 검증

**Files:** 없음(확인만)

- [ ] **Step 1: 카운트·문구 재확인**

Run: `grep -c "city:'별내'" index.html; grep -c "city:'갈매'" index.html; grep -c "회기\|석계" index.html`
Expected: `100`, `50`, `0`

- [ ] **Step 2: 브라우저 전체 흐름**

- 도시탭: 전체/별내/갈매만, 총 150곳
- 업종칩·리뷰수필터·정렬 동작
- SVG 존 클릭 → 필터
- 오늘의 픽 3곳 + 다시뽑기
- 갈매 카드 사진 전수 로드(깨진 사진 0 — 콘솔 네트워크 확인)
- 다크/라이트, 모바일 필터 토글, 사진 갤러리 스와이프

- [ ] **Step 3: index ↔ korea 사본 일치 확인**

Run: `diff index.html korea-bar-guide.html && echo SAME`
Expected: `SAME`

- [ ] **Step 4: 공개 반영은 지시 시에만**

git push / GitHub Pages 반영은 사용자가 명시적으로 지시할 때만. 여기서는 로컬 커밋까지.

---

## Self-Review

- **Spec coverage:** 회기·석계 삭제(T1), 갈매 50 실데이터(T3), CITY_ORDER(T1), SVG 2구역맵(T4), 오늘의 픽 전체 150 랜덤 3곳 지도밴드(T5), 텍스트 정리(T2), 타이틀 유지(변경 안 함), 검증(T6) — 전 항목 매핑됨.
- **Placeholder scan:** 갈매 실데이터 값(리뷰수/URL/사진)은 리서치 산출물이라 T3에서 실측 — 플랜상 의도된 런타임 수집, 플레이스홀더 아님. 그 외 TBD 없음.
- **Type consistency:** `pickCity(city)`·`naverPlaceUrl(item)`·`item.score`·`DATA`·`CITY_ORDER`·`data-city`·`.zone-cnt` 명칭 태스크 간 일치.
