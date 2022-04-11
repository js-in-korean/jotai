# Primitives

Jotai에서 상태는 atom의 조합으로 구성된다. 즉 하나의 atom은 전체 상태의 일부분이다. React의 `useState`와 다르게 Jotai의 atom은 특정 컴포넌트에 종속적이지 않다. atom을 어떻게 정의하고 사용하는지 알아보자.

## atom

jotai 패키지는 `atom` function을 export하는데, 이 function은 atom config를 생성하기 위해 사용된다. 굳이 'config'를 덧붙이는 이유는 그것이 단지 정의에 불과하고 아직 실제 값을 가지고 있는 것은 아니기 때문이다. 하지만 이하로는 문맥이 명확하다면 단순히 atom이라고 명칭하겠다.

기본 atom (config)를 생성하기 위해서 해야할 것은 그저 atom의 초기값을 전달하는 것이다.

```ts
import { atom } from 'jotai';

const priceAtom = atom(10);
const messageAtom = atom('hello');
const productAtom = atom({ id: 12, name: 'good stuff' });
```

기본 atom 외에 파생 atom을 생성할 수도 있는데, 여기에는 세가지 패턴이 있다.

- 읽을 수만 있는 atom
- 쓸 수만 있는 atom
- 읽고 쓸 수 있는 atom

파생 atom을 생성하기 위해서는 read function과 선택적으로 write function을 전달해야한다.

```ts
const readOnlyAtom = atom((get) => get(priceAtom) * 2);
const writeOnlyAtom = atom(
  null, // 관례적으로 첫번째 파라미터로 null을 전달한다.
  (get, set, update) => {
    // `update`는 이 atom을 업데이트할 때 전달받는 파라미터이다.
    set(priceAtom, get(priceAtom) - update.discount);
  }
);
const readWriteAtom = atom(
  (get) => get(priceAtom) * 2,
  (get, set, newPrice) => {
    set(priceAtom, newPrice / 2);
    // 이 곳에서 몇 개의 atom이든 한번에 업데이트 할 수 있다.
  }
);
```

read function의 `get`은 atom의 값을 읽기 위해 사용되는데 대상 atom에 대한 종속성을 자동으로 관리하며 반응형으로 동작한다.
write function의 `get` 역시 atom의 값을 읽기 위해 사용되지만 대상 atom에 대한 종속성을 자동으로 관리하지는 않는다. 또한 해결되지 않은 비동기 값은 읽을 수 없다. 비동기 동작과 관련해서는 [여기](https://jotai.org/docs/basics/async)를 참조하자.
write function의 `set`은 atom의 값을 쓰기 위해 사용되는데 대상 atom의 write function을 호출한다.

atom configs는 어디서든 생성될 수 있고 동적으로 생성될 수도 있으나 referential equality를 유의하는 것이 중요하다. 따라서 atom을 render function 내부에서 생성하고자 한다면 `useMemo`나 `useRef` 등을 이용하여 reference값을 유지해줄 필요가 있다.

> 번역자 comment
>
> referential equality는 메모리 주소의 일치를 의미한다. 번역 시 오해의 여지가 있을 수 있어 원문을 그대로 썼다.

```ts
const Component = ({ value }) => {
  const valueAtom = useMemo(() => atom({ value }), [value]);
  // ...
};
```

## useAtom

`useAtom` hook은 상태 내 atom값을 읽기 위해 사용된다. 상태는 atom config와 atom값들로 이루어진 하나의 WeakMap으로 볼 수 있다.

`useAtom`은 atom값과 이 값을 업데이트하기 위한 function을 tuple로 반환하는데 이는 React의 `useState`와 유사하다. 한편 `useAtom`은 파라미터로 `atom()` 의 결과로 생성된 atom config를 전달받는다.

최초에는 atom에 어떤 값도 부여되지 않는다. 오직 atom이 `useAtom`에 의해 사용된 이후에만 초기값이 상태에 저장된다. 만약 atom이 파생 atom이라면 이 과정에서 초기값을 구하기 위해 read function이 호출된다. 어떤 atom을 사용하는 모든 React 컴포넌트들이 unmount 했을 때와 같이, 더 이상 atom이 사용되지 않고 atom config 역시 더 이상 존재하지 않는다면 상태 내 atom값 역시 garbage collected 된다.

```ts
const [vaelu, updateValue] = useAtom(anAtom);
```

`updateValue`는 오직 하나의 파라미터를 전달받는데, 이는 atom config 내 write function의 세번재 파라미터로 전달된다.
해당 값이 어떻게 활용될 지는 write function의 구현에 달려있다.

## Provider

Provider는 React 컴포넌트 서브트리에 상태를 제공하기 위해 사용된다. 다수의 Provider들이 다수의 컴포넌트 서브트리들을 위해 사용될 수 있으며, 서로 중첩될 수도 있다. 마치 React Context와 동일하게 동작한다.

만약 컴포넌트 트리 내 Provider를 적용하지 않은 체 atom을 사용하게 된다면, atom은 기본 상태를 사용하게 된다. 이는 Provider-less 모드라고 불린다.

Provider는 세 가지 이유로 사용될 수 있다.

1. 서로 다른 컴포넌트 서브트리 간 별개의 상태를 제공하고자 하는 경우

2. 몇 가지 debug 정보를 얻기 위해

3. atom들의 초기값을 전달받기 위해

```tsx
const SubTree = () => (
  <Provider>
    <Child />
  </Provider>
);
```
