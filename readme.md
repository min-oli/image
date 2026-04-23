# Release Branch Strategy

## 핵심 요약

이 브랜치 전략의 목적은 개발 테스트는 자유롭게 하되, 실제 배포 후보는 엄격하게 관리하는 것이다.

```text
모든 작업은 main에서 시작한다.
개발 중 통합 테스트는 dev에서 자유롭게 한다.
배포 후보는 release-yymmdd로만 모은다.
QA, STG, PRD는 동일한 release-yymmdd 브랜치를 기준으로 순차 배포한다.
```

전체 흐름은 다음과 같다.

```text
feat/* -> dev
feat/* -> release-yymmdd -> QA -> STG -> PRD -> main
```

`dev`는 배포 경로에 포함되지 않는다. `dev`는 개발 중 통합 테스트용 브랜치일 뿐이다.

## 브랜치 역할

### main

`main`은 운영 환경의 기준 브랜치다.

- 운영 배포가 완료된 최종 코드만 존재한다.
- 모든 기능 브랜치는 `main`에서 생성한다.
- 직접 push를 금지한다.
- 평소 개발자가 직접 수정하지 않는다.
- `release-yymmdd` 또는 `hotfix/*`를 통해서만 변경된다.

### release-yymmdd

`release-yymmdd`는 이번 배포의 후보 브랜치다.

- 매주 정해진 요일에 최신 `main` 기준으로 생성한다.
- 이번 배포에 포함될 기능만 PR로 머지한다.
- QA, STG, PRD는 모두 동일한 `release-yymmdd` 기준으로 배포한다.
- PRD 배포 완료 후 `main`에 머지한다.
- 직접 push를 금지한다.

예:

```text
release-260427
```

### dev

`dev`는 개발 중 통합 테스트용 브랜치다.

- 개발자가 `feat/*`를 임시로 머지해서 Dev 서버에서 확인하는 용도다.
- 배포 후보로 간주하지 않는다.
- QA, STG, PRD 배포에 사용하지 않는다.
- 매주 PRD 배포 완료 후 최신 `main` 기준으로 reset한다.

중요한 코드는 반드시 `feat/*` 브랜치에 보관해야 한다. `dev`는 주기적으로 초기화될 수 있다.

### feat/[ticket]

`feat/[ticket]`은 개별 기능 개발 브랜치다.

- 반드시 `main`에서 생성한다.
- 개발 중 Dev 서버 확인이 필요하면 `dev`로 머지한다.
- 배포가 확정되면 `release-yymmdd`로 PR을 올린다.
- `dev`나 `release-yymmdd`의 변경사항을 원본 `feat/*`로 가져오지 않는다.

예:

```text
feat/ABC-123
feat/OYG-456
```

### fix/[ticket]

`fix/[ticket]`은 QA, STG 또는 release 통합 과정에서 발견된 문제를 수정하는 브랜치다.

- 일반적으로 `main`에서 생성한다.
- release에 합쳐진 뒤에만 발생하는 통합 문제라면 `release-yymmdd`에서 생성할 수 있다.
- 수정 후 `release-yymmdd`로 PR을 올린다.
- `release-yymmdd`에 직접 커밋하지 않는다.

예:

```text
fix/ABC-123-qa-bug
fix/release-260427-payment-issue
```

### hotfix/[ticket]

`hotfix/[ticket]`은 운영 긴급 수정 브랜치다.

- 반드시 `main`에서 생성한다.
- 검증 후 `main`으로 PR을 올린다.
- 운영 배포 후 현재 진행 중인 `release-yymmdd`와 `dev`에도 반영한다.

예:

```text
hotfix/ABC-999
```

## 기본 개발 및 배포 흐름

### 1. 기능 개발 시작

최신 `main`에서 기능 브랜치를 생성한다.

```bash
git fetch origin
git checkout -b feat/ABC-123 origin/main
```

### 2. Dev 서버 테스트

개발 중 통합 테스트가 필요하면 `feat/*`를 `dev`로 머지한다.

```text
feat/ABC-123 -> dev
```

`dev`는 자유롭게 테스트하는 공간이다. 단, `dev`의 변경사항을 다시 `feat/*`로 가져오면 안 된다.

### 3. release PR 생성

배포가 확정된 기능만 `release-yymmdd`로 PR을 올린다.

```text
feat/ABC-123 -> release-260427
```

### 4. QA 배포

`release-yymmdd` 브랜치를 기준으로 QA 환경에 배포한다.

```text
release-260427 -> QA
```

### 5. STG 배포

QA를 통과한 동일한 `release-yymmdd` 브랜치를 STG 환경에 배포한다.

```text
release-260427 -> STG
```

### 6. PRD 배포

STG를 통과한 동일한 `release-yymmdd` 브랜치를 PRD 환경에 배포한다.

```text
release-260427 -> PRD
```

### 7. main 반영

PRD 배포가 완료되면 `release-yymmdd`를 `main`에 머지한다.

```text
release-260427 -> main
```

### 8. dev 초기화

배포 완료 후 `dev`는 최신 `main` 기준으로 reset한다.

```text
dev = main
```

## 케이스별 흐름 그림

### 케이스 1. 정상 기능 개발 및 배포

기능은 `main`에서 시작하고, 개발 중 확인은 `dev`에서 하며, 실제 배포 후보는 `release-yymmdd`에 모은다.

```text
main
  |
  | 1. feat 브랜치 생성
  v
feat/ABC-123
  |
  | 2. 개발 중 Dev 서버 확인
  v
dev
  |
  | dev는 테스트장일 뿐, 배포 경로에 포함되지 않음
  |
  +------------------------------------+
                                       |
feat/ABC-123                          |
  |                                    |
  | 3. 배포 확정 후 PR                 |
  v                                    |
release-260427                        |
  |                                    |
  | 4. 동일 브랜치로 순차 배포         |
  v                                    |
QA -> STG -> PRD                      |
  |                                    |
  | 5. 배포 완료 후 main 반영          |
  v                                    |
main                                  |
  |                                    |
  | 6. dev 초기화                      |
  +-----------------------------------> dev = main
```

요약:

```text
main -> feat/* -> release-yymmdd -> QA -> STG -> PRD -> main
             \
              -> dev
```

### 케이스 2. release PR 충돌

`feat/ABC-123`를 `release-260427`에 넣으려는데 이미 먼저 머지된 기능과 충돌이 나는 상황이다.

```text
main
  |
  +--------------------+
  |                    |
  v                    v
feat/AAA-111       feat/ABC-123
  |                    |
  | 먼저 머지          | release PR 충돌
  v                    |
release-260427 <-------+
```

이때 원본 `feat/ABC-123`에 `release-260427`을 머지하지 않는다.

잘못된 처리:

```text
release-260427
      |
      | 금지: release를 원본 feat로 머지
      v
feat/ABC-123
```

올바른 처리:

```text
release-260427
  |
  | 1. release 기준 임시 브랜치 생성
  v
merge/ABC-123-into-release-260427
  |
  | 2. feat 변경분을 이 브랜치로 머지
  v
merge/ABC-123-into-release-260427
  |
  | 3. 충돌 해결 후 PR
  v
release-260427

feat/ABC-123
  |
  | 원본 feat는 깨끗하게 유지
  v
feat/ABC-123
```

최종 PR 방향:

```text
merge/ABC-123-into-release-260427 -> release-260427
```

### 케이스 3. QA 또는 STG 중 버그 수정

`release-260427`이 QA 또는 STG에 올라간 뒤 버그가 발견된 상황이다.

```text
release-260427
  |
  v
QA
  |
  | 버그 발견
  v
fix/ABC-123-qa-bug
  |
  | 수정 후 PR
  v
release-260427
  |
  | 다시 배포 및 검증
  v
QA -> STG -> PRD
```

원칙:

```text
release-260427에 직접 커밋하지 않는다.
fix/* 브랜치에서 수정하고 release-260427로 PR을 올린다.
```

### 케이스 4. 이번 배포에서 기능 제외

`release-260427`에 A, B, C 기능이 들어갔는데 C 기능을 이번 배포에서 제외해야 하는 상황이다.

현재 상태:

```text
main
  |
  v
release-260427
  |
  +-- feat/A 포함
  +-- feat/B 포함
  +-- feat/C 포함

결정: feat/C는 이번 배포 제외
```

원칙적인 처리:

```text
main
  |
  | 1. release 브랜치 재생성
  v
release-260427
  |
  | 2. 배포할 기능만 다시 반영
  +-- feat/A 머지
  +-- feat/B 머지
  |
  | feat/C는 제외
  v
QA -> STG -> PRD
```

예외적인 처리:

```text
release-260427
  |
  | QA 후반 또는 STG 이후라 재생성이 부담되는 경우
  v
revert feat/C
  |
  | 전체 회귀 테스트
  v
QA -> STG -> PRD
```

기본 선택지는 `release` 재생성이고, `revert`는 예외적으로만 사용한다.

### 케이스 5. 운영 hotfix

운영 장애 또는 긴급 수정은 `main`에서 시작한다.

```text
main
  |
  | 1. 긴급 수정 브랜치 생성
  v
hotfix/ABC-999
  |
  | 2. 검증 후 PR
  v
main
  |
  | 3. 운영 배포
  v
PRD
```

현재 진행 중인 release와 dev가 있다면 hotfix를 다시 반영한다.

```text
hotfix/ABC-999
  |
  +--> main
  |
  +--> release-260427
  |
  +--> dev
```

## 허용되는 브랜치 흐름

```text
feat/* -> dev
feat/* -> release-yymmdd
fix/* -> release-yymmdd
release-yymmdd -> main
hotfix/* -> main
hotfix/* -> release-yymmdd
hotfix/* -> dev
```

## 금지되는 브랜치 흐름

```text
dev -> feat/*
dev -> release-yymmdd
dev -> main
release-yymmdd -> feat/*
feat/* -> main
main 직접 push
release-yymmdd 직접 push
```

특히 아래 규칙은 반드시 지킨다.

```text
dev에서 테스트했다고 dev 코드를 feat 브랜치로 가져오지 않는다.
release 충돌을 해결한다고 원본 feat 브랜치에 release를 머지하지 않는다.
```

## release 충돌 처리

`feat/*`를 `release-yymmdd`에 PR 했는데 충돌이 발생하면 원본 `feat/*` 브랜치에서 해결하지 않는다.

원칙:

```text
release 충돌은 release 기준의 임시 merge 브랜치에서 해결한다.
원본 feat 브랜치는 깨끗하게 유지한다.
```

예:

```text
원본 브랜치: feat/ABC-123
배포 브랜치: release-260427
충돌 해결 브랜치: merge/ABC-123-into-release-260427
```

처리 흐름:

```bash
git fetch origin
git checkout -b merge/ABC-123-into-release-260427 origin/release-260427
git merge origin/feat/ABC-123
```

충돌을 해결한 뒤:

```bash
git add .
git commit
git push origin merge/ABC-123-into-release-260427
```

이후 PR을 다시 생성한다.

```text
merge/ABC-123-into-release-260427 -> release-260427
```

이 방식의 목적:

- `feat/*` 브랜치를 오염시키지 않는다.
- 실제 배포 후보인 `release-yymmdd` 기준으로 충돌을 해결한다.
- 충돌 해결 내용이 PR에 드러난다.
- 문제가 생기면 `merge/*` 브랜치만 버리면 된다.

## QA 또는 STG 중 버그 수정

QA 또는 STG에서 버그가 발견되어도 `release-yymmdd`에 직접 커밋하지 않는다.

원칙:

```text
모든 수정은 fix 브랜치에서 작업하고 release-yymmdd로 PR을 올린다.
```

기능 자체의 버그라면:

```text
fix/ABC-123-qa-bug -> release-260427
```

release에 합쳐진 뒤에만 발생하는 통합 문제라면:

```text
fix/release-260427-integration-issue -> release-260427
```

## 배포 제외 처리

`release-yymmdd`에 여러 기능이 들어간 뒤 특정 기능을 이번 배포에서 제외해야 할 수 있다.

예:

```text
release-260427에 A, B, C 기능이 들어감
C 기능은 이슈로 인해 이번 배포에서 제외 결정
```

기본 원칙:

```text
release 브랜치를 최신 main 기준으로 다시 만들고, 배포할 기능만 다시 머지한다.
```

즉, A와 B만 다시 반영한다.

```text
main -> release-260427 재생성
feat/A -> release-260427
feat/B -> release-260427
```

예외적으로 QA 후반 또는 STG 이후라면 `revert`를 사용할 수 있다.

단, `revert`는 다음 조건을 만족할 때만 사용한다.

- 제외할 기능이 다른 기능과 강하게 얽혀 있지 않다.
- DB migration, 설정 변경, feature flag가 독립적이다.
- revert 후 전체 회귀 테스트가 가능하다.
- 릴리즈 담당자가 승인했다.

## release에 머지하는 기준

`release-yymmdd`에는 이번 배포가 확정된 기능만 머지한다.

권장 기준:

- Dev 서버에서 기본 동작 확인이 끝났다.
- 이번 배포 포함 여부가 확정됐다.
- QA 시나리오가 준비됐다.
- DB migration 영향이 확인됐다.
- 배포 제외가 필요할 경우 영향 범위를 설명할 수 있다.

불확실한 기능은 `dev`에서만 테스트하고 `release-yymmdd`에는 늦게 머지한다.

## 시스템으로 강제할 규칙

문서만으로는 실수를 막기 어렵다. 아래 규칙은 GitHub, GitLab, CI 또는 배포 시스템에서 강제하는 것을 권장한다.

### 브랜치 보호

`main`:

- 직접 push 금지
- PR 필수
- required checks 필수
- 승인 필수
- force push 금지

`release-*`:

- 직접 push 금지
- PR 필수
- required checks 필수
- release manager 승인 필수
- force push 금지

`dev`:

- 팀 정책에 따라 직접 push 또는 PR 방식을 선택한다.
- force push는 관리자 또는 릴리즈 담당자만 허용한다.

### PR 방향 검사

CI에서 PR source/target branch 조합을 검사한다.

허용:

```text
feat/* -> dev
feat/* -> release-*
fix/* -> release-*
release-* -> main
hotfix/* -> main
hotfix/* -> release-*
hotfix/* -> dev
```

그 외 흐름은 실패 처리한다.

### release 브랜치 자동 생성

매주 정해진 요일에 최신 `main` 기준으로 `release-yymmdd` 브랜치를 자동 생성한다.

예:

```text
매주 월요일 오전 10시
main 기준 release-260427 생성
릴리즈 채널에 공지
```

### dev 초기화 자동화

PRD 배포 완료 후 `dev`를 최신 `main` 기준으로 reset한다.

```text
PRD 배포 완료
release-yymmdd -> main 머지
dev = main
```

### release 충돌 해결 스크립트

release 충돌 처리는 스크립트로 제공하는 것을 권장한다.

예:

```bash
./scripts/resolve-release-conflict.sh feat/ABC-123 release-260427
```

개발자는 충돌 발생 시 아래 원칙만 따르면 된다.

```text
release 충돌 = 충돌 해결 스크립트 실행
원본 feat 브랜치는 수정하지 않음
생성된 merge/* 브랜치로 release에 PR 생성
```

## 최종 원칙

```text
main은 운영 원본이다.
dev는 개발 테스트장이다.
release-yymmdd는 배포 기차다.
feat/*는 개인 작업장이다.

시작은 main에서 한다.
실험은 dev에서 한다.
배포는 release-yymmdd로 한다.
배포 완료 후 main에 반영한다.
dev는 주기적으로 main 기준으로 초기화한다.
```
