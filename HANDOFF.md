# HANDOFF — HI-log/play

> 2~4세 유아 동반 가족 가이드 정적 사이트. 다른 사람이 이어받아 유지·확장할 수 있도록 정리한 문서.
> 최종 업데이트: 2026-05-19

## 1. 프로젝트 한눈에

| 항목 | 값 |
|---|---|
| 배포 URL | https://hi-log.github.io/play/ |
| 호스팅 | GitHub Pages (main 브랜치 root, HTTPS 강제) |
| 백엔드 | **없음** (정적 HTML만) |
| 외부 API | Open-Meteo (날씨), Open-Meteo Air Quality (미세먼지) — 둘 다 키 불필요·무료 |
| CDN | Leaflet 1.9.4 (지도), OpenStreetMap 타일 |
| 빌드 시스템 | **없음** (파일 그대로 푸시) |
| 검색엔진 색인 | 차단됨 (robots.txt + meta robots) |

## 2. 파일 구조

```
play/
├── index.html              # 랜딩 (두 페이지로 진입)
├── resort/index.html       # 리조트 가이드 (~25곳)
├── experience/index.html   # 체험공간 가이드 (~162곳, 26테마)
├── robots.txt              # User-agent: * / Disallow: /
├── README.md
└── HANDOFF.md              # (이 문서)
```

URL 구조:
- `/` → 랜딩
- `/resort/` → 리조트
- `/experience/` → 체험공간

## 3. 기술 스택 / 의존성

순수 HTML/CSS/Vanilla JS. 외부 의존성:

| 용도 | 라이브러리/API | 비고 |
|---|---|---|
| 지도 | [Leaflet 1.9.4](https://leafletjs.com) + OpenStreetMap | unpkg CDN에서 로드 |
| 날씨 | [Open-Meteo Forecast](https://open-meteo.com/en/docs) | `current=temperature_2m,precipitation,weather_code` |
| 미세먼지 | [Open-Meteo Air Quality](https://open-meteo.com/en/docs/air-quality-api) | `current=pm2_5` |
| 위치 | 브라우저 Geolocation API | HTTPS 필수 (Pages는 HTTPS) |
| 상태 저장 | localStorage | 다크모드 toggle (페이지마다 별도 키) |

**API 키 필요한 거 없음.** 누가 fork해도 그대로 동작.

## 4. 데이터 모델 (experience)

`experience/index.html` 안의 `const DATA = [...]` (현재 약 162개). 한 항목 형식:

```js
{
  name: "장소명",
  icon: "🐑",                       // 카드에 표시되는 카테고리 이모지
  region: "metro",                  // gangwon | metro | other | jeju (4개 권역)
  regionLabel: "경기·용인",          // UI에 표시되는 권역 라벨
  type: "outdoor-farm",             // 테마 키 (아래 표 참조)
  location: "용인시 처인구 포곡읍",   // 카드 location, 좌표 매핑에도 쓰임
  experience: "체험 설명…",
  price: "성인 7,000원",
  tags: ["양", "산책로"],            // 검색·표시용

  // 옵션 필드
  free: true,        // 💸 무료/저렴 — 무료 배지 + free 필터
  indoor: true,      // 🏠 실내 — 실내 배지 + indoor 필터 + 우천 추천 후보
  isNew: true,       // ✨ 신상 — NEW 배지 + 펄스 애니메이션
  newDate: "2026-04-28",  // 신상 카드에 표시
  months: [4, 5],    // 시즌 자동 매핑 오버라이드 (없으면 type 기반 자동)
  startDate: "2026-04-01",  // 한정 이벤트 시작일 — 이전엔 "📅 다가옴" 배지
  endDate: "2026-06-01",    // 한정 이벤트 종료일 — 지나면 자동 숨김 (D-7 이내엔 "⏰ D-N 종료" 빨간 배지)
}
```

### 한정 이벤트(영화/기획전/축제 1회성) 처리

`endDate` 필드 하나만 넣으면 끝:
- `endDate < 오늘` → **카드 자동 숨김** (필터링 단계에서 빠짐, 모든 뷰·추천에 적용)
- 7일 이내 종료 → "⏰ D-N 종료" 빨간 배지로 강조
- 그 외 진행 중 → "~ 2026-06-01" 회색 배지
- `startDate > 오늘` → "📅 2026-06-15 시작" 노란 배지 (예고 노출)

예시: 베베핀 영화 같은 한정 상영
```js
{
  name: "베베핀 무비 콘서트",
  icon: "🎪",
  region: "metro", regionLabel: "서울·송파",
  type: "show",
  location: "롯데월드몰 시네마",
  experience: "...",
  price: "...",
  startDate: "2026-05-25",
  endDate: "2026-06-01",
}
```
2026-06-02부터는 페이지 새로고침만 해도 자동으로 안 보임.

### 4-1. 26개 테마 키

| 키 | 라벨 | 이모지 |
|---|---|---|
| outdoor-farm | 야외 목장 | 🐑 |
| indoor-zoo | 실내 동물원 | 🏠 |
| harvest | 수확/농장 | 🌾 |
| insect | 곤충생태 | 🦋 |
| aqua | 수족관·식물원 | 🐠 |
| mudflat | 갯벌 | 🌊 |
| museum | 어린이박물관 | 🏛️ |
| science | 과학관 | 🔬 |
| astro | 천문대 | 🌌 |
| forest | 숲·휴양림 | 🌳 |
| job | 직업체험 | 👷 |
| craft | 공예·전통 | 🎨 |
| cooking | 요리·베이킹 | 🍞 |
| transport | 교통박물관 | 🚂 |
| amusement | 테마파크·워터파크 | 🎢 |
| nature | 공원·전망대 | 🌷 |
| culture | 문화·테마관 | 🎭 |
| kidscafe | 키즈카페·실내놀이 | 🧸 |
| village | 농촌·어촌마을 | 🏘️ |
| cave | 동굴 | 🦇 |
| art | 미술관 | 🖼️ |
| camping | 캠핑·글램핑 | 🏕️ |
| temple | 사찰·템플스테이 | ⛪ |
| cruise | 유람선·해양 | 🚢 |
| winter | 겨울 액티비티 | ❄️ |
| show | 공연·인형극 | 🎪 |

## 5. 자주 하는 작업

### 5-1. 장소 1개 추가
`experience/index.html`의 적절한 카테고리 섹션 안에 위 형식의 객체 추가. JS 객체 한 줄짜리라 git diff 깔끔.

### 5-2. 새 카테고리(테마) 추가
1. `<div id="typeFilter">` 안에 `<span class="chip" data-type="새키">🆕 새이름</span>` 추가
2. `intro` 섹션에 한 줄 추가 (텍스트 안내)
3. `DATA`에 새 type 항목들 추가
4. 시즌 매핑 필요하면 `DEFAULT_MONTHS_BY_TYPE`에 키 추가
5. 카운트 표시는 자동

### 5-3. 신상(NEW) 표시 기준
- 최근 3개월 이내 개관 → `isNew: true, newDate: "YYYY-MM-DD"`
- 기간 지나면 두 필드만 삭제하면 됨
- 펄스 애니메이션·주황 테두리·"오픈 날짜 표기"가 자동 적용
- 옵션 필터 "✨ 신상만"으로 모아보기 가능

### 5-4. 시즌 캘린더 동작
1. 각 항목의 `months: [3, 4]`가 명시되면 그걸 사용
2. 없으면 `getMonths(d)` 함수가:
   - 키워드 매칭("벚꽃","튤립","워터파크" 등 — `experience` + `tags`에서 검색)
   - 그래도 없으면 `DEFAULT_MONTHS_BY_TYPE[d.type]`
   - 실내(`indoor:true`)는 사계절
   - 야외 기본은 3~11월

특정 장소의 시즌이 부정확하면 `months: [...]` 명시로 우선 적용.

### 5-5. 지도 좌표 매핑 동작
- 데이터에 좌표 필드 없음. JS의 `CITY_COORDS` 객체(약 100개 도시)에서 `location` + `regionLabel` 문자열로 키워드 매칭
- 매칭되면 그 도시 좌표 + name 해시 기반 deterministic jitter(±0.03°)로 마커 겹침 방지
- 매칭 안 되면 권역 fallback 좌표
- 새 도시 추가는 `CITY_COORDS` 객체에 한 줄 추가하면 됨

### 5-6. 오늘 추천 동작 (요약)
사용자가 위치 권한 허용 OR 도시 dropdown 선택 → `runRecommend(lat, lng, name)`:
1. `fetchWeather` — Open-Meteo 호출 (날씨 + 미세먼지)
2. `decideIndoorMode` — 임계값으로 실내/야외 모드 결정
   - 임계값 변경하려면 이 함수 안의 값 수정: 0℃, 32℃, PM2.5 50
   - 날씨 코드 분류는 `weatherInfo` 함수 (WMO 코드 기준)
3. 후보 = DATA → 현재 월 시즌 통과 + (실내 모드면 indoor만)
4. 거리 계산 (Haversine, `haversineKm`)
5. `pickWithMode`로 가까운 3 (야외 모드면 야외 우선 + 부족 시 실내 보충, 실내 모드면 다양성만) + 먼 2 (80km+, region 다양성)
   - 야외 모드인데 실내가 섞이면 경고 노트가 카드 위에 표시됨
6. `recoCardHTML`로 렌더링

## 6. 검색 차단

| 위치 | 내용 |
|---|---|
| `robots.txt` | `User-agent: * / Disallow: /` |
| 각 HTML `<head>` | `<meta name="robots" content="noindex, nofollow, noarchive, nosnippet, noimageindex">` 등 4개 |

GitHub repo 자체는 public이라 GitHub 검색에는 노출됨 — 페이지 내용은 가려져 있음.

## 7. 배포 흐름

```bash
git add -A
git commit -m "변경 설명"
git push
```

push 후 약 30초~1분 안에 GitHub Pages 빌드 완료. 빌드 상태:

```bash
gh api /repos/HI-log/play/pages/builds/latest --jq .status
# built / building / errored
```

문제 시 https://github.com/HI-log/play/actions 에서 빌드 로그.

## 8. 로컬에서 확인

```bash
cd /path/to/play
python3 -m http.server 8000
# http://localhost:8000 접속
```

`file://` 로 직접 열어도 대부분 동작하지만, **지도·오늘추천은 안 됨** (Geolocation·CORS는 HTTPS/localhost 필요).

## 9. 알려진 한계

- 좌표가 **도시 단위**라 같은 도시 안 마커는 jitter로 약간 흩어져 있을 뿐 실제 위치는 부정확.
- 시즌 매핑이 키워드 휴리스틱이라 부정확한 케이스 있음 → 해당 항목에 `months: [...]` 명시로 해결.
- localStorage 키는 페이지별로 분리됨 (`theme-experience`, `theme-hilog` 등). 페이지 통합 시 키 정리 필요.
- Leaflet은 unpkg CDN 의존. CDN 장애 시 지도 안 뜸. 안정성 원하면 파일 직접 호스팅.

## 10. 다음 개선 후보 (브레인스토밍 결과)

우선순위 높은 순:

1. **즐겨찾기 / 방문 체크** (Quick Win) — localStorage 토글, ⭐/✓ 버튼
2. **URL 공유** — 필터 상태를 query string으로 → 카톡 공유
3. **카카오맵 길찾기 외부 링크** — 카드에 📍 버튼
4. **거리순 정렬** — Geolocation으로 전체 리스트 거리 정렬 (현재는 추천 탭만)
5. **하루 코스 빌더** — 2~3곳 선택 → 지도에 순서 배치
6. **큐레이션 컬렉션** — "비 오는 날 BEST", "어린이날 코스" 미리 짠 묶음
7. **PWA화** — manifest.json + Service Worker, 홈 화면 설치
8. **신뢰도·최신성 라벨** — "마지막 확인: 2026-05" 표시

## 11. 외부 API 레퍼런스

- Open-Meteo Forecast: https://api.open-meteo.com/v1/forecast?latitude={lat}&longitude={lng}&current=temperature_2m,precipitation,weather_code&timezone=Asia%2FSeoul
- Open-Meteo Air Quality: https://air-quality-api.open-meteo.com/v1/air-quality?latitude={lat}&longitude={lng}&current=pm2_5
- Leaflet docs: https://leafletjs.com/reference.html
- WMO Weather Codes (날씨 코드 표): https://open-meteo.com/en/docs (페이지 하단)

## 12. 폐기 / 대체 가능 결정

| 의존 | 대체 옵션 |
|---|---|
| Open-Meteo | OpenWeatherMap (키 필요), 기상청 단기예보(인증 필요) |
| OpenStreetMap 타일 | Kakao Map / Naver Map (정적 사이트에서 키 노출 감수) |
| Leaflet | Kakao Map JS SDK / MapLibre |
| GitHub Pages | Cloudflare Pages / Vercel (private repo 무료) |

## 13. 연락처

- 원작자: wonism
- repo: https://github.com/HI-log/play

문서 끝.
