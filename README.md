# 202130311 안상하

## 2025.10.22 10주차

### server 및 client component 
* `인터리빙(Interleaving)`은 일반적으로 여러 데이터 블록이나 비트를 섞어서 전송하거나 처리하여 오류 발생 시 영향을 최소화하는 기술
* 특히 데이터 통신에서 버스트 오류(연속적인 오류)를 줄이고 오류 정정 코드를 효과적으로 사용하기 위해 사용됩니다.
* 프로그래밍이나 문서에서는 server 컴포넌트와 client 컴포넌트가 섞여서(interleaved) 동작하는 것을 의미
  * server component를 client component에 props를 통해 전달할 수 있습니다.
  * 이를 통해 client component 내에서 server에서 렌더링된 UI를 시각적으로 중첩할 수 있음
  * `ClientComponent`에 공간(slot)을 만들고 children을 끼워넣는 패턴이 일반적
* 예를 들어, client의 state를 사용하여 표시 여부를 전환(toggle)하는 `<Modal>` component 안에 server에서 데이터를 가져오는 `<Cart>` component가 있음
* 그 다음 부모 server component`(예: <Page>)` 안에 `<Modal>`의 자식으로 `<Cart>`를 전달할 수 있음
* Modal을 불러오는 곳이 Page이기 때문에 Page가 parent가 되는 것
~~~ts
// app/page.tsx
import Modal from './ui/modal'
import Cart from './ui/cart'

export default function Page() {
  return (
    <Modal>
      <Cart />
    </Modal>
  )
}
~~~

### Context란 무엇인가?
* 전역 상태 관리
  * Context를 사용하면 애플리케이션 전체에서 공유해야 하는 데이터를 중앙 집중적으로 관리할 수 있음 (예: 사용자 정보, 테마 설정 등)
* props drilling 문제 해결
  * 컴포넌트 트리가 깊어질수록 props를 계속 전달해야 하는 번거로움을 줄여줌
  * Context를 사용하면 필요한 컴포넌트에서 바로 데이터를 가져올 수 있으므로, 코드의 가독성을 높이고 유지 보수를 용이
* React 컴포넌트에서 사용
  * Context는 React에서 제공하는 기능이기 때문에 Next.js에서도 React 컴포넌트를 사용하여 구현

### Context provider (컨텍스트 제공자)
* React Context는 일반적으로 아래 테마처럼 전역 상태를 공유하는데 사용
* 그러나 server component에서는 React Context가 지원되지 않음
* Context를 사용하려면 children을 허용하는 client component로 만들어야 함
~~~ts
// app/theme-provider.tsx

'use client'

import { createContext } from 'react'

export const ThemeContext = createContext({})

export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode
}) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
~~~

* 추가된 부분
~~~ts
'use client'

import { createContext, useEffect, useState } from 'react'

export const ThemeContext = createContext<{
  theme: 'light' | 'dark'
  toggleTheme: () => void
}>({
  theme: 'light',
  toggleTheme: () => {},
})

export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode
}) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light')

  useEffect(() => {
    if (typeof window !== 'undefined') {
      document.documentElement.dataset.theme = theme
    }
  }, [theme])

  const toggleTheme = () => { setTheme((t) => (t === 'dark' ? 'light' : 'dark')) }
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}
~~~


## 2025.10.17 9주차

### Introduction
* 기본적으로 `layout`과 `page`는 `server component`입니다.
* `server`에서 데이터를 가져와 UI의 일부를 렌더링할 수 있고, 선택적으로 결과를 `cache`한 후 `client`로 스트리밍할 수 있습니다.
* 상호작용이나 브라우저 API가 필요한 경우 `client component`를 사용하여 기능을 계층화

### server 및 client component를 언제 사용해야 하나
* `client` 환경과 `server` 환경은 서로 다른 기능을 가지고 있음
* `server` 및 `client component`를 사용하면 사용하는 사례에 따라 각각의 환경에서 필요한 로직을 실행할 수 있음
  * 다음과 같은 항목이 필요할 경우에는 `client component`를 사용합니다.
    * state 및 event handler. `예)` `onClick` `onChange`
    * lifecycle logic. `예)` `useEffect`
    * 브라우저 전용 API. `예)` `localStorage` `window` `Navigator.geolocation` 등
    * 사용자 정의 `Hook`
  * 다음과 같은 항목이 필요할 경우에는 `server component`를 사용
    * 서버의 데이터베이스 혹은 API에서 data를 가져오는 경우 사용
    * API key, token 및 기타 보안 데이터를 client에 노출하지 않고 사용
    * 브라우저로 전송되는 `JavaScript`의 양을 줄이고 싶을 때 사용
    * 콘텐츠가 포함된 첫 번째 페인트`(First Contentful Paint-FCP)`를 개선하고, 콘텐츠를 client에 점진적으로 스트리밍

### server 및 client component를 언제 사용해야 하나요?
* 예를 들어 `<Page> component`는 게시물에 대한 데이터를 가져와서, `client` 측 상호 작용을 처리하는 `<LikeButton>에 props`로 전달하는 `server component`
* `@/ui/like-button`은 `client component`이기 때문에 `use client`를 사용하고 있음

~~~ts
// app/[id]/page.tsx
import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id)

  return (
    <div>
      <main>
        <h1>{post.title}</h1>
        {/* ... */}
        <LikeButton likes={post.likes} />
      </main>
    </div>
  )
}
~~~

~~~ts
// app/ui/like-button.tsx
'use client'

import { useState } from 'react'

export default function LikeButton({ likes }: { likes: number }) {
  // ...
}
~~~

### [ Optimistic Update(낙관적 업데이트) ]
* 사용자에 의해서 이벤트(예: 좋아요 버튼 클릭)가 발생 하면, 서버 응답을 기다리지 않고 클라이언트(브라우저)의 UI를 즉시 변경(업데이트)
* 서버에 보낸 요청의 성공을 낙관(optimistic)한다고 가정해서 먼저 화면에 변화를 보여줌
* 서버에서 응답이 없으면, UI를 원래 상태로(rollback).
* 네트워크 지연 동안에도 앱이 “빠르게 반응”하도록 느끼게 하는 것이 목적
* (장점)
  * 서버 응답 속도와 관계없이 즉각적인 피드백을 제공하여 사용자 경험을 향상
  * 네트워크 상태가 나쁘거나 응답 시간이 길어도 사용자에게 체감되는 속도가 빠름
* (단점)
  * 서버에서 오류가 발생하면, 사용자에게는 잠시 동안 잘못된 정보가 표시될 수 있음
  * 오류 발생 시 복구 로직이 필요

### [ Pessimistic Update(비관적 업데이트) ]
* 이벤트가 발행하면 먼저 서버에 요청을 보내고, 서버에서 성공 응답을 받은 후에 클라이언트의 UI를 업데이트
* (장점)
  * 서버의 응답을 기반으로 하기 때문에 데이터의 일관성이 보장
  * 오류가 발생할 가능성이 낮고, 잘못된 정보가 표시될 염려가 없음
* (단점)
  * 사용자는 서버의 응답을 기다려야 하므로, 응답이 늦어지면 사용자 경험이 저하될 수 있음
  * 특히 네트워크 지연이 발생할 경우 체감 속도가 느려짐

### like-button.tsx
* `@/ui/like-button.tsx`에서는 `state`를 2개 사용
* `count`는 `like 버튼`을 클릭한 횟수
* `isLiking`은 "서버에 요청이 진행 중인지"를 나타내는 `state`

### isLiking state의 주요 역할 
* 중복 클릭 방지 : isLiking이 true인 동안은 버튼을 disabled로 만들어 중복 요청 즉, 중복 낙관적 업데이트를 막는 역할
* UI 피드백 : 로딩 상태 표시(스피너나 문구)를 위해 사용이 가능
* 상태 안정화 : 서버에 요청이 끝날 때까지 추가 상태 변경을 잠시 멈추게 해서, 일관된 동작을 보장

### like-button.tsx
* 버튼을 클릭하면 handleClick이 호출되며, isLiking state를 **true**로, count state를 +1 변경
* line20에서 네트워크 시뮬레이션을 위해 isLiking state를 **300ms동안 true**를 유지
  * 즉, 버튼이 중복 클릭되는 것을 네트워크에 연결될 때까지 disable로 유지하기 위한 시뮬레이션
* 테스트 할 때는 300ms로는 힘들기 때문에 3000ms 정도로 조정해서 테스트 하는 것이 좋음 ➔ 3초 동안은 클릭해도 반응이 없음

~~~ts
const handleClick = async () => {
  // 낙관적 업데이트
  setIsLiking(true)
  setCount((c) => c + 1)

  // 실제 저장 로직(API 호출 등)이 있다면 이곳에서 호출할 수 있습니다.
  // 예: await fetch('/api/like', { method: 'POST', body: JSON.stringify({ id }) })

  // 예제에서는 짧은 지연 후 버튼 상태만 해제합니다.
  setTimeout(() => setIsLiking(false), 300)
}
~~~

### Next.js에서 server와 client component는 어떻게 작동할까?

* server component의 작동
  * server에서 Next.js는 React의 API를 사용하여 렌더링을 조정
  * 렌더링 작업은 개별 라우팅 세그먼트 별 묶음(Chunk)으로 나눔 ( layout 및 page )
  * server component는 `RSC Payload(React Server Component Payload)`라는 특수한 데이터 형식으로 렌더링 됩니다.
  * client component 와 RSC Payload는 HTML을 `미리 렌더링(prerender)`하는데 사용됩니다.
* React Server Component Payload(RSC)란 무엇인가?
  * RSC 페이로드는 렌더링된 React server component 트리의 압축된 바이너리 표현입니다.
  * client에서 React가 브라우저의 DOM을 업데이트하는데 사용

### RSC(RSC Payload)는 JSON인가, 바이너리인가?
* 과거 : JSON 기반
  * RSC 초기에는 JSON 형식의 문자열로 데이터를 전달 `예: { type: "component", props: { title: "Hello" } }`
* 현재 : 바이너리 형식으로 최적화
  * 최신 React, 특히 Next.js App Router는 RSC payload를 `compact binary format`으로 전송합니다.
  * JSON이 아니라, React 전용 이진 포맷으로 스트림(stream)을 통해 전달
  * 이 방식은 JSON보다 용량이 작고, 빠르게 파싱할 수 있음

### Next.js에서 server와 client component는 어떻게 작동할까?
* client component의 작동 (첫 번째 load)
  * HTML은 사용자에게 경로의 비대화형 미리보기를 즉시 보여주는데 사용
  * RSC 페이로드는 client와 server component 트리를 조정하는데 사용
  * JavaScript는 client component를 `hydration`하고, 애플리케이션을 대화형으로 만드는 데 사용
* Hydration이란 무엇인가?
  * `Hydration`은 이벤트 핸들러를 DOM에 연결하여 정적 HTML을 인터랙티브하게 만드는 React의 프로세스

### Next.js에서 server와 client component는 어떻게 작동합니까?
* 후속 네비게이션
  * 후속 탐색을 할 때 :
    * RSC 페이로드는 즉시 탐색할 수 있도록 `prefetch 및 cache`
    * client component는 server에서 렌더링된 HTML 없이 전적으로 client에서 렌더링

### 예시
* client component 사용
  * 파일의 맨 위, 즉 import문 위에 "use client" 지시문을 추가하여 client component를 생성할 수 있음
  * `use client`는 server와 client 모듈 트리 사이의 경계를 선언하는 데 사용
  * 파일에 "use client"로 표시되면 해당 파일의 모든 import와 자식 component는 client 번들의 일부로 간주
    * 즉, client를 대상으로 하는 모든 component에 이 지시문을 추가할 필요가 없음
  
~~~ts
// app/ui/counter.tsx

'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count} likes</p>
      <button onClick={() => setCount((count) => count + 1)}>Click me</button>
    </div>
  )
}
~~~

### client component 사용
* 문서의 코드는 /app/ui/counter.tsx를 작성했지만, src 디렉토리를 사용하는 경우는 다음과 같이 관리하는 것이 일반적
  * src/app/ 아래에는 라우팅 페이지만 작성하고 관리합니다.
  * 기타 사용자 정의 `component`나 `library`는 src/ 아래에 작성하고 관리

~~~ts
// src/components/counter.tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count} likes</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
~~~

### JS bundle 크기 줄이기
* client JavaScript 번들의 크기를 줄이려면 UI의 큰 부분을 client component로 표시하는 대신 특정 대화형 component에 “use client”를 추가
* 예를 들어, 다음 예제의 `<Layout> component`는 로고와 탐색 링크와 같은 정적 요소가 대부분이지만 대화형 검색창이 포함되어 있음
* `Search />`는 대화형이기 때문에 client component가 되어야 하지만, 나머지 layout은 server component로 유지될 수 있음
* `나머지 layout은 server component로 유지`

### JS 번들 크기 줄이기
`<Search />`는 사용자와의 상호작용이 바로 이루어질 가능성이 있기 때문에 Client Component로 사용하고, <Logo />는 상대적으로 중요하지 않고 이미지 등 용량이 크기 때문에 Server Component로 사용하는 것이 좋음

~~~ts
//app/layout.tsx

// Client Component
import Search from './search'
// Server Component
import Logo from './logo'

// Layout is a Server Component by default
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <Search />
      </nav>
      <main>{children}</main>
    </>
  )
}
~~~

~~~ts
// app/ui/search.tsx

'use client'

export default function Search() {
  // ...
}

~~~

### server에서 client component로 데이터 전달
* `props`를 사용하여 server component에서 client component로 데이터를 전달할 수 있음
* 앞에서 작성한 PostPage(/[id]/page.tsx) server component는 line28즈음에 client component인 LikeButton으로 `likes props`를 전달하고 있는 것을 확인할 수 있음

~~~ts
// app/[id]/page.tsx

import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id)

  return (
    <div>
      <main>
        <h1>{post.title}</h1>
        {/* ... */}
        <LikeButton likes={post.likes} />
      </main>
    </div>
  )
}
~~~

~~~ts
//app/ui/like-button.tsx

'use client'

export default function LikeButton({ likes }: { likes: number }) {
  // ...
}

~~~

### server에서 client component로 데이터 전달
* `알아두면 좋은 정보` : client component에 전달되는 Props는 React로 직렬화가 가능해야 함

### 직렬화(serialization)란 무엇인가?
* 일반적으로는 메모리에 있는 복잡한 데이터를 바이트의 연속 형태로 변환하는 과정을 말함
  * 즉, 자바스크립트의 객체나 배열처럼 구조가 있는 데이터를 파일로 저장하거나, 네트워크로 전송하기 쉽게 만드는 과정
* React나 Next.js 같은 프레임워크는 컴포넌트의 상태나 트리 구조를 서버에서 직렬화하여 클라이언트로 전송하고, 클라이언트에서 역직렬화 하는 과정을 자주 수행

## 2025.10.01 6주차

### Client-side transitions(클라이언트 측 전환)
* 일반적으로 서버 렌더링 페이지로 이동하면 전체 페이지가 로드
* Next.js는 `<Link>` 컴포넌트를 사용하는 클라이언트 측 전환을 통해 이를 방지
* 다음과 같은 방법으로 콘텐츠를 동적 업데이트
  * 공유 레이아웃과 UI 유지
  * 현재 페이지를 미리 가져온 로딩 상태 또는 사용 가능한 경우 새 페이지로 변경

### 동적 경로 없는 loading.tsx
* 동적 경로로 이동할 때 클라이언트는 결과를 표시하기 전 서버 응답을 기다림
  * 사용자 입장에서는 앱이 응답하지 않는다는 인상을 받을 수 있음
* 로딩 UI를 표시할려면 동적 경로에 loading.tsx를 추가하는 것이 좋음

### 동적 세그먼트 없는 generateStaticParams
* 동적 세그먼트는 사전 렌더링 될 수 있지만 `generateStaticParams`가 누락되어 사전 렌더링 되지 않는 경우, 해당 경로는 요청 시점에 동적 렌더링으로 대체

### await이 없어도 async를 붙여 두는 이유
* Next.js에서의 Server Component는 비동기 렌더링을 전제
* page.tsx 안에서 데이터를 fetch 하는 경우가 많아 async를 붙여도 문제 없음
1. 일관성 유지 : async function으로 일관성 유지
2. 확장성 : DB,API에서 데이터를 가져올 경우 수정할 필요가 없음
3. React Server Component 호환성 : async가 붙어 있어도 불필요한 오버헤드가 없음

### generateStaticParams 유무 비교
* generateStaticParams가 없는 경우
   * Next.js는 slug 값을 빌드 시점에는 알지 못함
   * slug 페이지에 접속하면, Next.js는 서버에서 요청이 올 때마다 해당 페이지를 동적으로 렌더링
   * 이 방식으로는 빌드 결과물로 HTML 파일이 생성되지 않음
* generateStaticParams가 있는 경우
  * Next.js에게 빌드 시점에 미리 생성할 slug 목록을 알려줄 수 있음
  * 지정된 slug에 대한 정적 HTML과 JSON 파일이 빌드 시점에 생성
  * 사용자가 처음 페이지에 접근할 때 서버 사이드 렌더링(SSR) 없이 미리 만들어진 페이지를 즉시 제공

### 느린 네트워크
* 네트워크가 느리거나 불안정한 경우 사용자가 링크를 클릭하기 전에 프리페칭이 완료되지 않을 수 있음
* 정적 경로, 동적 경로 둘 다 영향을 줄 수 있음
* loadimg.tsx 파일이 아직 프리페칭이 되지 않았기 때문에 즉시 표시되지 않을 수 있음
* 체감 성능을 개선하기 위해 `useLinkStatus Hook`을 사용하여 전환이 진행되는 동안 사용자에게 시각적인 피드백을 표시
* 예시
~~~ts
// app/loading-indicator.tsx

'use client'

import { useLinkStatus } from 'next/link'

export default function LoadingIndicator() {
  const { pending } = useLinkStatus()

  return pending ? (
    <div aria-label="loading" aria-live="polite" className="spinner" />
  ) : null
}
~~~
* 초기 애니메이션에 지연 시간(예: 100ms)을 추가하고, 초기 상태를 보이지 않게(opacity: 0) 설정하면 로딩 표시기를 `디바운스(debounce)` 할 수 있음
* 로딩 표시기는 내비게이션(페이지 전환)이 지정된 지연 시간보다 오래 걸리는 경우에만 표시
* `debounce란?`: 연속적으로 발생하는 이벤트를 그룹화하여, 마지막 이벤트가 발생한 후 일정 시간이 지나면 함수를 한 번만 실행하는 기술. 사용자 인터페이스에서 과도한 이벤트 발생을 막고 성능을 최적화하기 위해 사용
* 예시
~~~css
.spinner {
  opacity: 0;
  animation:
    fadeIn 500ms forwards,
    rotate 1s linear infinite;
  animation-delay: 100ms;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
~~~

### 프리페칭 비활성화
* `<Link>` 컴포넌트에서 `prefetch prop`을 `false`로 설정하여 프리패치를 사용하지 않도록 선택할 수 있음
* 대량의 링크 목록(예: 무한 스크롤 테이블)을 렌더링할 때 불필요한 리소스 사용을 방지하는 데 유용함
~~~html
<Link prefetch={false} href="/blog">Blog</Link>
~~~
* 프리패칭을 비활성화하면 다음과 같은 단점이 있음
  * 정적 라우팅: 사용자가 링크를 클릭할 때만 페이지를 가져옴
  * 동적 라우팅: 클라이언트가 해당 경로로 이동하기 전에 서버에서 먼저 렌더링
* 마우스 오버(hover) 시에만 프로그래밍 방식으로 프리패치를 사용하는 것이 좋음
* 사용자가 방문할 가능성이 높은 경로로만 프리패치를 제한할 수 있음

### Hydration이 완료되지 않음
* `<Link>`는 클라이언트 컴포넌트이기 때문에, 라우팅 페이지를 `프리패치(prefetch)`하기 전에 먼저 `하이드레이션(hydration)`되어야 함
* 초기 방문 시 내려받는 자바스크립트 번들 용량이 크면 하이드레이션이 지연되어 프리패칭이 바로 시작되지 않을 수 있음
* React의 `선택적 하이드레이션(selective hydration)`을 통해 이를 완화하여, 다음과 같은 방법으로 이점을 더욱 개선
  * `@next/bundle-analyzer` 플러그인을 사용하면 대규모 종속성을 제거하여 번들 크기를 식별하고 줄일 수 있음
  * 가능하다면 클라이언트에서 서버로 로직을 이동

### Hydration?
* 서버에서 생성된 정적 HTML에 JavaScript 로직을 추가하여 동적으로 상호작용이 가능하도록 만드는 과정을 의미
* React, Vue 등 프론트엔드 라이브러리나 프레임워크에서 많이 사용되는 용어. `서버 사이드 렌더링(SSR)`으로 생성된 정적 HTML에 클라이언트 측에서 JavaScript를 통해 이벤트 리스너, 상태 관리 등을 주입하여 인터랙티브한 웹 페이지로 변환하는 과정

### SSR과 Hydration
* SSR은 서버에서 미리 HTML을 생성하여 사용자에게 전달하는 방식
* 초기 로딩 속도가 빠르다는 장점이 있지만, 서버에서 생성된 HTML은 정적인 상태이므로 JavaScript 코드를 통해 동적인 상호작용을 구현하려면 추가적인 작업이 필요

### Hydration의 역할
* Hydration은 SSR로 생성된 정적 HTML에 클라이언트 측 JavaScript를 연결하여, 페이지가 로드된 후에도 사용자와의 상호작용이 가능하도록 만듬

### Examples - 네이티브 히스토리 API
* Next.js는 기본적으로 `window.history.pushState` 및 `window.history.replaceState` 메서드를 사용하여, 페이지를 다시 로드하지 않고도 브라우저의 기록 스택을 업데이트 할 수 있음
* `pushState` 및 `replaceState` 호출은 Next.js 라우터에 통합되어 `usePathname` 및 `useSearchParams`와 같은 훅(Hook)과 동기화

### window.history.pushState
* 이 함수를 사용하여 브라우저의 기록 스택에 새 항목을 추가 
* 사용자는 이전 상태로 돌아갈 수 있음
* 사용자는 이전 상태로 돌아갈 수 있음

### window.history.replaceState
* 브라우저의 기록 스택에서 현재 항목을 바꾸려 할 때 이 함수를 사용
* 사용자는 이전 상태로 돌아갈 수 있음
* 예시: 앱의 로케일(Locale)을 전환하는 경우에 사용
  * Locale: 사용자의 언어, 지역, 날짜/시간 형식, 숫자 표기법 등 사용자 인터페이스에서 사용되는 다양한 설정을 정의하는 문자열

## 2025.09.24 5주차

### searchParams
* URL의 쿼리 문자열을 읽는 방법
* 예: /products?category=shoes&page=2
  * 위에서 category=shoes, page=2가 `search parameters`

### 왜 동적 렌더링이 되는가?
* searchParams는 요청이 들어와야만 값을 알 수 있기에 Next.js는 이 페이지를 정적으로 미리 생성할 수 없고 요청이 올 때 마다 새로 렌더링을 함

### 코드 예시
~~~ts
export default async function ProductsPage({
  searchParams
}: {
  searchParams: Promise<{ id?: string; name?: string}>
}) {
  const {id = "non id", name = "non name" } = await searchParams;
  return (
    <div>
      <h1>Products Page</h1>
      <p>id : {id}</p>
      <p>name : {name}</p>
    </div>
  ) 
}
~~~

### Linking between pages(페이지 간 연결)
* `<Link>` 컴포넌트를 사용하여 경로 사이를 탐색할 수 있음
* `<Link>`는 HTML `<a>`태그를 확장하여 prefeching 및 client-side navigation 기능을 제공하는 Next.js의 기본제공 컴포넌트

### 네비게이션 작동 방식
* Server Rendering 
* Prefetching
* Streaming
* Client-side transitions 

### Server Rendering
* 초기 네비게이션 및 후속 네비게이션 할 때, 서버 컴포넌트 페이로드는 클라이언트로 전송되기 전에 서버에서 생성됨
* 정적 렌더링은 빌드 시점이나 재검증 중에 발생
* 동적 렌더링은 클라이언트 요청에 대한 응답으로 요청 시점에 발생
* Next.js는 사용자가 방문하 가능성이 높은 경로를 미리 가져 오고(Prefetching), 클라이언트 측 전환(Client-side transitions)을 수행하여 지연 문제를 해결

### Prefetching 미리 가져오기
* 사용자가 해당 경로로 이동하기 전에 백그라운드에서 해당 경로를 로드하는 프로세스
* 팔요한 데이터가 이미 준비되어 있기에 사용자 입장에서는 즉각적으로 느껴짐
* 미리 가져오는 경로의 양은 정적 경로인지 동적 경로인지에 따라 달라짐
  * 정적 경로: 전체 경로
  * 동적 경로: 프리페치를 건너 뛰거나, `loading.tsx`가 있는 경우 부분적으로

### Streaming
* 스트리밍을 사용하면 서버가 전체 경로가 렌더링될 때 까지 기다리지 않고, 동적 경로의 일부가 준비되는 즉시 클라이언트에 전송할 수 있음
* 페이지 일부가 로드 중이더라도 사용자는 더 빨리 콘텐츠를 볼 수 있음 
* Next.js는 백그라운드에서 page.tsx 콘텐츠를 `<Suspense>` 경계로 자동 래핑
* 미리 가져온 대체 UI는 경로가 로드되는 동안 표시되고 준비가 되면 실제 콘텐츠로 대체

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