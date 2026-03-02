"HTML Form 전송" 방식이 "전통적인 AJAX/Fetch POST 요청"과 비교했을 때 아키텍처적으로 어떤 비평을 받고 있는지

**"아키텍처적으로 파편화되어(Disconnected) 보인다"**는 느낌은 정확히 **"암시적(Implicit) 데이터 흐름 vs 명시적(Explicit) 데이터 흐름"**의 논쟁입니다.

---
### **1. 비평의 핵심: Form 전송은 왜 "아키텍처적 퇴보"로 보이는가?**

전통적인 `fetch('/api/post')` 방식에 익숙한 개발자들이 Form 방식을 비판하는 주된 논리는 다음과 같습니다.

#### **① "네트워크 계층이 숨겨져 있다 (The Network Layer is Hidden)"**

- **Fetch 방식:** `axios.post()`를 호출하면 헤더, 바디, 응답 코드가 코드에 명시적으로 보입니다. "여기서 네트워크 요청이 나간다"는 것이 명확합니다.
    
- **Form 방식:** `<form action={fn}>`은 겉보기에 그냥 함수 호출 같습니다. **직렬화(Serialization), 네트워크 전송, 에러 핸들링** 과정이 Next.js 내부로 숨겨집니다(Magic).
    
- **비평:** "숨겨진 마법(Abstraction)"이 많을수록, 나중에 커스텀 헤더를 넣거나 복잡한 인증 로직을 태울 때 프레임워크와 싸워야 합니다.
    

#### **② "디버깅과 모니터링의 어려움"**

- **Fetch 방식:** 브라우저 Network 탭에서 깔끔한 JSON Payload를 볼 수 있습니다.
    
- **Form 방식:** Server Actions는 내부적으로 `multipart/form-data`나 Next.js 고유의 인코딩 방식을 씁니다. 페이로드를 확인하고 디버깅하기가 훨씬 까다롭습니다.
    

#### **③ "재사용성의 부재 (Lack of Reusability)"**

- **Fetch 방식:** API 엔드포인트(`/api/user`)는 웹, 모바일 앱, 타사 서비스 어디서든 호출할 수 있는 **"독립적인 자원"**입니다.
    
- **Form 방식:** Server Action 함수는 Next.js 클라이언트 컴포넌트와 **강하게 결합(Coupled)**되어 있습니다. 이를 모바일 앱에서 호출하려면 별도의 API를 또 만들어야 하는 **이중 작업**이 발생할 수 있습니다.
    

---

### **2. 핵심 논쟁을 다룬 글 (접속 확인됨)**

이 주제(Form vs Fetch 아키텍처)를 깊이 있게 비평한 글들입니다.

#### **🔗 [비판] "Next.js의 추상화는 소프트웨어 엔지니어링 원칙을 위배하는가?"**

- **제목:** **Why Next.js Falls Short on Software Engineering**
    
- **저자:** Harshal Patil
    
- **내용:** 작성자님이 느끼신 "동떨어진 느낌"을 정확히 지적합니다.
    
    - Server Actions(Form)가 **"Client와 Server의 경계(Boundary)"**를 흐릿하게 만들어, 개발자가 코드를 짤 때 "이게 어디서 실행되는 거지?"라는 인지 부하(Cognitive Load)를 준다고 비판합니다.
        
    - 명시적인 API 계층이 사라짐으로써 발생하는 구조적 혼란을 다룹니다.
        
- **링크:** [https://blog.webf.zone/why-next-js-falls-short-on-software-engineering-d3575614bd08](https://blog.webf.zone/why-next-js-falls-short-on-software-engineering-d3575614bd08)
    

#### **🔗 [옹호] "데이터 변이(Mutation)의 본질: Form은 네비게이션이다"**

- **제목:** **Remix Data Flows (Form vs Fetch)**
    
- **출처:** Remix Documentation (Next.js Form 철학의 원류)
    
- **내용:** 왜 굳이 `fetch`를 버리고 `Form`으로 돌아갔는지 아키텍처 관점에서 설명합니다.
    
    - **핵심 논리:** `fetch`를 쓰면 로딩 상태, 에러 처리, 데이터 갱신(Revalidation)을 개발자가 수동으로 관리해야 하지만, `Form`을 쓰면 브라우저와 서버가 이를 자동으로 처리해준다는 **"선언적(Declarative) 아키텍처"**의 장점을 설명합니다.
        
- **링크:** [https://remix.run/docs/en/main/discussion/data-flow](https://remix.run/docs/en/main/discussion/data-flow)
    

#### **🔗 [보안 비평] "Form Action은 숨겨진 공용 API다"**

- **제목:** **Next.js Server Action Security**
    
- **출처:** Arcjet Blog
    
- **내용:** 아키텍처적으로 Form Action을 "내부 함수"처럼 취급할 때 생기는 위험성을 경고합니다.
    
    - 개발자는 Form을 UI의 일부로 보지만, 해커는 이를 **공개된 POST 엔드포인트**로 봅니다. 이 괴리감에서 오는 보안 취약점을 다룹니다.
        
- **링크:** [https://blog.arcjet.com/next-js-server-action-security/](https://blog.arcjet.com/next-js-server-action-security/)
    

---

### **3. 결론: "동떨어진 느낌"을 해소하는 아키텍처 제안**

작성자님의 느낌은 **"API가 UI에 종속되어 파편화되는 현상"** 때문입니다. 이를 해결하려면 Form을 쓰더라도 **"API처럼 관리하는 패턴"**이 필요합니다.

**추천 패턴: "Server Actions as Controllers"**

Form Action을 비즈니스 로직이 아닌, 단순한 **연결 통로(Controller)**로만 사용하세요.

TypeScript

```
// 1. services/post.ts (순수 비즈니스 로직 / 재사용 가능 / 테스트 가능)
// 모바일 앱용 API 라우트에서도 이걸 import 해서 씀
export async function createPostService(data: PostData) {
  return db.post.create({ data });
}

// 2. actions/post.ts (Form을 위한 연결 통로)
// 여기서는 오직 "Form 데이터 파싱"과 "캐시 갱신"만 담당
'use server'
import { createPostService } from '@/services/post';
import { revalidatePath } from 'next/cache';

export async function createPostAction(formData: FormData) {
  const data = parse(formData); // Zod 검증 등
  await createPostService(data);
  revalidatePath('/posts'); // Next.js 특화 로직
}

// 3. UI (Form)
// UI는 Action만 바라봄
<form action={createPostAction} />
```

이렇게 계층을 나누면, Form 방식의 편리함(캐시 갱신)은 누리면서도, 핵심 로직은 전통적인 API 구조처럼 **중앙 집중형**으로 관리할 수 있습니다.