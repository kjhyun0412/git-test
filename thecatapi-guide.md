# The Cat API 사용법 정리

- 공식 사이트: https://thecatapi.com/
- 공식 문서: https://docs.thecatapi.com/
- API 레퍼런스: https://docs.thecatapi.com/reference
- OpenAPI 스펙: https://api.thecatapi.com/openapi-json
- API 키 발급: https://thecatapi.com/signup (이메일로 키 발송)

## 1. 기본 정보

| 항목 | 내용 |
|---|---|
| Base URL | `https://api.thecatapi.com/v1` |
| 인증 | 모든 요청에 `x-api-key: YOUR-API-KEY` 헤더 |
| 이미지 동기 업로드 호스트 | `https://upload.thecatapi.com/v1/images/upload-sync` |

### 요금제
| 플랜 | 가격 | 한도 | 비고 |
|---|---|---|---|
| Free | $0 | 월 10,000 요청 | 학습/취미용, 품종 핵심 필드만 제공 |
| Startup | $99/월 | 월 100,000 요청 | 상업용 라이선스, 전체 품종 데이터, `lang` 지원 |
| Enterprise | 협의 | 무제한 | 사진 분석, 건강 기록 등 |

> 홈페이지 샘플은 `DEMO-API-KEY`를 쓰지만, 실제 사용 시에는 발급받은 키를 사용한다.
> 주의: 프런트엔드(JS)에 키를 직접 넣으면 노출된다. 학습용이 아니라면 서버를 경유하는 것이 안전하다.
> 스펙상 `/images/search`는 키가 선택 사항처럼 표기되어 있으나, 키가 있어야 `limit` 100까지 사용 가능하다.

## 2. 이미지 (Images)

### 이미지 검색 `GET /images/search`
```bash
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2"
```

| 파라미터 | 설명 |
|---|---|
| `limit` | 반환 개수 (1~100, 기본 1) |
| `page` | 페이지 번호 (0부터 시작) |
| `order` | `ASC` / `DESC` / `RAND` |
| `has_breeds` | `1`이면 품종 정보가 있는 이미지만 |
| `include_breeds` | `1`이면 품종 객체 포함 |
| `include_categories` | `1`이면 카테고리 객체 포함 |
| `breed_ids` | 쉼표 구분 품종 ID (예: `abys,beng`) |
| `breed_id` | 단일 품종 ID |
| `mime_types` | `jpg`, `png`, `gif` |
| `size` | `thumb`/`small`/`med`/`full` (호환용) |
| `sub_id` | 업로드 시 지정한 sub_id로 필터 |

예: 아비시니안 이미지 2장 + 품종 상세
```bash
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/images/search?limit=2&breed_ids=abys&include_breeds=1"
```

### 기타 이미지 API
| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/images/{id}` | 이미지 단건 조회 |
| GET | `/images` | 내가 업로드한 이미지 목록 (`?sub_id=` 필터) |
| POST | `/images/upload` | 이미지 업로드(비동기, multipart `file`) |
| POST | `/images/upload-sync` | 업로드 후 분석 완료까지 대기 (upload 호스트 사용) |
| GET | `/images/{id}/status` | 처리 상태 조회 |
| GET | `/images/{id}/labels` | AI 라벨(품종/카테고리) |
| DELETE | `/images/{id}` | 업로드 이미지 삭제 |

```bash
curl -X POST -H "x-api-key: YOUR-API-KEY" -F "file=@/path/cat.jpg" https://api.thecatapi.com/v1/images/upload
```

### 응답 필드 (ImageDto)
필수: `id`, `url`, `width`, `height`. 선택: `breeds`, `categories`, `colours`, `tags`, `sub_id`, `created_at` 등.

## 3. 품종 (Breeds)

| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/breeds` | 품종 목록 (`limit` 1~1000, `page`, `order`, `breed_groups`, `exclude_breed_groups`, `lang`) |
| GET | `/breeds/search?q=siamese` | 이름 검색 (`q` 1~30자) |
| GET | `/breeds/{id}` | 품종 상세 (예: `abys`, `beng`, `siam`) |
| GET | `/breeds/breed-groups` | 품종 그룹 목록 |
| GET | `/breeds/{id}/colours` | 품종별 색상/무늬 |

```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds/abys
curl -H "x-api-key: YOUR-API-KEY" "https://api.thecatapi.com/v1/breeds?breed_groups=Short-haired,Rex"
```

응답 예시(축약):
```json
{
  "id": "abys",
  "name": "Abyssinian",
  "life_span": "14-17",
  "temperament": "Active, Energetic, Independent, Intelligent, Gentle, Curious, Playful",
  "origin": "Egypt",
  "country_code": "EG",
  "breed_group": "Short-haired",
  "reference_image_id": "KWdLHmOqc",
  "image": {
    "id": "KWdLHmOqc",
    "url": "https://cdn2.thecatapi.com/images/KWdLHmOqc.jpg",
    "width": 3114,
    "height": 2609
  }
}
```

- Free: 이름, 성격, 원산지, 수명, 설명, 이미지 등 핵심 필드
- Startup 이상: 1~5점 특성 점수(`energy_level`, `affection_level`, `grooming` 등), 불리언 특성(`hypoallergenic`, `indoor` 등), `lang`(en, de, es, fr, it 등) 다국어

## 4. 투표 (Votes)

| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/votes` | 투표 목록 (`limit`, `page`, `order`, `sub_id`) |
| POST | `/votes` | 투표 생성 |
| GET | `/votes/{vote_id}` | 투표 단건 조회 |

```bash
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc", "value": 1}' \
  https://api.thecatapi.com/v1/votes
```
- Body: `image_id`(필수), `value`(필수, `1` 또는 `-1`), `sub_id`, `country_code`
- 투표는 삭제할 수 없다. 취소하려면 같은 이미지에 반대 `value`로 다시 투표한다.

## 5. 즐겨찾기 (Favourites)

| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/favourites` | 목록 (`limit`, `page`, `order`, `sub_id`, `image_id`, `attach_image`) |
| POST | `/favourites` | 생성 |
| GET | `/favourites/{id}` | 단건 조회 |
| DELETE | `/favourites/{id}` | 삭제 |

```bash
curl -X POST -H "Content-Type: application/json" -H "x-api-key: YOUR-API-KEY" \
  -d '{"image_id": "KWdLHmOqc", "sub_id": "user-123"}' \
  https://api.thecatapi.com/v1/favourites
```
- `sub_id`로 사용자별 즐겨찾기를 구분할 수 있다.

## 6. 그 밖의 엔드포인트

| 경로 | 설명 |
|---|---|
| `GET /facts`, `/facts/random`, `/facts/breed/{breedId}` | 고양이 상식 (Startup 이상 가능성 있음) |
| `GET /categories`, `/categories/{id}` | 이미지 카테고리 |
| `GET /health-tips` | 건강 팁 |
| `GET /accounts/features`, `/accounts/quota` | 사용 가능 기능/업로드 쿼터 |
| `/pets/*`, `/calculators/*`, `/webhooks/*` | 반려동물 분석·웹훅 (주로 Enterprise) |

## 7. JavaScript(fetch) 예제

```js
const API_KEY = 'YOUR-API-KEY';
const BASE = 'https://api.thecatapi.com/v1';

async function getCats(limit = 5) {
  const res = await fetch(`${BASE}/images/search?limit=${limit}&has_breeds=1&include_breeds=1`, {
    headers: { 'x-api-key': API_KEY },
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json(); // [{ id, url, width, height, breeds: [...] }, ...]
}

getCats(3).then(cats => {
  cats.forEach(c => console.log(c.url, c.breeds?.[0]?.name));
});
```

## 8. 참고 및 주의
- 오류 코드/재시도 정책: https://docs.thecatapi.com/docs/errors
- 월 호출 한도(Free 10,000회)를 넘기지 않도록 결과를 캐시하는 것을 권장한다.
- 스펙의 일부(`FavouriteDto` 등)는 확인하지 못했으므로, 세부 응답 필드는 API 레퍼런스에서 재확인이 필요하다.
