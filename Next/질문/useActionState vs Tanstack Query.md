**"Next.js의 내장 기능만으로 충분한가, 아니면 강력한 도구가 필요한가?"**의 싸움입니다.

결론부터 말씀드리면, **Next.js App Router 환경(Server Actions)에서는 `useActionState`가 표준**이고 훨씬 가볍습니다. 하지만 **복잡한 클라이언트 사이드 상태 관리나 낙관적 업데이트가 많다면 React Query**가 여전히 강력합니다.

이 둘의 차이를 **기능, 코드 스타일, 그리고 결정 기준**으로 명확히 비교해 드립니다.

---

### **1. 핵심 차이 한눈에 보기**

|**비교 항목**|**useActionState (React 내장)**|**TanStack Query (useMutation)**|
|---|---|---|
|**주 목적**|**Form 제출 결과** 및 UI 상태 관리|비동기 작업의 **전역 상태 관리** 및 캐싱|
|**데이터 범위**|**해당 컴포넌트 내부** (Local State)|**앱 전체** (Global Cache)|
|**데이터 갱신**|`revalidatePath` (서버 캐시 갱신)|`queryClient.invalidateQueries` (클라이언트 캐시 갱신)|
|**낙관적 UI**|`useOptimistic`과 함께 사용해야 함|`onMutate` 옵션으로 자체 지원|
|**번들 사이즈**|**0 KB** (React 내장)|약 13 KB (추가 라이브러리)|
|**난이도**|쉬움 (기본 훅 사용)|중간 (설정 및 보일러플레이트 필요)|

---

### **2. 코드 스타일 비교 (로그인 기능)**

#### **A. useActionState (Next.js 표준)**

- **특징:** 코드가 간결하고 `<form>` 태그와 자연스럽게 연결됩니다. JS가 없어도 동작(Progressive Enhancement)할 수 있습니다.
    

```typescript
'use client'
import { useActionState } from 'react';
import { loginAction } from './actions'; // Server Action

export default function LoginForm() {
  // state: 서버에서 리턴한 에러 메시지 등
  // formAction: 폼에 연결할 함수
  // isPending: 로딩 상태
  const [state, formAction, isPending] = useActionState(loginAction, null);

  return (
    <form action={formAction}>
      <input name="email" />
      <input name="password" />
      <button disabled={isPending}>로그인</button>
      {state?.error && <p>{state.error}</p>}
    </form>
  );
}
```

#### **B. TanStack Query (useMutation)**

- **특징:** `onSubmit` 핸들러를 따로 만들어야 하고 코드가 길어지지만, `onSuccess`, `onError` 같은 사이드 이펙트 제어가 매우 강력합니다.
    

```typescript
'use client'
import { useMutation } from '@tanstack/react-query';
import { loginAction } from './actions';

export default function LoginForm() {
  const mutation = useMutation({
    mutationFn: (formData: FormData) => loginAction(formData),
    onSuccess: (data) => {
      // 성공 시 토스트 메시지 띄우기, 페이지 이동 등
      toast.success('로그인 성공!');
    },
    onError: (error) => {
      toast.error('로그인 실패');
    }
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    mutation.mutate(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" />
      <input name="password" />
      <button disabled={mutation.isPending}>로그인</button>
      {mutation.error && <p>{mutation.error.message}</p>}
    </form>
  );
}
```

---

### **3. 언제 무엇을 선택해야 할까? (결정 가이드)**

#### **👉 `useActionState`를 선택하세요 (권장)**

1. **단순한 데이터 전송:** 로그인, 회원가입, 게시글 작성 등 "제출하고 끝나는" 기능.
    
2. **Next.js 캐시 활용:** `revalidatePath`를 통해 서버 데이터를 갱신하고 화면을 새로고침하는 흐름일 때.
    
3. **번들 사이즈 최적화:** 외부 라이브러리 없이 가볍게 가고 싶을 때.
    
4. **Form 중심 UI:** `<form>` 태그를 사용하는 표준적인 웹 UI일 때.
    

#### **👉 TanStack Query를 선택하세요**

1. **복잡한 사이드 이펙트:** 저장 성공 후 → 다른 쿼리를 다시 불러오고(refetch) → 토스트를 띄우고 → 모달을 닫는 등 **연쇄 작업**이 복잡할 때.
    
2. **낙관적 업데이트가 매우 빈번함:** 좋아요 버튼, 투두리스트 체크 등 서버 응답을 기다리지 않고 즉시 UI를 바꿔야 하는 인터랙션이 많을 때. (`useOptimistic`보다 `onMutate`가 더 직관적일 수 있음)
    
3. **폴링(Polling)이나 윈도우 포커스 재요청:** 서버 액션 자체보다는 데이터를 주기적으로 확인해야 하는 기능과 엮여 있을 때.
    
4. **이미 프로젝트에 React Query가 깔려 있을 때:** 일관성을 위해 굳이 섞어 쓰지 않고 Query로 통일하는 경우.
    

---

### **4. 실무적인 결론 (Hybrid 접근)**

최근 Next.js 실무 트렌드는 **"기본은 useActionState, 특수한 경우만 React Query"**입니다.

- **대부분의 Mutation(C/U/D):** `useActionState` + Server Actions를 사용하세요. 가장 깔끔하고 Next.js의 철학(서버 캐시)과 잘 맞습니다.
    
- **실시간성이 강한 UI:** 채팅, 실시간 대시보드, 복잡한 필터링 목록 등에는 **React Query**를 도입하여 클라이언트 측 경험을 강화하세요.
    