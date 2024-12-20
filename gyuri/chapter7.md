# 사용사례 시나리오1: Zustand

## 모듈 상태와 불변 상태 이해하기

상태 객체 속성을 갱신할 수 없는 불변 상태 모델을 기반으로 한다. 상태를 변경하기 위해서는 새 객체를 생성해서 대체해야하며, 수정하지 않은 객체는 재사용해야한다.
불변상태 모델의 장점은 상태 객체의 참조에 대한 동등성만 확인하면 변경여부를 알수 있으므로 객체의 값 전체를 확인할 필요가 없다는 것이다.
내부적으로 Object.assign()으로 구현된다.
store 함수에서 남은 부분은 store.subscribe로, 이 함수를 사용하면 store의 상태가 변경될 때마다 호출되는 콜백 함수를 등록할 수 있다.

## 리액트 훅을 이용한 리렌더링 최적화

전역 상태를 사용하는 경우 모든 컴포넌트가 전역 상태를 사용하는 것은 아니기 때문에 리렌더링 최적화가 필요하다.

```tsx
import create from "zustand';

export const useStore = create((set) => ({
  count: 0,
  text: "hello",
}));

const Component = () => {
    const count = useStore((state) => state.count);
    return <div>count: {count}</div>;
};
```

선택자 기반 리렌더링 제어 = 수동 렌더링 최적화

## 읽기 상태와 갱신 상태 사용하기

1. 선택자 사용 : `const selectCount1 = (state: StoreState) => state.count1;`
2. store에 함수를 추가해서 몇 개의 로직이 컴포넌트 외부에 위치하게 할 수 있다.

```tsx
const useStore = create<StoreState>((set) => ({
    count1: 0,
    count2: 0,
    inc1: () => set(
    (prev) => ({ count1: prev.count1 + 1 })
    inc2: () => set(
    (prev) => ({ count2: prev.count2 + 1 })
    }));
```

3. 파생 상태로 사용할 수 있다.
   예시의 경우, 합계가 변경될 때만 리렌더링 된다.

```tsx
const selectTotal = (state: StoreState) => state.count1 + state.count2;

const Total = 0 => {
    const total = useStore(selectTotal);
    return (
        <div>
            total: {total}
        </div>
    );
}
```

4. store에서 합계를 생성할 수도 있다.
   여러 속성을 동시에 계산한 후 동기화 상태를 유지
   Jotai가 좀 더 잘한다.

```tsx
const useStore = create((set) => ({ count1: 0,
    count2: 0,
    total: 0,
    inc1: () => set((prev) => ({ ...prey,
    count1: prev.count1 + 1,
    total: prev.count1 + 1 + prev.count2,
    }))'
    inc2: 0 => set((prev) => ({
    ...prev,
    count2: prev.count2 + 1,
    total : prev.count2 + 1 + prev.count1,
    })),
}));
```

## 구조화된 데이터 처리하기

만들고자 하는 기능의 객체 타입을 정한다. 이것을 바탕으로 StoreState 타입을 정의한다.

예제 : 투두 리스트

```tsx
type StoreState = {
  todos: Todo[];
  addTodo: (title: string) => void;
  removeTodo: (id: number) => void;
  toggleTodo: (id: number) => void;
};
```

addTodo, removeTodo, toggleTodo 함수가 불변 방식으로 구현돼 있음
기존 객체와 배열을 변경하지 않고 새로운 객체와 배열을 생성
store 의 불변상태갱신과 리액트가 제공하는 memo함수덕분에 리렌더링을 최적화할 수 있다.

## 이 접근 방식과 라이브러리의 장단점

zustand의 읽기 및 쓰기 상태는 다음과 같다.

- 읽기 상태: 리렌더링을 최적화하기 위해 선택자 함수를 사용한다.
- 쓰기 상태: 불변 상태 모델을 기반으로 한다.

선택자 함수를 사용한 Zustand의 렌더링 최적화도 불변성을 기반으로 한다. 즉, 선택자 함수가 참조적으로 동일한 객체(또는 값)를 반환하면 객체가 변경되지 않은 것으로 간주하고 리렌더링을 하지 않는다.

Zustand는 리액트와 동일한 모델을 사용해 라이브러리의 단순성과 번들 크기가 작다는 점에서 큰 이점이 있다.

Zustand의 한계는 선택자를 이용한 수동 렌더링 최적화이다.
객체 참조 동등성을 이해해야 하며, 선택자 코드를 위해 보일러플레이트 코드를 많이 작성해야 할 필요가 있다.

## 참고

이번 장에서는 store 생성자에 일부 기능을 제공할 수 있는 미들웨어와 리액트 생명주기에서 store를 생성하는 비모듈 상태 사용 등 Zustand의 다른 몇 가지 기능에 대해서는 다루지는 않았다. 이는 라이브러리를 선택할 때 고려해야 할 또 다른 사항이 될 수 있다.

네, zustand의 미들웨어와 비모듈 상태에 대해 설명해드리겠습니다.

## 미들웨어 (Middleware)

미들웨어는 store의 동작을 확장하거나 수정할 수 있게 해주는 기능입니다.

주요 미들웨어 예시:

### 1. persist 미들웨어

```typescript
import create from "zustand";
import { persist } from "zustand/middleware";

const useStore = create(
  persist(
    (set) => ({
      bears: 0,
      addBear: () => set((state) => ({ bears: state.bears + 1 })),
    }),
    {
      name: "bear-storage", // 로컬 스토리지에 저장될 키 이름
      getStorage: () => localStorage, // 스토리지 타입 설정
    }
  )
);
```

- 상태를 localStorage나 sessionStorage에 자동으로 저장하고 복원합니다.
- 페이지 새로고침 후에도 상태를 유지할 수 있습니다.

### 2. devtools 미들웨어

```typescript
import create from "zustand";
import { devtools } from "zustand/middleware";

const useStore = create(
  devtools((set) => ({
    bears: 0,
    addBear: () => set((state) => ({ bears: state.bears + 1 })),
  }))
);
```

- Redux DevTools와 연동하여 상태 변화를 추적할 수 있습니다.
- 디버깅에 매우 유용합니다.

## 비모듈 상태 (Non-module State)

비모듈 상태는 React 컴포넌트의 생명주기 내에서 store를 동적으로 생성하고 관리하는 방식입니다.

### 사용 예시:

```typescript
import create from "zustand";
import { createContext, useContext } from "react";

const createStore = () =>
  create((set) => ({
    bears: 0,
    addBear: () => set((state) => ({ bears: state.bears + 1 })),
  }));

const StoreContext = createContext(null);

export const StoreProvider = ({ children }) => {
  const [store] = useState(createStore);
  return (
    <StoreContext.Provider value={store}>{children}</StoreContext.Provider>
  );
};

// 컴포넌트에서 사용
const Component = () => {
  const store = useContext(StoreContext);
  const bears = store((state) => state.bears);
  return <div>{bears} bears</div>;
};
```

### 비모듈 상태의 장점:

1. 서버 사이드 렌더링(SSR) 지원이 용이합니다
2. 여러 store 인스턴스를 동시에 사용할 수 있습니다
3. 컴포넌트 트리별로 독립된 상태 관리가 가능합니다

### 주의사항:

- 비모듈 상태를 사용할 때는 store의 생명주기 관리에 주의해야 합니다
- 메모리 누수를 방지하기 위해 적절한 정리(cleanup) 작업이 필요할 수 있습니다
