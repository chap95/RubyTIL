### Open Class
정의가 끝난 클래스를 확장하는 기능
이 글의 출처 [강씨아저씨 티스토리](https://idea-sketch.tistory.com/36?category=662021)
***
```ruby
def to_alphanumeric(s)
  s.gsub(/[^\w\s]/, '')
end
```
위와 같이 텍스트에서 숫자와 문자만을 추출하는 로직이 있다고 가정해보자 해당 함수는 공용 함수로 존재해도 상관 없지만 `String` 클래스에 위치해 있는게 객체 지향에 맞는 구현이다. 그렇기 때문에 `String` 클래스에 해당 함수를 추가한다.

```ruby
class String
  def to_alphanumeric
    s.gsub(/[^\w\s]/, '')
  end
end
```

```ruby
'#3, league of legend !!!'.to_alphanumeric
```

위와 같이 사용이 가능하다.

오픈 클래스는 클래스를 수정할 수 있어 편리하지만 `to_alphanumeirc`이 이미 정의되어 있다면 문제가 발생한다. 내가 새롭게 추가한 함수를 나만 사용한다면 문제가 되지 않지만 타인이 작성한 코드에서 이전 버전의 함수가 사용되고 있다면 오류가 발생하고 왜 오류가 발생하는지 이유를 찾기 힘들다.
