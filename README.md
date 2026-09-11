# 🚦 수직적 피쳐 슬라이스 하네스 (Vertical Feature Slicing Harness)

> "I/O를 중심으로 기능마다 하나의 Function처럼 슬라이스한 Feature"를 기반으로 프로젝트를 구현하기 위한 AI 코딩 하네스 템플릿.

이 레포는 **템플릿**입니다. `Use this template`으로 복제한 뒤 `project/`를 실제 프로젝트에 맞게 채우고, `project-all-docs/`에 설계 문서를 작성해 사용합니다.

---

## 핵심 기능

- **최소 컨텍스트를 통한 환각/망각 이슈 해소** — 전체 문서 대신 슬라이스당 1장의 컨텍스트 카드만 세션에 제공
- **모노레포 및 슬라이스를 통한 병렬 구현** — 의존성이 낮은 슬라이스를 워크트리로 동시에 구현
- **결정론적 검증** — 사람의 인상이 아니라 통과/실패로 판정. 채점 기준(계약 테스트·E2E)은 세션 쓰기 권한 밖

## 핵심 원칙

1. **세션에는 전체 맥락 대신 해당 피쳐 + 앞뒤 맥락만** — I/O 계약, 사용 테이블, 상하류, 레퍼런스 슬라이스만 넣는다.
2. **피쳐는 무조건 세로로 자른다** — 클라이언트 요청 → 백엔드 → 클라이언트 출력까지가 한 슬라이스. 가로(프론트/백)로 잘라야 할 만큼 크면 슬라이스를 잘못 잡은 것.
3. **중간중간 하네스로 조인다** — 방치하고 결과만 받지 않는다. 실수가 나올 때마다 원인을 기계적으로 막는다.
4. **TC로 결정론적 검증** — 판정 기준은 통과/실패 값. 채점 기준은 에이전트 쓰기 권한 밖에 둔다.
5. **루프 엔지니어링** — N개 피쳐마다 개선 루프. 기준을 넘을 때까지 반복하되 M회 도달 시 중단 후 로그 출력.

---

## 디렉토리 구조

```
/project-all-docs                  # 세션 접근 X (총괄 문서, 사람만 읽음)
  /context-card                    # 슬라이스당 1장 → 프롬프트로 복사
    _template.md                   # 컨텍스트 카드 템플릿
    order-cancel.md                # 작성 예시
  api.md                           # 전체 API 명세
  erd.md                           # 전체 ERD
  feature-graph.md                 # 피쳐 의존 그래프
  checklist.md                     # 슬라이스 완료 체크리스트
  loop-config.md                   # 목표 지표 / 최대 반복 횟수
  approvals.md                     # 계약·공용 변경 승인 기록

/project                           # 레포 root (세션은 여기서 연다)
  CLAUDE.md                        # 자동 주입. 짧게 (쓰기 X)
  package.json                     # dev / test / test:e2e (쓰기 X)
  .eslintrc                        # 슬라이스 간 import 차단 (쓰기 X)
  .claudeignore                    # docs/generated 제외 (쓰기 X)
  .husky/pre-commit                # 커밋 시점 경계 검사 (쓰기 X)
  .claude/settings.json            # 툴 호출 시점 경계 검사: deny / ask (쓰기 X)
  /backend
    /src
      /common                      # 타입만. 로직 X (쓰기 X)
        /entity
        /dto
        /enums
        /error
      /shared                      # 인프라. 로직 (쓰기 X)
        /security                  # 인증 필터, 권한, 현재 사용자
        /exception                 # 전역 예외 핸들러
        /response                  # 공통 응답
        /config
      /feature
        /{slice}                   # 슬라이스별 (쓰기 O)
    /test
      /contract                    # 컨텍스트 카드로 만든 TC. hook으로 수정 차단
      /unit
        /{slice}                 # 세션이 TDD로 작성. 자기 슬라이스 폴더만 (쓰기 O)
  /front                           # backend 폴더와 구조 동일
    /src
      /common
        /type
      /shared
        /auth
        /ui
        /api
      /feature
        /{slice}
    /test
      /contract
      /unit
        /{slice}
  /e2e                             # 통합 테스트. 동결 (쓰기 X)
  /docs
    /generated                     # OpenAPI·ERD 생성물 (claudeignore 대상)
    /prompts                       # 테스트 세션 / 구현 세션 프롬프트 템플릿
```

**핵심은 두 가지.** `common/`·`shared/`는 세션 쓰기 금지, `test/contract/`는 실구현 전에 컨텍스트 카드를 보고 승인 후 동결. 나머지는 프로젝트에 맞게 조정.

- `project-all-docs/`는 원래 **레포 밖**에 두는 것이 원칙이다. 이 템플릿에서는 편의상 같은 레포의 형제 폴더로 두었으며, 세션은 반드시 `project/`에서 연다. 동기화가 필요할 때만 최상위(real-root)에서 새 세션을 연다.
- `docs/`는 사람용이라 세션에 통째로 들어가지 않는다. **세션은 프롬프트로 컨텍스트 카드만 제공받는다.**

---

## 워크플로우

1. **피쳐 함수 단위 설계** — 세로 슬라이스 분해, I/O 확정, 공용 레이어와 구현 순서 결정 (사람)
2. **API · ERD 문서화** — 사람이 조망하는 용도. 여기서 나온 타입/DTO/enum은 `common/`에 코드로 내려놓음
3. **의존 그래프** — 순서·병렬 묶음 정리. 시각화는 사람용, 상하류 한 줄은 세션 프롬프트로
4. **테스트 세션** — 깨끗한 세션이 명세만 보고 계약 테스트 작성 → 사람 승인 → 동결
5. **구현 세션** — 피쳐별 세션. 의존 없는 것끼리 병렬 (1인 기준 동시 2~3개 상한)
6. **루프 엔지니어링** — 슬라이스별 1회 + 통합 후 1회
7. **E2E + 육안 확인** — 슬라이스 완료 게이트

---

## 테스트 3계층

| 계층 | 범위 | 루프 안 | 작성 주체 |
|---|---|---|---|
| 단위/로직 | 백·프론트 각자 | O | 구현 세션 (TDD) |
| API 계약 | 백엔드 응답 형태 | O | 테스트 세션 → 동결 |
| E2E | 슬라이스 전체 흐름 | X (게이트) | 테스트 세션 |

API 계약 테스트가 타입 공유의 빈자리를 메운다. 백엔드/프론트가 언어가 달라 컴파일러 보호가 없는 유일한 구간이라, 여기가 뚫리면 통합 시점에야 발견된다.

### 계약 테스트 원칙

- **작성 세션은 구현 코드를 보면 안 된다.** 컨텍스트 카드만 보고 작성
- **사람 승인 후 동결.** 여기서 걸러지는 건 테스트보다 명세 오해. 두 AI가 같은 문서를 다르게 읽었다면 문서가 모호한 것
- **피쳐당 3~7개.** 계약과 주요 엣지케이스만
- **세션 쓰기 권한 밖에 둔다**

---

## 하네스 개입 트리거

**하드 스톱**

- 테스트 파일 수정 시도
- 슬라이스 밖 파일 수정 / 공용 레이어 변경

**경고**

- TC·E2E 2회 연속 실패
- 계약 테스트 실패 (명세 오해 신호 — 세부 테스트 실패와 구분)
- 같은 파일 3회 이상 반복 수정
- `any` / `@ts-ignore` / 테스트 skip 처리
- 임의 구현, 의존성 추가, 스키마·마이그레이션 변경
- 컨텍스트 임계 도달

1인이 세션 3개를 눈으로 볼 수는 없다. **git hook, 린트 룰, 파일 권한처럼 기계가 막는 쪽으로 최대한 옮긴다.**

### 기계적 강제

`CLAUDE.md`는 강제력이 없다. 실제로 막으려면 따로 장치가 필요. 경계는 **두 등급**으로 나뉘고, 툴 호출 시점(`.claude/settings.json`)과 커밋 시점(`.husky/pre-commit`)에서 같은 등급으로 막는다.

| 등급 | 경로 | 툴 호출 시점 | 커밋 시점 | 해제 |
|---|---|---|---|---|
| **하드** | `test/contract/` | `permissions.deny` — 프롬프트 없이 거부 | `exit 2` — 항상 차단 | 없음. 사람이 세션·훅 밖에서 처리 |
| **소프트** | `src/common/`, `src/shared/`, `e2e/`, 설정 파일 (`CLAUDE.md`, `package.json`, `.eslintrc`, `.claudeignore`, `.husky/`, `.claude/`) | `permissions.ask` — 사용자 승인 프롬프트 | `exit 1` — `HARNESS_APPROVED=1`이면 통과 | 사용자가 프롬프트에서 승인 → `HARNESS_APPROVED=1 git commit` → `approvals.md` 기록 |
| (선택) | 다른 슬라이스 `feature/{other}/`, `test/unit/{other}/` | `PreToolUse` 훅 + `HARNESS_SLICE` | — | 미설정 시 검사 생략 |

`deny`와 `ask`의 차이:

| | deny | ask |
|---|---|---|
| 세션이 수정 시도 | 즉시 거부 | 사용자에게 승인 프롬프트 |
| 사용자 승인 | 불가 (settings 수정 필요) | 그 자리에서 y/n |
| auto-accept 모드 | 거부 | 그래도 프롬프트 |
| `--dangerously-skip-permissions` | 거부 유지 | 프롬프트 생략됨 |

계약 테스트만 `deny`인 이유: 이건 채점 기준이라 세션 안에서 풀리는 순간 "동결"이 아니게 된다. 나머지는 슬라이스를 구현하다 보면 실제로 변경이 필요해지므로(DTO 필드 추가 등) "차단하되 사람이 풀 수 있는" 등급에 둔다. 훅 우회 명령(`--no-verify`, `HARNESS_APPROVED=`)도 `ask`라 세션이 시도하면 사용자에게 보인다.

`.claude/settings.json`은 [project/.claude/settings.json](project/.claude/settings.json)에 들어 있다. pre-commit은 git 기준 2차 방어선이며, exit 1/2는 git 입장에선 같지만(0이 아니면 차단) 로그에서 등급을 구분하기 위한 컨벤션이다.

그 외 강제 수단:

| 대상 | 수단 |
|---|---|
| 슬라이스 간 import | `.eslintrc` 린트 룰 |
| 실행 방법 일관성 | `package.json` 스크립트 고정 |

#### 세션 쓰기 범위

`order-cancel` 세션 기준:

```
backend/src/feature/order-cancel/**     O
front/src/feature/order-cancel/**       O
backend/test/unit/order-cancel/**       O   (TDD)
front/test/unit/order-cancel/**         O

backend/src/common/**, shared/**        △   ask — 사용자 승인 시 가능
front/src/common/**, shared/**          △
e2e/**                                  △
CLAUDE.md, package.json, .eslintrc,
.claudeignore, .husky/**, .claude/**    △
backend/src/feature/order-create/**     △   다른 슬라이스 (HARNESS_SLICE 설정 시 검사)

backend/test/contract/**                X   deny — 세션 안에서 해제 불가
front/test/contract/**                  X
```

#### 남는 구멍

- 계약 테스트 **삭제**·**이동**은 Edit/Write deny로 안 잡힌다. pre-commit은 `--name-only`라 삭제도 잡지만, settings로 막으려면 `Bash(rm *)`, `Bash(git rm *)`, `Bash(mv *)`도 deny해야 한다.
- `npm run test`가 전체 테스트를 돌리면 다른 슬라이스의 깨진 테스트를 고치러 갈 수 있다. 자기 슬라이스 범위만 돌리는 스크립트(`test -- $HARNESS_SLICE`)를 두는 게 낫다.
- `--dangerously-skip-permissions`로 세션을 열면 `ask`가 전부 무력화된다. 이 하네스에서는 쓰지 않는다.
- settings·훅은 Claude Code 기준이다. 다른 에이전트를 쓰면 같은 규칙을 그 도구의 권한 체계로 옮겨야 하며, git hook과 린트만 도구 무관하게 동작한다.

---

## 루프 안전장치

목표 점수만 걸면 그 점수만 오른다. 캐시로 정합성을 깨거나, 검사 범위를 줄여 취약점 점수를 올리는 식.

- **목표 점수** — 올리려는 지표
- **불변 게이트** — 계약 테스트 + E2E. 쓰기 권한 밖. 하나라도 깨지면 이터레이션 무효
- **최대 반복 횟수** — 도달 시 성공 여부와 무관하게 중단, 사람에게 인계
- **diff 리뷰** — 점수가 올랐어도 어떻게 올렸는지는 본다
- **지표당 루프 하나** — 성능과 취약점을 동시에 걸면 상충하는 최적화가 나와 원인 추적 불가

---

## 보조 규칙

- **읽기 전용 경계** — 공용 레이어는 세션에서 수정 금지. 필요하면 구현하지 말고 보고 후 중단
- **레퍼런스 슬라이스** — 첫 슬라이스는 병렬로 뿌리지 말고 혼자 구현. 이후 세션에 "이 폴더와 같은 구조로"가 어떤 컨벤션 문서보다 잘 먹힘 (`feature/_reference-slice/`)
- **풀네임 명명** — 축약어 금지. 모델은 이름의 토큰을 그대로 신호로 쓴다
- **DTO와 엔티티 분리** — 엔티티를 그대로 응답으로 내보내면 스키마 변경이 API 계약으로 새어나감
- **enum은 반드시 enum으로** — String으로 두면 세션마다 다른 문자열을 쓰고 컴파일러가 못 잡음
- **실행은 고정 스크립트로** — 세션이 실행 방법을 추측하면 포트를 바꾸거나 목으로 때움

---

## 운영 절차

**브랜치** — 슬라이스당 브랜치 1개, 순차 머지. 병렬 세션이 같은 브랜치에서 작업하면 서로 덮어쓴다.

**계약 변경 요청 처리** — 세션이 "공용 레이어 변경 필요"를 보고했을 때

1. 사람이 판단
2. 공용 타입 수정
3. **같은 타입 쓰는 다른 슬라이스 확인**
4. 끝난 슬라이스면 재작업 여부 결정
5. 컨텍스트 카드 갱신 후 세션 재개 (`project-all-docs/approvals.md`에 기록)

---

## 템플릿 사용법

1. `project-all-docs/api.md`, `erd.md`, `feature-graph.md`를 작성한다. (사람)
2. 도출된 엔티티·DTO·enum을 `project/backend/src/common/`, `project/front/src/common/`에 코드로 내려놓는다.
3. 슬라이스마다 `project-all-docs/context-card/_template.md`를 복사해 카드를 작성한다. **빈칸이 남으면 설계가 덜 됐다는 신호.**
4. `project/`에서 세션을 열고 `project/docs/prompts/test-session.md` 프롬프트 + 카드로 계약 테스트를 작성 → 승인 → 동결.
5. `project/docs/prompts/implementation-session.md` 프롬프트 + 카드로 구현 세션을 연다.
6. `project-all-docs/checklist.md`를 모두 통과하면 머지.

---

## 장점 / 단점

**장점**

- **병렬처리:** 의존성이 낮은 두 슬라이스를 워크트리로 병렬 구현 가능
- **최소 컨텍스트:** 토큰 비용 절감, 환각/망각 최소화
- **검증 최적화:** I/O 기반 슬라이스는 최소 TC와 피쳐 단위 E2E를 가능하게 하고, 린트·`.claudeignore`로 슬라이스 규칙을 강제

**단점**

- 선행 설계가 복잡하다
- 병렬 처리로 인한 리뷰가 어렵다. 팀 단위 개발에는 부적합 (1인 개발용으로 설계)
- 슬라이스마다 컨텍스트 카드를 제공해야 하는 운영 부담
