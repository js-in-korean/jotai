# Introduction

`npm install jotai` or `yarn add jotai`

Jotai는 "joe-tie"로 발음되며 일본어로 상태를 의미한다.

다음 링크에서 라이브 데모를 확인해 볼 수 있다. [Demo 1](https://codesandbox.io/s/jotai-demo-47wvh) | [Demo 2](https://codesandbox.io/s/jotai-demo-forked-x2g5d)

### Jotai가 Recoil보다 나은 점

- 더 간소화된 API
- string key 불필요
- TypeScript에 친화적

## 기본 atom 정의

Jotai에서 상태는 atom으로 표현된다. atom을 정의하기 위해 우리는 그저 `atom` function에 초기값을 전달하기만 하면 되는데 이는 string, number, object 또는 array와 같은 원시타입의 값이 될 수 있다.

```ts
import { atom } from 'jotai';

const countAtom = atom(0);
const countryAtom = atom('Japan');
const citiesAtom = atom(['Tokyo', 'Kyoto', 'Osaka']);
const mangaAtom = atom({ 'Dragon Ball': 1984, 'One Piece': 1997, Naruto: 1999 });
```

## 컴포넌트 내 atom 사용

atom은 `React.useState`와 유사한 형태로 사용할 수 있다.

```tsx
import { useAtom } from 'jotai';

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return (
    <h1>
      {count}
      <button onClick={() => setCount((c) => c + 1)}>one up</button>
    </h1>
  );
}
```

## 파생 atom 정의

`atom` function의 첫번째 파라미터로 read function을 전달함으로써 이미 존재하는 atom으로부터 파생되는 atom을 정의할 수 있다. `get`을 통해 다른 atom의 값을 참조한다.

```tsx
const doubledCountAtom = atom((get) => get(countAtom) * 2);

function DoubleCounter() {
  const [doubledCount] = useAtom(doubledCountAtom);
  return <h2>{doubledCount}</h2>;
}
```

> ✔️ 번역자 코멘트
>
> read function 내부에서 참조하고 있는 atom의 값이 갱신되면 해당 파생 atom의 값도 갱신된다.

## 여러 atom로부터 파생 atom 정의

하나의 파생상태를 정의하기 위해 여러 상태가 조합될 수도 있다.

```ts
const count1 = atom(1);
const count2 = atom(2);
const count3 = atom(3);

const sum = atom((get) => get(count1) + get(count2) + get(count3));
```

만약 functional programming에 익숙하다면 아래와 같이 사용할 수도 있다.

```ts
const atoms = [count1, count2, count3, ...otherAtoms];
const sum = atom((get) => atoms.map(get).reduce((acc, count) => acc + count));
```

## 비동기 atom 정의

`Promise`를 반환하는 read function을 전달함으로써 비동기 atom을 정의할 수 있다.

```ts
const urlAtom = atom('https://json.host.com');
const fetchUrlAtom = atom(async (get) => {
  const response = await fetch(get(urlAtom));
  return await response.json();
});

function Status() {
  // urlAtom의 값이 갱신되면 fetchUrlAtom의 값도 갱신되어 리랜더링을 유발한다.
  const [json] = useAtom(fetchUrlAtom);
}
```

> ✔️ 번역자 코멘트
>
> 비동기 atom은 read function이 최초로 해결되기 전까지는 값을 결정지을 수 없다. 따라서 비동기 atom을 컴포넌트 내에서 사용하기 위해서는 반드시 컴포넌트 상위 경로에 `React.Suspense`를 감싸주어야 한다.

## 값을 변경할 수 있는 파생 atom 정의

`atom` function의 두번째 파라미터로 write function을 전달함으로써 값을 변경할 수 있는 파생 atom을 정의할 수 있다. `get`을 통해 다른 atom의 값을 참조할 수 있으며 `set`을 통해 atom의 값을 변경할 수 있다.

```tsx
const decrementCountAtom = atom(
  (get) => get(countAtom),
  (get, set, _arg) => set(countAtom, get(countAtom) - 1)
);

function Counter() {
  const [count, decrement] = useAtom(decrementCountAtom);
  return (
    <h1>
      {count}
      <button onClick={decrement}>Decrease</button>
    </h1>
  );
}
```

> ✔️ 번역자 코멘트
>
> 위 예시 코드에서 write function은 `decrement` 가 호출될 때 실행되며, `decrement`에 전달된 파라미터는 write function의 세번째 파라미터(`_arg`)로 전달된다.

## 쓰기 전용 atom 정의

`atom` function의 첫번째 파라미터로 null을 전달함으로써 값을 변경할 수만 있는 atom을 정의할 수 있다.

```tsx
const multiplyCountAtom = atom(null, (get, set, by) => set(countAtom, get(countAtom) * by));

function Controls() {
  const [, multiply] = useAtom(multiplyCountAtom);
  return <button onClick={() => multiply(3)}>triple</button>;
}
```

## 비동기 액션

read function과 마찬가지로 write function도 `Promise`를 반환하도록 정의할 수 있다.

```tsx
const fetchCountAtom = atom(
  (get) => get(countAtom),
  async (_get, set, url) => {
    const response = await fetch(url);
    set(countAtom, (await response.json()).count);
  }
);

function Controls() {
  const [count, compute] = useAtom(fetchCountAtom);
  return <button onClick={() => compute('http://count.host.com')}>compute</button>;
}
```
