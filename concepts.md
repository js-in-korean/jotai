# Concepts

Jotai는 React의 불필요한 리랜더링 이슈를 해결하기 위해 탄생했다. 랜더링 결과로 생성된 UI가 이전과 동일하여 아무 차이를 발견할 수 없다면 이를 불필요한 리랜더링이라고 말할 수 있다.

이 이슈를 React context(`useContext` + `useState`)로 해결하려고 하면 많은 수의 context를 필요로 하게 되는데 그로인해 아래와 같은 이슈들을 수반하게 된다.

- Provider 지옥: 최상위 컴포넌트는 많은 수의 context Provider를 가져야 할 수 있다. 이는 기술적으로는 문제가 없지만 경우에 따라서는 서브트리에 따라 context를 구분하는 것이 더 바람직하다.

- 동적인 추가/제거: 새로운 context를 runtime에 추가하는 것은 별로 바람직하지 않은데 이는 새로운 Provider를 감싸게 되면 이하의 모든 컴포넌트들이 다시 mount되기 때문이다.

위 이슈들에 대한 전형적인 top-down 방식의 해결법은 selector 인터페이스를 적용하는 것이다. 예컨대 [use-context-selector](https://github.com/dai-shi/use-context-selector) 라이브러리가 있다. 그런데 이 접근법에는 한가지 골치아픈 점이 있는데, 컴포넌트의 리랜더링을 유발하지 않기 위해서는 selector function이 항상 동일한 값의 참조를 반환해야한다는 점이며 이는 많은 경우 memoization 기법을 요구하게 된다.

이와 다르게 Jotai는 Recoil에서 영감을 받아 atomic model을 기반으로한 bottom-up 방식의 접근법을 취한다. 상태는 atom들의 조합으로 구성되며 atom 간의 종속성에 따라 랜더링은 자동으로 최적화된다. 자연히 memoization과 같은 별도의 기법도 필요가 없다.

Jotai는 다음과 같은 두가지 원칙을 가진다.

- 원시적인: Jotai의 기본 인터페이스는 `React.useState`와 같이 매우 단순하다.

- 유연한: 파생 atom을 활용하면 여러 atom을 조합할 수도 있고 `React.useReducer`와 유사하면서 부수적인 기능을 탑재한 형태를 구현할 수도 있다.

Jotai의 핵심 API는 매우 단순하기 때문에 이것들을 활용하여 다양한 유틸리티 function들을 어렵지 않게 만들 수 있을 것이다.

[여기](https://jotai.org/docs/basics/comparison)에서 Jotai와 다른 라이브러리들과의 차이점을 확인해보자.
