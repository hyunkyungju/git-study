
## .git에 정보가 어떻게 저장되나
- .git 디렉토리: Git 저장소
- https://www.pankajtanwar.in/blog/how-does-github-store-millions-of-repo-and-billions-of-files
- All data in a Git repo is stored in a Direct Acyclic Graph. 
- Every object in this highly dense tree, is indexed in the database by SHA-1 hash of their content.
- 네모 박스의 내용이 커밋 객체에 포함됨
![](https://i.imgur.com/h8vwQ4B.png)


### .git 폴더의 구성
![](https://i.imgur.com/9czgrXF.png)
- 핵심은 파일이 저장된 object 디렉토리이다. objects 디렉토리를 꼼꼼히 보고 커밋에 이름 붙이는 refs 디렉토리를 간단히 살펴보았다. 
- https://junwoo45.github.io/2019-07-07-git%EC%9D%98%EB%82%B4%EB%B6%80/
- 기본적인 내용이 설명된 블로그. 

#### objects directory
- 파일이 저장되는 디렉토리다. 
- objects 디렉토리에 저장되는 것: 트리 객체(파일 목록), 커밋 객체(커밋 정보), blob 객체(파일 내용)
- Git은 모든 파일을 BLOB 객체로 저장한다. 
- 파일을 저장할 때 SHA-1로 체크섬을 계산한다. 
- 체크섬으로 계산한 해시값의 첫 두 자리를 디렉터리 이름으로 사용하고 나머지를 파일 이름으로 사용한다. 
- (참고만 하기. 만들어보니까 그냥 비어있음) object 디렉터리의 info 디렉터리와 pack 디렉터리에는 메타 정보와 Packfile이 저장된다. 
- objects 디렉터리에 객체가 생성되는 시점은 작업 디렉터리에 파일을 저장하는 시점이 아니다. git add 명령어로 Staging Area에 파일을 등록할 때 생성된다. 
- https://www.raywenderlich.com/books/advanced-git/v1.0/chapters/1-how-does-git-actually-work
- In Git, the most common objects are:
    - Commits: Structures that hold metadata about your commit, as well as the pointers to the parent commit and the files underneath.
    - Trees: Tree structures of all the files contained in a commit.
    - Blobs: Compressed collections of files in the tree.

##### tree object
- 파일 시스템에서의 디렉토리. 파일이름과 파일내용의 hash(blob) 을 가지고 있다.
- 트리 객체는 커밋 시점에 존재한 모든 파일에 대한 목록을 담고 있다
- 트리 객체는 다른 트리 객체를 하위에 가질 수 있다. 
- keeps a snapshot of the working directory in the moment when the commit was created. 
- This tree links to the sub-tree (folders & files) and sub-trees recursively links to other sub-trees.
![](https://i.imgur.com/c0j61PG.png)

##### commit object
- 커밋 객체는 특정 최상위 트리 객체와 저자 등의 메타 정보 및 부모 커밋의 정보를 가진다.
- 커밋 객체의 정보를 확인해 보면 하위에 트리 객체 하나를 가리키고 있는 것을 볼 수 있다.
-  이전 커밋에 대한 정보가 커밋 객체에 포함된다. 이전 커밋을 가리키는 부모 커밋 아이디(parent)가 있다.
-  파일 시스템 상태에 대한 스냅샷이다. 따라서 root Tree 의 hash 를 가지고 있다. 추가적으로 버전 컨트롤을 위해서 부모 커밋의 hash 도 가지고 있다. 커밋메세지 및 작성자 등 커밋 메타 정보 또한 가지고 있다.
- 착각하기 쉬운 것이 commit 이 차이라고 생각하는 것이다. 하지만 우리가 보게되는 차이는 두개의 스냅샷간의 차이를 tree 순회 를 통해서 계산한 것 이다.
- 찍어보면 tree 해시값이 항상 다름. 파일이 변경되었기 때문. 
- Every commit has a link with it’s parent commit.
![](https://i.imgur.com/zux6UsQ.png)


##### blob object
- 파일의 내용을 담고 있다.
![](https://i.imgur.com/9xWcHC5.png)



##### object 사용 예시
- 마지막 커밋은 main.py 파일만 바꾸었음. 다른 tree와 blob은 상태가 그대로임!
- 커밋 기록
![](https://i.imgur.com/XIKbKJO.png)
- 이전 커밋
![](https://i.imgur.com/O7k5RKK.png)
- main.py만 바꾼 커밋
![](https://i.imgur.com/YpSIMbT.png)
- 개인적으로 이 부분이 참 흥미롭다고 생각함. 여기서는 파일을 단위로 새로 스냅샷을 썼지만, 우리는 파일 1개를 보통 관리할거니까 파일 내의 페이지를 단위로 스냅샷을 적용하는 것도 괜찮을듯?
- 근데 ppt나 word가 페이지 단위로 xml로 나오는지는 모르겠다. 

#### refs directory
- Git의 커밋 아이디는 파일 이름과 마찬가지로 SHA-1 값을 사용한다. 이 값은 길고 복잡해서 직접 커밋 아이디를 이용해 작업한다면 매우 불편하다. 따라서 특별한 의미가 있고 사용 빈도가 높은 커밋은 별도의 이름을 지정해서 사용하는데 이를 레퍼런스(references 또는 refs)라 한다. 레퍼런스가 가리키는 커밋 아이디는 refs 디렉터리 아래에 레퍼런스별 파일로 저장된다. 이 파일의 경로(.git 디렉터리가 기준인 상대 경로)를 레퍼런스로 사용한다. 결국 레퍼런스는 특정 커밋에 대한 포인터이며 축약어이자 커밋 아이디를 저장한 파일의 경로인 셈이다.
- 예를 들어 기본으로 존재하는 마스터 브랜치는 refs/heads/master 파일에 마지막 커밋의 아이디가 저장되며 이 상대 경로가 그대로 레퍼런스로 사용된다. 브랜치나 태그의 경우에는 좀 더 간편한 방법을 제공한다. 상대 경로를 모두 사용할 필요 없이 브랜치나 태그의 이름만 사용해도 된다. 즉, refs/heads/master를 사용할 필요 없이 master만 사용해도 된다. 이를 브랜치 레퍼런스라고 한다.
- 정리하자면 커밋을 별도로 이름을 지정한 것을 정리한 것. 위의 내용은 네이버 D2 블로그에서 가져옴. 

## 파일 크기에 따라서 어떻게 전송/수신하나 -> 이걸 찾아봐야함

## github같은 곳은 git 정보를 어디에 저장하나
- git: 로컬에서 버전 관리
- github:  git 저장소를 관리하는 클라우드 기반 호스팅 서비스
- 이것도 2번 질문이랑 비슷한 것 같음

