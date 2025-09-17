# 202130311 안상하

## 2025.09.17 4주차

### 페이지 만들기
* Next.js는 파일 시스템 기반 라우팅을 사용하기 때문에 폴더와 파일을 사용하여 경로를 정의

~~~ts
export default function RootPage() {
  return(
    <div>
      <h1>Hello World</h1>
    </div>
  )
}
~~~

### 레이아웃 만들기
* 레이아웃은 여러 페이지에서 공유 되는 UI
* 네비게이션에서 state 및 상호작용을 유지하며, 다시 렌더링 되지는 않음
~~~ts
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <header>header</header>
        <main>{children}</main>
        <footer>footer</footer>
      </body>
    </html>
  )
}
~~~

### 중첩 라우트 만들기
* 중첩 라우트는 다중 URL 세그먼트로 구성된 라우트
* 폴더는 URL 세그먼트에 매핑되는 경로 세그먼트를 정의하는데 사용

### `[slug]`의 이해
* `slug`는 사이트의 특정 페이지를 쉽게 읽을 수 있는 형태로 식별하는 URL의 일부
* 문서의 경로 `/blog/[slug]`의 `[slug]` 부분은 불러올 데이터의 key를 말함
* async function : 함수를 async 선언해야 내부에서 await를 쓸 수 있음
* 데이터 소스가 크타면 `.find`는 0(n)이므로 DB 쿼리로 바꿔야 함

### 동적 세그먼트 만들기
* 동적 세그먼트를 사용하면 데이터에서 사용된 경로를 만들 수 있음

### 검색 매개변수를 사용한 렌더링
* `searchParams`를 사용하면 해당 페이지는 동적 렌더링으로 처리됨
    * URL의 쿼리 파라미터(search parameters)를 읽기 위해 요청이(request) 필요
* 클라이언트 컴포넌트는 `useSearchParams` hook을 사용하여 검색 매개변수를 읽을 수 있음

## 2025.09.10 3주차

### 최상위 폴더 Top-level folders
* 최상위 폴더는 애플리케이션의 코드와 정적 자산을 구성
    * app : 앱 라우터
    * pages : 페이지 라우터
    * public : 제공될 정적 리소스
    * src : 선택적 애플리케이션 소스 폴더
* 라우팅 파일
* 중첩 라우팅
* 동적 라우팅
* 라우팅 그룹 및 비공개 폴더
* 병렬 및 차단 라우팅
* 앱 아이콘
* Open Graph 및 Twitter 이미지

### Open Graph Protocol
* 웹사이트나 페이스북, 인스타그램, X, 카카오톡 등에 링크를 전달할 때 `미리보기`를 생성하는 프로토콜

### component의 계층 구조
* 특수 파일에 정의된 component는 특정 계층 구조로 렌더링

### 프로젝트 구성
* component는 중첩된 라우팅에서 재귀적으로 렌더링
* 라우팅 세그먼트의 component는 부모 세그먼트의 component 내부에 중첩
* 프로젝트 파일을 app 디렉토리의 라우팅 세그먼트 내에 안전하게 배치하여 실수로 라우팅 되지 않도록 할 수 있음

### 비공개 폴더
* 비공개 폴더는 폴더 앞에 밑줄을 붙여서 만듬 `ex) _foldername`

### 라우팅 그룹
* 폴더를 괄호로 묶어 라우팅 그룹을 만들 수 있음
* 구성 목적으로 사용되는 것을 의미, 라우터의 URL 경로에 포함되지 않아야 함
* 페이지 섹션, 목적 별로 구성 할 때 유용

### 레이아웃에 특정 세그먼트 선택
* 특정 라우트를 레이아웃에 포함하려면 새 라우팅 그룹에 만들고 동일한 레이아웃을 공유하는 라우팅 폴더들을 이 그룹으로 이동
* 그룹 외부 라우팅 폴더에는 레이아웃을 공유하지 않음

### loding skeletons (스켈톤 로딩)
* 콘텐츠가 로드되기 전 콘텐츠가 표시 될 위치에 회색이나 반투명한 상자 또는 영역을 표시하여 시각적 안내를 주는 일종의 와이어 프레임 

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
* `LCP(Largest Contentful Paint)` : 뷰포트 내에서 가장 큰 페이지 요소를 표시하는 데 걸리는 시간
* `FID(First Input Delay)` : 사용자가 웹페이지와 상호작용을 시도하는 첫 번째 순간부터 웹페이지가 응답하는 시간
* `CLS(Cumulative Layout Shift)` : 방문자에게 콘텐츠가 얼마나 불안정한 지 측정한 값, 레이아웃 이동 빈도를 측정

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