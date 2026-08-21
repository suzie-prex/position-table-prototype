# KOL Insight — Prototype Handoff

> 프로토타입에 실측 데이터를 입히기 위한 가이드

---

## 1. 프로토타입 접근

- **GitHub Pages**: https://suzie-prex.github.io/position-table-prototype/kol-insight/
- **로컬**: `kol-insight/index.html`을 브라우저에서 직접 열기 (빌드 불필요)
- **구조**: 단일 HTML 파일 (CSS + JS 인라인)

---

## 2. 화면 플로우

```
Home (KOL Insight 캐러셀)
  ├── 카드 탭 → Detail (이슈 상세)
  ├── More 카드 탭 → List (전체 리스트)
  └── See all → List

List (이슈 리스트)
  └── 카드 탭 → Detail

Detail (이슈 상세)
  ├── Back → 진입 화면으로 복귀
  ├── See other issues → List
  └── 심볼 행 탭 → Orderform
```

---

## 3. 데이터 위치

모든 이슈 데이터는 **`KOL_ISSUES` 배열** (index.html 약 1188행)에 있다.
이 배열 값을 수정하면 Home / List / Detail 화면에 바로 반영된다.

```js
var KOL_ISSUES = [
  { ... },  // Issue 1 (멘션 수 1위)
  { ... },  // Issue 2
  ...
];
```

심볼 메타 정보(아이콘 색상, 풀네임)는 **`SYM_META` 객체** (약 1196행)에 있다.

```js
var SYM_META = {
  BZ:  { name: 'Brent',   color: '#8B6914' },
  XAU: { name: 'Gold',    color: '#C9A834' },
  BTC: { name: 'Bitcoin', color: '#F7931A' },
  ...
};
```

---

## 4. 이슈 데이터 필드

| 필드 | 타입 | 설명 | UI 반영 위치 |
|------|------|------|-------------|
| `title` | string | 이슈 제목 (리스트용) | Home 카드, List 카드 |
| `dtitle` | string | 이슈 제목 (디테일용, 줄바꿈 허용) | Detail 타이틀 |
| `hot` | boolean | 화제성 표시 여부 | 🔥 badge-kol-hot |
| `arrow` | `'up'` \| `'down'` \| `'flat'` | 센티먼트 방향 | badge-kol-sentiment |
| `tone` | string | 톤 라벨 (표시용) | Detail badge row |
| `tclass` | `'bl'` \| `'br'` \| `'nu'` | 톤 분류 (bl=Bullish, br=Bearish, nu=Mixed) | 뱃지 색상 결정 |
| `mentions` | number | 총 멘션 수 | 카드 meta, Detail badge |
| `sources` | number | 원문 소스 수 | Detail "N sources" |
| `editorial` | string | AI 요약 텍스트 | Detail 인용 블록 |
| `vol` | `'Expanding'` \| `'Stable'` | 화제성 추세 | Detail "Buzz growing" / "Buzz steady" |
| `tags` | string[] | 관련 태그 | (현재 미사용, 확장용) |
| `related` | array | 관련 심볼 목록 | Detail "Mentions and movement" |

### `related[]` 항목 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `sym` | string | 심볼 티커 (SYM_META 키와 일치해야 함) |
| `m` | number | 해당 심볼 멘션 수 |
| `tone` | `'bl'` \| `'br'` \| `'nu'` | 심볼별 톤 |
| `tl` | string | 톤 라벨 (표시용) |
| `w1` | string | 7일 가격 변동률 (예: `'+3.80%'`) |

---

## 5. 데이터 교체 방법

### 이슈 추가/수정

`KOL_ISSUES` 배열에 객체를 추가하거나 기존 값을 수정한다.
배열 순서 = 멘션 수 내림차순 (1번째가 Issue 1).

```js
var KOL_ISSUES = [
  {
    title: '리스트에 표시될 제목',
    dtitle: '디테일에 표시될 제목',
    hot: true,
    arrow: 'up',
    tone: 'Bullish-leaning',
    tclass: 'bl',
    mentions: 500,
    sources: 4,
    vol: 'Expanding',
    editorial: 'AI 요약 텍스트를 여기에 입력',
    tags: [],
    related: [
      { sym: 'BTC', m: 300, tone: 'bl', tl: 'Bullish-leaning', w1: '+5.20%' },
      { sym: 'ETH', m: 200, tone: 'nu', tl: 'Mixed', w1: '-1.10%' }
    ]
  },
  // ... 추가 이슈
];
```

### 새 심볼 추가

`related`에서 새 심볼을 사용하려면 `SYM_META`에도 추가해야 한다.

```js
var SYM_META = {
  ...
  TSLA: { name: 'Tesla', color: '#CC0000' },
};
```

`name`은 Detail 심볼 sublabel에 표시되고, `color`는 아이콘 배경색.

---

## 6. 상태 확인 (토글)

화면 우하단 토글로 특수 상태를 확인할 수 있다.

| 화면 | 토글 | 동작 |
|------|------|------|
| Home | **Empty** | 이슈 0건 상태 (··· Collecting issues) |
| List / Detail | **Skeleton** | API 로딩 중 상태 (shimmer) |

---

## 7. 주의사항

- `tclass` 값은 반드시 `'bl'`, `'br'`, `'nu'` 중 하나여야 한다. 다른 값을 넣으면 뱃지가 표시되지 않는다.
- `related[].sym`은 `SYM_META`에 등록된 키와 일치해야 아이콘이 정상 표시된다.
- 이슈가 3개를 초과하면 Home 캐러셀에 자동으로 More 카드가 추가된다.
- 이슈가 0개이면 Home에서 자동으로 Empty 상태가 표시된다.
