# Spelling Corrector


## 소스 코드
소스코드 파일 : [corrector.py](corrector.py)
``` python
import re, collections


def words(text):
    return re.findall('[a-z]+', text.lower())


def train(features):
    model = collections.defaultdict(lambda: 1)
    for f in features:
        model[f] += 1
    return model


NWORDS = train(words(file('big.txt').read()))

alphabet = 'abcdefghijklmnopqrstuvwxyz'


def edits1(word):
    splits = [(word[:i], word[i:])
              for i in range(len(word) + 1)]

    deletes = [a + b[1:]
               for a, b in splits if b]
    transposes = [a + b[1] + b[0] + b[2:]
                  for a, b in splits if len(b) > 1]
    replaces = [a + c + b[1:]
                for a, b in splits for c in alphabet if b]
    inserts = [a + c + b
               for a, b in splits for c in alphabet]

    return set(deletes + transposes + replaces + inserts)


def known_edits2(word):
    return set(e2 for e1 in edits1(word)
               for e2 in edits1(e1) if e2 in NWORDS)


def known(words):
    return set(w for w in words if w in NWORDS)


def correct(word):
    candidates = known([word]) \
                 or known(edits1(word)) \
                 or known_edits2(word) \
                 or [word]
    return max(candidates, key=NWORDS.get)


# Test corrector
print 'speling =>', correct('speling')

```

## 확률 이론
`correct` 함수는 단어가 주어졌을대 가장 가능성이 높은 단어를 찾아내야한다.
만약에 'lates'라는 단어가 주어지면, 'late'랑 'latest'중에 무엇으로 교정해야할까?
최대한 가능성이 높은 단어를 선택하기위해서 확률을 이용한다.
단어 w가 주어졌을때 각 교정 결과 후보 c에 조건부 확률 값을 배정하고 그 중 확률을 최대화하는 후보를 선택하는것으로 해보자.
이런 c를 다음과 같이 쓸 수 있다.

(1) ![](https://latex.codecogs.com/gif.latex?%5Cmax_%7Bc%7D%20P%5Cleft%28c%7Cw%5Cright%29)

베이즈 정리에 의하면 이것을 아래처럼 고쳐 쓸 수 있다

(2) ![](https://latex.codecogs.com/gif.latex?%5Carg%5Cmax_%7Bc%7D%20%5Cfrac%7B%20P%28w%7Cc%29%20%5Ccdot%20P%28c%29%20%7D%7B%20P%28w%29%20%7D)

이 식에서 P(w)는 모든 후보 c에 대해 같은 값이므로 생략.
그러면 아래 식을 얻을 수 있다.

(3) ![](https://latex.codecogs.com/gif.latex?%5Carg%5Cmax_%7Bc%7D%20P%28w%7Cc%29%20%5Ccdot%20P%28c%29)

**언어 모델 : P(c)**
영어 텍스트에서 c가 나타날 확률을 뜻한다.
예를 들어, "the"는 영어 텍스트에서 7% 정도 있다.
`P(the)=0.07`이다.

**오류 모델 : P(w|c)**
c를 치려다가 실수로 w를 입력할 확률을 뜻한다.
예를 들면 P(teh|the)은 높지만, P(theeexyz|the)는 낮다.

P(c|w)를 계산하기 위해서는 두개의 요소를 고려해야한다
예를 들어 w="thew"가 입력되었다고 하자.
그럼 두 개의 후보 c="the"와 c="thaw" 중 확률 P(c|w)가 높은 것은 어느 쪽일까?
a를 e로 바꾸기만 하면 되니 "thaw"가 그럴듯해 보일수도 있다.
하지만 "the"도 그럴듯해 보인는 것이, the는 영어에서 굉장히 흔하게 쓰이고, e를 치면서 실수로 옆에 있는 w를 같이 눌렀을 수도 있다.
이렇듯 P(c|w)를 직접 계산하려고 해도 위에서 언급한 두 개의 요소를 모두 고려해야 하므로, 차라리 명시적으로 이들을 분리하고 이름을 붙이는게 낫다.

### P(c)를 계산하는 부분
이 코드는 백만개 가량의 단어를 포함하는 텍스트 파일 'big.txt'를 읽어들인다.
'big.txt'를 읽어들인 후 텍스트에 개별 단어들을 `words` 함수를 써서 추출한다.
이 함수는 알파벳 문자열들을 단어로 분리하고, 모든 알파벳을 소문자로 변환한다.
파일을 읽어들인 뒤 `train` 함수에서 확률 모델을 훈련한다. (단순히 몇번 등장하는지 카운팅)

``` python
def words(text):
    return re.findall('[a-z]+', text.lower())


def train(features):
    model = collections.defaultdict(lambda: 1)
    for f in features:
        model[f] += 1
    return model


NWORDS = train(words(file('big.txt').read()))
```

`NWORD[w]`는 파일안에서 단어 `w`가 몇 번이나 출현했는데 `dict` 형태로 들어있다.
여기서 문제가 하나 있다.
NWORD에 안들어 있는 단어는 어떻게 처리할까 이다.
여러가지 방법이 있는데 가장 간단한 방법은 새로운 단어들은 모두 한 번 봤다고 가정하는 것이다.
이와 같은 기법은 단어들의 확률분포에서 0이어야 할 부분들을 최소값인 1로 늘림으로써 분포를 평평하게 하는 효과가 있어서,
일반적으로 **평활법(smoothing)**이라고 한다.

이 소스코드에서는 파이썬의 `dict`구조체 대신 `collections.defaultdict`을 사용해서 구현했다.
이 클래스는 `dict`와 똑같이 작동하지만 처음 보는 키의 기본키를 직접 지정할 수 있다는 차이점이 있다.
`collections.defaultdict(lambda: 1)`여기서 `lambda: 1`이라는 코드로 1로 지정했다.

일반적으로 두 단어가 얼마나 비슷한가를 측정할때 **편집거리(edit distance)**를 사용한다.
한 단어를 다른 단어로 바꾸기 위해 필요한 연산의 최소 수를 의미한다.
연산의 종류는 다음과 같다.

| 연산 | 설명 |
| :--: | :-- |
| delete | 한 글자 삭제 |
| transpose | 인접한 두 글자를 바꾸기 |
| replace | 한 글자를 다른 알파벳으로 변경 |
| insert | 새 글자를 삽입 |

구현 소스코드는 아래와 같다.

``` python
def edits1(word):
    splits = [(word[:i], word[i:])
              for i in range(len(word) + 1)]

    deletes = [a + b[1:]
               for a, b in splits if b]
    transposes = [a + b[1] + b[0] + b[2:]
                  for a, b in splits if len(b) > 1]
    replaces = [a + c + b[1:]
                for a, b in splits for c in alphabet if b]
    inserts = [a + c + b
               for a, b in splits for c in alphabet]

    return set(deletes + transposes + replaces + inserts)
```

이 함수는 각 연산의 결과를 모두 합쳐서 반환하기 때문에 꽤 클 수 있다.
단어의 길이가 n이라고 하면, n개의 삭제, n-1개의 뒤집기, 26n개의 변경, 26(n+1)개의 삽입 연산이 가능하므로 전부 합하면 54n+25개의 후보가 있을 수 있다.
예를 들어, `len(edits('something'))`으로 원소의 개수를 세보면 494가 나온다.

오타가 편집거리 1만에 교정될 수도 있지만, 2까지 갈 가능성이 있다.
`edits1`을 한번 더 호출하는 방식으로 2까지 검사할 수 있다.

``` python
def edits2(word):
    return set(e2 for e1 in edits1(word)
                  for e2 in edits1(e1))
```

이렇게 작성하면 편하지만, 계산량이 너무 많다.
`len(edits('something'))`은 무려 114,324가 된다.
모든 후보를 생성하는 대신, 실제로 우리가 알고 있는 단어들만 반환하도록 최적화를 진행해보자.
아래 코드는 최적화를 적용한 코드이다.

```python
def known_edits2(word):
    return set(e2 for e1 in edits1(word)
               for e2 in edits1(e1) if e2 in NWORDS)
```

이렇게 최적화를 하면 `known_edits2('something'`은 `edits2`의 11만개 대신 4개의 단어만을 반환한다.
이 최적화로 수행시가느이 10% 정도를 절약할 수 있었다.

남은 부분은 **오류 모델 P(w|c)**를 만드는 부분이다.
입력 단어에서 편집거리가 1인 모든 단어는 편집 거리가 2인 단어보다 가능성이 훨씬 높고, 편집거리가 0이라고 우리가 알고 있는 단어마다 가능성이 낮다고 가정한다.
여기서 우리가 알고 있는 단어란 언어 모델, 즉 훈련용 데이터에 포함된 단어를 나타낸다.
이 전략을 아래와 같이 구현할 수 있다.

``` python
def known(words):
    return set(w for w in words if w in NWORDS)


def correct(word):
    candidates = known([word]) \
                 or known(edits1(word)) \
                 or known_edits2(word) \
                 or [word]
    return max(candidates, key=NWORDS.get)
```

`correct` 함수는 주어진 입력 `word`에서 편집 거리가 가장 가까운 단어들의 목록은 `candidates`에 저장한다.
그 후에는 `NWORDS` 모델에 의해 알려진 대로 P(c)가 가장 큰 단어를 반환한다.

## 출처
Peter Norvig의 [How to Write a Spelling Corrector](http://norvig.com/spell-correct.html)을 공부하면서 정리했습니다.<br/>
[여기](http://theyearlyprophet.com/spell-correct.html)에서 개선된 소스코드와 설명을 참고했습니다.