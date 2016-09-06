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

![](https://latex.codecogs.com/gif.latex?%5Cmax_%7Bc%7D%20P%5Cleft%28c%7Cw%5Cright%29)

베이즈 정리에 의하면 이것을 아래처럼 고쳐 쓸 수 있다

![](https://latex.codecogs.com/gif.latex?%5Carg%5Cmax_%7Bc%7D%20%5Cfrac%7B%20P%28w%7Cc%29%20%5Ccdot%20P%28c%29%20%7D%7B%20P%28w%29%20%7D)

이 식에서 P(w)는 모든 후보 c에 대해 같은 값이므로 생략.
그러면 아래 식을 얻을 수 있다.

![](https://latex.codecogs.com/gif.latex?%5Carg%5Cmax_%7Bc%7D%20P%28w%7Cc%29%20%5Ccdot%20P%28c%29)


## 출처
Peter Norvig의 [How to Write a Spelling Corrector](http://norvig.com/spell-correct.html)을 공부하면서 정리했습니다
소스 코드는 [여기](http://theyearlyprophet.com/spell-correct.html)에서 참고했습니다.