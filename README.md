# MagicSquare (4×4)

부분적으로 채워진 **4×4 마방진**(빈칸 2개, `0` 표기)을 **계약된 형식**으로 완성하거나, 스키마·도메인 위반 시 **안정적인 오류**를 반환하는 연습용 프로젝트입니다. 요구·완료 기준의 정본은 [`docs/PRD.md`](docs/PRD.md)이며, 구현 시 **ECB + TDD + Dual-Track(UI 계약 / Logic 규칙)**을 따릅니다.

---

## 문제를 한 줄로 (Report/01 요약)

16칸에 1~16을 한 번씩 두었을 때 **모든 행·열·두 주 대각선의 합이 같음**을 만족하는 배치가 **마방진**입니다. 본 저장소는 그 판정과 “빈칸 두 곳에 남은 두 수를 어떻게 넣을지”를 **반복 실행·자동 검증 가능한 규칙과 입출력 계약**으로 고정하는 것을 목표로 합니다. ([`Report/01.MagicSquare_ProblemDefinition_Report.md`](Report/01.MagicSquare_ProblemDefinition_Report.md))

---

## 입출력 계약 (Report/02 · PRD §4)

| 구분 | 내용 |
|------|------|
| **입력** | `4×4` 정수 격자. `0` = 빈칸 **정확히 2개**. 각 칸은 `0` 또는 `1..16`, 비0 **중복 없음**. |
| **출력(성공)** | 길이 6의 `int[6]`: `[r1, c1, n1, r2, c2, n2]` — 좌표는 **1-index**(1~4). `n1`, `n2`는 누락된 두 수이며, (작은 수→첫 빈칸, 큰 수→둘째 빈칸) 조합이 마방진이면 그 순서, 아니면 **역순**이 계약에 맞는 출력입니다. |
| **완전 채움 마방진** | 행·열·주대각 합이 동일하고, 1~16이 각 한 번씩 사용됩니다(고전 4×4에서 합은 **34**). |

**경계 오류 코드** (`Report/02` §2.2~2.4, PRD §4.5와 정렬):

| `code` | 발생 조건 |
|--------|-----------|
| `ERR_SIZE` | 행 수 ≠ 4 또는 어떤 행 길이 ≠ 4 |
| `ERR_EMPTY_COUNT` | `0`의 개수 ≠ 2 |
| `ERR_VALUE_RANGE` | 원소가 0도 아니고 1..16도 아님 |
| `ERR_DUPLICATE` | 비0 값 중복 |
| `ERR_DOMAIN_UNSOLVABLE` | 입력은 유효하나 두 배치 모두 마방진 불가 |
| `ERR_INTERNAL` | 예기치 않은 오류(정책에 따라 문서화) |

**도메인 실패 분류** (`Report/02` §1.4): `InvalidInputInvariant`, `UnsolvablePuzzle` 등 — 경계에서는 위 `code`/`message`로 매핑합니다.

---

## 아키텍처 · TDD (`.cursorrules`)

- **패턴:** ECB — `boundary → control → entity` 단방향.
- **Boundary:** I/O·DTO·표현; **도메인 규칙을 넣지 않고** Control에 위임.
- **Control:** 검증·솔브·출력 변환 **오케스트레이션**.
- **Entity:** `Grid`, `Cell`, `Solution`, 검증·솔버 등 **순수 도메인**.
- **TDD:** RED → GREEN → REFACTOR; pytest, AAA, **커버리지 최소 80%** (도메인 라인 ≥ 95%는 PRD·Report/04 성공 기준).
- **권장 디렉터리:** 루트 `.cursorrules`의 `magic_square/{entity,control,boundary}` 및 `tests/` 미러 구조.

**Python:** 3.10+ · PEP 8 · 타입 힌트 · Google 스타일 docstring(public) · 줄 길이 88.

---

## 문서 맵

| 문서 | 용도 |
|------|------|
| [`docs/PRD.md`](docs/PRD.md) | 제품 요구·불변(INV-*)·NFR·성공 기준·문서 이력 |
| [`Report/01.MagicSquare_ProblemDefinition_Report.md`](Report/01.MagicSquare_ProblemDefinition_Report.md) | 문제 정의·왜 4×4·왜 프로그램인지 |
| [`Report/02.MagicSquare_CleanArchitecture_TDD_Design_Report.md`](Report/02.MagicSquare_CleanArchitecture_TDD_Design_Report.md) | 계약·도메인 API·D-01~D-12·U-01~U-10 |
| [`Report/03.MagicSquare_CursorRules_Report.md`](Report/03.MagicSquare_CursorRules_Report.md) | `.cursorrules`와 동기 스냅샷 |
| [`Report/04.MagicSquare_UserJourney_Epic_To_Technical_Report.md`](Report/04.MagicSquare_UserJourney_Epic_To_Technical_Report.md) | Epic → User Story → Gherkin·fixture 기대값 |
| [`Report/05.MagicSquare_PRD_Report.md`](Report/05.MagicSquare_PRD_Report.md) | PRD 메타·갱신 요약(PRD 본문은 `docs/PRD.md`와 동기) |

---

## 구현 후 실행 (패키지 추가 시)

프로젝트 루트에 `pyproject.toml`과 `magic_square` 패키지가 준비되면 예시는 다음과 같습니다.

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -e ".[dev]"
pytest
pytest --cov=magic_square --cov-fail-under=80
```

현재 저장소는 **문서·규약 중심**이며, 위 명령은 코드 스캐폴딩 후 사용하면 됩니다.

---

## To-Do 리스트 (Epic / User Story / Task + Requirements 추적)

이 섹션은 **체크박스·단계 번호·요구 추적**을 한곳에 모은 실행 계획입니다.

- **계층:** `Epic-001` → `US-00n` (User Story) → `TASK-00n` (구현 단위). 필요 시 `TASK-00n-k`로 **TDD 마이크로 스텝**(RED → GREEN → REFACTOR)을 쪼갭니다.
- **추적 흐름(C2C, Code to Customer):** 각 Task는 **요구(Req / INV-*)** → **시나리오(Gherkin·Report/04)** → **테스트(`test_*`, D-*, U-*)**와 연결합니다. 항목을 `[x]`로 닫을 때마다 대응 테스트를 **추가하고 녹색**으로 유지합니다.
- **정본:** 입출력·불변·에러 코드는 [`docs/PRD.md`](docs/PRD.md), 설계·테스트 ID는 [`Report/02`](Report/02.MagicSquare_CleanArchitecture_TDD_Design_Report.md)·[`Report/04`](Report/04.MagicSquare_UserJourney_Epic_To_Technical_Report.md)를 따릅니다.

현재 저장소는 **문서·규약 중심**이며, 아래 구현 체크는 기본적으로 미완료(`[ ]`)입니다. 문서 작업만 완료된 경우 PR/커밋 메시지에 **어떤 REQ·시나리오를 반영했는지**를 한 줄이라도 남기면 PRD §2.2 추적성에 도움이 됩니다.

---

### Phase 0 — 저장소 스캐폴딩·추적성 (선행)

| ID | 설명 | 완료 기준(요약) |
|----|------|----------------|
| **TASK-000** | `magic_square/{entity,control,boundary}` 및 `tests/` 미러 구조 생성 (`.cursorrules` 예시) | 패키지 import·pytest 수집 가능 |
| **TASK-000b** | `pyproject.toml`: Python 3.10+, pytest, (선택) coverage, `pip install -e ".[dev]"` | README §「구현 후 실행」 명령이 동작 |
| **TASK-000c** | D-/U- ID ↔ `test_*` 이름 매핑을 `docs/` 또는 PR 본문에 1회 이상 기록 | PRD §2.2「매핑 가능」 충족 |

- [ ] **TASK-000** ~ **TASK-000c** (위 표 전부)

---

### Epic-001: 4×4 마방진 검증·완성 시스템

한 줄 목표: 부분 채움 격자에 대해 **규칙 위반은 안정적 오류**, 유효 입력은 **계약된 `int[6]`** 또는 도메인 불가로 일관되게 응답한다.

#### US-001: 스키마·도메인 불변 + **행·열·대각 합(마방진) 검증**

입력이 허용 집합인지 먼저 판정하고, 완전 채움에 대해 **마방진 합·퍼뮤테이션** 불변을 검증하는 쪽의 Entity·검증기를 만든다. (슬라이드의「합 검증」+ SquareValidator에 해당.)

- [ ] **TASK-001 — 도메인 모델 정의 (MagicSquare / Grid / Cell 등)**  
  - ECB에서 **Entity**에 둘 순수 자료·행동의 최소 집합(격자, 좌표, 필요 시 `Solution` 타입)을 타입 힌트와 함께 정의한다.  
  - Boundary에는 도메인 규칙을 넣지 않는다.

- [ ] **TASK-002 — 검증기(예: `SquareValidator` 또는 동등) 구현**  
  완전 채움·부분 채움에서 호출될 **합·범위·중복** 관련 판정을 한곳에 모은다. TDD로 쪼갤 때는 아래처럼 단계를 밟는다.  
  - [ ] **TASK-002-1 (RED):** 실패 케이스부터 `test_*` 추가 — 예: 비마방진·잘못된 합 → `is_valid()`(또는 동등 API)가 기대대로 실패.  
  - [ ] **TASK-002-2 (GREEN):** 최소 구현으로 테스트 통과.  
  - [ ] **TASK-002-3 (REFACTOR):** 중복 제거·명명 상수(`INV-MAGIC-CONSTANT` 등 PRD 표현과 정렬).

- [ ] **TASK-003a — Story 1: 입력 불변 (Report/04 Story 1 · PRD §4.1)**  
  - `4×4` 아님 → 스키마 실패 (D-04/D-05 맥락; 빈칸 개수는 별 INV).  
  - 빈칸(`0`)이 정확히 2개가 아님 → `INV-EXACTLY-TWO-ZEROS` (Gherkin: 빈칸 개수 오류).  
  - 값이 `{0}∪{1..16}` 밖 → `INV-DOMAIN-VALUES`.  
  - 비0 중복 → `INV-NO-DUP-NONZERO`.  
  - 단위 테스트: **D-04 ~ D-08** 대응.

- [ ] **TASK-003b — Story 4: 마방진 판정 (Report/04 Story 4)**  
  - 완전 채움: 행·열·주대각 동일 + 1~16 퍼뮤테이션 (`INV-MAGIC-CONSTANT`, `INV-PERMUTATION-WHEN-FILLED`).  
  - 단위 테스트: **D-11, D-12**.

#### US-002: **빈칸 탐색** (좌표·순서 고정)

PRD §4.3: 두 빈칸은 **행 우선**, 동행이면 **열 우선**으로 `(첫 빈칸, 둘째 빈칸)`을 고정한다.

- [ ] **TASK-004 — MissingFinder(또는 동등): `0`인 칸 2곳 좌표·순서**  
  - **선택(연습):** 백트래킹·제약 전파 등 다른 퍼즐 솔버 사고를 연습하려면 *별도 실험 브랜치*에서 N-Queen 등과 비교해 볼 수 있으나, **본 과제 계약상 필수 알고리즘은 아님**.  
  - **체크포인트:** 대표 fixture 기준 단일 케이스 **엔티티 탐색·정렬 경로가 100ms 미만**인지 로컬에서 한 번 확인(정본 수치는 팀/CI에서 재조정 가능).  
  - 단위 테스트: **D-09** (빈칸 위치 다양성).

#### US-003: 누락 숫자 (Report/04 Story 3 · UC-02)

- [ ] **TASK-005:** `{1..16}`에서 격자에 없는 두 수 `(min, max)` 반환.

#### US-004: 두 배치 시도·`int[6]` 계약 (Report/04 Story 5 · UC-04)

- [ ] **TASK-006:** 작은 수→첫 빈칸·큰 수→둘째 시도 후, 실패 시 **역순** 시도 → `Solution` / `solution_to_contract_array` (**1-index**).  
- [ ] **TASK-007:** 통합·도메인 테스트 — Gherkin「첫 배치로 완성」기대 `[1, 2, 3, 1, 4, 13]`;「역순으로만 완성」기대 `[3, 3, 6, 4, 4, 1]` (Report/04 fixture 표).  
- [ ] **TASK-008:** 두 배치 모두 실패 → `UnsolvablePuzzle`(또는 동등) — **D-10**.  
- [ ] **TASK-009:** 단위 테스트 **D-01, D-02, D-03** (해 있음·첫 배치만 성공·역만 성공).

#### US-005: Control 오케스트레이션

- [ ] **TASK-010:** parse → **불변 검증** → 빈칸 → 누락 수 → 솔브 → 계약 배열. 불변 실패 시 Entity 솔버 **미호출**을 테스트로 고정할 수 있으면 이상적.

#### US-006: Boundary + 계약 테스트 (Dual-Track UI, Report/02 §2.3)

- [ ] **TASK-011:** Boundary에서 형식 검증 후 Control 위임; 성공 시 `int[6]` 스키마.  
- [ ] **TASK-012:** 계약 테스트 **U-01 ~ U-06** (크기·빈칸 수·범위·중복 → 해당 `ERR_*` + 고정 `message`).  
- [ ] **TASK-013:** **U-07 ~ U-09** (Mock 성공·Unsolvable·응답 길이).  
- [ ] **TASK-014 (선택):** **U-10** 정책을 문서에 고정한 뒤 구현·테스트 한 가지로 합의.

#### 마무리 — PRD §2.2 · Report/04 성공 기준

- [ ] **TASK-015:** Domain(entity) 라인 커버리지 **≥ 95%**.  
- [ ] **TASK-016:** 입력 경계(계약) 테스트 **100%** 통과.  
- [ ] **TASK-017:** 프로젝트 커버리지 **≥ 80%**, `print()` 디버깅 및 `.cursorrules` forbidden 패턴 준수.

---

### Requirements 추적 매트릭스 (예시 · 구현 시 채움)

아래는 **Task → Req(또는 INV) → 시나리오 레벨 → 테스트** 연결의 예시입니다. 실제 `test_*` 이름을 정하면 **Test Name** 열을 고정하고, CI에서 **Status**를 갱신합니다.

| Task ID | Req / INV (요약) | Scenario (레벨) | Test Name (예시) | Status |
|:--------|:-----------------|:----------------|:-----------------|:-------|
| TASK-001 | 도메인 경계 명확화 | L1 설계 | (모듈 스모크) | TODO |
| TASK-002 | 완전 격자 마방진 판정 | L3 Fail / L1 Happy | `test_invalid_square` / `test_valid_ms` | TODO |
| TASK-004 | PRD §4.3 빈칸 순서 | L2 Edge | `test_missing_cells_order` | TODO |
| TASK-006 | §4.2 `int[6]` 계약 | L1 Happy | `test_contract_array_first_order` | TODO |
| TASK-011 | §4.5 경계 `ERR_*` | L1 UI 계약 | `test_boundary_err_size` 등 | TODO |

**운영 팁:** PR이나 `docs/traceability.md`(선택)에 위 표를 복사해 **Test Name**만 리포지토리 규약에 맞게 바꾸면, PRD의「INV / 계약 ID ↔ 테스트 ID 매핑 가능」을 증명하기 쉽습니다.

---

## 라이선스

미정 — 팀 정책에 맞게 추가하세요.
