# @manifesto-ai/projection Monorepo PRD v0.1

## 1. 목적 (Purpose)

`@manifesto-ai/projection` monorepo는 **Manifesto Core의 Domain Model과 Snapshot을 다양한 소비자(UI, API 등)로 “투영(Projection)”하기 위한 패키지 집합**을 관리한다.

이 monorepo의 목적은 다음과 같다:

* Projection 계층을 **core / agent와 물리적으로 분리**
* React, Vue, GraphQL 등 다양한 Projection 구현을 **일관된 구조**로 관리
* 빌드, 테스트, 배포를 **단일 모노레포 파이프라인**으로 통합
* Projection 구현 간 **의존성 역전 및 결합 방지**

---

## 2. 범위 (Scope)

### 포함 범위 (In Scope)

* pnpm + turbo 기반의 모노레포 초기 구성
* Projection 패키지들의 기본 폴더 구조 정의
* 공통 설정(tsconfig, eslint, build) 분리
* `@manifesto-ai/core`에 대한 **단방향 의존성** 설정
* 각 Projection 패키지의 **책임과 금지 사항 명문화**

### 제외 범위 (Out of Scope)

* 각 Projection의 상세 API 설계
* React/Vue 컴포넌트 구현
* GraphQL 스키마 생성 로직
* Agent 연동 로직
* 성능 최적화 및 벤치마크

> 본 PRD는 **“구조를 고정하는 것”**이 목표이며,
> Projection의 기능 구현은 이후 PRD에서 다룬다.

---

## 3. 아키텍처 원칙 (Architecture Principles)

### 3.1 단방향 의존성

```
@manifesto-ai/projection-*  ───▶  @manifesto-ai/core
```

* Projection은 **core만 의존**한다
* agent, 다른 projection 패키지에 의존하지 않는다
* core는 projection의 존재를 알지 못한다

---

### 3.2 Projection의 역할 정의

Projection은 다음 역할만 수행한다:

* Snapshot → 소비자 친화적 표현으로 **변환**
* Domain Model → UI / API 모델로 **투영**
* 읽기 전용 View 생성

Projection은 다음을 **절대 수행하지 않는다**:

* 상태 변경 (Snapshot mutation ❌)
* Effect 실행 ❌
* Action 정의 ❌
* LLM 호출 ❌

---

### 3.3 Consumer-Agnostic Core 유지

* core는 Projection의 요구사항에 맞춰 변경되지 않는다
* Projection은 core의 public API만 사용한다
* Projection별 편의 로직은 **Projection 내부에서 해결**

---

## 4. 모노레포 구성 (Monorepo Structure)

### 4.1 루트 구조

```
@manifesto-ai/projection/
├─ packages/
│  ├─ react/
│  │  └─ package.json
│  ├─ vue/
│  │  └─ package.json
│  ├─ graphql/
│  │  └─ package.json
│  └─ shared/
│     └─ package.json
│
├─ configs/
│  ├─ tsconfig/
│  │  └─ base.json
│  ├─ eslint/
│  │  └─ base.cjs
│  └─ build/
│     └─ tsup.config.ts
│
├─ package.json
├─ pnpm-workspace.yaml
├─ turbo.json
└─ README.md
```

---

### 4.2 패키지 역할

#### `packages/react`

* React 환경에서 Snapshot을 UI 상태로 투영
* Hook / Adapter / ViewModel 제공
* React-specific 의존성 허용

#### `packages/vue`

* Vue 환경에서 Snapshot을 UI 상태로 투영
* Composition API 기반 Adapter 제공

#### `packages/graphql`

* Domain Model / Snapshot을 GraphQL Schema로 투영
* Query / Mutation 생성 지원

#### `packages/shared`

* Projection 간 **공통 유틸리티**
* 순수 함수만 포함
* 프레임워크 의존성 ❌

---

## 5. 패키지 규칙 (Package Rules)

### 5.1 공통 규칙

모든 `@manifesto-ai/projection-*` 패키지는 다음을 만족해야 한다:

* `@manifesto-ai/core`만 runtime 의존성으로 허용
* public API는 명확히 export
* Snapshot은 **읽기 전용**
* Effect / Runtime import 금지

---

### 5.2 package.json 기본 형태

```json
{
  "name": "@manifesto-ai/projection-react",
  "version": "0.1.0",
  "private": false,
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "dependencies": {
    "@manifesto-ai/core": "^0.1.0"
  }
}
```

---

## 6. 빌드 및 개발 환경

### 6.1 패키지 매니저

* **pnpm**
* workspace 기반 의존성 관리

`pnpm-workspace.yaml`:

```yaml
packages:
  - "packages/*"
  - "configs/*"
```

---

### 6.2 빌드 오케스트레이션

* **turbo**
* 패키지 간 캐시 및 병렬 빌드

`turbo.json` (초기):

```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "lint": {},
    "test": {}
  }
}
```

---

### 6.3 TypeScript 설정

* 공통 `base tsconfig`를 `configs/tsconfig/base.json`에 정의
* 각 패키지는 이를 extend

---

## 7. README 구성 가이드 (루트)

루트 `README.md`에는 다음만 포함한다:

* 이 monorepo의 목적
* Projection의 개념 요약
* 포함된 패키지 목록
* core / agent와의 관계 다이어그램

> 상세 사용법은 각 패키지 README에서 다룬다.

---

## 8. 성공 기준 (Success Criteria)

이 PRD가 완료되었다고 판단하는 기준:

* [ ] pnpm + turbo 기반 monorepo가 정상 동작한다
* [ ] Projection 패키지들이 독립적으로 빌드된다
* [ ] core 외의 의존성이 없다
* [ ] 새로운 projection 패키지를 쉽게 추가할 수 있다
* [ ] core / agent와의 경계가 명확하다

---

## 9. 다음 단계 (Next Steps)

* `@manifesto-ai/projection-react` PRD 작성
* Projection 공통 인터페이스 초안 정의
* core public API surface 점검
* Example App (optional) 분리 검토

---

*This document defines the structural foundation of the `@manifesto-ai/projection` monorepo.
Implementation details are intentionally deferred.*

