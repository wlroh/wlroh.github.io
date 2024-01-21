---
layout: post
title: "테스트를 위한 프로덕션 코드는 괜찮을까?"
date: 2022-08-19
excerpt: "테스트를 위한 프로덕션 코드가 괜찮은지 다뤄보자"
categories: Spring
---

이번에는 테스트를 위한 프로덕션 코드에 대해 다뤄볼까합니다.

요즘에는 대부분의 개발자분들이 개발을 할 때 테스트 코드 또한 필수로 작성하는 분위기인 것 같습니다.
저 또한 테스트 코드를 작성하면서 개발을 하는데 이번 넥스트 스텝의 ATDD, 클린 코드 with Spring 과정을 진행 중에 리뷰어로부터 관련 주제로 피드백을 받으며 고민하면서 정리하게 되었습니다.

<br/>

## 테스트를 위한 프로덕션 코드는 괜찮을까?

테스트를 위한 프로덕션 코드는 `지양해야 한다` 라고 생각합니다. 이부분에 대해서는 대부분의 분들이 공감해주실 것 같습니다.

테스트 코드를 작성하는 이유는 다양한데, 우선 단어 그대로 **기능이 잘 구현되었는 지 테스트를 하기 위해 작성하는 코드**입니다. 그 기능을 구현한 것이 프로덕션 코드이고요.
그런 상황에서 테스트 코드를 작성하는 과정에서 불편함이 있어 테스트 코드를 위해 프로덕션 코드에 건들이게 되면 불필요한 유지보수 비용이 증가하게 되며, 주객이 전도되는 상황이 발생합니다.

만약 그런 상황이 생긴다면 그것은 `Bad Smell`이 나는 것일 수 있으며, 이것은 리팩터링이 필요한 상황일 수도 있습니다.

<br/>

## 그럼 테스트를 위한 프로덕션 코드는 괜찮지 않다고 결론이 나는 것인가?

프로그래밍은 수학처럼 정답이 정해져 있는 것은 없다라는 말처럼 테스트 코드를 위한 프로턱션 코드는 지양해야한다고 말씀드렸는데, 저는 예외로 테스트를 위한 프로덕션 코드를 만드는 경우가 있습니다.
어떤 경우에 만드는지와 그 이유에 대해 아래 코드를 보면서 이야기를 하고자 합니다.

```java
public class Lotto {

    private static final int LOTTO_NUMBER_COUNT = 6;
    private List<Integer> lottoNumbers;

    public Lotto(List<Integer> lottoNumbers) {
        if(lottoNumbers.size() != LOTTO_NUMBER_COUNT) {
            throw new IllegalArgumentException();
        }
        this.lottoNumbers = lottoNumbers;
    }

    ...
}
```

위의 코드처럼 `lottoNumbers`를 프로퍼티로 가지는 Lotto 도메인이 있습니다. Lotto 생성할 때 6개의 로또 번호가 있어야한다는 조건이 있어 테스트 코드를 작성하면 아래와 같이 작성할 수 있습니다.

```java
class LottoTest {

    @Test
    @DisplayName("로또를 생성할 때 6개의 로또 번호가 있어야 한다.")
    void invalidLotto() {
        assertThatThrownBy(() -> new Lotto(List.of(1, 2, 3, 4, 5, 6))).isInstanceOf(IllegalArgumentException.class);
    }

    @Test
    @DisplayName("로또를 생성할 때 6개의 로또 번호가 있어야 한다.")
    void invalidLotto_2() {
        assertThatThrownBy(() -> new Lotto(1, 2, 3, 4, 5, 6)).isInstanceOf(IllegalArgumentException.class);
    }
}
```

두가지 테스트 코드를 만들어 보았는데 혹시 이렇게만 봤을 때 차이점과 어떤 부분은 테스트를 위한 프로덕션 코드를 만드는지 눈치채신 분들도 있을 것 같습니다!

저는 생성자의 경우 테스트 코드에만 쓰이더라도 프로덕션 코드에 생성자 코드를 추가하고 있습니다.

테스트 코드는 **기능이 잘 구현되었는지 테스트 하기 위한 목적** 으로 사용되기도 하고, **프로덕션 코드에 대한 명세를 위한 목적** 으로도 사용됩니다.
결국 테스트코드 또한 작성만 하고 끝이 아닌 프로덕션 코드처럼 끊임없이 유지보수가 들어가야하는 코드입니다.

그럼 두가지의 테스트 코드중에 어떤 것이 더 유지보수 하기 좋을까요? 유지보수를 하기 쉽다는 것은 여러 의미가 있는데 그 중에 읽기 쉬운 코드 또한
유지보수하기 쉬운 코드로 볼 수 있을 것 같습니다. 그런 의미에서 저는 두 번째 테스트 코드가 유지보수 하기 쉬운 테스트 코드로 보여집니다.

또한 **다양한 생성자가 있을수록 더 유연하게 개발** 을 할 수 있게 됩니다. 현재는 테스트 코드에만 쓰이는 생성자이지만, 추후 요구사항이 더 생기면서 Lotto 객체를 다른 곳에서
생성하는 경우가 생길 수 있는데 다양한 생성자가 있으면, 유연하게 개발을 할 수 있는 장점이 생기며 프로덕션 코드를 위한 생성자로 쓰일 수도 있습니다.

```java
public class Lotto {

    private static final int LOTTO_NUMBER_COUNT = 6;
    private List<Integer> lottoNumbers;

    public Lotto(int... lottoNumbers) {
        this(List.of(lottoNumbers));
    }

    public Lotto(List<Integer> lottoNumbers) {
        if(lottoNumbers.size() != LOTTO_NUMBER_COUNT) {
            throw new IllegalArgumentException();
        }
        this.lottoNumbers = lottoNumbers;
    }

    ...
}

class LottoTest {

    @Test
    @DisplayName("로또를 생성할 때 6개의 로또 번호가 있어야 한다.")
    void invalidLotto_2() {
        assertThatThrownBy(() -> new Lotto(1, 2, 3, 4, 5, 6)).isInstanceOf(IllegalArgumentException.class);
    }
}
```

그래서 최종적으로 이렇게 테스트 코드를 위해 프로덕션 코드에 새로운 생성자를 추가하는 것에 대해서는 괜찮다고 생각합니다.
이렇게 나름의 개발철학을 만들며 성장 중에 ATDD 과정 중 리뷰어로부터 피드백을 받으며 고민이 생겼던 것이 있었습니다.

JPA의 경우 Entity마다 id를 만들어주어야 하는데 해당 id값은 개발자가 할당하지 않는 값입니다. 이 상황에서 id를 사용하는 테스트가 있을 경우
이 경우도 생성자로 추가해도 괜찮을지 고민이 되었습니다.

<br/>

## 프로덕션 코드로 사용가능성이 없는 생성자의 경우도 괜찮을까?

위에서 언급했듯이 저는 생성자의 경우 테스트를 위해 추가하는 것에 대해서는 두가지 이유로 허용해주고 있습니다.

1. 생성자가 다양해짐으로써 유연한 개발이 가능하여 추후 프로덕션 코드로써도 사용이 가능해진다.
2. 테스트 코드도 유지보수 범위에 포함되어 있기 때문에 유지보수를 고려해서 작성해야한다.

하지만 id의 경우 프로덕션 코드로 사용될 일이 나중에서도 없을 것 같습니다. 물론 2번째 사유로 인해 생성자로 추가해도 괜찮지만,
저는 찜찜한 부분이 있어 리뷰어님의 피드백을 바탕으로 2가지 해결책을 찾았습니다.

```java
public void addLineStation(Long lineId, LineStationCreateRequest request) {

    ...

    line.addLineStation(lineStation);
}
```

### Stubbing

```java
@DisplayName("지하철 노선에 역을 등록한다.")
@Test
void addLineStation() {
    // given
    Station station = mock(Station.class);
    when(station.getId()).thenReturn(1L);
    when(stationRepository.findAllById(anyList())).thenReturn(Lists.newArrayList(station));
    ...
}
```

첫번째는 `Stubbing`을 이용하는 방법입니다. 하지만 Stubbing을 사용하기 위해서는 테스트 코드에서 상세 구현을 알아야한다는 점때문에 Stubbing 대신 두번째로 설명할 `Reflection` 을 활용하는 방법을
더 좋아합니다.

### ReflectionTestUtils

```java
@DisplayName("지하철 노선에 역을 등록한다.")
@Test
void addLineStation() {
    // given
    Station station = createStation(1L, "강남역");
    when(stationRepository.findAllById(anyList())).thenReturn(Lists.newArrayList(station));
    ...
}

private Station createStation(Long id, String name) {
    Station station = new Station(name);
    ReflectionTestUtils.setField(station, "id", 1L);
    return station;
}
```

`ReflectionTestUtils`를 이용해서 필요한 값들을 추가로 할당하는 방법입니다. ReflectionTestUtils를 이용해서 별도로 생성메서드를 분리해서 사용하면 재사용성도 증가되며, 프로덕션 코드로 사용가능성이 없는 생성자를 추가하지 않고 해결이 가능합니다.

<br/>

## 결론

이야기가 길어져 정리를 다시 해보려고 합니다. 물론 결론이 정답은 아니고 다른 방법들이 더 있을 수 있습니다.
수업을 들으면서 테스트를 위한 프로덕션 코드에 대한 제 의견을 정리한 것입니다.

1. 테스트를 위한 프로덕션 코드는 **지양**해야 한다
2. **생성자**의 경우 테스트를 위한 프로덕션 코드가 추가하는 경우는 **허용**한다.
3. 다만, 프로덕션 코드로 사용이 될 가능성이 없는 생성자의 경우 `ReflectionTestUtils` 를 이용한다.

이렇게 세 줄로 정리할 수 있지 않을까 생각합니다.
