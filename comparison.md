# Comparison

## Jotai는 Zustand와 무엇이 다를까?

### 이름

Jotai는 "상태"의 일본어,
Zustand는 "상태"의 독일어이다.

### 유사한 라이브러리

Jotai는 Recoil과 유사,
Zustand는 Redux와 유사하다.

### 상태가 관리되는 곳

Jotai의 상태는 React component tree내에서 관리되는 반면,
Zustand의 상태는 React 외부의 공간에서 별도로 관리된다.

### 상태가 구성되는 방식

Jotai의 상태는 bottom-up 방식으로써 atom들로 구성되는 반면,
Zustand의 상태는 top-down 방식으로써 하나의 객체로 구성된다.

### 기술적인 차이

둘 사이의 가장 큰 차이는 상태 모델에 있다. Zustand는 (물론 원한다면 여러개의 저장소를 만들 수도 있겠지만 기본적으로는) 단일 저장소인 반면, Jotai는 여러 atom들을 정의하고 그것들을 서로 조합하여 상태를 구성한다. 이런 의미에서, Jotai와 Zustand 사이의 차이는 어떤 프로그래밍 mental model을 채택할 것이냐의 문제라고 할 수 있다.

Jotai는 React의 `useState`와 `useContext` 조합의 대체제가 될 수 있다. 여러 context를 생성하는 대신 atom들이 하나의 큰 context를 공유할 수 있다.
Zustand는 하나의 외부 저장소로서 기능하며, hook등을 이용하여 이 외부 저장소와 React를 연결할 수 있을 것이다.

### 언제 무엇을 사용해야 될까?

- 만약 `useState` + `useContext`의 대체제가 필요하다면, Jotai가 더 잘 어울릴 것이다.
- 만약 React 외부에서 상태를 업데이트 하고 싶다면, Zustand가 더 잘 동작할 것이다.
- 만약 code splitting이 중요하다면, Jotai를 사용했을 때의 성능이 더 좋을 것이다.
- 만약 Redux 개발자도구를 선호한다면, Zustand가 좋은 선택이 될 것이다.
- 만약 React의 `Suspense`를 사용하고자 한다면, Jotai를 선택해야 할 것이다.

## Jotai는 Recoil과 무엇이 다를까?

> (밑밥을 좀 깔자면 저자는 Recoil 사용 경험이 많지 않다. 그렇기 때문에 아래 내용은 다소 Jotai에게 편향되고 부정확할 수 있다.)

### 개발자

- Jotai는 Poimandres(전 react-spring)라는 몇 개발자들로 구성된 공동체에 의해 개발되었다.
- Recoil은 Facebook에 의해 개발되었다.

### 지향점

- Jotai는 쉬운 학습을 위해 기본 API에 초점을 맞추고 있으며 특정 프로그래밍 방식에 얽매이지 않는다. (이는 Zustand의 철학과도 일치한다.)
- Recoil은 대규모 애플리케이션을 위해 필요한 많은 기능들을 제공하며 그로인해 복잡한 사용법을 요구한다.

### 기술적인 차이

- Jotai는 atom을 식별하기 위해 메모리 참조 주소를 활용한다.
- Recoil은 atom을 식별하기 위해 string key를 활용한다.

### 언제 무엇을 사용해야 될까?

- 만약 무언가 새로운 것을 배우고 싶다면, 무언들 어떨까? 둘다 해보자.
- 만약 Zustand를 좋아한다면, Jotai 역시 맘에 들 것이다.
- 만약 애플리케이션이 매우 무겁고 (저장소, 서버, URL 등에 상태를 저장하기 위해) 상태 직렬화가 요구된다면, Recoil이 이를 위한 좋은 기능을 제공한다.
- 만약 React context의 대체제가 필요한 것이라면, Jotai가 충분한 기능을 제공한다.
- 만약 새로운 라이브러리를 개발해볼 요량이라면, Jotai가 좋은 기반이 될 수 있다.
- 이도 저도 아니라면, Jotai와 Recoil은 목표하는 것과 기반 기술이 매우 비슷하니 그냥 둘 다 사용해보고 당신의 의견을 공유해주면 좋겠다.
