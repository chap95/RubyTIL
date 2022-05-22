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
`&` 연산자는 다음의 2가지 상황에서 사용한다.
- 1. 정의된 block을 다른 함수에서 사용하고 싶을 때
- 2. 정의된 block을 Proc으로 변환시키고 싶을 때

```ruby
def my_method(&the_proc)
  the_proc
end

p = my_method {|name| "Hello, #{name}"}
p.call('tom') # => Hello, tom
```

아래 예시는 `&`연산자를 사용하지 않은 경우다.
```ruby
def my_method(the_proc)
  the_proc
end

p = my_method {|name| puts( "Hello, #{name}")}
p.call('tom') # => Hello, tom
```
위 로직을 실행하면 다음과 같은 에러를 만날 수 있다.
```
main.rb:1:in `my_method': wrong number of arguments     
(given 0, expected 1) (ArgumentError)
```

my_method의 인자 갯수는 1개지만 해당 로직에서 my_method를 인자 없이 호출했다는 의미다. 즉 block을 `&`연산자 없이 함수의 인자로 할당이 불가능하다는 의미다. `&`연산자 없이 사용하려면 아래와 같이 block을 Proc으로 만들어주면 된다.

```ruby
def my_method(the_proc)
  the_proc
end

current_proc = -> (name){ puts("Hello, #{name}") }
p = my_method(current_proc)
p.call('tom') # => Hello, tom
```
`->`키워드를 통해서 lambda로 Proc을 생성해주고 난 후 my_method에 인자로 넘겨주면 `&`연산자 없이도 정상적인 실행이 가능하다.

그렇가면 Proc을 block으로 변환하는 방법을 알아보자 위와 동일하게 `&`연산자를 사용하면 되지만 사용하게 되는 위치가 조금 다르다.

```ruby
def my_method(greeting)
 puts("#{greeting}, #{yield}")
end

my_proc =proc{"tom"}
my_method("Hello ", &my_proc)
```

my_proc을 Proc으로 만들어 둔 후에 my_method를 호출하는 부분에서 `&`를 사용하여 block으로 만들어 준 후에 `yield` 키워드로 넘겨주는 방식이다.

### proc VS lambda
지금까지 Proc을 만드는 방법으로 `Proc.new`,`lambda`,`& 연산자`에 대해서 알아보았다. 그렇다면 Proc.new 로 생성한 proc과 lambda로 생성한 proc에는 차이가 있을까? 있다면 어떤 차이가 있는지 알아보자

- return
proc 과 lambda는 내부에서 사용한 return에 차이가 있다.

```ruby
def test(callable_obj)
  callable_obj.call * 2
end

some_lambda -> { return 10 }
puts(test(some_lambda)) # => 20
```
lambda에서의 return은 우리가 흔히 알고 있는 return과 동일하다. lambda 내부에서 return 시 proc 객체를 빠져나온다.

```ruby
def test
  some_proc = Proc.new { return 10 }
  result = 0
  puts("before call Proc result => #{result}")
  result = some_proc.call
  puts("after call Proc result => #{result}")
end

test
```

하지만 proc에서의 return을 하면 proc 객체를 빠져나오는 것 뿐 아니라 proc 객체의 scope 까지 나와버리기 때문에
`after call Proc result => #{result}` 문은 실행되지 않으며
`result = some_proc.call`에서 함수가 종료되어 버린다.

```ruby
def test(callable_obj)
  puts("call test")
  callable_obj.call * 2
end

some_proc = proc { return 10 }
puts(test(some_proc))
```
위 함수에서도 `callable_obj.call` 문이 실행이 되고 그 후에 있는 2를 곱하는 연산은 실행되지 않고 `test` 함수가 종료되어 버린다.

- parameter

proc 내부로 전달된 파라미터가 정확하게 일치하지 않아도 무시하지만 lambda 내부로 전달된 파라미터가 정확하게 일치하지 않으면 에러가 발생한다. 
```ruby
some_proc = Proc.new {|a,b| puts("a => #{a}, b => #{b}")}
some_proc.call(1) # a => 1, b => 
some_proc.call(1,2) # a => 1, b => 2
some_proc.call(1,2,3) # a => 1, b => 2
```

```ruby
some_lambda = -> (a,b){puts("a => #{a}, b => #{b}")}
some_lambda.call(1) # main.rb:1:in `block in <main>': wrong number of arguments (given 1, expected 2) (ArgumentError)
some_lambda.call(1,2)
some_lambda.call(1,2,3)
```

lambda로 인자 갯수를 맞춰주지 않으면 에러가 발생하고 아래 코드가 실행되지 않는다.

proc 보다 lambda가 함수와 형태가 가까워 proc보다 lambda를 사용하는 것이 이슈를 트래킹하기 더 편하다. 그래서 일반적인 경우에는 lambda를 사용하는 것이 더 좋지만 특수한 경우에는 proc을 사용해야 한다.

### method
callable object의 마지막은 `method`이다. 루비에서는 클래스이 메소드도 뽑아낼 수 있다. 다음 예제를 보자.
```ruby
class Test
  def initialize(value)
    @value1 = value
  end

  def my_method
    @value1
  end
end

some_test = Test.new(1)
some_test_method = some_test.method :my_method
puts(some_test_method.class) # => Method
puts(some_test_method.call) # => 1
method_proc = some_test_method.to_proc
puts(method_proc.class)
puts(method_proc.lambda?)
```

위 예제에서는 뽑아낸 메소드를 `to_proc`을 활용하여 proc으로 만들었다. to_proc을 이용해서 만든 proc은 lambda 지만 lambda 키워드로 생성한 lambda와 scope에서 차이를 보인다.

lambda 키워드로 생성한 매소드는 정의된 시점의 scope 를 통해서 binding 하는 `closure`지만 `to_proc`으로 생성한 메소드는 scope가 메소드를 소유한 객체에 있다.

해당 scope를 끊어 내려면 `UnboundMethod`를 획득해야 한다. 일반적으로 클래스나 모듈 내부에서 정의된 method는 `BoundMethod`인데 이는 위의 설명과 같이 특정 객체에 scope가 존재한다. 하지만 `UnboundMethod`는 특정 객체에 scope가 바인딩 되어 있지 않는 메소드다.

`UnboundMethod`를 획득하려면 `instance_method`를 활용하면 되는데 사용하는 방법은 다음과 같다.

```ruby
module MyModule
  def my_method
    1
  end
end

some_unbound_method = MyModule.instance_method :my_method
puts(some_unbound_method.class) # => UnboundMethod
String.class_eval do
  define_method :custom_method, some_unbound_method
end

puts("some string".custom_method) # => 1
```

위와 같이 뽑아낸 `UnboundMethod`는 `UnboundMethod#bind` 또는
`Module#define_method`를 통해서 바인딩을 할 수 있다.
