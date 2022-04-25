## github같은 곳은 git 정보를 어디에 저장하나
- git: 로컬에서 버전 관리
- github:  git 저장소를 관리하는 클라우드 기반 호스팅 서비스
- 그렇다면 github는 로컬의 .git 폴더 정보를 어떻게, 어디에 저장하는가?

### How the GitHub store your repository files?
https://stackoverflow.com/questions/30451559/how-the-github-store-your-repository-files

- GitHub uses Git to store repositories, and accesses those repos from their Ruby application
- repository가 .git 폴더를 의미하는 것인듯. 
- 깃허브는 원래 Grit이라는 루비 라이브러리를 사용했음. 지금은 rugged를 사용함.
- There are Git reimplementations in other languages like JGit for Java 
- If you wanted to store Git repositories(.git 폴더), what you'd want to do is store them on a filesystem (or a cluster thereof) and then have a pointer in your database to point to where the filesystem is located, then use a library like Rugged or JGit or Dulwich to read stuff from the Git repository.
- .git 폴더를 서버의 파일시스템에 저장하라는 뜻인듯. 
- pointer를 db에 가지라는 것이 무슨 말인지 잘 이해가 안감
- .git 폴더를 가져오면 JGit 같은 라이브러리로 내용에 접근할 수 있는 듯 하다. 
- There are plugins for a lot of popular web frameworks for doing user file uploads and file management. 
- 사용자가 다운로드할 수 있는 파일만 다운로드할 수 있도록 적절한 사용 권한을 구축하는 것이 중요하다. 다양한 프레임워크 플러그인이 어떻게 작동하는지 살펴볼 필요가 있다.

### JGit
https://git-scm.com/book/ko/v2/%EB%B6%80%EB%A1%9D-B%3A-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%97%90-Git-%EB%84%A3%EA%B8%B0-JGit


### How does Github store millions of repo and billions of files?
https://www.pankajtanwar.in/blog/how-does-github-store-millions-of-repo-and-billions-of-files
- 고퀄의 글이다.

#### Github Architecture
- 한 문장 한 문장을 이해할 수 있으면 얼마나 좋을까 싶은 단락이다. 자기 전에 매일 보고싶은 정도. DNS, 로드 밸런서 이런 내용은 뺐음
- Github 레포 URL을 브라우저에 치면, 무슨 일이 일어난 후에 repo 페이지가 보일까 

##### Here is the quick flow 
- https://github.com/github/docs url 넣음
- 프론트 서버 중 하나가 request 받음 ( typically 8 Core CPU, 16 GB RAM Bare Metal Server)
- 프론트 서버의 Nginx가 (reverse proxy server) 요청을 보냄 to Unix Domain Socket (16 Unicorn worker processes are running on it) 
- 워커 한명이 백엔드로 요청 보냄 (Running on Ruby on Rails) 백엔드는 MySQL, 캐시랑 연결되어있음. 
- 캐시 미스 발생 시 **Grit library**를 이용해서 데이터 가져옴(이게 rugged로 바뀐거 아녔음?)
- Grit makes RPC call (Remote procedure call) to **Smoke** Service (filesystem에 direct하게 접근& mapped to frontend machines)
    - 운영체제를 공부하다 보면 프로세스간 통신을 위해 IPC(inter-Process Communication)을 이용하는 내용을 볼 수 있는데요, RPC는 IPC 방법의 한 종류로 원격지의 프로세스에 접근하여 프로시저 또는 함수를 호출하여 사용하는 방법을 말합니다.
    - Smoke Service를 검색하면 담배 게시글 밖에없다. 자료가 잘 없는 듯 
- 프론트 서버는 4개의 프록시 서버 인스턴스를 돌린다. 
- 프록시 서버가 request에서 repo의 유저 이름을 알아냄 
- **Chimney** (A library which gives route to Smoke service) talks to Redis for route and gives back to Smoke
- Smoke establishes a transparent proxy with proper file server
- The response is sent back through smoke proxy to Ruby on rails app
- It sends back response to client via Nginx (not through load balancer)
- Slave file servers keeps memcache server.
- IMPORTANT - Github uses Rackspace instead of Amazon EC2 as Amazon Elastic Block Store was not nearly as fast as bare metal when they ran benchmarks for handling high disk IO operations.
- 뒤로 갈수록 어렵다..  


#### New Github Storage
- shared file system -> federated(연합) storage 
- federated storeage: a collection of autonomous(자율적인) storage resources which are being monitored by a common management system that provides rules, how data is stored, managed and migrated throughout the storage network. 중간에 common management system이 전체적으로 관리하고, 자율적인 스토리지가 여러개 있다는 뜻. 
- In the new system, they could add as many additional machines (Linux machines running ext3/ext4) they wanted, without hitting the performance. 병렬처리로 성능을 끌어올릴 수 있었다는 뜻인 것 같은데 이유가 뭐지?

#### Spokes
- 목적: to store multiple distributed copies of a github repo across data centres.
- Spoke는 repo의 여러 복제본을 저장하고 모든 복제본을 sync로 관리함. sync가 동시 발생의 의미에 가까운듯. DB의 상태를 같게 유지시킨다는 말 같다. 
- 복제를 Git application level에서 수행함. 이전에는 file system block level에서 수행하였는데. 
- GitHub makes use of the three-phase commit protocol in order to update replicas, and also a distributed lock to ensure the correct update order

#### How data is stored in file storage?
- Git 레포의 모든 데이터는 DAG(Direct Acyclic Graph) 구조로 저장됨. 
- 커밋은 부모 커밋(이전의 커밋)과 연결되고, 워킹 디렉토리의 스냅샷 트리를 가짐. 
- 각 트리는 내부의 트리(폴더)를 갖고 recursiveley하게 tree간 연결되어있다. 
- 모든 tree object들의 DB 인덱싱은 SHA-1 해시로 되어있음. SHA1이면 160bit아닌가 엄청크네
- Spoke는 3개의 다른 서버에 깃헙 레포를 3개 카피해서 저장함. 
- 서버 2개가 다운되어도 작동할 수 있도록 한다. 
- For each push to a Git repo, it goes through a proxy which is responsible for replicating the change. 
![](https://i.imgur.com/pTFrPMf.png)
- Github Spokes’s design, with optimization of the performance of distributed reference updates, now allows Spokes to replicate Git repositories effectively. This improves robustness, speed, and flexibility. 최적화 했다는 문장인듯! 

