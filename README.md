# AI Global Rules

Cursor와 Windsurf에서 공통적으로 사용하는 글로벌 규칙 설정 문서입니다.
해당 문서는 [ai-rules.md](https://gist.github.com/devbrother2024/30e2d7784316f367f06953070abf29d5)의 규칙을 기반으로 하며, 기존 Next.js 설정 규칙을 고도화 하고 React 공통 프로젝트에 적합한 세부 규칙과 React 전용 규칙을 추가하여 작성되었습니다.

## 설정 파일 생성

- Cursor: 최상단 루트 폴더에 `.cursorrules` 파일 생성
- Windsurf: 최상단 루트 폴더에 `.windsurfrules` 파일 생성

## Next.js / React 공통 규칙

### TypeScript 사용

- 프로젝트 전반에 TypeScript를 사용하세요
- 타입 안정성을 위해 모든 컴포넌트와 서버 로직에 TypeScript를 적용하세요

---

### TypeScript 인터페이스 정의 규칙

- 인터페이스 정의 시 이름 앞에 'I'를 접두사로 추가하세요
- 인터페이스 생성은 `types/index.ts` 파일에 작성하세요

```typescript
export interface IComment {
  id: string;
  text: string;
  author: string;
}
```

---

### 함수 작성 규칙

#### 단일 책임 원칙 (Single Responsibility)

- 단일 책임 원칙에 따라 함수는 하나의 작업만 수행하도록 작성하세요
- 함수 이름은 수행하는 작업을 명확하게 표현하세요

#### 순수 함수 지향

- 가능한 한 순수 함수로 작성하세요
- 부수 효과는 별도의 함수로 분리하세요

---

### 컴포넌트 생성

- 모든 UI 컴포넌트는 ShadCN을 우선으로 생성하세요
- ShadCN 컴포넌트 생성 CLI 명령어는 `npx shadcn@latest add`를 사용하세요
- Toast 관련 컴포넌트 구조:
  ```plaintext
  components/ui/toast.tsx      # Toast 기본 컴포넌트
  components/ui/toaster.tsx    # Toast 컨테이너 컴포넌트
  hooks/use-toast.ts          # Toast 커스텀 훅
  ```

---

### axios 사용 규칙

- 사용자 요청 시 **axios instance**를 생성하여 기본 베이스로 사용하세요
- API 요청 시 **try-catch 문 대신 React Query의 `onError` 옵션**을 사용하여 에러 처리를 위임하세요
- API 호출 함수는 순수 함수 형태로 작성하여 재사용성을 높이고, React Query와 결합하여 상태 관리를 진행하세요

---

### React Query 사용 규칙

- 사용자 요청 시 **TanStack Query(React Query)**를 사용하세요
- 쿼리 키는 명확하고 일관성 있게 정의하세요
- 상태가 자주 변경되지 않는 데이터는 `staleTime`과 `cacheTime`을 적절히 설정하세요
- API 요청 에러 처리는 React Query의 `onError` 옵션을 활용하세요
- `useQuery`는 데이터 조회에, `useMutation`은 데이터 생성, 수정, 삭제에 사용하세요

#### 병렬 쿼리 처리

- 여러 개의 병렬적인 쿼리를 동시에 실행해야 할 경우 `useQueries`를 사용하세요
- `useQueries`는 각 쿼리의 상태를 개별적으로 관리하며, 모든 쿼리의 결과를 배열로 반환하세요

### 전역 상태 관리 규칙

#### 기본 원칙

- 불필요한 전역 상태 관리는 최소화하세요
  - 컴포넌트 로컬 상태로 충분한 경우 `useState` 나 `useReducer`를 사용하세요
  - 컴포넌트 트리 깊이가 깊지 않은 경우 props drilling이 더 명확할 수 있습니다
- 전역 상태관리가 필요하다고 판단될 때 Zustand를 사용하세요
  - 복잡한 상태 로직이 필요한 경우
  - 여러 컴포넌트에서 공유되는 상태가 많은 경우

#### Zustand 스토어 구독 최적화

- 전체 스토어 구독은 피하고 필요한 상태만 선택적으로 구독하세요
- 여러 상태 구독 시 `useShallow` 훅과 배열 선택자를 사용하세요

```typescript
// ❌ 잘못된 예시: 전체 스토어 구독
const { nuts, honey } = useBearStore();

// ✅ 좋은 예시: Array 방식의 useShallow로 필요한 상태만 구독
const [nuts, honey] = useBearStore(
  useShallow((state) => [state.nuts, state.honey])
);
```

### 성능 최적화 가이드라인

#### 메모이제이션

- 비용이 많이 드는 계산에는 `useMemo`를 사용하세요
- 함수 props에는 `useCallback`을 사용하세요
- 순수 컴포넌트에는 `React.memo`를 사용하세요

#### 에셋 최적화

- 아이콘과 간단한 일러스트레이션에는 SVG를 사용하세요
- 적절한 이미지 로딩 전략을 구현하세요 (지연 로딩, 적절한 크기 조정)
- 가능하다면 최신 이미지 포맷을 사용하세요 (WebP와 대체 이미지)

### useEffect 사용 지양 케이스

#### Props 변경 시 State 초기화

```typescript
// ❌ 잘못된 예시
useEffect(() => {
  setComment('');
}, [userId]);

// ✅ 좋은 예시: key prop 사용
export default function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  const [comment, setComment] = useState('');
}
```

#### Props나 State로 파생된 State 계산

```typescript
// ❌ 잘못된 예시
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// ✅ 좋은 예시: 렌더링 중 계산
const fullName = firstName + ' ' + lastName;
```

#### props 또는 state에 따라 state 값을 업데이트하는 경우

```typescript
// ❌ 잘못된 예시
const [visibleTodos, setVisibleTodos] = useState([]);
useEffect(() => {
  setVisibleTodos(getFilteredTodos(todos, filter));
}, [todos, filter]);

// ✅ 좋은 예시: useMemo 사용
const visibleTodos = useMemo(
  () => getFilteredTodos(todos, filter),
  [todos, filter]
);
```

### Git 커밋 메시지 작성 규칙

**포맷:**

```
<type>: <subject>

<body>
```

**커밋 타입 (Type):**

- feat: 새로운 기능 추가
- fix: 버그 수정
- docs: 문서 수정
- style: 코드 포맷팅, 세미콜론 누락, 코드 변경이 없는 경우
- refactor: 코드 리팩토링
- test: 테스트 코드, 리팩토링 테스트 코드 추가
- chore: 빌드 업무 수정, 패키지 매니저 수정

**제목 (Subject):**

- 변경 사항에 대한 간단한 설명을 작성하세요
- 50자 이내로 작성하세요
- 마침표 없이 작성하세요
- 현재 시제를 사용하세요

**본문 (Body):**

- 변경 사항에 대한 자세한 설명을 작성하세요
- 어떻게 보다는 무엇을, 왜 변경했는지 설명하세요
- 여러 줄의 메시지를 작성할 땐 "-"로 구분하세요

**예시:**

```plaintext
feat: 로그인 화면 키보드 UX 개선
- TextInput ref를 사용하여 자동 포커스 기능 추가
- returnKeyType 설정으로 키보드 엔터키 동작 개선
- 전화번호 입력 후 자동으로 비밀번호 입력창으로 포커스 이동
- 비밀번호 입력 후 엔터키로 로그인 가능하도록 개선
```

## Next.js 프로젝트 규칙

### Route Handler 우선 사용

- 모든 API 엔드포인트는 Route Handler를 사용하여 구현하세요
- 데이터베이스 작업, 외부 API 호출, 인증 등 복잡한 서버 작업은 반드시 Route Handler를 사용하세요
- Server Action은 단순 폼 제출 또는 간단한 데이터 처리에만 사용하세요

### App Router 사용

- 프로젝트 내 라우팅은 Pages Router 대신 App Router를 사용하세요

### 프로젝트 구조

- Next 프로젝트 구조는 다음과 같이 설정하세요
- `src` 폴더는 사용하지 않습니다
- 이미 생성된 폴더 구조가 존재하는 경우 변경하지 않습니다

- **프로젝트 내 라우팅은 Pages Router 대신 App Router를 사용하세요.**

## 프로젝트 구조: 주요 폴더 구조 예시

- **Next 프로젝트 구조는 다음과 같이 설정하세요. `src` 폴더는 사용하지 않습니다.**
- **이미 생성된 폴더 구조가 존재하는 경우 변경하지 않습니다.**

```

your-nextjs-project/
│
├── app/                      # App Router 라우트 폴더
│ ├── api/                    # API 엔드포인트 관련 폴더
│ │ ├── dashboard/              # 개별 페이지 폴더 예시 (재사용되지 않는 컴포넌트 포함)
│ │ └─├── page.tsx              # dashboard 페이지
│ │   └── DashboardStats.tsx    # 페이지 전용 컴포넌트
│ │
│ ├── components/               # 공통 컴포넌트 모음
│ │ ├── ui                      # ShadCN 공통 UI 컴포넌트
│ │ │ ├── button.tsx
│ │ │ ├── input.tsx
│ │ │ ├── select.tsx
│ │ │ ├── toast.tsx
│ │ │ ├── toaster.tsx
│ │ │ ├── layout/                 # 레이아웃 관련 공통 컴포넌트
│ │ │ │ ├── header.tsx
│ │ │ │ ├── footer.tsx
│ │ │ │ ├── sidebar.tsx
│ │ │ │ ├── OptionsDropdown.tsx
│ │ │ │ ├── PromptInput.tsx
│ │ │ │ └── GeneratedImagePreview.tsx
│ │ │ │
│ │ │ │ ├── state/                    # 상태 관리 관련 폴더
│ │ │ │ │ ├── mutations/              # React Query mutations 폴더
│ │ │ │ │ │ ├── auth/                # 인증 관련 mutations
│ │ │ │ │ │ │ ├── useLoginMutation.ts
│ │ │ │ │ │ │ └── useSignupMutation.ts
│ │ │ │ │ │ └── community/           # 커뮤니티 관련 mutations
│ │ │ │ │ │   ├── useCreatePostMutation.ts
│ │ │ │ │ │   └── useUpdatePostMutation.ts
│ │ │ │ │ │
│ │ │ │ │ │ ├── queries/               # React Query queries 폴더
│ │ │ │ │ │ │ ├── auth/                # 인증 관련 queries
│ │ │ │ │ │ │ │ ├── useUserQuery.ts
│ │ │ │ │ │ │ │ └── useProfileQuery.ts
│ │ │ │ │ │ │ └── community/           # 커뮤니티 관련 queries
│ │ │ │ │ │ │   ├── usePostsQuery.ts
│ │ │ │ │ │ │   └── usePostDetailQuery.ts
│ │ │ │ │ │ │
│ │ │ │ │ │ ├── store/                 # Zustand store 폴더
│ │ │ │ │ │ │ ├── auth/                # 인증 관련 store
│ │ │ │ │ │ │ │ ├── useAuthStore.ts
│ │ │ │ │ │ │ │ └── types.ts
│ │ │ │ │ │ │ ├── ui/                  # UI 관련 store
│ │ │ │ │ │ │ │ ├── useUIStore.ts
│ │ │ │ │ │ │ │ └── types.ts
│ │ │ │ │ │ │ └── theme/               # 테마 관련 store
│ │ │ │ │ │ │   ├── useThemeStore.ts
│ │ │ │ │ │ │   └── types.ts
│ │ │ │ │ │ │
│ │ │ │ │ │ └── index.ts               # 상태 관리 유틸리티 및 타입 정의
│ │ │ │ │ │
│ │ │ │ │ ├── hooks/                    # 커스텀 훅 폴더
│ │ │ │ │ │ ├── use-toast.ts            # 토스트 관련 훅
│ │ │ │ │ │ ├── use-auth.ts             # 인증 관련 훅
│ │ │ │ │ │ └── use-media.ts            # 미디어 쿼리 등 UI 관련 훅
│ │ │ │ │ │
│ │ │ │ │ ├── prisma/                   # Prisma 관련 폴더
│ │ │ │ │ │ ├── schema.prisma          # Prisma 스키마 정의 파일
│ │ │ │ │ │ └── migrations/            # 데이터베이스 마이그레이션 파일들
│ │ │ │ │ │
│ │ │ │ │ ├── lib/                     # 유틸리티 및 설정 파일
│ │ │ │ │ │ ├── prisma.ts             # Prisma 클라이언트 설정
│ │ │ │ │ │ └── utils.ts              # 기타 유틸리티 함수
│ │ │ │ │ │
│ │ │ │ │ ├── public/                   # 정적 파일 (이미지, 폰트 등)
│ │ │ │ │ │ └── favicon.ico
│ │ │ │ │ │
│ │ │ │ │ ├── styles/                   # 글로벌 스타일 (CSS, SCSS, Tailwind 등)
│ │ │ │ │ │ └── globals.css
│ │ │ │ │ │
│ │ │ │ │ ├── types/                    # 공통 인터페이스 및 타입 정의
│ │ │ │ │ │ └── index.ts                # 여러 파일에서 사용할 공통 타입 및 인터페이스 정의 파일
│ │ │ │ │ │
│ │ │ │ │ ├── utils/                    # 유틸리티 함수 모음
│ │ │ │ │ │ ├── fetcher.ts              # API 호출 등 유틸리티 함수
│ │ │ │ │ │ └── mockData.ts             # 목업 데이터 관리
│ │ │ │ │ │
│ │ │ │ │ ├── middleware.ts             # 미들웨어 설정 파일
│ │ │ │ │ ├── .env                      # 환경 변수 설정 파일
│ │ │ │ │ ├── next.config.mjs           # Next.js 설정 파일
│ │ │ │ │ ├── package.json              # 프로젝트 패키지 정보
│ │ │ │ │ └── tsconfig.json             # TypeScript 설정 파일
│ │ │ │ │
│ │ │ │ └── index.ts                    # 프로젝트 진입점
│ │ │ │
│ │ │ └── index.ts                    # 프로젝트 진입점
│ │ │
│ │ └── index.ts                    # 프로젝트 진입점
│ │
│ └── index.ts                    # 프로젝트 진입점
│
```

### 서버 컴포넌트 / 클라이언트 컴포넌트 규칙

#### 서버 컴포넌트 관련

- 데이터 페칭이 필요한 경우 서버 컴포넌트를 사용하세요

```typescript
async function ServerComponent() {
  const data = await fetchData(); // 서버에서 직접 데이터 페칭
  return <div>{data}</div>;
}
```

- 민감한 정보를 다룰 때는 서버 컴포넌트를 사용하세요

```typescript
import 'server-only'; // 서버 전용 컴포넌트 강제

async function SecureComponent() {
  const data = await fetch(URL, {
    headers: { Authorization: process.env.API_KEY },
  });
  return <div>{/* ... */}</div>;
}
```

#### 클라이언트 컴포넌트 관련

- **서버 → 클라이언트 데이터 전달**

  ```typescript
  // ✅ 권장: 직렬화 가능한 데이터만 전달하세요
  export default async function ServerComponent() {
    const data = await fetchData();
    return <ClientComponent data={data} />;
  }
  ```

- **클라이언트 → 서버 컴포넌트 전달**

  ```typescript
  // ✅ 권장: props로 서버 컴포넌트를 전달하세요
  function Layout() {
    return (
      <ClientComponent>
        <ServerComponent />
      </ClientComponent>
    );
  }
  ```

  - 'use client' 지시문은 파일 최상단에 배치
  - 상태 관리나 이벤트 핸들링이 필요한 부분만 클라이언트 컴포넌트로 구현
  - 클라이언트 컴포넌트에서 서버 컴포넌트 직접 import는 하지 마세요

#### 클라이언트 컴포넌트 규칙

- 'use client' 지시문은 파일 최상단에 배치하세요
- 상태 관리나 이벤트 핸들링이 필요한 부분만 클라이언트 컴포넌트로 구현하세요
- 클라이언트 컴포넌트에서 서버 컴포넌트 직접 import는 하지 마세요

#### 성능 최적화

- 불필요한 클라이언트 컴포넌트 사용을 지양하세요
- 서버 컴포넌트를 최대한 활용하여 초기 페이지 로드를 최적화하세요
- 상태 관리 로직은 필요한 범위로 최소화하세요

---

### ORM: Prisma 사용

- 데이터베이스 작업을 위해 ORM으로 Prisma를 사용하세요
- Prisma를 사용하여 데이터베이스 모델을 정의하고, CRUD 작업을 구현하세요
- Prisma Studio를 활용하여 데이터베이스를 관리하세요

#### Prisma 설정

- 프로젝트 루트에 `prisma` 폴더를 생성하세요
- `schema.prisma` 파일에서 데이터베이스 스키마를 정의하세요
- 클라이언트 인스턴스는 싱글톤 패턴으로 관리하세요

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

#### Prisma 사용 예시

```typescript
// 데이터 조회
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: true },
});

// 데이터 생성
const newPost = await prisma.post.create({
  data: {
    title,
    content,
    authorId: userId,
  },
});

// 트랜잭션 처리
const [post, notification] = await prisma.$transaction([
  prisma.post.create({ data: postData }),
  prisma.notification.create({ data: notificationData }),
]);
```

---

### Clerk 인증: clerkMiddleware() 사용

- 모든 인증은 Clerk을 사용하세요
- middleware.ts 파일에서는 **clerkMiddleware()**를 사용하세요
- authMiddleware는 사용하지 마세요

#### 기본 미들웨어 설정

```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher(['/sign-in(.*)', '/sign-up(.*)']);

export default clerkMiddleware(async (auth, request) => {
  if (!isPublicRoute(request)) {
    await auth.protect();
  }
});

export const config = {
  matcher: ['/((?!.*\\..*|_next).*)', '/', '/(api|trpc)(.*)'],
};
```

---

### ClerkClient: 유저 정보 조회 규칙

- ClerkClient를 사용하여 유저 정보를 조회할 때는 다음 규칙을 따르세요

```typescript
import { clerkClient } from '@clerk/nextjs/server';

const client = await clerkClient();

// 단일 유저 조회
const user = await client.users.getUser(userId);

// 다수 유저 조회 (권장)
const users = await client.users.getUserList({
  userId: userIds, // string[] 타입
});
```

---

## React 규칙

### 폴더구조

- 이미 생성된 폴더 구조가 존재하는 경우 기존 폴더 구조를 유지하세요

```plaintext
your-react-project/
│
├── src/                      # 소스 코드의 루트 디렉토리
│ ├── pages/                  # 페이지 컴포넌트
│ │ ├── Dashboard/           # 페이지별 폴더
│ │ │ ├── index.tsx         # 메인 페이지 컴포넌트
│ │ │ ├── DashboardStats.tsx # 페이지 전용 컴포넌트
│ │ │ └── styles.ts         # 페이지별 스타일
│ │ └── Auth/
│ │   ├── Login.tsx
│ │   └── Register.tsx
│ │
│ ├── components/            # 공통 컴포넌트
│ │ ├── ui/                 # UI 컴포넌트
│ │ │ ├── Button/
│ │ │ │ ├── index.tsx
│ │ │ │ └── styles.ts
│ │ │ ├── Input/
│ │ │ ├── Select/
│ │ │ └── Toast/
│ │ ├── layout/            # 레이아웃 컴포넌트
│ │ │ ├── Header/
│ │ │ ├── Footer/
│ │ │ └── Sidebar/
│ │ └── common/           # 기타 공통 컴포넌트
│ │   ├── OptionsDropdown/
│ │   └── PromptInput/
│ │
│ ├── state/               # 상태 관리
│ │ ├── store/            # Zustand store
│ │ │ ├── auth/
│ │ │ │ ├── useAuthStore.ts
│ │ │ │ └── types.ts
│ │ │ ├── ui/
│ │ │ └── theme/
│ │ └── queries/          # React Query
│ │   ├── auth/
│ │   └── posts/
│ │
│ ├── hooks/              # 커스텀 훅
│ │ ├── useToast.ts
│ │ ├── useAuth.ts
│ │ └── useMedia.ts
│ │
│ ├── services/          # API 서비스
│ │ ├── api.ts          # axios 인스턴스 설정
│ │ ├── auth.ts         # 인증 관련 API
│ │ └── posts.ts        # 게시글 관련 API
│ │
│ ├── styles/           # 글로벌 스타일
│ │ ├── global.ts      # 글로벌 스타일
│ │ ├── theme.ts       # 테마 설정
│ │ └── variables.ts   # 스타일 변수
│ │
│ ├── types/           # 타입 정의
│ │ ├── auth.ts
│ │ ├── post.ts
│ │ └── index.ts
│ │
│ ├── utils/           # 유틸리티
│ │ ├── constants.ts   # 상수
│ │ ├── helpers.ts     # 헬퍼 함수
│ │ └── validation.ts  # 유효성 검사
│ │
│ ├── routes/          # 라우트 설정
│ │ ├── PrivateRoute.tsx
│ │ ├── PublicRoute.tsx
│ │ └── index.tsx
│ │
│ └── App.tsx          # 앱 진입점
│
├── public/            # 정적 파일
│ ├── images/
│ ├── fonts/
│ └── favicon.ico
│
├── .env              # 환경 변수
├── package.json
├── tsconfig.json
└── README.md
```

### 코드 스플리팅

- 라우트 기반 코드 분할에 따라 필요시 React.lazy 사용하세요
- 라우트 레벨에서 Suspense 경계를 사용하세요
- 큰 서드파티 라이브러리는 동적 임포트를 사용하세요

#### lazy loading 적용 기준

- 초기 로딩에 필수적이지 않은 페이지에 적용하세요
- 번들 사이즈가 큰 페이지 (복잡한 차트, 에디터 등)에 적용하세요
- 사용자가 자주 방문하지 않는 페이지 (관리자 페이지, 설정 페이지 등)에 적용하세요
