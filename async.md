# Async

Jotai는 최고수준의 비동기 지원을 제공하며 이를 위해 React의 Suspense를 최대로 활용한다.

> 기술적으로 React.lazy 이외의 Suspense 사용은 React 17에서 여전히 지원되지 않거나 문서화되지 않았다. 만약 이 기능이 차단된다면 [guides/no-suspense](https://jotai.org/docs/guides/no-suspense) 참고하자.

## Suspense

비동기 atom을 사용하기 위해서는 React 컴포넌트 트리를 `<Suspense>`로 감싸야 한다. 만약 `<Provider>` 를 사용중이라면 적어도 하나의 `<Suspense>`를 그 안에 위치시켜야 한다.

```tsx
const App = () => (
  <Provider>
    <Suspense fallback="Loading...">
      <Layout />
    </Suspense>
  </Provider>
);
```

물론 2개 이상의 `<Suspense>`를 컴포넌트 트리 내에 위치시키는 것도 가능하다.

## 비동기 읽기 atom

atom의 read function은 Promise를 반환할 수 있다. 이는 컴포넌트의 랜더링을 일단 유보시킨 뒤 Promise가 충족된 이후 리랜더링 되도록 한다.

주목해야할 부분은, `useAtom`이 오직 resolved된 값만을 반환한다는 점이다.

```ts
const countAtom = atom(1);
const asyncCountAtom = atom(async (get) => get(countAtom) * 2);
// 비록 read function이 Promise를 반환하고 있지만,

const Component = () => {
  const [num] = useAtom(asyncCountAtom);
  // `num`은 number type인 것이 보장된다.
};
```

어떤 atom의 read function이 비동기일 때 뿐만 아니라 하나 이상의 종속 atom이 비동기인 경우에도 해당 atom은 비동기 atom이 된다.

```ts
const anotherAtom = atom((get) => get(asyncCountAtom) / 2);
// 비록 이 atom의 read function은 Promise를 반환하지 않지만,
// `asyncCountAtom`이 비동기 atom이기 때문에 이 atom 역시 비동기로 동작한다.
```
