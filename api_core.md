## atom

atom config를 생성하기 위해서는 `atom`를 사용하라. atom config는 불변 객체다. atom config는 atom 값을 들고 있지 않는다. atom 값은 Provider 상태에 저장된다.

```tsx
// 기본 atom
function atom<Value>(initialValue: Value): PrimitiveAtom<Value>

// 읽기 전용 atom
function atom<Value>(read: (get: Getter) => Value | Promise<Value>): Atom<Value>

// 쓰기 가능한 파생 atom
function atom<Value, Update>(
  read: (get: Getter) => Value | Promise<Value>,
  write: (get: Getter, set: Setter, update: Update) => void | Promise<void>
): WritableAtom<Value, Update>

// 쓰기 전용 파생 atom
function atom<Value, Update>(
  read: Value,
  write: (get: Getter, set: Setter, update: Update) => void | Promise<void>
): WritableAtom<Value, Update>
```

- `initialValue`: atom의 값이 변경되기 전까지 반환할 초기값
- `read`: 다시 렌더할 때마다 호출되는 함수다. `read`의 시그니처는 `(get) => Value | Promise<Value>`이고, `get`은 atom config를 가져와서 이어 서술된 것처럼 Provider에 저장된 값을 반환하는 함수다. 의존성은 추적된다. `get`이 한번이라도 atom에서 사용된다면, `read`는 atom 값이 변경될 때마다 재실행될 것이다.
- `write`: 대부분 atom의 값을 변경할 때 사용되는 함수다. 자세한 설명을 하자면, `useAtom`이 반환하는 쌍의 두번째 값 `useAtom()[1]`을 호출할 때마다 호출된다. 기본 atom에서 함수의 기본값은 atom 값을 변경할 것이다. `write`의 시그니처는 `(get, set, update) => void | Promise<void>`이다. `get`은 위에 설명한 것과 비슷하지만 의존성을 추적하지 않는다. `set`은 atom config와 새로운 값을 가져와서 Provider에 있는 atom 값을 업데이트하는 함수다. `update`는 이 후에 설명하는 `useAtom`에 의해 반환되는 업데이트 함수로부터 받는 임의의 값이다.

```tsx
const primitiveAtom = atom(initialValue)
const derivedAtomWithRead = atom(read)
const derivedAtomWithReadWrite = atom(read, write)
const derivedAtomWithWriteOnly = atom(null, write)
```

두 종류의 atom이 있다. 쓰기 가능한 atom과 읽기 전용 atom이다. 기본 atom은 항상 쓰기 가능하다. 파생 atom은 `write`가 명시되어 있다면 쓰기 가능하다. 기본 atom의 `write`는 `React.useState`의 `setState`에 해당한다.

### debugLabel

생성된 atom config는 `debugLabel`이라는 선택 속성을 가질 수 있다. 디버그 레이블은 디버깅시에 atom을 나타내기 위해 사용된다. 자세한 정보는 [디버깅 가이드](./guides_debugging.md)를 보라.

메모: 디버그 레이블이 유일할 필요는 없지만 보통 구별할 수 있도록 만드는 것을 권장한다.

### onMount

생성된 atom config는 `onMount`라는 선택 속성을 가질 수 있다. `onMount`는 `setAtom` 함수를 가져와서 선택적으로 `onUnmount` 함수를 반환하는 함수다.

`onMount` 함수는 atom이 provider에서 처음 사용될 때 호출된다. 그리고 `onUnmount`는 더 이상 사용되지 않을 때 호출된다. 일부 엣지 케이스에서 atom은 unmount된 다음 즉시 mount될 수 있다.

```tsx
const anAtom = atom(1)
anAtom.onMount = (setAtom) => {
  console.log('atom is mounted in provider')
  setAtom(c => c + 1) // mount시에 count 증가
  return () => { ... } // 선택적으로 onUnmount 함수를 반환
}
```

`setAtom` 함수를 호출하는 것은 atom의 `write`를 작동시킬 것이다. `write`를 원하는 대로 만들면 동작을 변경할 수 있다.

```tsx
const countAtom = atom(1)
const derivedAtom = atom(
  (get) => get(countAtom),
  (get, set, action) => {
    if (action.type === 'init') {
      set(countAtom, 10)
    } else if (action.type === 'inc') {
      set(countAtom, (c) => c + 1)
    }
  }
)
derivedAtom.onMount = (setAtom) => {
  setAtom({ type: 'init' })
}
```

## Provider

```ts
const Provider: React.FC<{
  initialValues?: Iterable<readonly [AnyAtom, unknown]>
  scope?: Scope
}>
```

atom config는 값을 들고 있지 않는다. atom 값은 저장소에 분리되어 존재한다. Provider는 저장소를 가지고 컴포넌트 트리 아래에 atom 값을 제공하는 컴포넌트이다. Provider는 React context provider처럼 동작한다. 만약 Provider를 사용하지 않는다면, 기본값을 가지고 provider-less 모드로 동작한다. Provider는 다른 컴포넌트 트리마다 다른 atom 값을 들고 있어야 할 때 필요하게 된다. Provder는 또한 아래에서 설명하는 능력도 가지고 있는데, provider-less 모드에서는 존재하지 않는다.

```jsx
const Root = () => (
  <Provider>
    <App />
  </Provider>
)
```

### `initialValues` prop

Provider는 선택 속성으로 `initialValues`를 받는다. 이는 초기 atom 값을 지정할 수 있다. 이것의 사용 사례는 테스팅과 서버 사이드 렌더링이 있다.

#### Example

```jsx
const TestRoot = () => (
  <Provider
    initialValues={[
      [atom1, 1],
      [atom2, 'b'],
    ]}>
    <Component />
  </Provider>
)
```

#### TypeScript

`initialValues` prop은 타입 친화적이 아니다. 헬퍼 함수를 사용하는 것으로 완화할 수 있다.

```ts
const createInitialValues = () => {
  const initialValues: (readonly [Atom<unknown>, unknown])[] = []
  const get = () => initialValues
  const set = <Value>(anAtom: Atom<Value>, value: Value) => {
    initialValues.push([anAtom, value])
  }
  return { get, set }
}
```

### `scope` prop

Provider는 scope가 있는 Provider를 사용할 수 있는 `scope`라는 선택 속성을 받는다. scope 내에서 atom을 사용할 때 같은 scope에 있는 provider가 사용된다. scope 값에 대한 권장 사항은 고유 symbol이다. scope의 주요 사용 사례는 라이브러리 사용이다.

#### Example

```jsx
const myScope = Symbol()

const anAtom = atom('')

const LibraryComponent = () => {
  const [value, setValue] = useAtom(anAtom, myScope)
  // ...
}

const LibraryRoot = ({ children }) => (
  <Provider scope={myScope}>{children}</Provider>
)
```

## useAtom

```ts
// 기본 또는 쓰기 가능한 파생 atom
function useAtom<Value, Update>(
  atom: WritableAtom<Value, Update>,
  scope?: Scope
): [Value, SetAtom<Update>]

// 읽기 전용 atom
function useAtom<Value>(atom: Atom<Value>, scope?: Scope): [Value, never]
```

useAtom hook은 Provider에 저장된 atom 값을 읽기 위한 것이다. atom 값과 업데이트 함수를 useState처럼 튜플로 반환한다. `atom()`으로 생성된 atom config를 취한다. 처음에는 Provider에 저장된 값이 없다. atom이 `useAtom`으로 처음 사용되면, Provider에 초기 값이 추가될 것이다. 만약 atom이 파생 atom이라면 read 함수는 초기 값을 계산하기 위해 실행된다. atom이 더 이상 사용되지 않을 때, 즉 이를 사용하는 모든 컴포넌트가 unmounted 되고 atom config는 더 이상 존재하지 않는 경우, 값은 Provider에서 제거된다.

```js
const [value, updateValue] = useAtom(anAtom)
```

`updateValue`는 하나의 인수를 취하며, 이는 atom의 write 함수의 세번째 인수에 전달될 것이다. 동작은 write 함수가 구현되는 방식에 달려 있다.

---

## 메모

### atom 의존성이 작동하는 방식

To begin with, let's explain this. In the current implementation, every time we invoke the "read" function, we refresh the dependencies and dependents. For example, If A depends on B, it means that B is a dependency of A, and A is a dependent of B.
먼저 이것을 설명하자. 현재 구현에서는 매번 "read" 함수를 작동시키고 의존성과 종속성을 갱신한다. 예를 들어 A가 B에 의존한다면, B는 A에 의존성이고 A는 B의 종속성이다.

```js
const uppercaseAtom = atom((get) => get(textAtom).toUpperCase())
```

read 함수는 atom의 첫번째 파라메터이다. 의존성은 처음에 비어있을 것이다. 처음 사용할 때, read 함수를 실행하고 `textAtom`의 의존하는 `uppercaseAtom`를 알아본다. `textAtom`은 `uppercaseAtom`에 의존성을 가진다. 따라서 `textAtom`의 종속성에 `uppercaseAtom`를 추가한다. read 함수를 다시 실행하면(`textAtom` 의존성이 업데이트되기 때문에), 의존성이 다시 생성된다. 이 경우에는 동일하다. 그런 다음 오래된 종속성을 제거하고 최신의 것으로 교체한다.

### Atom은 필요시에 생성될 수 있다

여기 기본 예제에서는 컴포넌트 밖에서 전역으로 정의하는 atom을 보여줬는데, atom을 생성하는 장소와 때에 대한 제약은 없다. Atom이 객체 참조 식별에 의해 식별되는 것을 기억하는 한 언제든지 만들 수 있다.

만약 렌더 함수에서 atom을 생성한다면 보통 메모이제이션을 위해 `useRef`나 `useMemo`같은 hook을 사용할지 모른다. 그렇지 않으면 atom은 컴포넌트가 렌더할 때마다 재생성 될 것이다.

atom을 생성하고 `useState`나 심지어 다른 atom 저장할 수 있다. [issue #5](https://github.com/pmndrs/jotai/issues/5)에서 예제를 보라.

전역으로 어딘가에서 atom을 캐시할 수 있다. [이 예제](https://twitter.com/dai_shi/status/1317653548314718208)나 [다른 예제](https://github.com/pmndrs/jotai/issues/119#issuecomment-706046321)를 보라.

파라미터화된 atom을 위해서는 utils에서 [`atomFamily`](./api_utils.md#atom-family)를 확인하라.

### atom에 대한 더 많은 내용

- 만약 기본 atom을 생성한다면 `useState` 동작을 에뮬레이트하기 위해 미리 정의된 read/write 함수를 사용할 것이다.
- 만약 read/write 함수를 가진 atom을 생성한다면 그것들은 다음과 같은 제약사항을 가진 어떤 동작을 제공할 수 있다.
- `read` 함수는 React 렌더 단계동안 작동될 것이고, 순수해야만 한다. React로 순수한 것이 무엇인지는 [여기](https://gist.github.com/sebmarkbage/75f0838967cd003cd7f9ab938eb1958f)에서 설명하고 있다.
- `write` 함수는 처음 호출한 위치와 다음 호출 동안 useEffect에서 작동될 것이다. 그래서 렌더시에 `write`를 호출해서는 안된다.
- atom이 `useAtom`으로 처음 사용될 때, `read` 함수는 초기값을 얻기 위해 작동될 것이고, 재귀 프로세스다. 만약 Provider에서 atom 값이 존재한다면, `read` 함수를 작동시키는 대신 그 값이 사용될 것이다.
- 일단 atom이 사용되면(그리고 Provider에 저장된), 그 값은 오직 의존성이 업데이트된 경우(useAtom으로 직접 업데이트하는 것을 포함하여)에만 업데이트된다.
