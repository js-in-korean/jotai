> 임시: https://github.com/pmndrs/jotai/blob/main/docs/api/core.mdx

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

비고: 디버그 레이블이 유일할 필요는 없지만 보통 구별할 수 있도록 만드는 것을 권장한다.

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

Atom config는 값을 들고 있지 않는다. Atom 값은 저장소에 분리되어 존재한다. Provider는 저장소를 가지고 컴포넌트 트리 아래에 atom 값을 제공하는 컴포넌트이다. Provider는 React context provider처럼 동작한다. 만약 Provider를 사용하지 않는다면, 기본값을 가지고 provider-less 모드로 동작한다. Provider는 다른 컴포넌트 트리마다 다른 atom 값을 들고 있어야 할 때 필요하게 된다. Provder는 또한 아래에서 설명하는 능력도 가지고 있는데, provider-less 모드에서는 존재하지 않는다.

```jsx
const Root = () => (
  <Provider>
    <App />
  </Provider>
)
```

### `initialValues` prop

A Provider accepts an optional prop `initialValues`, with which you can specify
some initial atom values.
The use cases of this are testing and server side rendering.

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

The `initialValues` prop is not type friendly.
We can mitigate it by using a helper function.

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

A Provider accepts an optional prop `scope` that you can use for a scoped Provider.
When using atoms with a scope, the provider with the same scope is used.
The recommendation for the scope value is a unique symbol.
The primary use case of scope is for library usage.

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
// primitive or writable derived atom
function useAtom<Value, Update>(
  atom: WritableAtom<Value, Update>,
  scope?: Scope
): [Value, SetAtom<Update>]

// read-only atom
function useAtom<Value>(atom: Atom<Value>, scope?: Scope): [Value, never]
```

The useAtom hook is to read an atom value stored in the Provider. It returns the atom value and an updating function as a tuple, just like useState. It takes an atom config created with `atom()`. Initially, there is no value stored in the Provider. The first time the atom is used via `useAtom`, it will add an initial value in the Provider. If the atom is a derived atom, the read function is executed to compute an initial value. When an atom is no longer used, meaning all the components using it are unmounted, and the atom config no longer exists, the value is removed from the Provider.

```js
const [value, updateValue] = useAtom(anAtom)
```

The `updateValue` takes one argument, which will be passed to the third argument of writeFunction of the atom. The behavior depends on how the writeFunction is implemented.

---

## Notes

### How atom dependency works

To begin with, let's explain this. In the current implementation, every time we invoke the "read" function, we refresh the dependencies and dependents. For example, If A depends on B, it means that B is a dependency of A, and A is a dependent of B.

```js
const uppercaseAtom = atom((get) => get(textAtom).toUpperCase())
```

The read function is the first parameter of the atom.
The dependency will initially be empty. On first use, we run the read function and know that `uppercaseAtom` depends on `textAtom`. `textAtom` has a dependency on `uppercaseAtom`. So, add `uppercaseAtom` to the dependents of `textAtom`.
When we re-run the read function (because its dependency `textAtom` is updated),
the dependency is created again, which is the same in this case. We then remove stale dependents and replace with the latest one.

### Atoms can be created on demand

While the basic examples here show defining atoms globally outside components,
there's no restrictions about where or when we can create an atom.
As long as we remember that atoms are identified by their object referential identity,
we can create them anytime.

If you create atoms in render functions, you would typically want to use
a hook like `useRef` or `useMemo` for memoization. If not, the atom would be re-created each time the component renders.

You can create an atom and store it with `useState` or even in another atom.
See an example in [issue #5](https://github.com/pmndrs/jotai/issues/5).

You can cache atoms somewhere globally.
See [this example](https://twitter.com/dai_shi/status/1317653548314718208) or
[that example](https://github.com/pmndrs/jotai/issues/119#issuecomment-706046321).

Check [`atomFamily`](../api/utils.mdx#atom-family) in utils for parameterized atoms.

### Some more notes about atoms

- If you create a primitive atom, it will use predefined read/write functions to emulate `useState` behavior.
- If you create an atom with read/write functions, they can provide any behavior with some restrictions as follows.
- `read` function will be invoked during React render phase, so the function has to be pure. What is pure in React is described [here](https://gist.github.com/sebmarkbage/75f0838967cd003cd7f9ab938eb1958f).
- `write` function will be invoked where you called initially and in useEffect for following invocations. So, you shouldn't call `write` in render.
- When an atom is initially used with `useAtom`, it will invoke `read` function to get the initial value, this is recursive process. If an atom value exists in Provider, it will be used instead of invoking `read` function.
- Once an atom is used (and stored in Provider), it's value is only updated if its dependencies are updated (including updating directly with useAtom).
