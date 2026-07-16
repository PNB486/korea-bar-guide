# 회기·석계 술집 가이드 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 회기역·석계역 술집 각 20곳(총 40)을 네이버 플레이스 데이터 기반 단일 HTML 가이드로 제작.

**Architecture:** japan-food-guide의 `suntory-tatsujin-guide.html`(3285줄, 인라인 CSS/JS + DATA 배열)을 복제해 국내 스키마로 개조. 리서치는 병렬 웹서치 에이전트 2기(지역당 1기)가 수행하고 결과를 wiki에 백업한 뒤 DATA에 반영.

**Tech Stack:** 정적 HTML/CSS/JS 단일 파일, node(검증), Playwright(UI 검증, `music-dashboard/node_modules/playwright` + Edge executablePath).

## Global Constraints

- 평점 소스는 네이버 플레이스만: `visitorReviews`(방문자리뷰 수), `blogReviews`(블로그리뷰 수), `naverUrl` 딥링크. 별점 없음.
- 바·와인바 제외. 폐업 확인 필수.
- 카테고리(bucket)는 고정 세트 없음 — 리서치 결과 분포로 확정.
- photo URL은 HTTP 200 실검증 통과분만.
- 배포 없음: 로컬 git 커밋만. push/공개 금지.
- japan판의 산토리 tier / 타베로그 / 미슐랭 / STATION_MAP·ACCESS_MAP 번역 로직 전부 삭제.
- 스키마(1곳당): `{area:'회기'|'석계', name, bucket, genre, address, access, budget, closed?, visitorReviews, blogReviews, naverUrl, menu:[3], photo, card?, reserve?}`

---

### Task 1: 리서치 에이전트 2기 병렬 발사 + wiki 백업

**Files:**
- Create: `C:\Users\Soul\wiki\docs\topics\2026-07-16-hoegi-bar-guide-20-data.md` (에이전트 결과 백업)
- Create: `C:\Users\Soul\wiki\docs\topics\2026-07-16-seokgye-bar-guide-20-data.md`

**Interfaces:**
- Produces: 지역당 20곳 × 스키마 전 필드 + 지역 카테고리 분포 요약. Task 3이 이 wiki 파일을 파싱해 DATA 생성.

- [ ] **Step 1: 에이전트 2기 동시 발사** (general-purpose, 병렬, run_in_background)

프롬프트 골자(지역명만 치환):

```
<지역>역(서울) 도보 10분 내 술집 20곳 조사. 목적: 로컬 가이드 데이터.
제외: 바/와인바/칵테일바, 폐업·휴업 의심 업소, 프랜차이즈 위주 지양(로컬 우선).
포함 업종: 고기구이/횟집·해산물/전·막걸리/포차/이자카야/호프·치킨/곱창·막창 등 술+안주.
선별: 네이버 방문자리뷰 수 상위 + 블로그 언급 교차. 지역 실제 분포 반영(억지로 업종 균형 맞추지 말 것).
곳당 수집 필드:
- name(정확한 상호), genre(세부업종), address(도로명), access(<지역>역 N번출구 도보 M분)
- budget(1인 안주+술 대략 ₩표기), closed(정기휴무, 없으면 '없음')
- visitorReviews(네이버 방문자리뷰 수), blogReviews(네이버 블로그리뷰 수) — m.place.naver.com 또는 검색결과에서 실측
- naverUrl(네이버플레이스 개별 페이지 URL), menu(대표 안주 3개)
- photo(음식/외관 사진 URL — HTTP 200 확인, 배너/이벤트 이미지 금지, pstatic.net 계열 우선)
- card(카드결제, 확인될 때만), reserve(예약 가능/불가/필수, 확인될 때만)
출력: 파이프(|) 구분 표 + 하단에 업종 분포 카운트 요약.
네이버 차단 시 insane-search 스킬 패턴(모바일 URL, curl_cffi) 활용 가능.
```

- [ ] **Step 2: 결과 수신 후 wiki 2파일에 원문 그대로 백업**

- [ ] **Step 3: 검수** — 20곳 미달/중복/바 포함 여부 확인. 미달분은 보충 에이전트 1기 추가(삿포로 세션 패턴).

### Task 2: HTML 국내화 골격 (샘플 2곳으로 렌더 확인)

**Files:**
- Create: `C:\Users\Soul\Desktop\AI\Claude\korea-bar-guide\korea-bar-guide.html` (japan HTML 복제 후 개조)

**Interfaces:**
- Consumes: `Desktop/AI/Claude/japan-food-guide/suntory-tatsujin-guide.html` (베이스)
- Produces: `DATA` 배열(신 스키마), `card()`/`magazineCard()` 렌더러, 지역 탭(#cityChips 재사용, 회기/석계), 카테고리 칩(BUCKETS는 Task 3에서 확정값으로 교체).

- [ ] **Step 1: 복제 + 삭제** — 타베로그/산토리/미슐랭/마스터즈드림 배지·필터·레전드, STATION_MAP/ACCESS_MAP/FALLBACK_MAP/translateAccess, formatBudget의 円 로직, tabelogSearchUrl 제거. DATA는 샘플 2곳(회기1/석계1, 가짜 아님 — 리서치 결과 중 선발)만.
- [ ] **Step 2: 스키마 렌더 교체** — 점수줄(.c-score-row)을 `방문자리뷰 N · 블로그 M`으로, 업체명 링크→naverUrl, 주소·교통→구글맵 링크 유지. 타이틀 "위대한 AI님의 회기·석계 술집 가이드", 파비콘 🍶.
- [ ] **Step 3: `node --check`로 스크립트 문법 통과 + 브라우저 열어 2카드 렌더 확인.**
- [ ] **Step 4: Commit** `feat: 국내화 골격 (샘플 2곳)`

### Task 3: 카테고리 확정 + DATA 40곳 반영 + 검증

**Files:**
- Modify: `korea-bar-guide.html` (DATA 배열, BUCKETS)

- [ ] **Step 1: wiki 2파일 분포 합산 → bucket 세트 확정**(바 제외). BUCKETS 배열·필터칩 반영.
- [ ] **Step 2: wiki → JS 객체 변환, DATA 삽입.** join은 반드시 `',\n'`(삿포로 콤마누락 교훈). 삽입은 line-based(whole-rewrite 금지).
- [ ] **Step 3: node 검증** — `node --check`, DATA.length===40, 지역당 20, bucket 값이 BUCKETS에 전부 존재(코드포인트 일치), 중복명 0, 중복photo 0, 필수필드 누락 0.
- [ ] **Step 4: photo 전건 HTTP 상태 스캔** — 200 아닌 것 재조사 교체.
- [ ] **Step 5: Commit** `feat: 회기·석계 40곳 데이터 반영`

### Task 4: Playwright UI 검증 + 마무리 커밋

- [ ] **Step 1: 데스크톱(1280)/모바일(390) 스크린샷 + 지역탭 전환·카테고리칩·테마토글·모바일 타일펼침 클릭 검증** (탭 클릭 불안정 시 element.click() JS 디스패치 우회 — 세션13 패턴)
- [ ] **Step 2: 스크린샷 사용자 제시, 이슈 수정 후 Commit** `feat: UI 검증 완료`
