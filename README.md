# 202130311 안상하

## 2025.09.03 2주차

### IDE 플러그 인
* VS CODE에서 플러그인을 활성화하는 방법 
    1. 명령 팔레트 열기
    2. "TypeScript: TypeScript 버전 선택" 검색
    3. "Use WorkSpace Version" 선택

### ESLint 설정
* Next.js에는 `ESLint`가 내장
* `create-react-app`으로 필요한 패키지를 자동으로 설치, 적절한 설정 구성
* 기존 프로젝트에 ESLint를 수동으로 추가할땐느 `package.json`에 `next lint` 스크립트 추가
~~~json
{
    "scripts" : {
        "lint" : "next lint"
    }
}
~~~

### Installation
* Strict : Next.js의 기본 ESLint 구성, Core Web Vitals 규칙 세트를 포함
    * 처음 설정하는 개발자에게 권장되는 구성
* Base : Next.js의 기본 ESLint 구성을 포함
* Cancel : 구성을 건너뜀. 사용자 지정 ESLint를 구성하고 싶다면 이 옵션을 선택

### import 및 모듈의 절대 경로 별칭 설정
* Next.js에는 `tsconfig.json` 및 `jsconfig.json` 파일의 `"paths"` 및 `"baseUrl"` 옵션에 대한 지원을 내장
* 프로젝트 디렉터리를 절대 경로로 별칭하여 모듈을 더 쉽고 깔끔하게 가져올 수 있음

### Core Web Vitals
* LCP(Largest Contentful Paint) : 뷰포트 내에서 가장 큰 페이지 요소를 표시하는 데 걸리는 시간
* FID(First Input Delay) : 사용자가 웹페이지와 상호작용을 시도하는 첫 번째 순간부터 웹페이지가 응답하는 시간
* CLS(Cumulative Layout Shift) : 방문자에게 콘텐츠가 얼마나 불안정한 지 측정한 값, 레이아웃 이동 빈도를 측정

### .eslintrc.json vs eslint.config.mjs
* json은 복잡한 설정이 어려움
* `mjs`는 ESLint가 새롭게 도입한 방식, ESM형식
* 확장자 `mjs`는 `"module JavaScript"`
* 조건문, 변수, 동적 로딩 등 코드처럼 유연한 설정 가능
* 다른 설정 파일을 import해 재사용 가능
* 프로젝트 규모가 커질수록 유지보수 유리

### pnpm
* 고성능 Node 패키지 매니저
* 디스크 공간 낭비, 복잡한 의존성 관리, 느린 설치 속도 문제 개선
* pnpm의 특징 중에 하드 링크를 사용해서 디스크 공간을 효율적으로 사용