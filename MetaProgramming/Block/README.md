### Block

많은 언어에서 블록이란 개념을 찾아볼 수 있다. 하지만 Ruby에서 블록이라는 개념은 클로저 또는 익명함수라 생각하면 된다.
Ruby에서는 `do...end` 또는 `{ }` 으로 표현을 한다.

```ruby
myArray = [1,2,3,4,5]
myArray.each do |value|
  puts(value)
end
```

이 때, `do...end` 구문이 블록이다. 위 구문은 JS 또는 Python 에서는 익숙하지 않은 문법이다. `each`라는 method에 `do...end` 구문이 전달이 된다.

---

### yield

블록 개념에는 `yield` 키워드가 있다. 설명을 먼저하면 헷갈리니 예시를 보자

```ruby
def say_my_age(age)
  puts(age)
end
```

age에 따라서 나이를 출력해주는 함수다. 지금도 훌륭한 함수지만 조금 더 유연한 방식으로 사용하고 싶은 욕심이 든다. 그 때, `yield` 키워드를 사용한다.

```ruby
def say_my_age
  yield
end
```

위와 같이 yield를 사용하면 `say_my_age` 호출할 때 블록을 넘겨주면 `yield`에 넘어오게 된다. 그러면 조금 더 함수를 유연하게 사용할 수 있다.

---

### block_given

`yield`를 사용한 함수에 블록을 넘기지 않으면 어떻게 될까? `yield` 를 사용한 함수를 블록 없이 호출할 경우 `no block given` 에러를 뿜어 낸다. 그러면 함수를 볼 때 `yield` 키워드를 사용 여부를 일일이 판단하여 사용해야 할까? 그렇게 되면 너무 불편하다. 이 때, `block_given` 키워드를 사용하면 된다.
`block_given`은 block이 함수의 인자로 주어졌는지 구분할 때 사용한다.

```ruby
  def say_my_age
    if block_given?
      yield
    else
      "you didn't give me your age"
    end
  end
```

위와 같이 block이 인자로 주어졌으면 `yield` 키워드를 사용하고 아닌 경우 예외처리를 할 수 있다.

---

### &block

블록을 함수에 넘겨주어서 `yield` 키워드로 호출하는 방법 외에도 명시적으로 블록을 함수에 넘겨주는 방법이 있다. 바로 `&block` 키워드를 사용하면 된다.

```ruby
def say_my_age(&block)
  block.call
end
```

위의 예시는 `yield` 키워드를 사용한 예시와 같은 동작을 하는 로직이다. 뿐만 아니라 블록에 인자를 전달할 수 있다.

```ruby
class Array
  def our_own_each(&block)
    for i in self
      block.call i
    end
  end
end
```

***
### instance_exec, instance_eval

위에서 살펴본 `yield` 와 `&block` 키워드와는 다른 동작을 역할을 하는 `instance_exec` 와 `instance_eval` 키워드를 알아보자.

블록은 위에서 알려준 2가지 메소드에 의해서도 호출이 될 수 있다.

```ruby
class Cat
  def do_something(&block)
    instance_exec(&block)
  end
  
  def sit
    "I am sitting"
  end
end
```
사용법은 위와 같다. `Cat` 의 `sit` 메소드를 호출하려면 

```ruby
cat = Cat.new
cat.do_something { puts cat.sit }
```

이렇게 호출하면 됐다. 하지만 `instance_exec` 메소드로 블록을 호출하면 `Cat` 클래스에 참조를 끊어도 `sit` 메소드에 접근이 가능하다.

```ruby
cat = Cat.new
cat.do_something { puts sit }
```

이 부분이 `yield` 와 다른 점이다.

이러한 프로그래밍 기능이 왜 필요한 걸까?
***
### why need 'instance_eval'
루비 언어는 `scope gate`를 가지고 있는데 다음의 예시를 통해 이해해보자.

```ruby
v1 = 1
class MyClass
  v2 = 2
  local_variables #1)
  def my_method
    v3 = 3
    local_variables #2)
  end
  local_variables #3)
end

obj = MyClass.new
obj.my_method
local_variables #4)
```
위 코드에서 넘버링 된 부분의 지역 변수는 어떤 것일지 예상해보자
```
#1) === v2
#2) === v3
#3) === v2
#4) === v1, obj
```

`local_variables`는 scope인 변수를 가리키는 것인데 내가 예상한 것과 전혀 다른 변수들을 가리키고 있었다. 내부적으로 어떻게 동작하는지 알아보자

제일 처음부분에서 `v1`을 정의하며 scope는 `v1`이 있다. 하지만 `MyClass`를 정의하는 부분으로 들어가면서 scope는 `MyClass`로 넘어가게 된다. `MyClass`의 정의가 끝나게 되면 `scope`는 `top-level`으로 돌아온다. 마지막 `my_method`를 호출하게 되면 기존에 `top-level`에 머물던 scope는 버리고 MyClass 안의 `my_method` scope를 다시 생성한다. 그리고 실행이 종료되면 `my_method` scope를 버리고 `top-level` scope로 돌아오게 된다.

이처럼 Ruby는 scope가 변경될 때 기존에 바인딩 되어있던 정보를 버리고 새로운 정보를 바인딩 하게 된다. Ruby에서는 기존의 바인딩을 버리는 키워드가 있는데 이를 바로  `scope gates`라고 한다.

루비에는 3가지의 scope gates 가 있다.
```
1. class를 정의할 때
2. module을 정의할 때
3. 함수
```
위 3가지 시점에 기존의 바인딩을 버리게 된다.

```ruby
v1 = 1
class Test
  v2 = v1
  def test_method
    v3 = v1
  end
end
```
그런데 scope gates가 있더라도 위와 같이 사용하고 싶을 때가 있다.
여러가지 방법이 있는데 여기서는 `instance_eval`만 다루어 보겠다.

아래 예제를 확인해보자
```ruby
class MyClass
  def init
    @v1 = 1
  end
end

obj = MyClass.new
obj.instance_eval do 
  self
  @v
end

v = 2
obj.instance_eval { @v1 = 2 }
obj.instance_eval { @v1 }
```
위 예시를 보면 새로운 정보가 바인딩 되어야 하지만 `MyClass` 내부 변수에 접근도 하고 새로운 값을 할당할 수 있는 모습을 볼 수 있다.
`instacne_eval`은 위처럼 `scope gate`를 무시하고 변수에 접근 및 값을 할당 할 수 있다.