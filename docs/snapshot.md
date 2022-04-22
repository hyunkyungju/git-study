# Git 뜯어보기
## 할 것
- git 뜯어보기 ~4/25 [Git 뜯어보기](https://www.notion.so/Git-bced185e3ff24020b3e643598d6a21a5)
    - .git에 정보가 어떻게 저장되나
    - 파일 크기에 따라서 어떻게 전송/수신하나
    - github같은 곳은 git 정보를 어디에 저장하나


## 스냅샷
- 스냅샷이 폴더의 구조와 메타데이터만 저장하는 것인가, 거기에 더해 파일 내용까지 저장하는 것인가? 가 가장 궁금했음.
- 결론은 후자가 맞다. BLOB(binary large object) 형태로 파일 내용을 저장함. 
- VCS가 델타 기반과 스냅샷 기반으로 크게 나뉘고, git은 스냅샷 기반임. 스냅샷이 git의 큰 특징 중 하나인듯. 
- 파일이 변경되었으면 변경 내용을 추적하는게 아니라(이건 델타 방식) 새로 스냅샷을 찍음
1. https://tech.gluesys.com/blog/2020/12/16/storage_7_intro.html
- 스냅샷 자체에 대해 가장 상세히 설명된 글인데 안봐도 될듯. 

2. https://okky.kr/article/306493
- 개발 커뮤니티 글이긴 한데 기본적인 개념을 챙기기 좋음. 
- git은 모든것들은 hash object로 변환해서 저장합니다. 폴더 스트럭쳐 (filesystem tree nodes), 파일들, 태그...기타 등등. SHA-1 hash를 사용하여 hash object로 저장하게 되죠. 
- snapshot이라는 계념은 commit을 했을당시에 각각의 타입들 (파일들, 폴더들, 커밋들) 을 "고대로" 저장하게 되는겁니다. 예를들어, src/utils/hello_me.py라는 파일이 커밋 되었다면 src/tree/hello_me.py 파일이 있다는 object가 하나 저장되고, hello_me.py안에 있는 컨텐츠가 저장되고, 커밋 메세지가 따로 object로서 저장됩니다. 파일이 더 추가된다면 그에대한 오브젝트들이 더 생겨나는 거지요. 
- 델타가 더 느린 이유는 간단합니다. 저장을 하기 위해서 따로 작업을 안해도 되서 그렇습니다. "수정한 것"만 저장한다는게 빠를것 같지만, "수정한 것"을 추출하기 위해서는 처음 저장한 파일에 이제까지 수정 했던 모든 "델타"들을 적용한후, 현재에 파일과 비교해야 하지요. 그 이후에 "수정한 것"만 저장해야하니, history가 들어갈수록 시간이 늘어나는겁니다.반면에 깃은, 그냥 지금 상태를 그냥 저장해 버리면 되니, 빠를 수 밖에요.


3. http://dogfeet.github.io/articles/2012/git-delta.html
![](https://i.imgur.com/5sjPDej.png)

- 이렇게 저장하기 때문에 1k 짜리를 한 글자씩 10번 수정해서 커밋해도 거의 1k이다. GC가 수행되면 이렇게 델타로 저장할 뿐만 아니라 gzip으로 압축하기 때문에 실제로 저장하는 용량은 더욱 적다. Git은 이런 기법을 통해서 '저장소 크기가 가장 작다'라고 주장한다.
- 결국 깃헙도 delta 방식을 사용하긴 하는듯. 용량을 줄이기 위해서. 그리고 gzip으로 압축함.
- 그래서 결국 저장을 어떻게 하는지(DB에 어떻게 넣는지) 자료가 잘 안보인다. 


4. https://stackoverflow.com/questions/4964099/what-is-a-git-snapshot
- A snapshot is the state of something (e.g. a folder) at a specific point in time. In this case, snapshot means the current content of the test branch, this doesn't have to be the head revision.


5. https://d2.naver.com/helloworld/1859580
- 체크아웃하려는 버전의 파일이 온전히 BLOB(binary large object) 객체로 저장돼 있으므로 객체를 파일로 변환하기만 하면 된다. 해당 버전에서 변경 사항이 없는 파일은 단순히 이전 버전의 파일을 가리킨다. 따라서 각 버전은 그 시점의 파일 한 벌을 별도로 가지고 있는 셈이다.
- 흥미로움!


