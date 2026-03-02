Next.js에서 캐시가 **전체 공유(Shared)**인지 **유저 개별(Private)**인지 결정짓는 가장 큰 기준은 **"데이터가 어디에 저장되는가"**와 **"어떤 정보를 참조하는가"**입니다.

## 1. 전체 공유 캐시 (Shared Cache)

서버 컴퓨터에 저장되며, **전 세계 모든 유저가 동일한 결과물**을 나눠 갖습니다.

### **언제 발생하나?**

- **정적 데이터:** 공지사항, 상품 상세 정보, 블로그 포스트처럼 누가 접속해도 똑같은 내용을 보여줄 때.
- **빌드 시점 생성:** `npm run build` 할 때 이미 HTML이 만들어진 페이지.

### **주요 종류 및 코드**

1. **Full Route Cache (페이지 전체)**: 서버가 그린 HTML 결과물을 모든 유저에게 똑같이 던져줍니다.
2. **Data Cache (데이터 결과)**: API에서 받아온 JSON 데이터를 서버가 저장해두고 재사용합니다.

```typescript
// [전체 공유 예시] 상품 상세 페이지
// 어떤 유저가 들어와도 id가 1번인 상품은 똑같은 정보를 보여줍니다.
async function getProduct(id: string) {
  const res = await fetch(`https://api.com/products/${id}`, {
    cache: 'force-cache' // 이 데이터는 서버에 저장되어 모든 유저가 공유함
  });
  return res.json();
}

export default async function Page({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  return <div>{product.name}</div>;
}
```

---

## 2. 유저 개별 캐시 (Private Cache)

특정 유저의 **브라우저**나 **개별 요청 세션**에만 저장됩니다. 유저 A의 캐시가 유저 B에게 보일 일이 절대 없습니다.

### **언제 발생하나?**

- **개인화 데이터:** 내 프로필, 장바구니, 설정 등 "나"만 봐야 하는 정보일 때.
- **동적 요청:** 쿠키, 헤더, 검색 파라미터(`searchParams`) 등 유저마다 다른 정보를 바탕으로 응답할 때.

### **주요 종류 및 코드**

1. **Router Cache (브라우저 메모리)**: 내가 방문했던 페이지를 내 브라우저가 기억함.
2. **Request Memoization**: 내가 페이지 하나를 부를 때, 중복된 API 호출을 잠깐 합쳐줌.


```typescript
// [유저 개별 예시] 마이페이지
// 쿠키나 헤더를 쓰는 순간, Next.js는 "전체 공유 캐시"를 자동으로 해제합니다.
import { cookies } from 'next/headers';

export default async function MyPage() {
  const cookieStore = cookies();
  const sessionId = cookieStore.get('session-id'); // 유저별 쿠키 참조

  // 이 요청은 유저별로 다르게 처리되며, 서버에 공유 캐시를 남기지 않습니다.
  const res = await fetch('https://api.com/profile', {
    cache: 'no-store' // 혹은 유저별 데이터이므로 캐싱하지 않음
  });
  
  const profile = await res.json();
  return <div>안녕하세요, {profile.name}님!</div>;
}
```

---

## 3. 한눈에 비교하는 결정적 차이

|**구분**|**전체 공유 (Shared)**|**유저 개별 (Private)**|
|---|---|---|
|**저장 위치**|**서버** (Disk/Memory)|**유저 브라우저** 또는 **임시 메모리**|
|**주요 요인**|정적 경로, `force-cache`|쿠키, 헤더, `no-store`, `searchParams`|
|**보안**|공공 데이터용 (안전함)|개인 정보용 (보안 필수)|
|**속도**|**가장 빠름** (미리 그려둠)|빠름 (내 브라우저가 기억할 때만)|
|**Next.js 모드**|**Static Rendering**|**Dynamic Rendering**|

---

## 4. 실무 판단 가이드

1. **"로그인이 필요한가?"**
    
    - **Yes:** 유저 개별 캐시를 써야 합니다. (`cookies()` 사용 시 자동 전환)
        
    - **No:** 전체 공유 캐시를 써서 성능을 극대화하세요.
        
2. **"주소창 옆에 내 이름이 나오는가?"**
    
    - **Yes:** 무조건 유저별입니다. 전체 캐싱이 걸리면 다른 사람 이름이 나올 수 있습니다.
        
    - **No:** (예: 공지사항) 전체 캐싱을 걸어 서버 부하를 줄이세요.
        