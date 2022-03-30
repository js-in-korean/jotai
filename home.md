# Home

Jotai는 불필요한 리랜더링을 제거하며 React 내부의 상태 관리 기능을 그대로 사용하기 때문에 Suspense 및 Concurrent 기능을 모두 활용할 수 있다. 간단히 `React.useState`를 대체하는 데에 사용될 수 있을 뿐 아니라 복잡한 요구사항을 가진 대규모 애플리케이션에까지 적용될 수 있다.

## 서론

Jotai는 Recoil에서 영감을 받은 atomic 모델을 사용하여 상태관리에 bottom-up 접근법을 취한다. 상태를 여러 atom의 조합으로 구성하며 atom 간의 의존성에 따라 랜더링을 최적화한다. React context의 불필요한 리랜더링 이슈도 내부적으로 해결되는데, 그로 인해 memoization등의 최적화 기법을 따로 적용할 필요가 없다.

## 핵심 API

Jotai는 매우 간단하고 TypeScript 친화적인 API를 제공한다. React의 `useState` 만큼 사용하기 단순할 뿐만 아니라 모든 상태는 전역으로 공유될 수 있고 쉽게 파생 상태를 정의할 수도 있으며 불필요한 리랜더링은 자동으로 제거된다.

```ts
import { atom, useAtom } from 'jotai';

// 상태와 파생 상태를 정의한다.
const textAtom = atom('hello');
const uppercaseAtom = atom((get) => get(textAtom).toUpperCase());

// 어느 컴포넌트에서든 정의된 상태를 사용할 수 있다.
const Input = () => {
  const [text, setText] = useAtom(textAtom);
  const handleChange = (e) => setText(e.target.value);
  return <input value={text} onChange={handleChange} />;
};

const Uppercase = () => {
  const [uppercase] = useAtom(uppercaseAtom);
  return <div>Uppercase: {uppercase}</div>;
};

const App = () => {
  return (
    <>
      <Input />
      <Uppercase />
    </>
  );
};
```

## 추가 유틸리티

Jotai 패키지는 `jotai/utils` 번들도 제공하는데, 이를 통해 browser localStorage와 연동하거나 Redux와 비슷한 인터페이스로 상태를 정의할 수도 있다.

```ts
import { useAtom } from 'jotai';
import { atomWithStorage } from 'jotai/utils';

// localStorage 저장에 사용될 key와 상태의 초기값을 정의한다.
const darkModeAtom = atomWithStorage('darkMode', false);

const Page = () => {
  // 다른 상태들과 마찬가지로 동일한 인터페이스로 사용할 수 있다.
  const [darkMode, setDarkMode] = useAtom(darkModeAtom);
  const toggleDarkMode = () => setDarkMode(!darkMode);
  return (
    <>
      <h1>Welcome to {darkMode ? 'dark' : 'light'} mode!</h1>
      <button onClick={toggleDarkMode}>toggle theme</button>
    </>
  );
};
```

## Third-party 연동

여러 third-party 라이브러리와 연동하기 위한 번들 또한 제공된다. Immer, Optics, Query, XState, Valtio, Zustand, Redux 그리고 URQL 등이 지원된다. third-party 라이브러리와의 연동은 `atomWithImmer`와 같이 그저 set function이 대체된 형태로 수행되거나 또는 `atomWithStore`와 같이 외부의 상태 저장소와 양방향 data binding되는 형태로 수행되기도 한다.
