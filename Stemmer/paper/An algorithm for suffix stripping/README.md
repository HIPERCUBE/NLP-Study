# 접미사 스트리핑 알고리즘 논문

논문 원본은 [여기](originalPaper.txt)에서 보시면 됩니다.
정확하게 번역하기보다는 내용을 이해하고 공부하기 위해서 번역한거라서 번역상태가 좋지 않습니다. ㅜㅜ

## INTRODUCTION
접미사를 자동으로 없애는것은 정보 검색 분야에서 중요한 작업입니다.
In a typical IR environment, one has collection of documents, each described by the words in the document title and possibly by words in the document abstract.
단어가 정확히 어디서 나왔는지는 무시하고, 문서는 단어의 벡터로 표현된다고 말할 수 있다.
공통된 어근을 가진 용어는 비슷한 의미를 가진다.
아래 예시를 보자.

> CONNECT<br/>
> CONNECTED<br/>
> CONNECTING<br/>
> CONNECTION<br/>
> CONNECTIONS

용어 그룹이 이처럼 하나의 용어로 융합될때, IR(Information Retrieval) 시스템의 성능은 상향된다.
이것들은 'ED', 'ING', 'ION', 'IONS'같은 접미사들만 없애서 'CONNECT'로 만들면 바로 끝난다.
게다가 접미사 스트리핑 과정은 IR 시스템에 있는 용어들의 총 개수를 줄여줄 것이다. (위에 나온 단어들을 하나로 표시해서 개수를 줄인다는 뜻)
따라서 시스템의 데이터 복잡도와 크기가 줄어들기 때문에 이득이다.

많은 접미사 스트리핑 전략들에 대해서 이미 많은 논문이 나왔다. {(e.g. 1-6)}
작업의 성격은 stem 사전(줄기 사전, 한국어로 어떻게 해야할지..)이 쓰였는지, 접미사 리스트가 쓰였는지에 따라 상당히 변한다.
stem 사전을 만들어 사용하고, 작업의 목적은 IR 성능의 향상이라고 가정해보자, 그러면 접미사 스트립핑 프로그램은 일반적으로 명시적인 접미사 리스트(각 접미사들은 알맞은 줄기에서 나온 단어들을 지울 수 있게 할 수 있는)들을 제공한다.
이 논문에서는 이러한 접근법을 사용합니다.
현재 프로그램의 가장 큰 이점은 소스 코드가 라인수가 적고, 빠르다는 것이다. => 그래서 간단함.
여하튼 이 논문에서 알고리즘을 모두 설명하기에 충분히 간단하다.
논문대로 만든 프로그램의 속도는 연속된 텍스트의 큰 파일에 있든 모든 단어에 적용하기 현실적이다.

정보 검색 작업을 위한 접미사 스트립핑 프로그램에서, 2가지를 명심해야한다.
먼저, 접미사들은 IR의 성능을 향상시키기위해 간단하게 없애야한다.
이 말은 즉, 어떤 상황에서도 항상 접미사를 완벽하게 없앨 필요가 없다는 것이다.
자동으로 단어의 접미사를 정확하게 찾아냈더라도

아마도 W1, W2 두 단어의 접미사를 없애서 줄기 단어 S를 만드는 최선의 기준은 'a document is about W1'과 'a document is about W2' 2 문장 사이의 차이점이 없어야 한다는 것이다.
만약 W1이 CONNECTION이고 W2가 CONNECTIONS라면, 하나의 용어로 만들기 편해 보일것이다.
하지만 W1이 RELATE이고 W2가 RELATIVITY라면 어려워 보인다. (특히 이론 물리학에 관한 거라면)
이런한 경우들을 해결하기 위해, 과연 융합해야하는지에 대한 입장에 약간의 변화가 있다.
단순히 문서의 적절성만 따져서 뭐리를 날리면 될수도 있다는 입장도 있다.
접미사 스트립핑 시스템의 가치에 대한 평가는 correspondingly 어렵다.

두번재 관점은, 여기서 채택한 접근법에 관한 것이다.
즉, 다양한 규칙의 접미사 리스트를 사용하는 것이다.
접미사 스트립핑의 성공률은 꽤 낮지만, 
예를 들어, 만약 SAND와 SANDER를 합쳐본다고 하면, WAND와 WNADER가 될 확률이 높다.
WANDER의 ER이 접미사로 여겨졌기 때문에 에러가 발생한것이다.
접미사는 단어의 의미를 완전히 바꿔버리는 것과 같다.
이런 경우에는 삭제 작업이 unhelpful(도움이 안된다)하다.
PROBE와 PROBATE를 예시로 들어보면, 이 둘은 현대 영어에서 전혀 다른 의미를 가지고 있다.
(현재 이 프로그램의 알고리즘에서는 합치지 않는다.)
단어를 사용하는 분야에서 위같은 성능 하락이 발생하지 않도록 하기위해서, 몇가지 규칙을 추가해 접미사 스트립핑 프로그램의 성능을 향상시켜야한다.
이 현상이 발견되지 않는다면, 필요한 만큼 프로그램을 훨씬 복잡하게 하는것도 쉽게 가능하다.
또한 과도한 강조를 해서 중요해 보이게 하는것도 가능하지만, 오히려 드문것으로 판명된다.
예를 들어 추가적인 접미사로 단어의 어근에서 변하는 경우는 DECEIVE/DECEPTION, RESUME/RESUMPTION, INDEX/INDICES 등이 있다.
에러률이 예상된다는 관점에서 볼때, 이런한 케이스들을 다루기를 시도하는 것은 별로 가치있어 보이지 않는다.

현재 프로그매의 단순함을 결점이라고만 보기는 힘들다.
잘 알려진 Cranfield 200 collection{7}에서 테스트할때 검색 성능이 향상되었고, 1971년에 캠브릿지 대학교의 IR 연구에서 사용된것보다 훨씬 정교하다.
아래 방식으로 테스트를 진행했다.
먼저 타이틀의 단어들과 문서의 abstracts는 접미사 스트립핑 시스템을 거친다.
그 다음 결과 원형 단어들은 문서의 인덱스로 사용한다.
쿼리의 단어들은 같은 방식으로 원형 단어로 줄여진다.
그리고 문서들은 각 쿼리때마다 순위가 매겨진다.
이러한 랭킹과정으로부터 재현율과 정확도 값을 얻는다.
전체 과정은 이 논문에 나오있는 접미사 스트립핑 시스템으로 계속 반복된다.
그리고 결과값은 아래와 같다.

| earlier system || present system ||
| :--: | :--: | :--: | :--: |
| precision | recall | precision | recall |
|   0 | 57.24 |  0 | 58.60 |
|  10 | 56.85 |  10 | 58.13 |
|  20 | 52.85 |  20 | 53.92 |
|  30 | 42.61 |  30 | 43.51 |
|  40 | 42.20 |  40 | 39.39 |
|  50 | 39.06 |  50 | 38.85 |
|  60 | 32.86 |  60 | 33.18 |
|  70 | 31.64 |  70 | 31.19 |
|  80 | 27.15 |  80 | 27.52 |
|  90 | 24.59 |  90 | 25.85 |
| 100 | 24.59 | 100 | 25.85 |

성능은 엄청 다르지 않다.
정요한 점은 기존의 정교한 시스템은 이 간단한 시스템과 별로 다르지 않다는 것이다.

## ALGORITHM
접미사 스트립핑 알고리즘을 전부 보여주기 위해서 몇가지를 정의할 필요가 있다.</br>
아래 리스트에 있는 것들만 단어의 **consonant(자음)**이라고 한다.

- A,E,I,O,U를 제외한 단어
- 모음뒤에 있는 Y

예를 들어 `TOY`에서 **consonant(자음)**은 T와 Y다.</br>
`SYZYGY`에서는 S,Z,G이다.</br>
**vowel(모음)**은 **consonant(자음)**이 아닌 것을 말한다.

**consonant(자음)**은 **c**로 표시한다.</br>
**vowel(모음)**은 **v**로 표시한다.</br>
**c**가 연속적으로 나타나는 것은 **C**라고 표시한다.</br>
**v**가 연속적으로 나타나는 것은 **V**라고 표시한다.

어떤 단어든지 아래와 같은 형태로 나타난다.

- CVCV ... C
- CVCV ... C
- VCVC ... C
- VCVC ... V

이들 모두는 단일 형태로 표현될 수 있다.

> [C]VCVC ... [V]

대괄호안에 들어있는 것은 몇번 반복되는지를 나타낸다.</br>
예를 들어 `(VC){m}`는 VC가 m번 반복된다는 것을 나타낸다.</br>
아래처럼 사용할 수 있다.

> [C](VC){m}[V]

**m**은 **measure**의 약자로 단어나 부분이 몇번 반복되는지 뜻한다.</br>
'm=0'인 경우는 null word이다.</br>
아래에 몇몇 예시가 있다.

> m=0   TR(C), EE(V), TREE(CV), Y(V), BY(CV)</br>
> m=1   TROUBLE(CVCV), OATS(VC), TREES(CVC), IVY(VCV)</br>
> m=2   TROUBLES(CVCVC), PRIVATE(CVCVCV), OATEN(VCVC), ORRERY(VCVCV)</br>

[C](VC){m}[V]를 이렇게 이해하면 된다.</br>
[C], [V]는 있을수도 있고, 없을 수도 있다.</br>
(VC)가 m번 반복되는 것을 말하는 것이다.</br>

접미사 제거 규칙은 아래와 같은 형태로 제공된다.

> (condition) S1 -> S2

만약에 단어가 접미사 S1으로 끝나고, S1 앞에 있는 단어가 주어진 condition(조건)을 충족하면, S1은 S2로 교체된다.
condition(조건)은 일반적으로 m의 관점에서 주어진다.
예를 들면

> (m > 1) EMENT ->

여기서 S1은 'EMENT'이고, S2는 null이다.
'REPLACEMENT'는 'REPLAC'으로 될 것이다.
'REPLAC'은 m이 2이기 때문이다.
(REPLAC = CVCVC)

condition(조건) 부분은 아마 아래 내용들을 포함한다.

\*S - S로 끝나는 stem(원형 단어) 과 마찬가지인 다른 단어들

\*v\* - 모음을 포함한 stem
 
\*d - 자음 2개로 끝나는 stem(e.g -TT, -SS)

\*o - cvc로 끝나는 stem, 2번째 c가 W, X, Y가 아닌 단어(e.g -WIL, -HOP).

그리고 condition(조건) 파트는 **and**, **or**, **not** 표현들을 포함해야한다.
아래 예시를 보자.

> (m>1 and (\*S or \*T))

m이 1보다 크고, S나 T로 끝나는지 검사하는것이다.
> (\*d and not (\*L or \*S or \*Z))

L, S, Z가 아니라 이중 자음(consonant)로 끝나는지 검사하는 것이다.
이런 정교한 조건은 드물게 필요하다.

아래 나온 규칙들 중에서, 오직 하나만 순종한다.
그리고 이건 주어진 단어에서 S1과 가장 길게 매칭될것이다.
예를 들면 아래와 같다.

> SSES -> SS<br/>
> IES -> I<br/>
> SS -> SS<br/>
> S ->

(조건들이 전부 null인 경우) `CARESSES`는 `CARESS`와 매핑된다.
이유는 `SSES`가 S1과 가장 길게 매칭도기 때문이다.
즉 `CARESS`는 `CARESS`(S1='SS')와 매칭되고, `CARES`는 `CARE`(S1=S)와 매칭된다.

아래 규칙들들 어플리케이션에서 사용되는 예시를 보도록하자.
알고리즘은 아래와 같다.
"[]"안에 있는 거는 조건, 그 뒤에 나오는거는 예시이다.

**Stemp 1a**

> [SSES -> SS] caresses -> caress<br/>
> [IES -> I] ponies -> poni<br/>
> [IES -> I] ties -> ti<br/>
> [SS -> SS] caress -> caress<br/>
> [S -> ] cats  -> cat

**stem 1b**

> [(m>0) EED -> EE] feed -> feed<br/>
> [(m>0) EED -> EE] agreed -> agree<br/>
> [(\*v\*) ED -> ] plastered -> plaster<br/>
> [(\*v\*) ED -> ] bled -> bled<br/>
> [(\*v\*) ING -> ] motoring -> motor<br/>
> [(\*v\*) ING -> ] sing -> sing

만약 **step 1b**의 2,3번재 규칙이 성공적이면, 아래 단계가 진행된다.

> [AT -> ATE] conflat(ed) -> conflate<br/>
> [BL -> BLE] troubl(ed) -> trouble<br/>
> [IZ -> IZE] siz(ed) -> size<br/>
> (\*d and not (\*L or \*S or \*Z))<br/>
>    -> Single letter<br/>
> hopp(ing) -> hop<br/>
> tann(ed) -> tan<br/>
> fall(ing) -> fall<br/>
> hiss(ing) -> hiss<br/>
> fizz(ed) -> fizz<br/>
> [(m=1 and \*o) -> E] fail(ing) -> fail<br/>
> [(m=1 and \*o) -> E] fil(ing) -> file

단일 문자에 매핑하는 규칙은 이중 문자의 글자 쌍을 하라를 제거한다.
`-AT`, `-BL`, `-IZ`뒤에 오는 `-E`, 즉 접미사 `-ATE`, `-BLE`, `-IZE`는 나중에 인식될 수 있다.
이 `E`를 지우는 것은 4번째 단계이다.

**Step 1c**

> [(\*v\*) Y -> I] happy -> happi<br/>
> [(\*v\*) Y -> I] sky -> sky 

Step 1은 복수형과 과거 분사를 다룬다.
이렇게 하면 후속단계가 훨씬 간단해진다.

**Step 2**

> [(m>0) ATIONAL -> ATE] relational -> relate<br/>
> [(m>0) TIONAL -> TION] conditional -> condition<br/>
> [(m>0) ATIONAL -> ATE] rational -> rational<br/>
> [(m>0) ENCI -> ENCE] valenci -> valence<br/>
> [(m>0) ANCI -> ANCE] hesitanci -> hesitanc<br/>
> [(m>0) IZER    ->  IZE ]          digitizer      ->  digitiz<br/>
> [(m>0) ABLI    ->  ABLE]          conformabli    ->  conformabl<br/>
> [(m>0) ALLI    ->  AL  ]          radicalli      ->  radica<br/>
> [(m>0) ENTLI   ->  ENT ]          differentli    ->  differen<br/>
> [(m>0) ELI     ->  E   ]          vileli        - >  vil<br/>
> [(m>0) OUSLI   ->  OUS ]          analogousli    ->  analogou<br/>
> [(m>0) IZATION ->  IZE ]          vietnamization ->  vietnamiz<br/>
> [(m>0) ATION   ->  ATE ]          predication    ->  predicat<br/>
> [(m>0) ATOR    ->  ATE ]          operator       ->  operat<br/>
> [(m>0) ALISM   ->  AL  ]          feudalism      ->  feuda<br/>
> [(m>0) IVENESS ->  IVE ]          decisiveness   ->  decisiv<br/>
> [(m>0) FULNESS ->  FUL ]          hopefulness    ->  hopefu<br/>
> [(m>0) OUSNESS ->  OUS ]          callousness    ->  callou<br/>
> [(m>0) ALITI   ->  AL  ]          formaliti      ->  forma<br/>
> [(m>0) IVITI   ->  IVE ]          sensitiviti    ->  sensitiv<br/>
> [(m>0) BILITI  ->  BLE ]          sensibiliti    ->  sensible

문자열 S1에 대한 테스트는 프로그램이 테스트된 단어의 2번제 글자를 스위치함으로써 빨라질 수 있다.
이런 방식은 때때로 문자열 S1의 가능한 값의 고장을 유발하기도 한다.
step 2의 문자열 S1은 끝에서 2번째 글자의 알파벳순 정렬된다.
비슷한 기술이 다른 단계에서도 적용될 수 있다.

**Step 3**

> [(m>0) ICATE ->  IC]              triplicate     ->  triplic<br/>
> [(m>0) ATIVE ->    ]              formative      ->  form<br/>
> [(m>0) ALIZE ->  AL]              formalize      ->  formal<br/>
> [(m>0) ICITI ->  IC]              electriciti    ->  electric<br/>
> [(m>0) ICAL  ->  IC]              electrical     ->  electric<br/>
> [(m>0) FUL   ->    ]              hopeful        ->  hope<br/>
> [(m>0) NESS  ->    ]              goodness       ->  good

**Step 4**

> [(m>1) AL    ->             ]     revival        ->  reviv<br/>
> [(m>1) ANCE  ->             ]     allowance      ->  allow<br/>
> [(m>1) ENCE  ->             ]     inference      ->  infer<br/>
> [(m>1) ER    ->             ]     airliner       ->  airlin<br/>
> [(m>1) IC    ->             ]     gyroscopic     ->  gyroscop<br/>
> [(m>1) ABLE  ->             ]     adjustable     ->  adjust<br/>
> [(m>1) IBLE  ->             ]     defensible     ->  defens<br/>
> [(m>1) ANT   ->             ]     irritant       ->  irrit<br/>
> [(m>1) EMENT ->             ]     replacement    ->  replac<br/>
> [(m>1) MENT  ->             ]     adjustment     ->  adjust<br/>
> [(m>1) ENT   ->             ]     dependent      ->  depend<br/>
> [(m>1 and (*S or *T)) ION ->]     adoption       ->  adopt<br/>
> [(m>1) OU    ->             ]     homologou      ->  homolog<br/>
> [(m>1) ISM   ->             ]     communism      ->  commun<br/>
> [(m>1) ATE   ->             ]     activate       ->  activ<br/>
> [(m>1) ITI   ->             ]     angulariti     ->  angular<br/>
> [(m>1) OUS   ->             ]     homologous     ->  homolog<br/>
> [(m>1) IVE   ->             ]     effective      ->  effect<br/>
> [(m>1) IZE   ->             ]     bowdlerize     ->  bowdler

이제 접미사들을 삭제되었다.
이제 남은것은 간단한거 밖에 없다.

**Step 5a**

> [(m>1) E     ->       ]          probate        ->  probat<br/>
> [(m>1) E     ->       ]          rate           ->  rate<br/>
> [(m=1 and not *o) E ->]          cease          ->  ceas

**Step 5b**

> [(m > 1 and *d and *L) -> single letter]<br/>
>                                     controll       ->  control<br/>
>                                     roll           ->  roll

스템이 너무 짧은 경우 알고리즘은 접미사를 제거하지 않도록 주의한다.
스템의 길이수를 m이라고 측정한다.
이 접근법에는 언어적인 기초가 없다.
이건 단순하게보인다.
m이 접미사를 없앨지 말지 결정하는데 꽤 효과적인 도움이 된다.
예를 들면 아래에 2가지 리스트가 있다.

| List A | List B |
| :-- | :-- |
| RELATE   |  DERIVATE |
| PROBATE  |  ACTIVATE |
| CONFLATE |  DEMONSTRATE |
| PIRATE   |  NECESSITATE |
| PRELATE  |  RENOVATE |

List B의 단어들에서 `-ATE`는 삭제된다.
하지만 List A의 단어들에서는 그렇지 않다.
이것은 DERIVATE/DERIVE, ACTIVATE/ACTIVE, DEMONSTRATE/DEMONSTRABLE, NECESSITATE/NECESSITOUS 쌍은 서로 conflate결합한다는 것이다.

The fact that no attempt is made to identify prefixes can make the results look rather
inconsistent. Thus PRELATE does not lose the -ATE, but ARCHPRELATE becomes
ARCHPREL. In practice this does not matter too much, because the presence of
the prefix decreases the probability of an erroneous conflation.

Complex suffixes are removed bit by bit in the different steps. Thus
GENERALIZATIONS is stripped to GENERALIZATION (Step 1), then to GENERALIZE
(Step 2), then to GENERAL (Step 3), and then to GENER (Step 4). OSCILLATORS
is stripped to OSCILLATOR (Step 1), then to OSCILLATE (Step 2), then to
OSCILL (Step 4), and then to OSCIL (Step 5). In a vocabulary of 10,000
words, the reduction in size of the stem was distributed among the steps as
follows:

    Suffix stripping of a vocabulary of 10,000 words
    ------------------------------------------------
    Number of words reduced in step 1:   3597
                  "                 2:    766
                  "                 3:    327
                  "                 4:   2424
                  "                 5:   1373
    Number of words not reduced:         3650

The resulting vocabulary of stems contained 6370 distinct entries. Thus the
suffix stripping process reduced the size of the vocabulary by about one
third.

## REFERENCIES

1.  LOVINS, J.B. Development of a Stemming Algorithm. \Mechanical
    Translation and computation Linguistics\. \11\ (1) March 1968 pp 23-31.

2.  ANDREWS, K. The Development of a Fast Conflation Algorithm for English.
    \Dissertation for the Diploma in Computer Science\, Computer
    Laboratory, University of Cambridge, 1971.

3.  PETRARCA, A.E. and LAY W.M. Use of an automatically generated authority
    list to eliminate scattering caused by some singular and plural main
    index terms. \Proceedings of the American Society for Information
    Science\, \6\ 1969 pp 277-282.

4.  DATTOLA, Robert T. \FIRST: Flexible Information Retrieval System for
    Text\. Webster N.Y: Xerox Corporation, 12 Dec 1975.

5.  COLOMBO, D.S. and NIEHOFF R.T. \Final report on improved access to
    scientific and technical information through automated vocabulary
    switching.\ NSF Grant No. SIS75-12924 to the National Science
    Foundation.

6.  DAWSON, J.L. Suffix Removal and Word Conflation. \ALLC Bulletin\,
    Michaelmas 1974 p.33-46.

7.  CLEVERDON, C.W., MILLS J. and KEEN M. \Factors Determining the
    Performance of Indexing Systems\ 2 vols. College of Aeronautics,
    Cranfield 1966.