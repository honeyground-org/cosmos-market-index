# 코스모스 마켓 인덱스

여기 있는 `index.json`이 **마켓의 목록 전체**다. 앱은 이 파일 하나를 받아(캐시)
목록·검색·필터를 그린다. 게시란 이 파일에 항목 한 개를 추가하는 PR이다.

> 코드는 우리가 호스팅하지 않는다. 인덱스는 **당신의 저장소를 가리키는 포인터**이고,
> 설치는 그 저장소에서 이루어진다. 에이전트를 만드는 방법은
> [AGENT_GUIDE.md](../docs/AGENT_GUIDE.md)에 있다 — 이 문서는 **등록 절차**만 다룬다.
>
> 현재 `agents`는 비어 있다. 이 저장소의 `market/index.json`이 공식 인덱스의 원본이며,
> 별도 공개 저장소(`cosmos-market-index`)로 승격하는 것이 Phase E4다.

## 등록 절차

1. 당신의 저장소 루트에 `cosmos-agent.yaml`을 두고 **버전 태그를 단다**(`v1.2.0`).
2. 로컬에서 규격을 통과하는지 확인한다:
   ```bash
   python scripts/verify_market.py /path/to/your-agent            # 매니페스트
   python scripts/verify_market.py --index market/index.json      # 인덱스
   ```
3. `index.json`의 `agents`에 항목을 추가해 PR을 연다. 검토 후 병합되면 즉시 목록에 뜬다.

**병합 기준은 하나다: 검증기의 `errors`가 비어 있을 것.** 경고(`warnings`)는 병합을
막지 않지만, 대부분은 사용자에게 손해가 가는 신호이므로 고치는 편이 좋다.

## 항목 스키마 (`schema_version: 1`)

```json
{
  "schema_version": 1,
  "updated": "2026-07-28",
  "agents": [
    {
      "id": "acme/weather-plus",
      "repo": "https://github.com/acme/weather-plus",
      "ref": "v1.2.0",
      "type": "feature",
      "contract": "plugin/v1",
      "title": "날씨 플러스",
      "description": "기상청 단기예보를 읽어 오늘 입을 옷까지 알려준다.",
      "author": "Acme",
      "license": "MIT",
      "homepage": "https://github.com/acme/weather-plus",
      "category": "생활",
      "capabilities": ["network"],
      "requires_desktop": false,
      "price": {"amount": 0, "currency": "KRW"},
      "verified": true
    },
    {
      "id": "acme/kanban-brain",
      "repo": "https://github.com/acme/kanban-brain",
      "ref": "v0.3.1",
      "type": "brain",
      "contract": "memory/v1",
      "title": "칸반 브레인",
      "description": "할 일을 보드 구조로 기억하는 교체형 브레인.",
      "author": "Acme",
      "license": "Apache-2.0",
      "category": "생산성",
      "capabilities": ["memory"],
      "requires_desktop": false,
      "price": {"amount": 4900, "currency": "KRW"}
    }
  ]
}
```

| 필드 | 필수 | 의미 |
|---|---|---|
| `id` | ✅ | `소유자/이름` — **저장소 소유자와 일치해야 한다**(남의 이름으로 등록 방지). 소문자, 설치 경로가 된다 |
| `repo` | ✅ | `https://` 저장소 URL. http는 거부(중간자 공격) |
| `ref` | ✅ | 설치할 **고정 참조** — 태그(`v1.2.0`)나 커밋 SHA. `main`은 경고: 어제와 오늘이 다른 코드는 감사도 재현도 불가능하다 |
| `type` | ✅ | `feature` \| `brain` \| `engine` \| `persona` |
| `contract` | ✅ | 타입에 대응하는 계약 축과 메이저(`plugin/v1` 등). 매니페스트와 같아야 한다 |
| `title`·`description`·`author` | ✅ | 목록에 그대로 표시된다 |
| `category` | ✅ | 자유 문자열. **카테고리 목록은 인덱스에서 발견한다** — 코드에 고정 목록이 없으므로 새 카테고리가 필터에서 사라지지 않는다 |
| `capabilities` | ✅ | 매니페스트 선언의 **사본**. 설치 화면은 코드를 받기 전에 이걸 보고 고지한다 → 설치 시 실물과 대조해 **실물이 더 넓으면 설치가 중단된다** |
| `requires_desktop` | | 로컬 전용이면 `true`. 웹 셸에서는 도구가 노출되지 않는다 |
| `license`·`homepage`·`requires` | | 권장. 라이선스가 없으면 사용자가 쓸 수 있는지 판단할 수 없다 |
| `price` | | `{amount, currency}`. 생략 = 무료. 결제 연동 전까지는 표시·필터에만 쓰인다 |
| `verified` | | **우리만 쓴다.** 사용자가 추가한 비공식 인덱스의 값은 앱이 강제로 내린다 |
| `installs`·`stars` | | 집계가 채우는 자리. 인덱스에 값을 박지 말 것 — PR로만 갱신되는 파일의 숫자는 그 순간부터 낡는다 |

## 이 인덱스에 없는 것

- **코드**: 우리는 호스팅하지 않는다. 설치는 당신 저장소에서 이루어진다.
- **자동 심사**: `verified`는 사람이 검토한 항목에만 붙는다. 미검증 항목도 설치할 수
  있지만 앱이 그 사실과 요구 권한을 **기본으로 표시**한다.
- **서명·샌드박스**: 아직 없다(로드맵). 그래서 설치는 곧 코드 실행이라는 사실을
  숨기지 않는 쪽을 택했다 — [AGENT_GUIDE.md §안전](../docs/AGENT_GUIDE.md) 참조.
