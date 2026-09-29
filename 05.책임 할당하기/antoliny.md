# 5장 책임 할당하기

- 데이터보다 행동을 먼저 결정하라
- 협력이라는 문맥 안에서 책임을 결정하라

→ **데이터에 집중하지 말고 객체의 책임과 협력에 초점을 맞추라**

## 데이터보다 행동을 먼저 결정하라

**“이 객체가 수행해야 하는 책임은 무엇인가?”** → **“이 책임을 수행하는데 필요한 데이터는 무엇인가?”**

## 올바른 도메인 모델이란 존재하지 않는다.

> 💡 도메인 모델은 도메인을 개념적으로 표현한 것이지만 그 안에 포함된 개념과 관계는 구현의 기반이 돼야 한다. 이것은 도메인 모델이 구현을 염두에 두고 구조화되는 것이 바람직하다는 것을 의미한다. 반대로 코드의 구조가 도메인을 바라보는 관점을 바꾸기도 한다.
>
> → “구현의 기반이 돼야 한다”, “구현을 염두에 두고 구조화…”
>
> **생각보다 도메인을 프로그래밍적인 해석에서의 개입이 괜찮고 오히려 해야한다는 의미인가 ? (구현적으로 실용성이 높은 + 더 명확하게 구조화하는)**
>
> <details open>
> <summary>AI와 함께한 예시</summary>
>
> **도서관 대출 시스템에서 ‘책’을 모델링하는 과정**이 좋은 예시입니다. 처음부터 도서관에 같은 책이 여러 권 있다는 사실을 알고 있었다고 가정해 볼게요.
>
> ## 1. 도메인 모델이 구현의 기반이 된다
>
> 처음에는 도메인을 이렇게 이해합니다.
>
> > “회원은 책을 대출한다. 대출에는 대출일과 반납 예정일이 있다.”
>
> 이를 모델로 표현하면 다음과 같습니다.
>
> ```text
> 회원 ── 대출 ── 책
> ```
>
> 객체지향으로 구현한다면 이런 구조를 생각할 수 있습니다.
>
> ```java
> class Loan {
>     Member member;
>     Book book;
>     LocalDate borrowedOn;
>     LocalDate dueOn;
> }
> ```
>
> 여기서는 **‘대출’이라는 도메인 개념과, 회원·책 사이의 관계가 코드의 구조를 결정**했습니다.
>
> “회원이 책을 이용한다”라는 막연한 설명에서 나아가, 무엇을 연결하고 어떤 정보를 관리하는지 구현할 수 있을 만큼 구체화한 것입니다.
>
> ## 2. 구현하면서 도메인을 바라보는 관점이 바뀐다
>
> 그런데 `Book`을 구현하다 보면 질문이 생깁니다.
>
> > “『어린 왕자』가 세 권 있다면, `Book` 객체는 하나여야 할까, 세 개여야 할까?”
>
> 하나로 만들면 각 권의 대출 상태를 구별하기 어렵습니다. 세 개로 만들면 제목·저자 같은 공통 정보와 각 권의 대출 상태를 함께 관리하게 됩니다.
>
> 이 과정에서 **우리가 ‘책’이라는 말로 서로 다른 두 개념을 표현하고 있었다**는 사실이 드러납니다.
>
> | 개념                 | 의미                         | 해당 정보            |
> | -------------------- | ---------------------------- | -------------------- |
> | 도서 정보 `BookInfo` | 어떤 책인가                  | 제목, 저자           |
> | 소장본 `BookCopy`    | 도서관에 있는 구체적인 한 권 | 관리 번호, 분실 여부 |
>
> 이제 도메인을 이렇게 다시 이해하게 됩니다.
>
> > “회원이 대출하는 것은 도서 정보가 아니라, 도서관이 소장한 특정 한 권이다.”
>
> 바뀐 모델은 다음과 같습니다.
>
> ```text
> 도서 정보 ── 소장본 여러 개
>                  │
>                 대출 ── 회원
> ```
>
> 코드에서도 대출 대상이 달라집니다.
>
> ```java
> class Loan {
>     Member member;
>     BookCopy copy;   // 구체적인 한 권을 대출한다
>     LocalDate borrowedOn;
>     LocalDate dueOn;
> }
> ```
>
> ## 3. 이 예시가 문장과 연결되는 지점
>
> 핵심은 **같은 책이 여러 권 있다는 새 요구사항이 추가된 게 아니라는 점**입니다. 이미 알고 있던 사실인데, 구현하면서 식별과 상태를 정확하게 다루려다 보니 기존 모델의 모호함이 드러난 것입니다.
>
> 즉, 다음과 같은 순환입니다.
>
> > **도메인에 대한 이해 → 코드 구조 → 구현에서 모호함 발견 → 도메인에 대한 이해 정교화 → 코드 구조 개선**
>
> “구현을 염두에 두고 모델링한다”는 것은 **개념의 의미와 관계를 코드로 표현할 수 있을 만큼 명확하게 한다**는 뜻이고, “코드 구조가 관점을 바꾼다”는 것은 **그 과정에서 발견한 구분이 도메인을 이해하는 새로운 기준이 된다**는 뜻입니다.
>
> </details>

## GRASP

### INFORMATION EXPERT(정보 전문가)

객체에게 책임을 할당하는 첫 번째 원칙은 책임을 수행할 정보를 알고 있는 객체에게 책임을 할당하는 것.

즉, **“전문가”** 에게 맡기는 것이다.

여기서 사용한 **“전문가”** 라는 단어 하나만으로 객체지향 관점에서 누구에게 책임을 맡겨야할지에 대한 의미가 떠오른다. → 책임을 수행하는데 필요한 정보를 가지고 있는 객체

그런면에서 **“전문가”** 라는 단어가 너무나도 적절하게 느껴진다.

## 구현을 통한 검증

```java
public class Screening {
  private Movie movie;
  private int sequence;
  private LocalDateTime whenScreened;

  public Reservation reserve(Customer customer, int audienceCount) {
    return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
  }

  private Money calculateFee(int audienceCount) {
    return movie.calculateMovieFee(this).times(audienceCount);
  }
}
```

> 💡 Screening을 구현하는 과정에서 Movie에 전송하는 메시지의 시그니처를 calculateMovieFee(Screening screening)으로 선언했다는 사실에 주목하라. 이 메시지는 수신자인 Movie가 아니라 송신자인 Screening 의 의도를 표현한다. 여기서 중요한 것은 Screening 이 Movie의 내부 구현에 대한 어떤 지식도 없이 전송할 메시지를 결정했다는 것이다. 이처럼 Movie의 구현을 고려하지 않고 필요한 메시지를 결정하면 Movie의 내부 구현을 깔끔하게 캡슐화할 수 있다.

클래스 속성이 서로 다른 시점에 초기화되거나 일부만 초기화된다는것은 응집도가 낮다는 증거

메서드들이 사용하는 속성에 따라 그룹이 나뉜다면 클래스의 응집도가 낮다는 증거

→ **속성 그룹과 해당 그룹에 접근하는 메서드 그룹을 기준으로 코드를 분리해보자**

하나의 클래스가 여러 타입의 행동을 구현하고 있는 것처럼 보인다면 클래스를 분해할 필요가 있다.

각 타입별로 구체적인 클래스로 만들어내고 그 클래스들을 묶는 역할을 만들어내자.

그리고 우리는 그 역할에 결합시키면 된다.

> 💡 **데이터가 아닌 책임을 중심으로 설계하자**

## 책임 주도 설계의 대안

책임과 객체 사이에서 방황할 때 돌파구를 찾기 위해 선택하는 방법은 최대한 빠르게 목적한 기능을 수행하는 코드를 작성하는 것.

→ 일단 돌아가는 무언가를 만드는것.

크게 만들고 천천히 분리해보는식(처음은 응집도 높은 메서드로)

---

## 객체지향 관점에서 리팩토링하다 실패한 청년

## 소스코드

```typescript
import {
  getWebPushPublicKey,
  registerWebPushSubscription,
  deactivateWebPushSubscription,
} from "./api";

export const PUSH_SUBSCRIPTION_ID_KEY = "chongchong:push-subscription-id";

// 푸시알림 구독
export async function enablePush() {
  // 지원하는 환경인가 ?
  if (
    !("Notification" in window) ||
    !("serviceWorker" in navigator) ||
    !("PushManager" in window)
  ) {
    throw new Error(
      "알림을 지원하지 않는 환경입니다. iPhone에서는 홈 화면에 추가한 앱에서 사용해 주세요.",
    );
  }

  if (Notification.permission === "denied") {
    throw new Error("기기 또는 브라우저 설정에서 알림을 허용해 주세요.");
  }

  // 알림 권한을 확인하고 'granted'가 아니라면 알림 허용 여부를 요청한다
  const permission =
    Notification.permission === "granted"
      ? "granted"
      : await Notification.requestPermission();

  // 거절하면 함수를 종료한다
  if (permission !== "granted") return false;

  // 알림 작업을 수행하는 서비스워커를 가져온다
  const scope = new URL("/push/", window.location.origin).href;
  const sw = await navigator.serviceWorker.getRegistration(scope);

  // 서비스 워커가 존재하지 않은 경우거나 상태가 'activated'가 아닌 경우 에러를 발생시킨다
  if (sw?.scope !== scope || sw.active?.state !== "activated") {
    throw new Error("알림 준비 중입니다. 잠시 후 다시 눌러 주세요.");
  }

  // VAPID 키를 가져온다
  const publicKey = await getWebPushPublicKey();

  // 브라우저와 알림 서비스를 구독한다
  const subscription = await sw.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: publicKey,
  });

  try {
    const { endpoint, keys } = subscription.toJSON();

    if (!endpoint || !keys?.p256dh || !keys.auth) {
      throw new Error("푸시 구독 정보를 확인하지 못했어요.");
    }

    // 백엔드와 알림 서비스를 구독한다
    const subscriptionId = await registerWebPushSubscription({
      endpoint,
      keys: {
        p256dh: keys.p256dh,
        auth: keys.auth,
      },
    });

    // 구독 상태를 보존하기 위해 로컬스토리지에 저장한다
    localStorage.setItem(PUSH_SUBSCRIPTION_ID_KEY, String(subscriptionId));
  } catch (error) {
    await subscription.unsubscribe().catch(console.error);
    throw error;
  }

  return true;
}

// 푸시알림 구독 해제
export async function disablePush() {
  // 로컬 스토리지에서 구독 키를 가져온다
  const subscriptionId = localStorage.getItem(PUSH_SUBSCRIPTION_ID_KEY);

  // 백엔드와 알림 서비스를 구독 해제한다.
  if (subscriptionId) {
    await deactivateWebPushSubscription(Number(subscriptionId));
  }

  if ("serviceWorker" in navigator) {
    // 구독관련 작업을 수행하는 서비스 워커를 가져온다
    const scope = new URL("/push/", window.location.origin).href;
    const sw = await navigator.serviceWorker.getRegistration(scope);

    // 브라우저와 알림 서비스의 구독을 해제한다
    if (sw?.scope === scope) {
      const subscription = await sw.pushManager.getSubscription();
      await subscription?.unsubscribe();
    }
  }

  // 로컬 스토리지에서 구독 키를 제거한다
  localStorage.removeItem(PUSH_SUBSCRIPTION_ID_KEY);
}
```

### enablePush

- 지원하는 환경인지 확인한다
- 알림 권한이 ‘granted’인지 확인한다 -> 권한이 ‘granted’ 가 아니라면 권한요청을 수행한다 ->
  - 권한이 ‘granted’ 가 아니라면 권한요청을 수행한다
- 권한이 ‘granted’가 아니면 false를 반환한다  -> (push알림을 허용하지 않겠습니다)
- 스코프를 가져온다(서비스워커 스코프)
- 스코프를 통해 특정 스코프로 되어있는 서비스워커를 가져온다
- 가져온 서비스 워커의 스코프를 확인하고 ‘activated’ 상태인지 확인한다
- 퍼블릭 키를 가져온다 (VAPID) -> Voluntary Application Server Identification Key
- 서비스워커 푸시매니저에 구독한다
- 구독한걸 백엔드에도 등록한다
  - 구독 정보를 분해한 뒤 구독 정보가 존재하는지 확인한다. (Endpoint -> Push Service가 발급한 전송 주소, keys -> 메시지 암호화용 공개 키)
  - 웹푸시 구독을 백엔드에도 등록한다
  - 응답받은 구독식별자를 로컬스토리지에 저장한다
  - 에러가 발생하면 백엔드와 일치하게 브라우저에서 푸시 알림 구독을 해제한다
- 위 과정에서 문제가 발생하지 않으면 true를 반환한다

---

### disablePush

- 구독아이디가 존재하는지 로컬스토리지에서 가져온다
- 구독아이디가 존재하다면 백엔드에 일단 먼저 구독해제 요청한다
- Navigator 객체에 서비스워커가 있다면
  - 스코프를 가져온다(서비스워커 스코프)
  - 스코프를 통해 특정 스코프로 되어있는 서비스워커를 가져온다
  - 스코프가 동일하다면 서비스워커를 가져와 구독을 해제한다
- 로컬스토리지에서 구독아이디를 제거한다.

### 핵심 비즈니스

- 푸시알림을 활성화한다

**“현재 이 기기에서 푸시알림을 받을 수 있게 해줘”**

- 푸시알림을 비활성화한다

**“현재 이 기기에서 푸시알림을 받지 않게 해줘”**

### 리팩토링 가치

낮다.

- 변경될 가능성이 상대적으로 낮다(하지만 언제나 예상하기 어렵다, 하지만 자주 변경될 가능성은 확실히 낮다)
- 코드가 하나의 책임으로 묶여있기 때문에 테스트하기 어렵다
  - 한곳을 테스트하기 위해 모든 동작을 거쳐야한다.
  - 각 동작들이 명시적으로 어떤걸 의미하는지 알기 어렵다.

### 리팩토링 핵심

- 푸시 알림을 다른 방식으로 구현해도 유연하게 대응할 수 있어야한다
  - 현재 방식이 FCM 으로 바뀐다 할지라도 유연하게 변경할 수 있어야한다.

### 메시지

- 지원하나요 ?
- 구독한다 (+ 키를 가져온다)
- 권한을 요청한다.

### 역할

아무리 생각해봐도 떠올리지 못했다.

### 궁금증

- 비즈니스의 경계를 알면 더 쉬울까?
  - 어디가 변경될지에 대한 경계를 잘 모르겠다. (메시지를 보고 역할이 쉽게 떠오르지 않는다)
  - 사실 메시지를 떠올리기도 어려웠다. 어디에 누군가에게 무언가를 맡겨야할지 감이 오지 않았다.

## 돌아보고 난뒤 드는 생각

이미 난 객체지향을 하고 있었다.

물론 위 코드가 좋은 코드라는건 아니다.

하지만 이미 하나의 책임을 다하고 있는 코드는 맞다. (약간의 버그는 있을 수 있음, 실제로 있음)

이 하나의 책임안에서 분리되어야할 책임을 찾기 어려웠고 그 사이에 역할또한 발견하지 못했다.

그 역할은 우리의 코드를 변경에 유연하게 대응시켜줄 하나의 부분이다.

물론 이 코드 자체를 처음 접해서 난이도가 높았던 것이기도 하다.

### 멀리서 바라보기

나는 알림을 구독, 해제 하는 기능을 만들고 있었다.

조금 더 멀리서 바라보니 나는 총총 앱에서 특정 버튼을 클릭하면 푸시 알림을 구독하고 해제하는 UI를 만들고 있었다.

그렇기에 처음에 내가 생각한 비즈니스는 더 넓었다.

단순히 구독만을 하는 기능이 아닌 사용자가 클릭하고 푸시알림을 구독할 수 있어야하며 현재 상태를 확인할 수 있는 UI를 만들고 있었다.

즉 이미 책임은 분리되어있다.

`enablePush`, `disablePush`는 푸시알림을 구독/해제 했을때의 어떻게 동작하는지에 대한 책임을 가지고 있다.

```tsx
export default function Switch(props: SwitchProps) {
  return (
    ...
  );
}
```

스위치 컴포넌트는 사용자에게 구독 상태를 그려주는 책임을 가지고 있다.

```typescript
export default function useNotificationEnabledState() {
   ...
}
```

`useNotificationEnabledState`는 푸시 알림의 구독 상태를 관리하는 책임을 가지고 있다.

![푸시 알림 스위치 UI](images/antoliny/push-notification-ui.png)

여기서 푸시 알림 상태를 나타내는 `Switch`는 본인의 상태가 어떻게 관리되는지 모른다.

이건 `useNotificationEnabledState`가 알고 있다.

또 `useNotificationEnabledState`도 어떤 방식으로 푸시 알림을 구독, 해제하는지 모른다.

FCM으로 하든 아니면 네이티브한 방법으로 하든 그냥 `enablePush`라는 함수를 호출할 뿐 내부에서 정확히 어떤 방식으로 구독을 수행하는지는 모른다.

만약 우리가 구독방식을 바꿔야 하거나, 구독 상태를 관리하는 방식을 바꿔야 하거나, 아니면 구독 상태를 표시하는 UI를 바꿔야하는 상황이 발생한다고 가정해보자.

그때 우리는 각 책임에 맞는 부분만 수정하면 될것이다.

그렇다.

조금 더 넓게 보았을때 나는 자연스럽게 객체지향을 하고 있었다.

그렇게 나는 코드를 다시 바라보았다.

차라리 `enablePush`, `disablePush`는 지금 당장은 ‘응집도/결합도’ 적인 관점보다 ‘가독성/예측 가능성’ 적인 관점에서 바라보고 리팩토링 해야했다는걸.
