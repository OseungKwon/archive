
# 🏗️ Final Architecture Specification: Scalable Cart Demo

## 1. 핵심 철학 (Core Principles)

1. **React는 I/O 장치다:** React는 데이터를 화면에 그리고(Output), 이벤트를 감지하는(Input) 역할을 할 뿐, 비즈니스 로직의 주인이 아닙니다.
    
2. **Domain은 순수하다 (FP First):** `entities` 계층은 프레임워크(React, Next.js)와 무관한 **순수 TypeScript 함수**와 **Type**으로만 구성됩니다.
    
3. **의존성은 단방향으로:** `Infrastructure` → `Presentation` → `Application` → `Domain`. 중심부로 갈수록 고수준 정책이며 외부 변화에 영향을 받지 않습니다.
    
4. **Hybrid BFF:** 보안과 효율성의 균형을 위해 트래픽 성격에 따라 통신 경로를 이원화합니다.
    

---

## 2. 폴더 구조 (Directory Structure: FSD)

가장 중요한 **Next.js `app`**과 **FSD `pages`**의 역할 분리가 반영된 구조입니다.

Plaintext

```
src/
├── app/                        # [Infrastructure] Next.js App Router (Routing & Entry)
│   ├── api/cart/route.ts       # (BFF) 보안/쓰기 작업용 Route Handler
│   ├── cart/page.tsx           # (Controller) Page.tsx: 데이터 Fetching & 메타데이터
│   └── layout.tsx              # (Layout) Auth Context Provider 주입
│
├── pages/                      # [Presentation Layer] 순수 UI 조립 (View)
│   └── cart/
│       └── ui/CartPage.tsx     # (View) 프레임워크 독립적 페이지 컴포넌트
│
├── widgets/                    # [Composition Layer] 독립적인 기능 블록
│   └── cart-list/              # (e.g., 장바구니 리스트 + 요약)
│       └── ui/                 # Dumb Components (Props로 데이터 받음)
│
├── features/                   # [Application Layer] 유저 시나리오 (Use Cases)
│   └── add-to-cart/
│       ├── model/hook.ts       # Custom Hook (React Query + Logic 연결)
│       └── ui/AddToCartBtn.tsx # Smart Component
│
├── entities/                   # [Domain Layer] 순수 비즈니스 로직 (No React)
│   └── cart/
│       ├── model/types.ts      # Data Structure (Readonly)
│       └── lib/logic.ts        # Pure Functions (calcTotal, validateQty)
│
└── shared/                     # [Shared Layer] 공용 유틸리티
    ├── api/                    # HTTP Client (Fetch Wrapper & Interface)
    ├── lib/                    # FP Utils
    └── ui/                     # Atomic Design Components
```

---

## 3. 핵심 아키텍처 결정 사항 (Key Decisions)

### ① Controller(`app`) vs View(`pages`) 분리 전략

Next.js의 라우팅 시스템과 FSD의 구조적 명확성을 공존시키기 위한 전략입니다.

| **구분**  | **Next.js (src/app/**/page.tsx)**                                                                                | **FSD (src/pages/**/ui/Page.tsx)**                                         |
| ------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **역할**  | **Controller (제어자)**                                                                                             | **View (뷰)**                                                               |
| **책임**  | • URL 라우팅 및 파라미터 파싱<br><br>• 서버 사이드 데이터 Fetching (RSC)<br>  <br>• 메타데이터 (`generateMetadata`)<br>  <br>• 리다이렉트 처리 | • 위젯(Widgets) 배치 및 레이아웃<br><br>• Props로 받은 데이터 렌더링<br><br>• **비즈니스 로직 없음** |
| **의존성** | Next.js, Headers, Cookies                                                                                        | React, Pure Props                                                          |


```typescript
// src/app/cart/page.tsx (Controller)
export default async function Page() {
  const cartData = await fetchCart(); // Infra Access
  return <CartPage cart={cartData} />; // Inject into View
}
```

### ② 데이터 통신 및 흐름 (Hybrid BFF)

|**트랙 (Track)**|**대상**|**흐름 경로**|**역할**|
|---|---|---|---|
|**Secure / Write**|장바구니 담기, 결제|`Client` → `Route Handler` → `Backend`|토큰 은닉, DTO 매핑, 도메인 로직 보호|
|**Static / Read**|상품 목록, 상세 페이지|`RSC` → `Backend`|SEO 최적화, 초기 로딩 속도(LCP) 개선|
|**Fast / Read**|단순 검색, 필터링|`Client` → `Proxy (Rewrites)` → `Backend`|비용 절감, 불필요한 서버 연산 제거|

### ③ 상태 관리 전략 (State Management)

|**상태 종류**|**관리 도구**|**접근 방식**|
|---|---|---|
|**Server State**|**TanStack Query**|데이터의 최신성 보장, 캐싱, 중복 제거.|
|**Auth/Session**|**Context API**|변하지 않는 전역 데이터. `Provider` 패턴으로 주입 (No useEffect).|
|**Client UI**|**Zustand**|모달 열림, 드래그 상태 등 휘발성 UI 상태.|

---

## 4. 계층별 구현 상세 (Implementation Details)

### ① Infrastructure Layer (Shared API)

- **역할:** 외부 세상(HTTP)과의 통신 규격 정의.
    
- **특징:** 토큰 주입 로직을 `Strategy Pattern`으로 주입받아, 클라이언트/서버 환경 모두 대응.
    

```typescript
// shared/api/http-client.ts
export class FetchHttpClient implements IHttpClient {
  constructor(
    private baseUrl: string, 
    private getToken?: () => Promise<string | null> // Strategy Injection
  ) {} 
  // ... request implementation with 'Authorization' header injection
}
```

### ② Domain Layer (Entities)

- **역할:** 변하지 않는 비즈니스 규칙.
    
- **구현:** 100% 순수 함수(Pure Function). 테스트 커버리지 100% 목표.
    

TypeScript

```typescript
// entities/cart/lib/logic.ts
export const calculateTotal = (items: CartItem[]): number => 
  items.reduce((acc, item) => acc + item.price * item.quantity, 0);
```

### ③ Application Layer (Features)

- **역할:** 사용자의 의도(Intent)를 처리하고, 도메인과 인프라를 연결.
    
- **구현:** Custom Hook 내에서 순수 함수와 상태 관리 도구를 조합.
    

---

## 5. 시니어 개발자를 위한 Action Plan

이 명세서는 확장성과 유지보수성을 최우선으로 설계되었습니다. 이제 다음 단계로 개발을 시작하시면 됩니다.

1. **Foundation:** `shared/api/http-client.ts` 작성 (인터페이스 및 구현체).
    
2. **Domain:** `entities/cart/model` 및 `logic.ts` 작성 (순수 함수 테스트 포함).
    
3. **Infra (BFF):** `app/api/cart/route.ts` 작성 (백엔드 연결).
    
4. **View & Controller:** `pages/cart/ui/CartPage.tsx` 작성 후 `app/cart/page.tsx`에서 조립.
    
5. **Feature:** `features/add-to-cart` 구현 (UI + Hook 연결).
    

설계는 완벽합니다. 이제 코드로 실현할 시간입니다. 행운을 빕니다!