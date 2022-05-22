# CallableObject
출처 : [강씨아저씨](https://idea-sketch.tistory.com/40?category=662021)

해당 정리에서 다룰 내용은 아래 3가지 이다.
- Proc
- Lambda
- method

block의 동작은 `먼저 정의 후 나중에 실행` 이 2단계로 나뉘는데 `block`만 이 2단계로 정의되는 것은 아니다.

위에서 언급한 3가지도 `먼저 정의 후 나중에 실행`이 가능하다. 위 3가지 + block을 포함해서 `callable object`라 부른다.


proc 클래스 인스턴스를 통해서 Callable Object를 만드는 법
```ruby
increase = Proc.new {|x| x + 1}
# some code
increase(2) # => 3
```
위와 같은 코드가 우리가 잘 알고 있는 지연평가 방식이다.

Ruby에서는 Proc 클래스의 인스턴스를 만드는 방법 외에도 proc 키워드나 lambda 키워드를 사용하는 방법이 있다.

```ruby
decrease = lambda {|x| x - 1 }
decrease.class # => Proc
decrease(2) # => 1


decrease = proc {|x| x - 1 }
decrease.class # => Proc
decrease(2) # => 1

decrease = -> (x) {x - 1} # as same as lambda {|x| x - 1}
```
***
### Proc Object

위에서 Proc을 선언하고 사용하는 방법을 간단하게 알아보았다. 이제 Proc이 정확히 무엇이며 왜 사용하는지 알아보자

Proc은 코드 블록을 캡슐화 한 것인데 이는 지역변수에 할당도 되고 함수나 다른 Proc에 인자로 전달도 가능하며 호출도 가능하다. Proc은 함수형 프로그래밍의 핵심적인 개념과 루비 라는 언어에 필수적인 요소다.

Proc은 `closure`이며 이는 해당 객체가 생성된 전체 context를 기억하고 사용할 수 있음을 의미한다.

설명을 정리해보았는데 `closure(클로저)`가 무엇인지 정확히 이해가 가지 않는다. 그래서 클로저에 대해서 간단하게 정리해 보겠다.

### Closure
클로저는 간단히 정리하면 내부 함수가 외부의 context(맥락)에 접근할 수 있음을 의미한다. 정확히 말하면 자유변수에 함수가 닫혀있는 것을 의미하는데 정확한 의미보다 위와 같이 클로저로 할 수 있는 것들로 이해하면 더욱 쉽게 이해할 수 있다.

다음은 `lambda`를 통해 클로저를 만든 예제이다. 클로저가 아닌 일반함수와의 비교 코드 또한 함께 구현했다.

```ruby
def outer
  x = 10;

  def innerGeneral(x)
    puts("parameter x => #{x}") # => 11
    x += 1
  end
  innerLambda = -> { x += 1 }
  puts("outer x => #{x}") # => 10
  innerLambda.call()
  puts("after call innerLambda x => #{x}") # => 11
  innerGeneral(x)
  puts("after call innerGeneral x => #{x}") # => 11
end

outer
```

위 예시에서 `innerGeneral` 함수는 일반 함수이고 `innerLambda`는 lambda 키워들 통해서 선언한 클로저이다.
outer 함수는 x라는 지역변수에 10을 할당하여 사용한다. innerLambda와 innerGeneral을 동일하게 x를 하나 증가시키는 로직을 수행하는데 일반함수를 실행하고 나면 x가 증가하지 않지만 lambda함수를 실행하고 나면 x가 증가했음을 확인 가능하다.


### & 연산자
`&`