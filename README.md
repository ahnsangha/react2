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