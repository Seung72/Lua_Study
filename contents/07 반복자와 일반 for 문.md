# 07. 반복자와 일반 for 문

## 7.1 반복자와 클로저

반복자는 집합의 각 요소를 훑는 구성물을 말한다. 루아에서 반복자는 호출될 때마다 집합으로부터 '다음' 요소를 반환하는 보통의 함수로 표현된다.

클로저는 자신을 감싼 환경의 지역 변수에 접근할 수 있는 함수이기 때문에, 반복자의 호출들 사이에서 현재 실행된 시점과 그 시점에서 계속 진행할 방법에 대해 훌륭한 방법을 제공한다.

클로저 생성에는 일반적으로 함수 두 개가 필요하다.

- 클로저 자신
- 클로저를 생성하는 공장 함수

```Lua
function values(t)
    local i = 0
    return fuction() i = i + 1; return t[i] end
end
```

- 이 경우 반복자는 `ipairs`와 다르게 각 요소의 인덱스를 반환하지 않고 값만 반환한다.
- `values`는 공장함수다. 이 함수를 호출할 때마다 새로운 클로저가 만들어진다.
  - 이 클로저는 외부 변수 t와 i의 상태를 계속 유지한다.
  - 반복자를 호출할 때마다 목록 t의 다음 차례에 해당하는 값을 반환한다.
  - 마지막 요소를 지나면 반복자는 nil을 반환하여 반복 처리의 끝을 알린다.

반복자를 `while`문 안에서 사용할 수도 있다.

```Lua
-- while문
t = {10, 20, 30}
iter = values(t)
while true do
    local element = iter()
    if element == nil then break end
    print(element)
end

-- 일반 for문
t = {10, 20, 30}
for element in values(t) do
    print(element)
end
```

보다시피 일반 for문을 사용하는 것이 더 쉽다. 일반 for문은 반복을 위한 모든 잡다한 관리 작업을 처리한다. 이 구문은 내부에 반복자 함수를 보관하므로, iter 변수가 필요 없다. 새로운 반복마다 반복자ㅏ를 호출하며, 반복자가 nil을 반환할 때 반복문을 중지한다.

## 7.2 일반 for문의 문법

위 반복문 예시들은 새로운 반복문마다 클로저를 새로 생성해야 하는 단점이 있다. 대부분의 경우 문제가 되지 않지만 특정 상황에서는 불편할 수 있다. 그럴 때 일반 for문 자신이 반복 상태를 유지하도록 지정한다. 일반 for문의 상태 유지 기능에 대해서 설명한다. 일반 for문에서 유지되는 것은 세 개의 값이다.

- 반복자 함수
- 불변 상태
- 제어 변수

일반 for문의 문법은 다음과 같다

```Lua
for <변수 목록> in <수식 목록> do
    <본체>
end
```

- 변수 목록: 한 개 이상의 변수 이름을 쉼표로 구분한 목록
- 수식 목록: 한 개 이상의 수식들을 쉼표로 구분한 목록

`for`문이 제일 먼저 하는 일은 in 연산자 뒤에 놓인 수식을 계산하는 것이다. 이 수식들은 `for`문이 유지한 세 개의 값이 되어야 한다. 다중 배정과 마찬가지로, 리스트의 마지막 요소는 한 개 이상의 값으로 끝날 수 있다.
이러한 초기화 단계가 끝난 뒤, `for`문은 불변 상태 값과 제어변수를 사용하여 반복자 함수를 호출한다. 그런 다음 `for`문은 변수 목록에 선언되어 있는 변수들에 반복자 함수의 반환값을 배정한다. 만약 제어 변수로 배정할 첫 번째 반환값이 nil이라면 반복문 실행은 멈춘다. 그렇지 않다면 `for`문은 본체 부분을 실행하고 반복자 함수를 다시 호출하고 처리 과정을 반복한다.

```Lua
for var_1, ..., var_n in <수식 목록> do <블록> end

-- 두 코드는 실질적으로 같다.

do
    local _f, _s, _var = <수식 목록>
    while true do
        local var_1, ..., var_n = _f(_s, _var)
        _var = var_1
        if var == nil then break end
        <블록>
    end
end
```

## 7.3 무상태 반복자

스스로 어떤 상태도 유지하지 않는 반복자를 말한다. 그러므로 다중 반복문에서 동일한 무상태 반복자를 사용한다면, 새로운 클로저를 생성하는 부담을 피할 수 있다.
매 회 실행 시 `for` 반복문은 자신의 반복자 함수를 불변 상태값과 제어 변수 초기값을 사용하여 호출한다. 무상태 반복자는 이 두 값만 사용하여 다음 차례 반복 요소를 생성한다. `ipairs`가 전형적인 예다.

```Lua
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end
```

`ipairs`와 반복자 함수 모두 아주 간단하다. 이 두 함수는 루아로 다음과 같이 작성할 수 있다.

```Lua
local function iter(a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end

function ipairs (a)
    return iter, a, 0
end
```

진행 순서는 다음과 같다.

- ipairs를 호출하면 반환값 세 개를 받는다.
- 그 다음 루아는 `iter(a, 0)`을 호출하여 1, a[1]을 반환받는다.
- 이런식으로 첫 번째 반환값이 `nil`이 될 때까지 반복한다.

테이블의 모든 요소를 반복 처리하는 `pairs` 함수는 반복자 함수가 루아에서 제공하는 기본 함수인 `next`라는 점을 제외하고 대부분 비슷하다.

```lua
function pairs(t)
    return next, t, nil
end
```

k가 테이블 t의 항목을 뜻하는 거라면 next는 테이블에서 k의 다음 키와 두 번째 반환 값으로서 그 키에 대응하는 값을 반환한다. next(t, nil)은 첫 번째 키-값 쌍을 반환한다. 다음 결과 쌍이 더 이상 없을 때 next는 nil을 반환한다.

## 7.4 복잡한 상태를 사진 반복자 함수

반복자 함수를 만들다 보면 하나의 불변 상태값과 하나의 제어 변수만으로 사용하는 경우보다 더 많은 상태값을 유지해야 할 경우를 자주 접하게 된다. 이 경우 클로저를 사용하는 것이 가장 간단한 해결책이다. 차선책은 필요한 모든 정보를 한 테이블에 넣고 이 테이블을 반복 처리의 불변 상태 값으로 사용하는 것이다. 상태값이 언제나 동일한 테이블이라고 하더라도 테이블 내용은 반복을 실행하는 도중에 변경할 수 있다.

```lua
local iterator 나중에 정의한다.
function allwords ()
    local state = {line = io.read() , pos = 1}
    return iterator, state
end

function iterator (state)
    while state.line do - 읽어 들일 줄이 남아 있을 동안 반복
        local s , e string.find(state.line, "%W+"， state.pos)
        if s then
            state.pos e + 1
            return string.sub(state.line, s, e)
        else
            state.line = io.read()
            state . pos = 1 -- 첫 번째 위치부터 시작
        end
    end
    return nil
end
```

가능하다면 언제나 for 변수들에 자신의 모든 상태값을 담는 무상태 반복자를 작성하는 방법을 시도해야만 한다. 이 방법을 사용하면 반복문을 시작할 때 새로운 객체들을 생성하지 않아도 된다. 만약 이 방법을 반복자로 처리할 수 없는 경우에는 클로저를 사용한다. 클러저는 테이블을 사용한 반복자보다 효율이 좋다. 그리고 테이블 필드에 접근하는 것보다 비지역 변수에 접근하는 것이 빠르다.

## 7.5 진짜 반복자

반복자(iterator)라는 이름은 단어 의미와 약간 다르게 사용된다. 실제로 반복되는 것은 반복자 함수가 아니라 for 반복문이기 때문이다. 값을 계속 찍어내기 때문에 발생자가 더 적절한 명칭이겠지만, 이미 여러 언어에서 반복자란 용어를 흔히 사용한다.
하지만 반복자가 내부에서 실제로 반복 처리를 빌드하는 또 다른 방법이 있다. 이러한 반복자의 사용은 반복 처리를 할 때마다 반복자가 해야 할 일을 설명하는 함수를 인수로 지정해서 반복자를 호출한다.

```Lua
function allwords (f)
    for line in io.lines() do
        for word in string.gmatch(line, "%w+") do
            f(word)
        end
    end
end
```

이 반복자를 사용하려면, 함수로 반복문 본체를 제공하여야 한다.

```Lua
allwords(print)
```

종종 본체 역할을 하는 함수로 익명 함수를 사용한다.

```Lua
local cnt = 0
allwords(function (w)
    if w == "hello" then cnt = cnt + 1 end
end)
print(cnt)

-- 반복자 형식으로 작성 --
local cnt = 0
for w in allwords() do
    if w == "hello" then cnt = cnt + 1 end
end
print(cnt)
```

---

출처
[프로그래밍 루아](https://search.shopping.naver.com/book/catalog/32492196601?query=%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%20%EB%A3%A8%EC%95%84&NaPm=ct%3Dlg0lm4tc%7Cci%3D4ff95860ae3926b4ecded18df1cd90c3f669c3f1%7Ctr%3Dboksl%7Csn%3D95694%7Chk%3D03a50a86d09e9d6f540477098e76363425b34610)
