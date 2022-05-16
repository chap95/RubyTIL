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
