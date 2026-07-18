# 별내 지역 확장 설계 (술집 + 밥집)

작성일: 2026-07-18

## 목적

기존 "회기·석계 술집 가이드"에 세 번째 지역 **별내**(남양주 별내신도시, 별내역)를 추가한다.
별내는 기존 두 지역과 달리 **술집 + 밥집 혼합**으로 수록한다.

## 범위 결정 (확정)

- **컨셉**: 별내만 술집+밥집 혼합. 회기·석계는 술집 전용 유지.
- **제목**: `회기·석계 술집 가이드` → `회기·석계·별내 맛집 가이드` (술집 → 맛집).
- **별내 구성**: 20곳 = 술집 10 + 밥집 10.
- **필터 UX**: 업종칩을 **도시별 동적**으로 변경 — 선택한 도시에 실제 존재하는 업종만 노출.

## 데이터 모델

기존 스키마 유지, `city` 값만 확장:

```
{city:'별내', name, bucket, genre, address, access, budget, closed?,
 visitorReviews, blogReviews, naverUrl, menu:[문자열3], photo, extraPhotos:[문자열3], card?, reserve?}
```

- `city:'별내'` 20건 추가.
- **술집 10**: 기존 `BUCKETS` 재사용 (고기구이/이자카야/요리주점/곱창·막창/횟집·해산물/전·막걸리/호프·치킨).
- **밥집 10**: 신규 `MEAL_BUCKETS`. 후보 — `백반·한식`, `국밥·탕`, `칼국수·면`, `돈까스·경양식`, `분식`, `중식`, `카페·브런치`. **실제 데이터 분포에 따라 최종 확정** (사용 안 하는 후보는 배열에서 제외).
- `type` 필드는 추가하지 않는다. 술/밥 구분은 bucket이 겸한다 (YAGNI).

## 코드 변경 (index.html + korea-bar-guide.html 동일)

두 파일은 사본 관계 → 동일하게 반영.

1. **CITY_ORDER** (L1531): `['회기','석계']` → `['회기','석계','별내']`
2. **CITY_EN** (L1532): `별내:'BYEOLLAE'` 추가
3. **BUCKETS / MEAL_BUCKETS** (L1533):
   - `BUCKETS` (술집) 유지.
   - `const MEAL_BUCKETS = [...]` 신규 (밥집, 실데이터 확정).
   - `deriveBucket`은 `BUCKETS.concat(MEAL_BUCKETS)` 기준으로 판정.
4. **buildChips 업종칩** (L1754~1766): 고정 `['전체', ...BUCKETS]` → **activeCity 기준 동적 파생**.
   - activeCity !== '전체': 해당 도시 DATA에 존재하는 bucket만.
   - activeCity === '전체': 전 도시 bucket 합집합.
   - 순서: 술집 buckets(BUCKETS 순) → 밥집 buckets(MEAL_BUCKETS 순).
   - 도시 전환 시 업종칩 재빌드 필요 → 도시 onclick에서 업종칩 다시 그리기 + activeBucket 유효성 리셋(현 도시에 없으면 '전체'로).
5. **하드코딩 문자열 갱신**:
   - title `<title>` (L6), hero-title (L1415), `document.title` 템플릿 (L2121): "회기·석계·별내 맛집 가이드".
   - heroCount (L1414): "3개 역세권 총 61곳".
   - map h2 (L1424): "3개 역세권 총 61곳".
   - map-city-row 별내 추가 (L1432 뒤): `data-city="별내"`, `별내 / BYEOLLAE · 신도시 상권`, `20곳`.
   - legend (L1469) / footer (L1478): 별내는 밥집 포함되므로 "분식 제외" 문구를 지역 한정으로 조정 (회기·석계 한정 제외, 별내는 밥집 수록).
   - 리서치 기준일 (L1479): 2026-07-18 병기.

## 데이터 수집 (최대 작업량)

- 대상: 별내신도시 / 별내역(4호선·경춘선·별내선) 도보권 네이버 플레이스.
- 술집 10 = 방문자리뷰 상위 + 블로그 교차. 밥집 10 = 방문자리뷰 상위 + 블로그 교차.
- 곳당 필드: name / bucket / genre / address / access(별내역 도보 N분) / budget / closed / visitorReviews / blogReviews / naverUrl / menu[3] / photo / extraPhotos[3] / card / reserve.
- 네이버 WAF 차단 시 insane-search 사용.
- **실측치만**. 리뷰수·가격 추정/하드코딩 금지. 사진 URL은 네이버 플레이스 등록본만.
- DATA 삽입은 line-based(한 곳 = 한 줄), single-quote. JSON.stringify·whole-rewrite 금지.

## 검증

- 브라우저에서 도시탭 별내 선택 → 밥집+술집 칩 노출, 필터 동작.
- 회기/석계 선택 시 밥집 칩 안 뜸.
- 카운트/제목 61곳 반영.
- 사진 로드·갤러리·네이버/구글맵 링크 정상.
- 비ASCII(한글) 매장명 깨짐 없는지 확인.

## 비목표

- 회기·석계에 밥집 추가 (이번 아님).
- 지도 실지오/좌표 연동.
- 데이터 자동 스크레이핑 파이프라인.
