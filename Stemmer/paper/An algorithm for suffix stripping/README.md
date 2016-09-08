# 접미사 스트리핑 알고리즘 논문

논문 원본은 [여기](originalPaper.txt)에서 보시면 됩니다.
정확하게 번역하기보다는 내용을 이해하고 공부하기 위해서 번역한거라서 번역상태가 좋지 않습니다. ㅜㅜ

## INTRODUCTION
접미사를 자동으로 없애는것은 정보 검색 분야에서 중요한 작업입니다.
In a typical IR environment, one has collection of documents, each described by the words in the document title and possibly by words in the document abstract.
단어가 정확히 어디서 나왔는지는 무시하고, 문서는 단어의 벡터로 표현된다고 말할 수 있다.
공통된 어근을 가진 용어는 비슷한 의미를 가진다.
아래 예시를 보자.

> CONNECT
> CONNECTED
> CONNECTING
> CONNECTION
> CONNECTIONS

용어 그룹이 이처럼 하나의 용어로 융합될때, IR 시스템의 성능은 상향된다.
이것들은 'ED', 'ING', 'ION', 'IONS'같은 접미사들만 없애서 'CONNECT'로 만들면 바로 끝난다.
게다가 접미사 스트리핑 과정은 IR 시스템에 있는 용어들의 총 개수를 줄여줄 것이다. (위에 나온 단어들을 하나로 표시해서 개수를 줄인다는 뜻)
따라서 시스템의 데이터 복잡도와 크기가 줄어들기 때문에 이득이다.
