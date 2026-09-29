### 1. "협력에 적합한 책임이란 메시지 수신자가 아니라 메시지 전송자에게 적합한 책임을 의미한다."

책에서는 **협력에 적합한 책임을 메시지의 수신자보다 전송자의 관점에서 생각해야 한다**고 이야기한다.

기존에는 모달을 열기 위해 그냥 상태를 바꿨다.

```tsx
setIsCreateFolderOpen(true);
```

지금은 이걸

```tsx
const folder = await openModal({
  type: "create-folder",
});
```

처럼 바꿨다.

그런데 책임 주도 설계 관점에서 생각해보면, 호출하는 쪽이 정말 원하는 건 `"모달을 열어라"`가 아니라 “폴더 생성을 위해 사용자 응답을 받아라”에 더 가까운 것 아닐까 하는 생각이 들었다.

그렇다면 `openModal`보다

```tsx
const folder = await requestCreateFolder();
```

처럼 호출하는 쪽의 행동에 더 가까운 이름이 적절한 걸까?

다만 지금 구조도 `openModal({ type })` 정도만 알고 있고, 실제로 어떤 상태를 바꾸고 어떤 컴포넌트를 렌더링할지는 모달 쪽에 숨겨두고 있다. 그래서 이것도 나름대로 선언적으로 잘 분리한 구조처럼 느껴진다.

**메시지의 이름도 반드시 호출자의 책임에 맞춰 추상화해야 하는지**, 아니면 프론트엔드에서는 `openModal`처럼 어느 정도 UI의 존재를 드러내는 것도 괜찮을지 고민해봤다.

---

### 2. 책임을 나누다 보니 오히려 결합이 강해졌다

책임으로 잘게 나누고,

`useCreateFolderAction`으로 폴더 생성 흐름을 하나의 책임으로 묶고 나니,

코드 자체는 이전보다 훨씬 읽기 쉬워졌다.

```tsx
const { openModal } = useGalleryModal();
const { refreshRoom } = useRoomQueryActions();

const handleCreateFolder = async () => {
  const folder = await openModal({
    type: "create-folder",
  });

  if (!folder) return;

  await refreshRoom();

  selectFolder(folder.id);
  clearSelection();
};

return { handleCreateFolder };
```

기존에는 모달 성공 콜백 안에 후처리가 섞여 있었는데, 지금은 **폴더를 생성한 뒤 어떤 일이 이어지는지**가 한 흐름으로 보인다.

그런데 책임을 하나로 모으다 보니 이 훅은 자연스럽게 여러 대상과 협력하게 됐다.

모달을 열어야 하고, 방 데이터를 갱신해야 하고, 새로 만든 폴더를 선택해야 하고, 기존 선택 상태도 초기화해야 한다.

즉, 책임은 더 명확해졌지만 그 책임을 수행하기 위한 결합은 오히려 늘어났다.

**책임을 잘 나눴는데 결합이 강해졌다면, 이 구조를 다시 개선해야 하는 걸까?**

결론적으로 책임이 모이는 곳이라면 자연스러운 구조라고 생각한다.

이들간의 추상화 정도가 비슷하게 이루어져 있다면 좋은 코드라고 생각한다.

---

### 3. 프론트엔드의 다형성

모달을 하나의 `openModal`로 처리하려다 보니, 모달마다 결과 타입이 다르다는 문제가 있었다.

예를 들어 폴더 생성은 `RoomFolder`를 반환하지만, 삭제 모달은 결과가 없을 수도 있다.

```tsx
type GalleryModalResult = {
  "create-folder": RoomFolder;
  "edit-folder": RoomFolder;
  "delete-folder": void;
  "delete-media": void;
};
```

그래서 요청한 모달의 `type`에 따라 결과 타입이 달라지도록 만들었다.

```tsx
const openModal = <T extends GalleryModalRequest["type"]>(
  modal: Extract<GalleryModalRequest, { type: T }>,
): Promise<GalleryModalResult[T] | null> => {
  // ...
};
```

사용하는 쪽에서는 같은 `openModal`을 호출하지만,

```tsx
const folder = await openModal({
  type: "create-folder",
});
// RoomFolder | null

await openModal({
  type: "delete-media",
});
// void | null
```

어떤 요청을 보내느냐에 따라 서로 다른 결과를 받게 된다.

5장에서 다형성 부분을 읽으면서 이 구조가 조금 떠올랐다.

다만 객체들이 동일한 메시지를 각자의 방식으로 처리한다기보다는, 하나의 `openModal`이 `type`을 보고 여러 경우를 처리하고 있다는 점에서 다형성을 적용해 볼 수 있겠다는 생각이 들었다.
