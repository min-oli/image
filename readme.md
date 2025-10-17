# oy-global-web

# 📖 목차

1. [프로젝트 개요](#-프로젝트-개요)
2. [설치](#-설치)
3. [실행](#-실행)
   - [Static Remote 방식](#1-static-remote-방식)
   - [Development Remote 방식 (HMR)](#2-development-remote-방식-hmr)
   - [포트 설정](#3-포트-설정)
4. [Legacy(JSP) 환경 로컬 실행](#-legacyjsp-환경-로컬-실행)
   - [oy-global-web 실행](#1-oy-global-web-실행)
   - [JSP 파일의 host 스크립트 URL 수정](#2-jsp-파일의-host-스크립트-url-수정)
   - [JSP에서 remote 컴포넌트 로드](#3-jsp에서-remote-컴포넌트-로드)
     - [방법 1: id/className 사용](#방법-1-id와-classname-사용)
     - [방법 2: windowmf-객체-사용](#방법-2-windowmf-객체-사용)
   - [oy-global-front 실행](#4-oy-global-front-실행)
5. [아이콘 추가 및 생성](#-아이콘-추가-및-생성)
6. [Storybook](#-storybook)
   - [Storybook 배포](#storybook-배포)
8. [Mock Service Worker (MSW)](#-mock-service-worker-msw)
9. [배포](#-배포)
10. [참고 문서](#-참고-문서)

## 📌 프로젝트 개요

이 프로젝트는 Module Federation을 기반으로 하는 마이크로 프론트엔드 아키텍처를 사용합니다. `host`와 `host-legacy` 두 개의 컨테이너 애플리케이션을 중심으로, 각 기능별 `remote` 애플리케이션(예: `remote-home`, `remote-cart`)을 동적으로 로드합니다.

- **`host`**: 순수 React 환경을 위한 메인 애플리케이션입니다.
- **`host-legacy`**: 기존 JSP 환경 위에서 React 컴포넌트를 렌더링하기 위한 브릿지 역할을 하는 애플리케이션입니다.

현재 운영 중인 글로벌몰 서비스를 `host-legacy` 애플리케이션을 활용하여 점진적으로 마이그레이션 하고, 최종적으로는 `host` 애플리케이션으로 전부 마이그레이션하는 것이 목표입니다.

## 📌 설치

프로젝트 루트에서 아래 명령어를 실행하여 필요한 모든 패키지를 설치합니다.

```bash
yarn install
```

## 📌 실행

개발 환경에서는 두 가지 방식으로 서버를 실행할 수 있습니다.

### 1. Static Remote 방식

`host` 앱만 단독으로 실행하여 개발을 진행합니다. 이 방식은 `remote` 앱들의 빌드된 결과물을 정적으로 제공받아 사용하므로, `remote` 앱을 별도로 실행할 필요가 없습니다. `host` 자체의 UI나 로직 개발에 집중할 때 유용합니다.

```bash
nx serve host
# 또는
nx serve host-legacy
```

### 2. Development Remote 방식 (HMR)

`host`와 특정 `remote` 앱들을 동시에 실행하여 함께 개발을 진행합니다. 이 방식은 HMR(Hot Module Replacement)을 지원하여, `remote` 앱의 코드를 수정하면 페이지 전체를 새로고침하지 않고도 변경 사항이 `host`에 실시간으로 반영됩니다. 여러 모듈에 걸친 기능을 통합 개발할 때 유용합니다.

`--devRemotes` 플래그에 원하는 `remote` 앱 이름을 쉼표로 구분하여 전달합니다.

```bash
# host와 remote-home, remote-cart를 함께 실행
nx serve host --devRemotes=remote-home,remote-cart

# host-legacy와 remote-home, remote-cart를 함께 실행
nx serve host-legacy --devRemotes=remote-home,remote-cart
```

### 3. 포트 설정

로컬 개발 시 각 애플리케이션은 다음 포트를 사용합니다.

| 애플리케이션          | Development Mode | Static Mode |
| --------------------- | ---------------- | ----------- |
| `host`, `host-legacy` | `4200`           | `4200`      |
| `remote-home`         | `4201`           | `4100`      |
| `remote-cart`         | `4202`           | `4100`      |
| `remote-account`      | `4203`           | `4100`      |
| `remote-support`      | `4204`           | `4100`      |

## 📌 Legacy(JSP) 환경 로컬 실행

`host-legacy`는 기존 JSP 기반의 `oy-global-front` 프로젝트 위에서 React 리모트 컴포넌트를 렌더링하기 위해 사용됩니다. 로컬에서 JSP 페이지에 리모트 컴포넌트가 올바르게 표시되는지 확인하려면 다음 단계를 따르세요.

### 1. oy-global-web 실행

### 2. JSP 파일의 host 스크립트 URL 수정

JSP 페이지에 `host-legacy`의 JavaScript 파일을 로드하는 스크립트 주소를 로컬용으로 변경합니다.

**AS-IS**

```html
<script type="module" src="${Const.MF_CDN_URL}/host-legacy/main.js?v=<%= System.currentTimeMillis() %>"></script>
```

**TO-BE**

```html
<script type="module" src="http://localhost:4200/main.js"></script>
<script type="module" src="http://localhost:4200/runtime.js"></script>
```

> **⚠️** 배포 시에는 `runtime.js`를 제거하고, 스크립트 주소를 다시 `${Const.MF_CDN_URL}`로 원복해야 합니다.

### 3. JSP에서 remote 컴포넌트 로드

`host-legacy`는 두 가지 방법으로 remote 컴포넌트를 로드하고 렌더링합니다.

#### 방법 1: id와 className 사용

`div` 태그에 `remoteName/moduleName` 형식의 id와 `remote-component` class를 추가하면 `div` 태그 하위에 remote 컴포넌트가 렌더링됩니다.

```html
<div id="remote-home/KPopSection" class="remote-component">
  <!-- KPopSection 컴포넌트가 여기에 렌더링됩니다. -->
</div>
```

#### 방법 2: `window.mf` 객체 사용

`window.mf` 객체에 정의된 함수를 직접 호출하여 컴포넌트를 렌더링하는 방식입니다. Vue 환경과의 호환성 이슈를 해결하기 위해 추가되었습니다.

```javascript
const checkMFInitialized = callback => {
  if (window.mf?.renderRemoteComponent) {
    callback();
  } else {
    // 'mf-initialized' 이벤트는 host-legacy가 준비되면 발생합니다.
    document.addEventListener('mf-initialized', callback);
  }
};

checkMFInitialized(() => {
  // window.mf.renderRemoteComponent(
  //   '렌더링될_엘리먼트_ID',
  //   '리모트_앱_이름',
  //   '컴포넌트_이름'
  // );
  window.mf.renderRemoteComponent('recommendation-product-list', 'remote-home', 'RecommendationProductList');
});
```

- `host-legacy` 로드가 완료되면 `window.mf` 객체에 `renderRemoteComponent` 함수가 정의되고 `mf-initialized` 커스텀 이벤트가 발생합니다.
- 위 예제처럼 이벤트 리스너를 등록하거나 `window.mf` 객체를 확인하여 원하는 시점에 컴포넌트 렌더링을 트리거할 수 있습니다.

### 4. oy-global-front 실행

## 📌 아이콘 추가 및 생성

1.  `libs/shared/components/src/assets/icons` 디렉터리에 새로운 `.svg` 파일을 추가합니다.
2.  아래 명령어를 실행하여 아이콘 컴포넌트를 자동으로 생성합니다.

    ```bash
    nx generate-icons components
    ```

## 📌 Storybook

### Storybook 배포

- PR 생성 시 **Storybook Preview**가 자동 배포되며, 아래와 같이 PR 코멘트에 미리보기 링크가 표시됩니다.

  _(예: https://oyg-dev.github.io/oy-global-web/pr-713)_

- PR이 **머지되면**, 해당 Preview Storybook은 자동으로 **정리(삭제)** 됩니다.

- `main` 브랜치에 머지되면, 최신 Storybook이 자동으로 아래 경로에 배포됩니다.  
  🔗 [https://oyg-dev.github.io/oy-global-web/main](https://oyg-dev.github.io/oy-global-web/main)

## 📌 Mock Service Worker (MSW)

API 모킹이 필요한 경우, MSW를 사용하여 개발할 수 있습니다.

1.  `libs/shared/mocks` 디렉터리에 새로운 핸들러(handler)를 추가합니다.
2.  `MSW=true` 환경 변수와 함께 개발 서버를 실행합니다.

    ```bash
    MSW=true nx serve host-legacy --devRemotes=remote-home,remote-cart
    ```

## 📌 배포

- **Development (dev)**: `main` 브랜치에 Pull Request가 머지되면 자동으로 개발 환경에 배포됩니다.
- **Staging (stg) / Production (prd)**: 스테이징 및 운영 환경 배포는 필요시 GitHub Actions의 `manual-ci-cd` 워크플로우를 통해 수동으로 실행해야 합니다. (**⚠️**: `module-deploy` 워크플로우는 사용하지 마세요.)

## 🔗 참고 문서

- [글로벌프로덕트개발팀 Frontend 개발 컨벤션](https://oyitsm.cj.net/confluence/pages/viewpage.action?pageId=561366282)
- [BFF Home Screen Service Swagger](https://dev-bff-home-screen.oliveyoung.com/swagger-ui/index.html)
- [OYG Design System MO](https://www.figma.com/design/qsI8dRZRkyyYI5NIlTtjxM/Components-MO?node-id=0-1&p=f&t=ZmlBYM2QPuo5FhkX-0)
- [OYG Design System PC](https://www.figma.com/design/3cXG9najSHKHzzX6DRh0u2/Components-PC?node-id=0-1&p=f&t=DHDUOwVUl6MopFap-0)
