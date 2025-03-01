# 디바운싱(debouncing)과 쓰로틀링(throttling)

### 들어가기

코드는 0과 1의 세계다. 즉, 0과 1의 이분법으로 명확하게 나누어지는 영역이다. 반면 우리가 살아가는 현실 세계는 이분법적으로만 구분하기 어려운 경우가 많다. 예를 들어 어떤 사람이 방에 들어오는 상황을 생각해보자. 한 발만 방에 걸쳐둔 상태를 ‘들어왔다’고 해야 할까, 아니면 온전히 방에 들어왔을 때를 ‘들어왔다’고 해야 할까?

아날로그를 디지털로 변환한다는 것은 미세한 부분까지 잘게 쪼개고 구분한다는 의미다. 그 작은 차이 하나하나도 나누어 인식하게 된다.

웹에서 발생하는 이벤트도 마찬가지다. 만약 “방에 들어오면 이벤트를 실행해 주세요”라고 코드를 작성한다면 ‘신발 끝이 들어왔다’, ‘신발이 전부 들어왔다’, ‘다리가 들어왔다’, 등 아주 사소한 단계마다 이벤트가 발생한다. 모든 단계마다 이벤트가 발생하고 API요청까지 해야한다면 불필요한 과부하가 발생한다. 이를 방지하고 효율적으로 이벤트를 관리하는 방법이 **디바운싱(debouncing)**과 **쓰로틀링(throttling)**이다.

오늘 살펴볼 디바운싱과 쓰로틀링은 이처럼 0과 1로 세분화된 디지털 세계에서 불필요한 부분은 걸러내고, 필요한 동작만 효율적으로 처리하기 위한 기법이다.

## 디바운싱

### 1. 어원

ON/OFF 스위치는 0과 1로 구분되어 아날로그에서 디지털로 전환하기 쉬워 보이지만, 사실 아날로그 스위치도 0과 1로 정확히 나뉘지 않는다. 왜냐하면 스위치를 켜고 끄는 과정에서 ‘켜지는 중’, ‘꺼지는 중’과 같은 아주 짧은 순간, 즉 접점이 여러 번 튀는 현상이 발생하기 때문이다. 예를 들어, “켜지는 0.0001초의 짧은 순간을 켜짐으로 볼지, 꺼짐으로 볼지”와 같은 문제가 생긴다. 이처럼 스위치가 접점에서 떨어지거나 붙는 사이에 ON과 OFF 상태가 반복되는 현상을 바운싱(bouncing) 이라고 한다. 그리고 이 바운싱을 방지하기 위해 고안된 기법이 바로 디바운싱(debouncing) 이다.

### 2. 정의

스위치를 ON으로 올리는 과정을 떠올려보면, 그 짧은 순간에는 ON/OFF가 번갈아 발생하지만 최종적으로는 ON 상태를 유지한다. 결국 **마지막에 멈춰선 상태가 중요하다.**

마찬가지로 코드에서 불필요하게 동일한 이벤트가 여러 번 호출될 때, 반복 호출을 무시하고 마지막에 발생한 이벤트만 유효하게 처리하고 싶을 때 디바운싱을 사용한다.

### 2. 활용

-   사용자 검색 기능
-   이메일 중복 확인

input창에 입력이 있을 때마다 API를 호출하지 않고, 입력을 마치고 잠시 멈췄을 때 API를 호출하면 불필요한 API 요청을 방지할 수 있다.

### 3. 코드

```js
const debounce = (func) => {
    let timer;
    return (e) => {
        if (timer) clearTimeout(timer);
        // 타이머가 있을 경우 clearTimeout 실행
        timer = setTimeout(() => {
            func(e);
        }, 500);
    };
};
```

위 코드는 setTimeout을 사용해 디바운싱을 간단히 구현한 예시다.

내부에 timer 변수를 두고, 함수가 호출될 때마다 이전 타이머가 있으면 clearTimeout(timer)로 초기화한다.
그 후 새롭게 타이머를 설정하고, 정해진 시간이 지나도록 더 이상 함수 호출이 없으면 그제서야 func(e)를 실행한다.

즉, 마지막 호출 후 설정된 시간(여기서는 500ms)이 지날 때까지 새 호출이 없으면 함수를 실행한다.

실무에서는 매번 디바운스 함수를 구현하기보다 Lodash 같은 라이브러리에서 제공하는 debounce 함수를 자주 사용한다. 예를 들어, Lodash에서 제공하는 debounce를 사용하면 아래와 같이 작성할 수 있다.

```js
const handleMouseEnter = debounce((id: number) => {
    if (hoverId === id) return;
    setHoverId(id);
}, 3000);
```

이 코드는 3초 동안 새로운 이벤트가 발생하지 않을 때에만 실제 로직(setHoverId)을 실행한다.

이처럼 디바운싱은 이벤트의 ‘마지막 상태’를 중요하게 여겨, 지정된 시간 내에 반복적으로 발생하는 이벤트를 무시하고, 최종 호출만 유효하게 처리한다. 이런 기법을 통해 불필요한 자원 낭비를 줄이고, 사용자 경험을 부드럽게 만들 수 있다.

## 쓰로틀링

### 1. 어원

Throttle은 '목을 조르다', '(밸브의) 조절판'이라는 뜻이 있다. 예를 들어, 물이 **콸콸** 쏟아질 때 밸브를 적당히 잠가 물이 **똑똑** 떨어지도록 조절하는 느낌이다.

즉, 밸브를 조절하듯 이벤트를 제어하는 기법이다. 디바운싱이 일정 시간 동안 이벤트를 완전히 막고 마지막 이벤트만 처리하는 것과 달리, 쓰로틀링은 특정 간격으로 이벤트를 실행한다.

### 2. 정의

이벤트가 콸콸콸 쏟아지는 상황을 생각해보자. 앞서 말한 마우스 이벤트를 생각해보면 마우스가 살짝만 움직여도 이벤트가 콸콸콸 쏟아진다. 이러한 이벤트를 적당히 조절해서 물이 똑-똑- 떨어지듯이 이벤트의 동작을 제어한다. 결국 이벤트가 **똑-똑 떨어지는 시점**이 언제인지가 중요하다.

마찬가지로 코드에서 불필요하게 동일한 이벤트가 여러 번 호출될 때, 쏟아지는 이벤트를 조절하여 마치 물이 1초에 한 방울만 떨어지는 것처럼, 일정 시간 당 딱 한번만 이벤트를 유효하게 처리하고 싶을 때 쓰로틀링을 사용한다.

즉, 이벤트가 실행되는 주기를 조절하는 것이 핵심이다.

### 2. 활용

-   마우스 호버 이벤트
-   무한 스크롤 기능

마우스 이벤트는 물이 콸콸콸 쏟아지는 것처럼 무수한 이벤트를 뱉어낸다. 조금만 마우스를 움직여도 이벤트가 콸콸콸 쏟아진다. 이러한 마우스 이벤트에 api 호출을 연결하면 어떻게 될까? 마우스를 마구 움직이면 그때마다 계속 api를 호출하게 될 것이다. 이 때, 특정 시점마다 딱 이벤트가 한번만 호출되도록 하고 싶은 경우 쓰로틀링을 사용할 수 있다.

### 3. 코드

```js
const throttle = (func, delay = 500) => {
    let lastTime = 0;

    return function () {
        const now = new Date().getTime();

        if (now - lastTime >= delay) {
            func();
            lastTime = now;
        }
    };
};
```

위 코드는 setTimeout을 사용하지 않고, 시간 차이를 계산하여 일정한 간격으로만 이벤트가 실행되도록 하는 쓰로틀링 기법이다. (timeout을 사용할 수도 있다)

현재의 시간이 최초 마지막 시간 + delay시간이 지나는 경우에 함수를 호출한다.

즉, 이벤트가 계속 발생하더라도 delay 간격마다 한 번씩만 실행된다.

Lodash의 throttle을 사용하면 다음과 같이 작성할 수 있다.

```js
const handleMouseEnter = throttle((id: number) => {
    if (hoverId === id) return;
    setHoverId(id);
}, 3000);
```

이처럼 throttle은 이벤트가 동작하는 '기간'을 중요하게 여겨, 지정된 시간 내에 딱 한번만 이벤트가 실행되도록 조절한다. 이런 기법을 통해 불필요한 자원 낭비를 줄이고, 사용자 경험을 부드럽게 만들 수 있다.

---

### 추가: Lodash에서 구현한 debounce와 throttle

공부하다보니 Lodash에서는 어떻게 debounce와 thrrotle을 구현했는지 궁금해져서 찾아보았다.

```JS
    function debounce(func, wait, options) {
      var lastArgs,
          lastThis,
          maxWait,
          result,
          timerId,
          lastCallTime,
          lastInvokeTime = 0,
          leading = false,
          maxing = false,
          trailing = true;

      if (typeof func != 'function') {
        throw new TypeError(FUNC_ERROR_TEXT);
      }
      wait = toNumber(wait) || 0;
      if (isObject(options)) {
        leading = !!options.leading;
        maxing = 'maxWait' in options;
        maxWait = maxing ? nativeMax(toNumber(options.maxWait) || 0, wait) : maxWait;
        trailing = 'trailing' in options ? !!options.trailing : trailing;
      }

      function invokeFunc(time) {
        var args = lastArgs,
            thisArg = lastThis;

        lastArgs = lastThis = undefined;
        lastInvokeTime = time;
        result = func.apply(thisArg, args);
        return result;
      }

      function leadingEdge(time) {
        // Reset any `maxWait` timer.
        lastInvokeTime = time;
        // Start the timer for the trailing edge.
        timerId = setTimeout(timerExpired, wait);
        // Invoke the leading edge.
        return leading ? invokeFunc(time) : result;
      }

      function remainingWait(time) {
        var timeSinceLastCall = time - lastCallTime,
            timeSinceLastInvoke = time - lastInvokeTime,
            timeWaiting = wait - timeSinceLastCall;

        return maxing
          ? nativeMin(timeWaiting, maxWait - timeSinceLastInvoke)
          : timeWaiting;
      }

      function shouldInvoke(time) {
        var timeSinceLastCall = time - lastCallTime,
            timeSinceLastInvoke = time - lastInvokeTime;

        // Either this is the first call, activity has stopped and we're at the
        // trailing edge, the system time has gone backwards and we're treating
        // it as the trailing edge, or we've hit the `maxWait` limit.
        return (lastCallTime === undefined || (timeSinceLastCall >= wait) ||
          (timeSinceLastCall < 0) || (maxing && timeSinceLastInvoke >= maxWait));
      }

      function timerExpired() {
        var time = now();
        if (shouldInvoke(time)) {
          return trailingEdge(time);
        }
        // Restart the timer.
        timerId = setTimeout(timerExpired, remainingWait(time));
      }

      function trailingEdge(time) {
        timerId = undefined;

        // Only invoke if we have `lastArgs` which means `func` has been
        // debounced at least once.
        if (trailing && lastArgs) {
          return invokeFunc(time);
        }
        lastArgs = lastThis = undefined;
        return result;
      }

      function cancel() {
        if (timerId !== undefined) {
          clearTimeout(timerId);
        }
        lastInvokeTime = 0;
        lastArgs = lastCallTime = lastThis = timerId = undefined;
      }

      function flush() {
        return timerId === undefined ? result : trailingEdge(now());
      }

      function debounced() {
        var time = now(),
            isInvoking = shouldInvoke(time);

        lastArgs = arguments;
        lastThis = this;
        lastCallTime = time;

        if (isInvoking) {
          if (timerId === undefined) {
            return leadingEdge(lastCallTime);
          }
          if (maxing) {
            // Handle invocations in a tight loop.
            clearTimeout(timerId);
            timerId = setTimeout(timerExpired, wait);
            return invokeFunc(lastCallTime);
          }
        }
        if (timerId === undefined) {
          timerId = setTimeout(timerExpired, wait);
        }
        return result;
      }
      debounced.cancel = cancel;
      debounced.flush = flush;
      return debounced;
    }
```

Lodash는 throttle을 구현하기 위해 debounce를 활용했다.

```js
function throttle(func, wait, options) {
    var leading = true,
        trailing = true;

    if (typeof func != 'function') {
        throw new TypeError(FUNC_ERROR_TEXT);
    }
    if (isObject(options)) {
        leading = 'leading' in options ? !!options.leading : leading;
        trailing = 'trailing' in options ? !!options.trailing : trailing;
    }
    return debounce(func, wait, {
        leading: leading,
        maxWait: wait,
        trailing: trailing,
    });
}
```

출처: [lodash Github](https://github.com/lodash/lodash/blob/4.17.21/lodash.js#L10965)
