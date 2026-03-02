> https://nextjs-ko.org/docs/app/api-reference/functions/revalidatePath

`revalidatePath` 함수는 특정 경로에 대해 [캐시된 데이터](https://nextjs-ko.org/docs/app/building-your-application/caching)를 필요에 따라 무효화할 수 있게 해줍니다.

**참고 사항**:

- `revalidatePath`는 [Node.js와 Edge 런타임](https://nextjs-ko.org/docs/app/building-your-application/rendering/edge-and-nodejs-runtimes) 모두에서 사용할 수 있습니다.
- `revalidatePath`는 포함된 경로가 다음에 방문될 때 캐시를 무효화합니다. 즉, 동적 경로 세그먼트를 가진 경로로 `revalidatePath`를 호출해도 즉시 많은 재검증이 발생하지 않습니다. 무효화는 경로가 다음에 방문될 때 발생합니다.
- 현재 `revalidatePath`는 [클라이언트 측 라우터 캐시](https://nextjs-ko.org/docs/app/building-your-application/caching#client-side-router-cache)의 모든 경로를 무효화합니다. 이 동작은 임시적이며, 특정 경로에만 적용되도록 업데이트될 예정입니다.
- `revalidatePath`를 사용하면 [서버 측 라우트 캐시](https://nextjs-ko.org/docs/app/building-your-application/caching#full-route-cache)에서 **특정 경로**만 무효화됩니다.